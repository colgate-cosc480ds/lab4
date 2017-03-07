# Lab 4

Welcome to the *fourth* COSC 480 Data Science Lab!  You are encouraged to work with a partner for this lab.

This lab is due **Monday, March 20 2017 at 11:59pm**.  Submission instructions are similar to previous labs.  *When you finish, please be sure to write a commit message!*  

## Setup Instructions

You should not need to make any updates to your VM.

## Overview

In this assignment, you will look for fraud in election returns from the disputed 2009 Iranian presidential election. You will examine the least significant digits of the vote totals — the ones place and the tens place.

The ones place and the tens place don't affect who wins. They are essentially random noise, in the sense that in any real election, each value is equally likely. Another way to say this is that we expect the the ones and tens digits to be uniformly distributed — that is, 10% of the digits should be 0, 10% should be 1, and so forth. If these digits are not uniformly distributed, then it is likely that the numbers were made up by a person rather than collected from ballot boxes. (People tend to be poor at making up truly random numbers.)

It is important to note that a non-uniform distribution does not necessarily mean that the data is fraudulent data. A non-uniform distribution is a great signal for fraudulent data, but it is possible for a non-uniform distribution to appear naturally.


## Task Details

### Task 1: Read and clean the election data

There were four candidates in the 2009 Iranian election: Ahmadinejad, Rezai, Karrubi, and Mousavi. The file `election-iran-2009.csv` contains data, reported by the Iranian government, for each of the 30 provinces. We are interested in the vote counts for each of these candidates. Thus, there are 120 numbers we care about in the file.

Write a function called `iran_votes` that takes no arguments and returns a list of integers corresponding to the votes for each candidate from each province.  The order of the integers does not matter.

Here is a sample of what your function should return (your order may differ):

	iran_votes()[:5]
	[1131111, 623946, 325911, 1799255, 199654]

Write a similar function called `us_votes` that does the same for the 2008 U.S. election which had six candidates (Obama, McCain, Nader, Barr, Baldwin, McKinney) in 54 voting regions (states and other voting blocks like D.C., Puerto Rico).  There are 302 non-blank numbers.

You will notice that the data contains double-quotes and commas. Furthermore, some entries are blank.  Blank entries should be *removed* and not simply counted as zero.

### Task 2: Make and plot histograms

With the voting data above, we only care about the ones and tens digits.  Write some code that extracts out the ones and tens digits from the votes that you extracted in the first step.  So for example, if you just had the following 5 vote tallies: 
	
	[1131111, 623946, 325911, 1799255, 199654]

you would want to extract out the following 10 digits: `[1, 1, 4, 6, 1, 1, 5, 5, 5, 4]`.

Then write some functions to plot the resulting digit distribution.  Specifically, write functions `plot_iran_ones_and_tens_histogram` and `plot_us_ones_and_tens_histogram`.  These functions take no arguments and should produce plots exactly as shown in the example files `example-iran-plot.png` and `example-us-plot.png` except that the line for Iran/US will be different (the example plots use totally made up data).

The y-axis in these plots reports the frequency of occurrence of each digit (x-axis).  The frequency is the number of times the digit appears (in either the ones or tens place) divided by the total number of digits.  The line labeled "ideal" refers to the ideal distribution assuming that the numbers are generated uniformly at random.

Call these functions in the python notebook.

### Aside: Smaller samples have more variation

After completing step 2, you should find that the Iran election data looks rather different from the expected flat line at Frequency of 0.1.  Is this different enough that we can conclude that the numbers are probably fake? No — you can't tell just by looking at the graphs we have created so far. We will use a more principled, statistical ways of making this determination.

With a small sample, the vagaries of random choice might lead to results that seem different than expected. As an example, suppose that you plotted a histogram of 20 randomly-chosen digits (10 random numbers, 2 digits per number).  This will look much *worse* than the Iran data, even though it is genuinely random.  (In fact, `example-us-plot.png` is exactly that: the 20 random digits.)  Of course, it would be incorrect to conclude from this experiment that the data for this plot is fraudulent and that the data for the Iranian election is genuine. Just because your observations do not seem to fit a hypothesis does not mean the hypothesis is false — it is very possible that you have not yet examined enough data to see the trend.

### Task 3: Hypothesis testing

We will use hypothesis testing as a tool to investigate election fraud.  Recall the ingredients of a hypothesis test:

- a null hypothesis
- a test statistic
- a procedure for estimating the p-value

For this problem, our null hypothesis `H_0` is that the ones and tens digits are independent and identically distributed random samples drawn from a uniform distribution over the numbers {0, 1, 2, ..., 9}.  We will test this null hypothesis on  first on the Iran data and then again on the U.S. data.

Our *test statistic* will be the sum of squared differences from the expected value.  Here, the expected value is 0.1 for each digit.  Thus, if we observe:

	[0.3, 0.2, 0.2, 0.0, 0.0, 0.0, 0.1, 0.1, 0.05, 0.05]

The sum of squared differences would be:

	(0.3-0.1)^2 + (0.2-0.1)^2 + (0.2-0.1)^2 + 
	(0.0-0.1)^2 + (0.0-0.1)^2 + (0.0-0.1)^2 + 
	(0.1-0.1)^2 + (0.1-0.1)^2 + (0.05-0.1)^2 + 
	(0.05-0.1)^2

which is equal to
	
	(0.2)^2 + 5 x (0.1)^2 + 2 x (0.0)^2 + 2 x (0.05)^2 = 0.095

In case you're curious, this test statistic is an example of the [Chi-square test statistic](https://en.wikipedia.org/wiki/Chi-squared_test).

Compute this test statistic on the Iran data.  Let's call this the *observed value of the test statistic* or *observed statistic* for short.  You should get something very close to `0.00739583333333`.  Is this unusually big?  Unusually small?  We can't simply look at this number and assess its significance.  However, we can see how likely it would be if the data really were sampled uniformly.  That's the idea of the next step.

Our *procedure for estimating the p-value* is as follows.  We want to calculate the following quantity: what is the probability, under the null hypothesis, of generating a test statistic at least as large as the *observed statistic*?  We can estimate this probability using simulation.

Repeat the following 10,000 times:

- generate a random sample of data from the uniform distribution over the digits {0..9}
- compute the test statistic on the sample
- increment a counter if the test statistic on the random sample is greater than or equal to the *observed statistic*

The p-value can be estimated as the fraction of times (in the 10,000 simulations) the test statistic on the random sample is greater than or equal to the *observed statistic*.

**Important detail**: the *size* of the random sample you take in the above simulation is critical.  If you take a random sample of 1,000,000 digits, their frequencies will be very close to 0.1 and the test statistic will be close to zero; if you take a random sample of 20, their frequencies may vary widely giving you test statistic values all over the place.  How many samples should you take?  The sample size should be *equal to the number of digits* that were used to calculate the *observed statistic*.  Note: this number is *different* for Iran and the U.S.

Write the functions `pvalue_iran` and `pvalue_us`.  These functions take no arguments and simply return the estimated p-value.

Call these functions in the python notebook.  Please include some formatting (i.e., some print statements describing what is being calculated).

### Task 4: Interpret your results

In the python notebook, write a brief (1-2 paragraphs) discussion of your results.  You might start with the outcome of the hypothesis test.  Then you might move on to the bigger question of election fraud.  

If you're curious, the Iran results were analyzed statistically for fraud.  You can read more about it [here](https://arxiv.org/abs/0906.2789).  The analysis differs somewhat from what we did here but the spirit is the same.



##### Acknowledgments

This assignment is adapted from an assignment by Bill Howe at the University of Washington.  Some of the text above was copied from that assignment.  Other aspects of the assignment have been changed considerably.

