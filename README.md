# A/B-Testing
## Experiment Overview: Free Trial Screener

In the experiment, Udacity tested a change where if the student clicked "start free trial", they were asked how much time they had available to devote to the course. If the student indicated **5 or more hours per week**, they would be taken through the checkout process **as usual**. If they indicated **fewer than 5 hours per week**, a message would appear indicating that Udacity courses usually require a greater time commitment for successful completion, and **suggesting that the student might like to access the course materials** for free. At this point, the student would have the option to continue enrolling in the free trial, or access the course materials for free instead. This screenshot shows what the experiment looks like.

![Experiment Screenshot](https://github.com/mcgradyjason/AB-Testing/blob/master/Final%20Project-%20Experiment%20Screenshot.png)

The **hypothesis** was that this might set clearer expectations for students upfront, thus reducing the number of frustrated students who left the free trial because they didn't have enough time—**without significantly reducing the number of students** to continue past the free trial and eventually complete the course.


## Metric Choice
### Invariant Metrics

* Number of cookies: That is, number of unique cookies to view the course overview page. (dmin=3000)
* Number of clicks: That is, number of unique cookies to click the "Start free trial" button (which happens before the free trial screener is trigger). (dmin=240)
* Click-through-probability: That is, number of unique cookies to click the "Start free trial" button divided by number of unique cookies to view the course overview page. (dmin=0.01)

The choice of invariant metrics are listed above. Since the unit of diversion is a cookie, it's better to choose number of cookies rather than user-ids as invariant metrics for sanity check, we definitely want to make sure that control group and experiment group have equal amount of users assigned. 

Number of clicks is also favored, because it's a population sizing metrics that should be equally assigned to control and experiment group. CTR is simply a number calculated by number of cookies and number of clicks, this metrics is not expected to change while number of cookies and number of clicks are fixed.

### Evaluation Metrics

* Gross conversion: That is, number of user-ids to complete checkout and enroll in the free trial divided by number of unique cookies to click the "Start free trial" button. (dmin= 0.01)
* Retention: That is, number of user-ids to remain enrolled past the 14-day boundary (and thus make at least one payment) divided by number of user-ids to complete checkout. (dmin=0.01)
* Net conversion: That is, number of user-ids to remain enrolled past the 14-day boundary (and thus make at least one payment) divided by the number of unique cookies to click the "Start free trial" button. (dmin= 0.0075)

Selected evaluation metrics are listed above, the reason why gross conversion is chosen is that this metrics is expected to be lower in the experiment group compared to control group. The number of students who fininsh check out after clicked free trial button are expected to be reduced due to the warning in the experiment group. Given that number of cookies is fixed, the gross conversion will be different between two groups, which is a part of null hypothesis.

Retention is expected to be different between control and experiment groups. Null hypothesis states that showing up the warning will not reduce the number of students who past the free trial and finish the course. Like we discuss above, number of user-id to complete checkout will be expected to different between control and experiment groups, the retention will be also different, which makes it a good metrics for testing null hypothesis.

Net conversion should be roughly the same, since number of user-ids pass the free trial and number of cookies to click the button is fixed according to null hypothesis like we discussed above, it is also a good evaluation metrics.

### Measuring Variability

Baseline values can be found [there](https://docs.google.com/spreadsheets/d/1MYNUtC47Pg8hdoCjOXaHqF-thheGpUshrFA21BAJnNc/edit#gid=0)

Given that baseline CTP on `Free Trial` button is `0.08`, and sample size of 5000 cookies visited homepage everyday. We can calculate number of cookies click `Free Trial` button is `5000 * 0.08 = 400`. The number of user-ids to remain enrolled past the 14-day boundary equals to `0.20625 * 400 = 82.5`.

The baseline gross conversion is `0.20625`, retention is `0.53` and net conversion is `0.1093125`. Assume that they follow binoimial ditribution, and standard deviation should be calculated based on `sqrt(p * (1-p) / N)`.

* Standard deviation of gross conversion: p = 0.20625, N = 400, SE = 0.0202
* Standard deviation of retention: p = 0.53, N = 82.5, SE = 0.0549
* Standard deviation of net conversion: p = 0.1093125, N = 400, SE = 0.0156

| Evaluation Metric | Standard Deviation |
|:-------------------:|:--------------------:|
| Gross Conversion  | 0.0202 |
| Retention         | 0.0549 |
| Net Conversion    | 0.0156 |

### Sizing
#### Number of Samples vs. Power

First, we decided not to use Bonferroni correction, since the evalution metrics we have chosen are closely correlated with each other, basically they are calcuated by three variables, which they will tend to move together while some variables are changing. And Bonferroni correction are more conservative when dealing with correlated evaluation metrics.

All the calculation is done by [online calculator](http://www.evanmiller.org/ab-testing/sample-size.html)

* Gross conversion: Given that `gross conversion = 20.625%`, `alpha = 0.05`, `1 - beta = 0.8`, `dmin = 1%`. Sample size needed per variation is `25835`. Noting that we should have both control and experiment groups with same sample size, and gross conversion captures number of cookies clicked the button while we need to calculate number of pageview. Recall that, `click-through-probability on button is 0.08`, so the number of pageview need is calculated by `25835 * 2 / 0.08 = 645875`.

* Retention: Similarly, given that that `retention = 53%`, `alpha = 0.05`, `1 - beta = 0.8`, `dmin = 1%`. Sample size needed per variation is `39115`. In this case, retention captures number of users who actually finish checkout, in order to convert it into number of pageview needed, we need `gross conversion` and `CTP on button` as followed: `39115 * 2 / 0.20625 / 0.08 = 4741212`.

* Net conversion: Given that `net conversion = 10.93125%`, `alpha = 0.05`, `1 - beta = 0.8`, `dmin = 0.75%`. Sample size needed per variation is `27413`, similarly, the number of pageview needed is `27413 * 2 / 0.08 = 685325`.

| Evaluation Metric | Pagenew Needed |
|:-------------------:|:--------------------:|
| Gross Conversion  | 645875 |
| Retention         | 4741212 |
| Net Conversion    | 685325 |

#### Duration vs. Exposure

When it comes to calculate duration, I soon realize that choosing `Retention` as evaluation metrics may not be appropriate in Audacity case. Even if we expose 100% of Audacity traffic to this experiment, which is `40000` unique cookies visited website per day, the duration of this experiment is `4741212 / 40000 = 118.53`, which is about 119 days. It is obviously too long to launch a single experiment with diverting full traffic of the whole site. On the other hand, it potentially exposes more business risk that customers might not like this change and lead to drop in pageview and enrollment. This will have a significantly negative impact on the business if this change will remain 100+ days.

At this stage, we should **only keep two evaluation metrics** : `Gross conversion` and `Net conversion`, since net converison requires more pageview than gross conversion, the duration is calcualted by `685325 / 40000 = 17.133125`, which is about 18 days with full site traffic. It is also practical to reduce the proportion of traffic exposed to experiment to lower the risk. In this case, I prefer to divert whole traffic to the experiment to get result as soon as possible.

## Experiment Analysis
### Sanity Checks

First, we sum up the [number](https://docs.google.com/spreadsheets/d/1Mu5u9GrybDdska-ljPXyBjTpdZIUev_6i7t4LRDfXM8/edit#gid=0) in control and experiment group seperately. The total number of cookies visited homepage in control group is `345543` and `344660` in the experiment group. The total number of cookies clicked "Free Trial" button in the control group is `28378` and `28325` in the experiment group.

* Number of cookies: number of cookies are expected to be equally assigned into control and experiment group, so the expected probability should be `0.5`. The error margin can be calcualted by `1.96 * sqrt(0.5 * 0.5 / (345543 + 344660)) = 0.0012`, the 95% confidence interval should be (0.4988, 0.5012), while observed value in the control group is `345543 / (345543 + 344660) = 0.5006`. We can conclude that sanity check is passed for this invariant metrics.

* Number of clicks: number of clicks are expected to be equally assigned into control and experiment group, so the expected probability should be `0.5`. The error margin can be calcualted by `1.96 * sqrt(0.5 * 0.5 / (28378 + 28325)) = 0.0041`, the 95% confidence interval should be (0.4959, 0.5041), while observed value in the control group is `28378 / (28378 + 28325) = 0.5005`. We can conclude that sanity check is passed for this invariant metrics.

* Click-through-probability: In the control group, CTP is `28378 / 345543 = 0.0821`. The error margin could be calculated similarly by `1.96 * sqrt(0.0821 * (1 - 0.0821) / 345543 = 0.0009`, the 95% confidence interval should be (0.0812, 0.0830), the observed value is `28325 / 344660 = 0.0822`, so the sanity check is passed.

| Metric | Observed Value | CI Lower Bound | CI Upper Bound | Result |
|:------:|:--------------:|:--------------:|:--------------:|:------:|
| Number of Cookies | 0.5006 | 0.4988 | 0.5012 | Pass |
| Number of clicks on "start free trial" | 0.5005 | 0.4959 | 0.5042 | Pass |
| Click-through-probability | 0.0822 | 0.0812 | 0.0830 | Pass |

### Check for Practical and Statistical Significance

First, let's sum up the total number of click on 'Free Trial' button, noting that enrollment and payment are only available from Oct 11 to Nov 2, so total number of click should be adjusted to this period of time, rather than the whole period that we used in sanity check.
The total number of click in the control group is `17293` and `17260` in the experiment group. The total number of enrollment in the control group is `3785` and `3423` in the experiment group. The total number of payment in the control group is `2033` and `1945` in the experiment group. 

Next, we need to calculate pooled probability and pooled standard error to find 95% confidence interval of each evaluation metrics, based on the formula as followed:

     p_pooled = (X_cnt + X_exp) / (N_cnt + N_exp)
     se_pooled = sqrt(p_pooled * (1-p_pooled) * (1./N_cnt + 1./N_exp))

* Gross conversion: pooled probability = `(3785 + 3423) / (17293 + 17260) = 0.2086`, the error margin is `1.96 * sqrt(0.2086 * (1 - 0.2086) * (1/17293 + 1/17260)) = 0.0086`. The 95% confidence interval is (-0.0291, -0.0120). Since the practical significance level = 0.01, we can conclude that the result is both statistically and practically significant. In the experiment group, the gross conversion is significant lower than control group, indicating that the change has negative impact by reducing gross conversion rate.

* Net conversion: pooled probability = `(2033 + 1945) / (17293 + 17260) = 0.1151`, the error margin is `1.96 * sqrt(0.1151 * (1 - 0.1151) * (1/17293 + 1/17260)) = 0.0067`. The 95% confidence interval is (-0.0116, 0.0018). Since the practical significance level = 0.0075, we can conclude that the result is neither statistically nor practically significant. In the experiment group, the net conversion is not significant different from control group, indicating that the change has on impact by reducing gross conversion rate.


| Metric | dmin | Observed Difference | CI Lower Bound | CI Upper Bound | Result |
|:------:|:--------------:|:--------------:|:--------------:|:--------------:|:------:|
| Gross Conversion | 0.01 | -0.0206 | -.0291 | -.0120 | Satistically and Practically Significant |
| Net Conversion | 0.0075 | -0.0049 | -0.0116 | 0.0018 | Neither Statistically nor Practically Significant |

### Sign Tests

We checked difference between control and experiment group day by day, and find that gross conversion in experiment group is smaller than control group in 19 out of 23 days. Based on [online calculator](http://graphpad.com/quickcalcs/binomial2/), the two-side p value is `0.0026` under the null hypothesis that this is totally by chance, so we could reject null hypothesis that conclude that result is significantly different. While, there are 13 days that the net conversion in experiment group is smaller than control group, and the p value is `0.6776`, meaning that it is likely that the outcome is simply by chance. So we should say the result is not statistically significant.

| Metric | p-value  | Statistically Significant|
|:------:|:--------------:|:--------------:|
| Gross Conversion | 0.0026 | Yes |
| Net Conversion | 0.6776 | No |

### Summary

We decided not to use Bonferroni correction, since the evalution metrics we have chosen are closely correlated with each other, basically they are calcuated by three variables, which they will tend to move together while some variables are changing. And Bonferroni correction are more conservative when dealing with correlated evaluation metrics.

### Recommendation

I recommend not to launch this experiment. 

The hypothesis was that this might set clearer expectations for students upfront, thus reducing the number of frustrated students who left the free trial because they didn't have enough time — without significantly reducing the number of students to continue past the free trial and eventually complete the course.

First, based on effect size test and sign test, the gross conversion in experiment group is significantly and practically lower than control group. In the other words, the number of students who choose to enroll in free trial indeed decreases due to the warning, which aligns with what null hypothesis expects to happen.

Then, since net conversion rate is not statistically significant, the number of students who eventually pass the free trial and make a payment is not different among experiment and control group, which is also stated in the null hypothesis. At this point, we can conclude that the hypothesis is correct. However, the lower bound of confidence interval is actually lower than lower bound of practical significance level, indicating this change might bring negative impact and we shoulb be cautious about this result. I highly suggest that we need more investigation to this before launching this change. So, at this point, it's better to not launch this change.

## Follow-Up Experiment ： How to Reduce Cancellation 

Recently I cancel my Spotify premium service, after I click the cancel button, it plays a song to show they're sad to know that I'm gonna leave. I think this is a great idea if Udacity could apply this when a student decide to quit free trial. New-enrolled student will see a welcome video right after they enroll in any Nanodegree, I think Udacity should play a similar video when student try to cancel their service. In the video, they could encourage students to keep up, tell them where they could find help if they get stuck or suggest other classes and study resources.

* They hypothesis is that playing this video when students choose to quit free trial will reduce the number of students who cancel the service during the free trial. The number of students who quit free trial are expected to be lower in experiment group compared to control group. The unit of diversion is user-id.

* The invariant metrics could be `number of user-id enrolled in free trial`.

* The evaluation metrics could be `retention`. `Net conversion` in the experiment group are expected to be higher compared to control group. 

* Duration: since this experiment is similar to what we just did, the duration is expected to be about 18 days with full site traffic.

* Before jumping into analyzing result, we should first apply sanity check on invariant metrics. `number of cookies visited the homepage`, `click-through-probability on 'Free Trial' button` and `gross conversion` should be equally assigned.

* After sanity check is passed, we could compare evaluation metrics with statistical significance level and practical significance level. Also, sign test should be performed to confirm with effect size test.

* Recommendation could be given based on analysis of evaluation metrics.
