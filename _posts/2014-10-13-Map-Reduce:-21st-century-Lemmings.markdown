---
layout: post
title:  "Map/Reduce: 21st century Lemmings"
date:   2014-10-13 17:54:02
categories: jekyll update
---
A while ago I started learning and setting up small-scale programming jobs on Hadoop/MapReduce. My first 
reaction to the new tools was "Wow, this is empowering!". For the first time since I got into the business of 
machine learning and data science, I felt I could reach things beyond the capabilities of my desktop PC. At 
the same time, my overall feeling in going through the logic of the Map Reduce framework, was a déjà vu from 
my teenage years, when a  friend and I would play [Lemmings](http://en.wikipedia.org/wiki/Lemmings_%28video_game%29) 
on his Atari for hours: You are given an army of small, dumb workers. Get them to work together the right way, 
and you are guaranteed to have awesome results and fun along the way. In this spirit, this will be the first of a number of posts on MapReduce.

As a general reminder, a Map/Reduce process consists of two parts. In the first part, a number of so-called "mappers" 
go through the data and produce `key,value` pairs. The mappers can each go through a different chunk of the data, so 
that they work in parallel. These pairs are then sorted on the keys, partitioned into new chunks, and passed to the 
reducers (note, however, that an additional "combiner" stage can be added after the mappers' job is done, e.g. to 
alleviate network traffic on a computer cluster). The reducers do some additional analysis on the `key,value` pairs 
(for example count how many different values appear for each key), and produce a final result which should normally 
be human-readable.

For example, imagine you have two decks of cards which are mixed together, and want to find out if any cards are 
misssing from one of the decks. One basic check is to go through the decks, and see if there are four cards for each 
figure and for each deck. To do this, you split the mixed-up deck in two and give one half to each of your two kids 
(the mappers). You tell them to go through the cards, and for each card write on a little note the type of figure 
and which deck it came from (the _key_). In this particular example it does not matter what you pass as a _value_, 
so the value can be blank (say null). At the end you have a large number of little papers saying _(Ace Deck 1, null)_, 
_(Eight Deck 2, null)_, ans so on. The next step is to sort the little pieces of paper, according to the figure value 
on them (e.g. _Ace_, _Two_, _Three_, etc.), followed by the deck value on them (i.e. all of the notes on _Deck 1_ come before 
the  papers from _Deck 2_). After this  is done, you split the stack of sorted paper notes in two. You take one half, 
and your spouse the other half. Now you two are the reducers: you each go through your stack of notes, and keep a 
register for each figure and deck. In other words, you record how many notes you saw saying _(Aces Deck 1, null)_, 
_(Eight Deck 2, null)_, and so on. Because of the sorting, you will see all the notes about Aces from _Deck 1_ before 
you see any other note, so as soon as you see a Two from _Deck 1_, you can write down how many Aces from _Deck 1_ you saw. 
If you end up with a three, you know one ace is missing from deck one, and you can note  this on a different piece of 
paper. The exact same thing holds for your spouse. In the end you can put your summary notes down to see how many 
figures of each kind are missing from each deck, and that's all there is. This is an example of solving a counting problem 
using Map/Reduce. Of course, in a real implementation there are several kinds of issues having to do with the fact that 
the machines doing the work are on a  cluster with certain communication constrains between the machines, and with some 
of the machines occasionally failing, but I will not go into this. 

I will not cover in detail how the Map/Reduce framework solves different kinds of problems. If you are  interested in a 
basic introduction, the Udacity course videos are excellent. Instead of repeating that material, I will show how the 
framework solves a particular kind of problem: creating an inverted index of words appearing on an online forum. The data 
I will work on is from the Udacity forum, and you can find the complete set [here](http://content.udacity-data.com/course/hadoop/forum_data.tar.gz). 
This forum is similar to the Stack Exchange websites, and was built using [OSQA](http://www.osqa.net/). Each post in 
the forum is a node which contains the text of the post, information about the author, the parent node for answer and 
comment posts, etc. My goal is to go through all the posts in the forum, and for each word, list all of the nodes on which 
the word appears. However, I want to be able to show the data as it is being processed, so I will work on a subset of the 
full forum data, which I can easily inspect on my text editor. You can find the input data [here](https://github.com/nikos-daniilidis/hadoop-mapreduce-o), 
under the name `testfile-forum`. 

To solve the problem using Map/Reduce, we will need to break it into two tasks. First we will process the raw data and output 
a series of `key,value` pairs (this is the mappers' job). Since we want to find the nodes in which a certain word appears, 
we will use the words as keys and the values will be node ids. The pseudocode for the task looks something like the following
 
        for each entry in forum
            find the entry text
            find the node id
            split the entry text into words
            for each word in the entry text
                print word, node id

In python, this looks as follows:
import sys, csv

		reader = csv.reader(sys.stdin, delimiter='\t')
		noNoList = ['.',',','!','?',':',';','\"','(',')','<','>','[',']','#','$','=','-','/']
		
		for line in reader:
			if line[0]=="id":
				continue
			elif len(line) == 19:
				nodeid = line[0]
				body = line[4]
			for ch in noNoList:
				body = body.replace(ch," ")
			body = body.replace("  "," ")
			wordlist = body.lower().split(" ")
			for word in wordlist:
				print "{0}\t{1}".format(word, nodeid)

You can find this as `mapper9.py` [here](https://github.com/nikos-daniilidis/hadoop-mapreduce-o). If you convert this python script 
to an executable file (using `chmod +x mapper9.py`), you can simulate the action of the map and sort by streaming the contents of 
the test file to the mapper, and sorting the output of the mapper (on a terminal this is done using `cat | ./mapper9.py | sort`). 
The output is pretty long, and there are a lot of "non-word" items, since  I did not make any particular effort to clean up the text 
in the mapper function. Here is a section of the output which should make the next step obvious:

		any	2741
		any	7185
		application	26454
		are	2312
		as	2312
		as	2312
		audio	2312
		be	2312
		be	6361
		being	2741

You can see that sorting the output of the mapper makes the life of the reducers much easier. They can be very dumb programs, using 
limited resources. If a reducer is given this segment of the mapper output, all that is needed is for it to make a list of all the 
indices, as long as the key is the same. When the key changes, it can dump the key and list of node ids to the putput, and start over 
with the next key. In Python, this looks as follows: 

		import sys, string

		oldKey = None
		nodeList = []
		listSeparator = ", "

		for line in sys.stdin:
			data_mapped = line.strip().split("\t")
			if len(data_mapped) != 2:
				# Something has gone wrong. Skip this line.
				continue

			thisKey, thisNode = data_mapped

			if oldKey and oldKey != thisKey:
				nodeList.sort()
				numNodes = len(nodeList)
				nodeStr = string.join(nodeList,listSeparator)
				print "{0}\t{1}\tTotal\t{2}".format(oldKey,nodeStr,str(numNodes))
				oldKey = thisKey; # reduntant but whatever
				nodeList = []        

			oldKey = thisKey
			nodeList.append(thisNode.strip('\"'))

		if oldKey != None:
			nodeList.sort()
			numNodes = len(nodeList)
			nodeStr = string.join(nodeList,listSeparator)
			print "{0}\t{1}\tTotal\t{2}".format(oldKey,nodeStr,str(numNodes))

When we feed the sorted output of the mapper to this function (using `cat | ./mapper9.py | sort | ./reducer9.py`), the result is, again, 
a long file. The section of the file pertaining to the section of mapper output we saw above, is here:

		any	2741, 7185	Total	2
		application	26454	Total	1
		are	2312	Total	1
		as	2312, 2312	Total	2
		audio	2312	Total	1
		be	2312, 6361	Total	2
		being	2741	Total	1

And that is all. As you can see, in this particular example I have asked the reducer to do a little bit more work and tell me how many 
times each word has appeared, but the essence doesn't change. 

### The caveat

I chose here two examples which are easy to follow through, but could give an oversimplified immpression of the actual issues in using the MapReduce framework. The types of problems I described above(counting and creating inverted index), are relatively easy, in that they do not tax the computer cluster with too much communication between the different machines carrying out individual map tasks. As an example of a situation which is more complicated, consider any algorithm which involves matrix multiplication, e.g. the matrix multiplication common in algorithms for calculating the page rank of web pages (see excellent discussion [here](http://www.mmds.org/)). In such cases, relatively efficient solutions exist within the MapReduce framework (by subdividing the matrices to small blocks which are processed in parallel). However, there  exist problems (matrix inversion) for which -to my knowledge- efficient implementations are still under way.  [Same post on wordpress](http://oligotropos.wordpress.com/2014/10/13/mapreduce-lemmings-in-the-21st-century/).


