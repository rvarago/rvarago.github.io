---
layout:	"post"
title:	"Moving Average Filter"
---

* * *

Hello,

Today, I'm going to talk about a simple and commonly used linear filter known
as moving average filter.

We'll discuss the importance and usage of this filter, some aspects of its
description and along the text, I'll give a implementation of a moving average
filter in MATLAB for smoothing a noisy signal.

### Motivation and Description

When analyzing a signal, it's often desired to understand its trend, rather
than its value at every point of the independent variable.

For example, in Economics, suppose that we want to see how the Gross Domestic
Product (GDP) moves in the long term, basically we're looking whether it goes
up, down or remains constant over a long period of time.

Nonetheless, a signal normally contains more than a trend component, but it
may has a combination of trend, seasonality and noise. This is the known as
classic model for time series modelling [1].

According to what was exposed above, we can understand a time series as a
sequence of values that evolve over time in a random manner, or in other
words, as a possible realization of some stochastic process. Here, we've used
time as an independent variable, but it could be any other ( _e.g._ spatial
coordinate), furthermore it could be a vector ( _e.g._ time and the three
spatial coordinates).

Let's consider a time series in the discrete domain.

Suppose for simplicity, that our time series only contains a deterministic
trend and a noise component. Suppose too, that our model is additive, that is,
the trend is summed with noise:

![](/assets/img/2016-09-05-moving-average-filter_0.png)

Our goal is to smooth:

![](/assets/img/2016-09-05-moving-average-filter_1.png)

That is, to reduce the noise effect, such that we can focus in the trend
component, seeing how it evolve in a long term analysis.

### Noise Component and Filtering

To provide a practical approach, let's consider our trend as a linear
function:

![](/assets/img/2016-09-05-moving-average-filter_2.png)

Considering the size of our time series as:

![](/assets/img/2016-09-05-moving-average-filter_3.png)

The following MATLAB script creates this series and plots its graph.

Its result is shown in Figure 1 and it shows that we have a growing trend.

![](/assets/img/2016-09-05-moving-average-filter_4.png)

Figure 1: Trend Component.

The above plot shows the best scenario, where we haven't any noise effect,
hence there's no need of using a filter to smooth the signal.

But this isn't the signal which we often encounter in the real world. In most
applications, we have a noisy signal, _i.e._ one that was corrupted by noise.

In the following, it's shown a MATLAB script that corrupts our previously
defined time series with a normally distributed noise:

![](/assets/img/2016-09-05-moving-average-filter_5.png)

Such that:

![](/assets/img/2016-09-05-moving-average-filter_6.png)

This noise, which follows a Gaussian distribution, with zero mean, equally
distributed in frequency ( _i.e._ there's no serial correlation along the
signal samples) and additive effect, has a particular importance in the
Digital Signal Processing (DSP) domain, and it's denominated Additive White
Gaussian Noise (AWGN).

Figure 2 shows the plot of the noisy signal.

![](/assets/img/2016-09-05-moving-average-filter_7.png)

Figure 2: Noisy Signal.

Now, it's harder to assert about the trend, it seems that there's a moderated
growth, but there's not a clear confirmation of this assumption, someone less
careful can even say that the signal is simply floating around some straight
line.

Is it the case? Let's investigate it in more detail.

It's common, but not necessary required, to have a noise component of a very
high frequency compared to the trend. In this way, we can achieve our
objective of smoothing the signal by passing it through a low pass filter.

A low pass filter is a system which leaves the signal unchanged up to certain
frequency:

![](/assets/img/2016-09-05-moving-average-filter_8.png)

Called the cut-off frequency, and suppresses the signal components with
frequency greater than cut-off [2].

The simplest kind of low pass filter is called ideal low pass filter, and it's
basically a system with frequency response:

![](/assets/img/2016-09-05-moving-average-filter_9.png)

That is **one** for frequencies below:

![](/assets/img/2016-09-05-moving-average-filter_10.png)

And **zero** elsewhere, _i.e._ :

![](/assets/img/2016-09-05-moving-average-filter_11.png)

Although its simplicity, it happens that this kind of filter has some
disadvantages, mainly due its non-causal impulse response, which makes
impossible implement a real time filter.

Fortunately, there are many others simple filters that can be adequately
applied, and besides its easy implementation, they provide sufficient results
for almost all situations. This is the case for investment analysis, where the
investor are observing the asset price and trying to guess its trend over
time.

One of the most used smoothing filters (low pass) is the Moving Average (MA)
filter.

For example, in the above case of a investor trying to guess the asset's
trend, an MA filter is one of the favorite choices, due to its easy
development and interpretation.

The MA causal impulse response:

![](/assets/img/2016-09-05-moving-average-filter_12.png)

For some defined number of taps:

![](/assets/img/2016-09-05-moving-average-filter_13.png)

Is defined by:

![](/assets/img/2016-09-05-moving-average-filter_14.png)

This kind of filter is generally classified as Finite Impulse Response (FIR)
filter.

We can interpret this filter as a system which adds the first:

![](/assets/img/2016-09-05-moving-average-filter_15.png)

Values, then divides this sum by:

![](/assets/img/2016-09-05-moving-average-filter_16.png)

That is, gets the average.

After this, it replaces the oldest value for the most recent entry, then
repeats the process of averaging. Therefore we have a average that is
"running" through the time series, and at every step, it replaces the oldest
value by the next one in the series.

One drawback of this filter, is that it has a delay of:

![](/assets/img/2016-09-05-moving-average-filter_17.png)

Due to the need of accumulating them for the averaging process. Furthermore,
its frequency characteristic is not so good in comparison with more
complicated filters, but because of its straightforward implementation, it's
very used anyway.

Finally, the code snippet below shows a implementation of the MA filter.

Where the filtered signal is:

![](/assets/img/2016-09-05-moving-average-filter_18.png)

The Figure 3 shows the plot of.

![](/assets/img/2016-09-05-moving-average-filter_19.png)

In this plot, we can see the delay effect previously described.

But more importantly, now we can make a more educated guess about the time
series trend, which is growth, in accordance with the deterministic trend
component shown in Figure 1.

### Conclusion

In this article, we'd talked about some concepts related to signals and the
problem of smoothing a time series corrupted by noise. After it, we introduced
the MA filter as a simple, but at some extend, efficient solution for this
issue.

Finally, we developed an implementation of an MA filter in MATLAB and it's
replicated as a whole here.

### Bibliography

[2] OPPENHEIM, A. V. and SCHAFER, R. W. Discrete-Time Signal Processing. 3E.
Published By Pearson.


***
*Originally published at https://medium.com/@rvarago*
