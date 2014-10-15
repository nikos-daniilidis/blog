---
layout: post
title:  "Data imputation: the brute, the stochastic, and the aggressive"
date:   2014-07-12 10:00:00
categories: jekyll update
---
I am having a second look at doctor ratings in the SF bay area, and using it as a platform to develop some methods for problems of this kind. This time, I collected more data, and I implemented a few additional methods to deal with missing data.  This was necessary before I could build algorithms working on the doctor ratings data, as the existing Pandas capabilities for dealing with missing data do not provide many options.

I used two methods of data imputation (filling in missing data) for the doctor ratings. The crude method was: just drop all doctors who have any missing ratings. Obviously, this ends up in a massive loss of data, and can add some bias. The second method I tried was mean imputation: for each doctor fill-in the missing ratings by using the mean of the existing ratings. This is still fairly crude and could also add some bias. A slight improvement is possible by adding a stochastic element: whenever there are missing values, generate the missing values as random draws from a normal distribution with the mean and the standard deviation of the existing values. I have implemented these two methods in functions which work on Pandas DataFrames, although the former method (let's call it aggressive imputation) is a one-liner in Pandas.

The first thing I did was collect more data, bringing the total number of doctors to 930, of which ratings are available for 750. Now I can plot the same combination of scatter plots and histograms as I did on the previous blog post.

![raw data](/assets/2014-07-12-imputation/new-data-raw.png)

The histograms and $$ R^2$$ values are slightly different, but a lot of scatter remains. Most of the variance in the data cannot be explained by the simple regression models of the form $$ Y = \beta_0+\beta_1 X$$ which I use here.

After aggressive imputation, I am left with 126 out of 930 doctors. However, the remaining doctors have all four ratings from the websites, which can make it easy to move ahead with data analysis. I proceed to inspect the data after aggressive imputation:

![aggressive imputation](/assets/2014-07-12-imputation/new-data-aggressive-imput.png)

The level  of correlation between different ratings did not increase after getting rid of the missing values. This is evident from the $$ R^2$$ values for the linear fit models, but also visually from the amount of scatter in the scatter plots.

I implemented two methods of mean imputation. "Brute" mean imputation replaces missing values with the means of existing corresponding values. "Stochastic" mean imputation replaces the missing values with random draws from a normal distribution with the mean and standard deviation of the existing corresponding values. Here I have a look at how they perform. Brute imputation results in this visualization:

![mean imputation](/assets/2014-07-12-imputation/new-data-brute-mean-imput.png)

The first thing to note is that I now get a large density of points near and along the diagonals of the scatter plots. This is very promising! It means that, although the ratings from individual websites show significant scatter when compared one by one, the aggregate picture which emerges by taking all ratings into account shows a large degree of correlation. A second way to see this is to look at the $$R^2$$ coefficients in the linear fits. The values have improved significantly over those I was getting in the original data without imputation, but also over the values I was  getting after aggressive imputation.

The large density of points on the diagonals of the scatter plots reveals a limitation of the "brute" mean imputation method. Here's why: If I am missing two out of four ratings for a particular doctor, then I will fill their values with the same number (the mean of the existing ones). When I make a scatter-plot, these values will appear on the diagonal, creating an illusion of perfect correlation - a kind of false optimism of the method. This is why stochastic mean imputation is a good idea: By adding a random element to the imputed values, I reduce this fictitious correlation and counter balance the false optimism. In terms of fitting models to the imputed data, we can expect that adding the stochastic element will reduce  the amount of bias we introduce because  of the fictitious correlations. Here  is how the stochastic method looks:

![stochastic imputation](/assets/2014-07-12-imputation/new-data-stoch-mean-imput.png)

As you can see, I am now getting a lower degree of correlation than with the brute method. This is clear from the $$ R^2$$ values in the linear fits, which are now 10-30% lower than they were after the "brute" mean imputation. This is a  good thing! It means we have partially solved the problem of over-optimism of our model.

One last thing to note, is that the linear regression model I am using to fit a line through the scatterplots suffers from "regression toward the mean". In other words, linear regression tends to give predictions which are closer to the mean than to the extremes. It is a conservative model! The  most obvious symptom of this is when your reaction to looking at a fit is: "this fit is wrong, it is way too flat". Looking at the scatter plots in both data sets with mean imputation, the linear fits certainly have this look. It is not a problem of the algorithm, but rather a feature  of the linear regression model that you are looking at.

Now that I have a "complete" data set, I am taking a next step into multiple linear regression. This will allow me to evaluate which of my ratings matter most, and whether I would be able to ignore some in designing a doctor-choosing algorithm.

In this work I used the Python Pandas and NumPy libraries. You can find the code on my GitHub repository, as [IPython notebook](https://github.com/nikos-daniilidis/find-md/blob/master/find_me_a_doc_imputation.ipynb), as well as converted to [html](nikos-daniilidis.github.io/find-md/find_me_a_doc_imputation.html). [Same post on wordpress](http://oligotropos.wordpress.com/2014/07/12/ihmo-continued-the-brute-the-stochastic-and-the-aggressive/).


