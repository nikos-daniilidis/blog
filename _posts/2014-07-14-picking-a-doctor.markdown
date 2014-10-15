---
layout: post
title:  "Picking a doctor!"
date:   2014-07-14 17:54:02
categories: jekyll update
---
After the two previous posts discussing the collection and cleaning up of doctor ratings data, it is time for the fun part: 
Picking the best rated doctor. The poor man's approach is: Given the data, average all the ratings for each doctor, and 
whoever has the largest average rating wins. But the question we really want to answer is: Given the data, which doctor has 
the _highest probability_ of having a high score, say above 4.8 out of 5?

More concretely, consider the following example:

    * Doctor A: 4 out of 4 ratings. Average rating: 4.5/5. Variance of ratings: 0.5/5.
    * Doctor B: 4 out of 4 ratings. Average rating: 4.6/5. Variance of ratings: 0.3/5.
    * Doctor C: 3 out of 4 ratings. Average rating: 4.5/5. Variance of ratings: 0.5/5.

It is not clear that one doctor is better than the others: the mean rating and standard deviation do not capture all of the 
information, and missing values complicate things even more. What we need is a _single measure_ of quality for each doctor. 
We also want to be able to estimate the _probability distributions_ of the doctors' quality measures, given the entire ensemble 
of ratings that we have in our possession. In what follows, I am using Principal Components Analysis (PCA) to derive a unique 
rating parameter from the data set, and boostraping to estimate the distribution of ratings for each doctor.

Let me try to break this down, in case it was too technical: You can think of PCA as a way to draw a 'best fit' line (or plane in 
higher dimensions) through a cloud of data, by treating all the variables on equal footing (as opposed to least squares linear 
regression, which singles-out one of the vartiables to find the best fit). PCA allows me to compact my data from a $P$-dimensional space to lower dimensions, by picking those combinations of parameters which explain the maximal amount of variance in the data. In plain English: with PCA I can find a single linear combination of the four ratings (not necessarily the mean) which explains as much of the variance in the data as can be explained using a single parameter.  Moreover, 
[under certain assumptions](http://en.wikipedia.org/wiki/Principal_component_analysis#PCA_and_information_theory), the 
transformation is optimal in terms of compacting the information in the data to as few parameters as possible: If I keep the first component, I am guaranteed to have the maximum amount of information that I could have in a single parameter, given my observations.

So, here is the strategy: I will perform PCA on the data and keep only the component which explains the maximum amount of variance in the data. Let's call this component the 'PCA score'. PCA is not suited to data with missing values, so I will use one of the data imputation methods I developed before. I do not want to throw  away data, but also do not want to introduce too much bias. So, I will use stochastic mean imputation, and repeat the analysis many times to acquire good statistics. The randomness I introduce this way, will influence all PCA scores in a way which reflects the missing information in the data. This way, the PCA score of each doctor will become a probability distribution, based on which I can then select the best doctors.

There are two more caveats here: First, by introducing normally distributed random ratings to impute the missing values, I am making an implicit assumption about the probability ditributions of missing ratings. Second, the algorithm described above, will be biased toward doctors who have all four ratings and have high scores in all four. For example, doctors with equally high, or higher ratings in only three websites will be less favored by the stochastic imputation step. To circumvent this, before the imputation I will add a step of randomly removing one out of four ratings from all doctors who have all four ratings. In summary, here is the algorithm I will use:

		get rating data
		repeat N times
		{
		randomly remove 1 out of 4 ratings where all 4 ratings are present
		do mean stochastic imputation on data
		do PCA on imputed data
		save PCA score for all doctors for this iteration
		}
		analyse the PCA score distributions from the N saved iterations

At the end of this process, I will have a _distribution_ of PCA scores for each doctor. Then, I can use several different criteria to rate the doctors based  on the PCA score distributions.

The first step is to check the implementation of a single iteration of the PCA algorithm. Here I perform PCA on a single realization of the data after stochastic mean imputation. I have implemented the PCA transformation in such a way that my PCA score is in the range between 0 and 1. The first thing is to check how the PCA score compares to the mean rating.

![pca score vs mean](/assets/2014-07-14-picking-doctor/pca_vs_mean.png)

It is reassuring to see that the trend is linear. We also see that the  relation is not purely monotonic. This reflects the difference between PCA and the mean, which I already alluded to.

The next thing to check is how the distribution of PCA scores looks for this particular realization of stochastic mean imputation.

![pca score histogram](/assets/2014-07-14-picking-doctor/single_pca_histogram.png)

The distribution looks similar to the ratings histograms from individual websites, which I showed in my previous posts. Of course, this is one particular draw from a distribution of PCA ratings.

This all looks as expected, and it is time to run the bootstrapping analysis on the ratings dataset. I ran the bootstrapping analysis twice. The first time, I did what I would call "single bootstrap". I only did stochastic mean imputation on the dataset, but I left all the doctors with four ratings untouched. As I said in the introduction, this adds bias in favour of the doctors who have all four ratings. That's what motivated the 'double boot-strapping' procedure: I will remove ratings at random to remove this bias (let's call this exputation). The question is, why stop at 3 ratings. You could be biased against 2 and 1 rating as well. My short answer to this is: I made this choice partially on intuitive, partially on practical grounds (e.g. cpu time). In general, one should try more sophisticated exputation/imputation schemes.

To highlight the difference between single and double bootstrap, I am plotting the histograms of PCA scores for two doctors, one of whom having all four ratings and the other one having only two ratings in the original dataset. Here are the histograms (Doctor 7 in this example had 2 missing ratings, while Doctor 6 had 0 missing ratings):

![single vs double bootstrapping](/assets/2014-07-14-picking-doctor/single_vs_double_bootstrap.png)

The histograms give me a hint that the double bootstrap analysis was a step in the right direction. I am now getting more similar score distributions between doctors with missing ratings, and those who had no missing ratings, so I can be more confident that I am comparing different doctors on the same footing.

If you have accepted what I've done so far, we are finally at the last turn of the road! The final step is to decide who are the best rated doctors based on their probability of having a score above a certain value. I set the score threshold to different thresholds between 0.99 and 0.80, and sum up the estimated probabilities that the PCA score is above a certain threshold. Here is an example output:

|  name   | P(score> 0.99) | P(score> 0.95) | P(score> 0.90) | P(score> 0.85) | P(score> 0.80) |
|---------|:---------------:|:--------------:|:--------------:|:--------------:|:--------------:|
| DOC#404 | 0.627 | 0.976  | 1 | 1 | 1 |
|---------|:---------------:|:--------------:|:--------------:|:--------------:|:--------------:|
| DOC#424 | 0.649 | 0.971  | 1 | 1 | 1 |
|---------|:---------------:|:--------------:|:--------------:|:--------------:|:--------------:|
| DOC#310 | 0.655 | 0.970  | 1 | 1 | 1 |
|---------|:---------------:|:--------------:|:--------------:|:--------------:|:--------------:|
| DOC#93  | 0.642 | 0.970  | 1 | 1 | 1 |
|---------|:---------------:|:--------------:|:--------------:|:--------------:|:--------------:|
| DOC#188 | 0.645 | 0.968  | 1 | 1 | 1 |
|---------|:---------------:|:--------------:|:--------------:|:--------------:|:--------------:|
| DOC#184 |	0.610  | 0.966 | 1 | 1 | 1 |
|---------|:---------------:|:--------------:|:--------------:|:--------------:|:--------------:|
| DOC#76  |	0.657 |	0.963  | 1 | 1 | 1 |
|---------|:---------------:|:--------------:|:--------------:|:--------------:|:--------------:|
| DOC#262 | 0.627 | 0.956 | 1 | 1 | 1 |
| DOC#506 | 0.585 | 0.935 | 1 | 1 | 1 |
|---------|:---------------:|:--------------:|:--------------:|:--------------:|:--------------:|
| DOC#486 | 0.528 | 0.921 | 1 | 1 | 1 |


And this is it, that's what I wanted. Now I can decide what is a reasonable threshold, and choose the 'best rated' doctor.

To do this analysis I used the following Python libraries: Pandas, Scikit-Learn, Numpy, and Matplotlib. You can find the code on which this work was based on my Github repository as an [IPython notebook](https://github.com/nikos-daniilidis/find-md/blob/master/find_me_a_doc_pca-score.ipynb), and converted to [html](http://nikos-daniilidis.github.io/find-md/find_me_a_doc_pca-score.html). [Same post on wordpress](http://oligotropos.wordpress.com/2014/07/14/picking-a-doctor/).


