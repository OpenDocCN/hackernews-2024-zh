<!--yml
category: 未分类
date: 2024-05-27 14:33:59
-->

# SI Units for Request Rate

> 来源：[https://two-wrongs.com/si-units-for-request-rate](https://two-wrongs.com/si-units-for-request-rate)

*Request rate* is the number of requests that arrive, or are serviced, or leave, during some period.

It’s surprisingly common for people to speak of a request rate without specifying what the length of the period is. I have even seen dashboards that don’t have a fixed period for the metrics query – the request rate becomes measured per whatever aggregation interval the dashboard deems appropriate for the window size at the time. If you zoom out, you get a higher request rate. If you move the window to a high-resolution screen, you get a lower request rate.

We should specify the period length in the query to the metrics database, so everyone sees the same request rate regardless of how many pixels their dashboard occupies at the time.

The good period length to use is the second. Request rates should be measured as the number of requests per second.¹ I have met some people who measure requests per minute. Don’t be those people. This sounds like it would have an si unit, i.e. we should be able to say something like “our request rate is 57 watts.” Except obviously not watts.

* * *

Turns out there are two si units that both could fit:

*   The hertz, or Hz, is the si unit of frequency. It is defined as one event per second.
*   The becquerel, or Bq, is the si unit of (radio)activity. It is also defined as one event per second.

Why are there two units for the same thing? A physicist that hears that an event occurs at 4 Hz will assume that there is one event exactly every 250 milliseconds. The hertz unit is strongly associated with periodic behaviour. Radioactive decay is not that well behaved, and will happen only on average with the given frequency. A sample that decays at 4 Bq may decay zero times one second, and then 9 times the next.² Assuming a Poisson distribution, these will be rare events, but if you look at the sample for an hour, you have a better than 50 % chance of observing that exact sequence of decays.

It makes sense then that we would say the request rate is 500 Hz when we talk about highly regular load testing, where one request is issued consistently every 2 ms. But if it’s about organic traffic that happens to arrive at an average of 500 times per second, then maybe saying 500 Bq is more appropriate.

This is also convenient when we are nearing request rates static web servers or caches handle. Saying “ninety kilobecquerel” and writing “90 kBq” is a lot more convenient than “ninety thousand requests per second” and “90,000 requests/s”.³ A reader suggested inventing the unit “rips” instead, which I like both the idea and sound of, but I’m a sucker for bending standards to my will.

* * *

*Except*! It seems that – in contrast to the hertz which is a general unit – the becquerel is meant to be specifically about radioactive decay. There’s no si unit for arbitrary events that happen on average with a certain frequency. I will keep using the becquerel for request rates, and I hope that 50 years from now, we will have forgotten the silly mistake of thinking it was only about nuclear decay.