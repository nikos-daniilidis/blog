---
layout: post
title:  "Physics in war and peace: Historic trends in the Physical Review Archive"
date:   2014-06-09 00:00:01
categories: jekyll update
---
This is going to be somewhat of a lengthy post, but I found doing the research behind it quite interesting. The subject is to identify historical trends in Physics research, based on the records of the Physical Review Archive. What I will write today comes from analysing a record of the titles of articles which appeared in the Physical Review from 1900 until 1970. The record consists of around 47000 article entries.

There are many cuts one can take through this data. Today I will only look at:

    * How many articles appeared each year?
    * How many unique terms appeared each year?
    * How did the typical probability of appearance  of a certain term a certain year vary?

The questions are answered in the two following figures:

![number of publications]({{ site.baseurl }}/assets/2014-06-01-phys-rev-1900-1970/numbers-vs-year-plot.png)

![term probabilities]({{ site.baseurl }}/assets/2014-06-01-phys-rev-1900-1970/probabiliites-vs-year-plot.png)

(note that in both figures the y axis shows the logarithm of the plotted quantity).

These two plots are fascinating! You can see the 20th century industrial and military history of the United States through the Physical Review publication titles. Moreover, you can get a glimpse of the trends within Physics itself.

Around the turn of the century there is a modest number of publications. Perhaps good publications were ending up in the prestigious European journals at the time. In addition, there is fairly small diversity in the topics which are covered in publications, some 0.5 % probability of a given term to appear in a randomly chosen title. In all, the statistics during this period is poor.

Around 1920, we seem to enter a period of intensified research. By looking at the terms which are the most popular during that time (in an upcoming post), it is pursuit of the early quantum physics which seems to prevail. The fashionable terms of the 20's are related to spectroscopy, as compared to electricity and thermodynamics which seemed to prevail before 1920.

The boom of quantum physics continues until the 1930's, when research seems to take a hit and stay flat in terms of quantity and diversity of publications. That's no surprise, with the entire economy sinking, why would Physics research stay afloat? Interestingly, the New Deal fails to give research even a fraction of its previous momentum, although it does make for a minor uptick starting between 1933 and 1935.

The most striking feature in these plots is what happens between 1940 and 1950. The war years cause an incredible dip to the number of works that get published (research becoming war oriented and secret?). The same is true of the number of terms found in the published works, pointing to reduced diversity of the published works (although the hit to diversity is less severe than the hit to publishability). Both these trends can probably be traced to a 'cut the fat' approach to research during the war years.

What is very interesting, is that the dip in diversity and publishability happens before the beginning of the Manhattan project in 1942. Thus, we cannot blame all of this change to 'nucular' technology. There were a number of technologies which were very essential to the war effort, and very much related to Physics research: RF engineering to develop Radar technology and communications systems, missiles and ballistics, submarine technology, optics and camouflage, 'War Metallurgy' of steels, and the effects of 'Impact and Explosion' [e.g.](http://www.loc.gov/rr/scitech/trs/trsosrd.html). Of course, we're always tempted to think of the Manhattan project as the most decisive contribution of Physicists to the war effort. That is not an unjustified conclusion: After the end of this war, and of the Manhattan project, we became for the first time able to destroy our entire planet and most life on it within a matter of days!

At the end of the war, research diversity and publishability starts a fast recovery which continues until around 1950. After that, the 1950's hit a mysterious period of stagnation, if not decline. Does this have to do with the unstable economic conditions of the 1950's and the slump at the end of the decade? Unfortunately I am a Physicist, not an Economist to say . The issue is possibly more complex than a simplistic poor economy, poor research relation. During the 1950's the economy was very much focused on making cars and electrical technology, areas where Physics innovation was not particularly useful, especially with the legacy of the war years. Indeed, the list of popular Physics research terms during the 1950's revolves around nuclear and high-energy Physics.

Continuing on to the 1960's, one understands why so many people are nostalgic about it! The stagnant trends in research turn around circa 1960. There is an upward trend in the number of Physics terms in use (a proxy of new ideas or diversity), but an even faster increase in the number of publications that appear each year. This probably has to do with the improvement of economic conditions in the 1960's but partially also due to the birth of electronics. Information technology and microelectronics take off during this era, and Physicists are frontrunners in this race. It is the birth of 'Solid State Physics'.

There is one strange aspect of the total number of publications and the total number of research-related terms in the sixties: It is the first time when the gap between the number of Physics terms and the number of articles closes (especially after 1967). Is this a merely a result of homogenizing and smoothing out of the terminology used in Physical Review publications, a result of changes in the diversity of papers appearing in the Physical Review, or does it signify a slowing down of the pace of expanding science to new realms -fewer new terms being introduced, while older terms are studied in more depth. Perhaps a good question for historians of science.

The  data for this post was kindly provided by the American Physical Society. To carry out the data analysis, I used the Python Pandas, Python Scikit-Learn, and Python NLTK libraries. You can see more about the technical details of the data analysis on the git repository for [this project](https://github.com/nikos-daniilidis/haystack) (notebook converted to [html](http://nikos-daniilidis.github.io/haystack/parse-pr-xml.html) is here). [Same post on wordpress](http://oligotropos.wordpress.com/2014/06/01/physics-in-war-and-peace-historic-trends-in-the-physical-review-archive/).
