# Number needed to recruit
Kristin Öster

## Background

In prediction, $`r^2`$ can be a misleading summary. In a recent post on
the Psychometrics Substack
(https://psychometrics.substack.com/p/r-and-r) the authors explain why
$`r^2`$ can lead to misleading conclusions about the usefulness of
personality testing in recruitment settings. A personality test that
correlates $`r = .19`$ with job performance explains only
$`r^2 \approx 3.6\%`$ of the variance, which sounds negligible. But
recruitment is about prediction, that is, ranking candidates. What we
want to know is how much a recruitment tool improves our chances of a
good hire, not how much variance it explains or how it causally relates
to the outcome.

Now, I am not a psychometrician nor do I work with recruitment. But this
post sparked my interest regarding how we can say something about the
uncertainty of the measure. Predictive models apply to the group level:
they describe what we can expect on average across many hires, not what
will happen for any single person. With $`r = .19`$, applicants scoring
+2 SD on the test are expected to perform $`0.19 \times 2 \approx 0.4`$
SD higher than average. Used consistently, the test should therefore
raise the success rate over time. But the individual outcomes remain
highly uncertain, and structured recruitment with testing is costly. If
an organization rarely recruits, the long-run gain may never
materialize. So if not $`r^2`$, what measure could we use to interpret
the uncertainty of the expected benefit?

In clinical research, the number needed to treat (NNT) complements mean
effect sizes by showing how many patients must be treated to gain one
additional favorable outcome (or prevent one unfavorable outcome). The
same logic can be applied to recruitment: how many hires must be made
with the test, rather than without it, to get one additional successful
hire?

``` math
\begin{aligned}
NNT &= \frac{1}{ARR} 
\\
ARR &= CER - EER
\end{aligned}
```

NNT = number needed to treat, ARR = Absolute risk reduction, CER =
Control event rate, EER = Experiment event rate

## Demonstration

Suppose we recruit retail sales staff, where success means closing at
least 64 sales per month (mean 60, SD 10; hypothetical figures). Assume
a conscientiousness test, standardized in the applicant pool, correlates
r = .19 with monthly sales. Applicants scoring X ≥ 2 SD are then
expected to close about 0.19 × 2 ≈ 0.4 SD, or roughly 4 sales more per
month than average. This is a group-level prediction. Many individuals
in both groups will succeed or fail regardless.

``` math
\begin{aligned}
Y(\text{monthly sales}) &= Normal(60,10)
\\
Y(\text{monthly sales, standardized}) &= Normal(0,1)
\\
X(\text{Conscientiosness, standardised}) &= Normal(0,1) \\
r_{x,y} &=0.19
\end{aligned}
```

Code for setting up the data:

<details class="code-fold">
<summary>Code</summary>

``` r
library(mvtnorm)
r  <- 0.19
S  <- matrix(c(1, r, r, 1), 2)

x_cut <- 2
y_cut <- 0.4
p_xlo <- pnorm(2)   # P(x <  2)
p_xhi <- 1-p_xlo   # P(x >= 2)
p_ylo <- pnorm(0.4) # P(Y <  0.4)
p_yhi <- 1 - p_ylo  # P(Y >= 0.4)
```

</details>

Calculate the success rates in the two groups, selecting those scoring
above ($`P(Y\ge0.4|X\ge2`$) and selecting at random ($`P(Y\ge0.4)`$)

<details class="code-fold">
<summary>Code</summary>

``` r
#joint probability: (P(Y>=0.4 & X>=2) 
p_yhi_and_xhi <- pmvnorm(lower = c(x_cut, y_cut), upper = c(Inf, Inf), sigma = S)[1]

#CER
p_yhi_given_xhi <- p_yhi_and_xhi / p_xhi #among those with a high (+2sd) x score, how many also have Y \ge 0.4?

#Base rate/EER: ($P(Y>=0.4)$)
p_base <- p_yhi   #Among those with any X score (selecting at random) how many has a Y \ge 0.4?


ARR  <- p_yhi_given_xhi - p_base  #Absolute success increase: Not a risk reduction, but success increase
nnt <- 1 / ARR
c(CER = p_base, EER = p_yhi_given_xhi , ARR = ARR, NNT = nnt)
```

</details>

          CER       EER       ARR       NNT 
    0.3445783 0.5206059 0.1760277 5.6809246 

In terms of success rates, about 52% of hires with high
conscientiousness scores (x ≥ 2) reach 64 sales (EER), compared with
about 34% of those recruited at random (CER). The difference of 18
percentage points (ARR, or in this case, absolute success increase)
gives a number needed to hire of about 5.6. In other words, we must hire
about six people from the high conscientiousness group, rather than at
random, to get one additional person who reaches the sales target (+4
above the average).

## Using a lower cut-off

Selecting on those scoring +2SD on a tests, means selecting only the top
2.3 % of candidates. Let’s assume you select the top 10 % to recruit
among.

<details class="code-fold">
<summary>Code</summary>

``` r
x_cut <- qnorm(p = 0.1, lower.tail = F)
y_cut <- x_cut*r
p_xlo <- pnorm(x_cut)   
p_xhi <- 1-p_xlo   
p_ylo <- pnorm(y_cut) 
p_yhi <- 1 - p_ylo  


#joint probability: (P(Y>=0.4 & X>=2) 
p_yhi_and_xhi <- pmvnorm(lower = c(x_cut, y_cut), upper = c(Inf, Inf), sigma = S)[1]

#CER
p_yhi_given_xhi <- p_yhi_and_xhi / p_xhi #among those with a high (+2sd) x score, how many also have Y \ge 0.4?

#Base rate/EER: ($P(Y>=0.4)$)
p_base <- p_yhi   #Among those with any X score (selecting at random) how many has a Y \ge 0.4?


ARR  <- p_yhi_given_xhi - p_base  #Absolute success increase: Not a risk reduction, but success increase
nnt <- 1 / ARR
c(CER = p_base, EER = p_yhi_given_xhi , ARR = ARR, NNT = nnt)
```

</details>

          CER       EER       ARR       NNT 
    0.4038111 0.5363407 0.1325296 7.5454821 

Choosing among the top 10 % scorers gives you a 54 % chance of hiring
someone who reaches 2 more sales than average, which is 13 percentage
points higher than the chances when recruiting at random. On average,
this benefit one additional successful hire (+2 sales compared to
average) is expected for every 8 hires (NNT).
