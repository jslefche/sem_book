---
title: "Categorical Variables"
author: "Jon Lefcheck"
date: "November 12, 2018"
output: html_document
---

# Categorical Variables

## Introduction to Exogenous Categorical Variables

While most examples using SEM consider continuous variables, it is often the case that the variables are discrete. These include binary (yes/no, failure/success, etc.), nominal (site 1, site 2), or ordinal levels (small < medium < large). There are two cases: categorical variables as exogenous or as endogenous. We will deal with the simpler case of exogenous categorical variables first, as they pose not so much of a computational issue, but a conceptual one.

A linear regression predicting y has the following standard form:

  $$y = \alpha + \beta_{1}*x_{1} + \epsilon$$

where $\alpha$ is the intercept, $\beta_{1}$ is the slope of the effect of $x$ on y, and $\epsilon$ is the residual error.

When $x$ is continuous, the intercept $\alpha$ is intepreted as the value of y when $x$ = 0. All good.

For categorical factors, the intercept $\alpha$ has a different interpretation. Consider a value of $x$ with $k$ levels. Since the levels of $x$ are discrete and can never assume a value of 0, $\alpha$ is instead the mean value of y at the 'reference' level of $x$. (In R, the reference level is the first level alphabetically, although this can be set manually.) The regression coefficients $\beta_{k}$ are therefore the effect of each other level *relative* to the reference level. So for $k$ levels, there are $k - 1$ coefficients estimated with the additional $\alpha$ term reflecting the $k$th level.

Another way to think about this phenomenon is using so-called 'dummy' variables. Imagine each level was broken into a separate variable with a value of 0 or 1: a two-level factor with levels "a" and "b" would then become two factors "a" and "b" each with the levels 0 or 1. (In R, this would mean transposing rows as columns.) 

Now imagine setting all the values of these dummy variables to 0 to estimate the intercept: this would imply the total absence of the factor, which is not a state. Another way of thinking about this is that the dummy variables are linearly dependent: if "a = 1" then by definition "b = 0" as the response variable cannot occupy the two states simultaneously. Hence the need to set one level as the reference, so that the effect of "a" can be interpreted relative to the absence of "b".

This behavior present a challenge for path diagrams: there is not a single coefficient for the path from $x$ -> y, nor are there enough coefficients to populate a separate arrow for each level of $x$ (because one level must serve as the reference). 

There are a few potential solutions:

* for binary variables, set the values as 0 or 1 and model as numeric, which would yield a single coefficient.

* for ordinal varaibles, set the values depending on the order of the factor, e.g., small = 1 < medium = 2 < large = 3, and then model as numeric, which would yield a single coefficient.

For both of these approaches, the coefficients will be interpreted as moving from one state (0) to another (1), or from one level (1) to the next (2). 

* create dummary variables for each level: this is procedurally the same as above (splitting levels into $k$ - 1 separate variables that occupy 0/1). The key here is not to create $k$ variables, to avoid the issue raised above about dependence among predictors. This is the default behavior of *lavaan*.

This approach becomes prohibitive with large number of categories and can greatly increase model complexity. Moreover, each level is treated as an independent variable in tests of direct separation, and thus will inflate the degrees of freedom for the test.

* for suspected interactions with categorical variables, a multigroup analysis is required. In this case, the same model is fit for each level of the factor, with potentially different coefficients (see Chapter: Multigroup Models).

* test for the effect of the categorical variable using ANOVA, but do not report a coefficient. This approach would indicate whether a factor is important, but omits important information about the direction and magnitude of change. For example, does a significant treatment effect imply an increase or decrease in the response, and by how much? For this reason, such an approach is not ideal.

A alternate approach draws on this final point, and involves testing and reporting the model-estimated, or marginal, means.

## Exogenous Categorical Variables as Marginal Means

All models can be used for prediction. In multiple regression, the predicted values of one variable are often computed while holding the values of other variables at their mean. Marginal means are the mean of these predicted values. In other words, it is the expected value of one variable given the other variables in the model.

For categorical variables, marginal means are particularly useful because they provide an estimated mean for each level of each factor.

Consider a simple example with a single response and two groups "a" and "b":


```r
set.seed(111)

dat <- data.frame(y = runif(100), group = letters[1:2])

model <- lm(y ~ group, dat)

summary(model)
```

```
## 
## Call:
## lm(formula = y ~ group, data = dat)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.48473 -0.21466 -0.01238  0.19715  0.54995 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  0.44677    0.03871  11.541   <2e-16 ***
## groupb       0.08551    0.05475   1.562    0.122    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.2737 on 98 degrees of freedom
## Multiple R-squared:  0.02429,	Adjusted R-squared:  0.01433 
## F-statistic:  2.44 on 1 and 98 DF,  p-value: 0.1215
```

Note that the summary output gives a simple coefficient, which is the effect of group "b" on y in the absence of group "a". The intercept is simply the average of y in group "a":


```r
summary(model)$coefficients[1, 1]
```

```
## [1] 0.4467679
```

```r
mean(subset(dat, group == "a")$y)
```

```
## [1] 0.4467679
```

The marginal means are the expected value of y in group "a" or group "b".


```r
predict(model, data.frame(group = "a"))
```

```
##         1 
## 0.4467679
```

```r
predict(model, data.frame(group = "b"))
```

```
##       1 
## 0.53228
```

Because this is a simple linear regression, these values are simply the means of the two subsets of the data, because they are not controlling for other covariates:


```r
mean(subset(dat, group == "a")$y)
```

```
## [1] 0.4467679
```

```r
mean(subset(dat, group == "b")$y)
```

```
## [1] 0.53228
```

Let's see what happens we add a continuous covariate:


```r
dat$x <- runif(100)

model <- update(model, . ~ . + x)
```

Here, the marginal mean must be evaluated while holding the covariate $x$ at its mean value:


```r
predict(model, data.frame(group = "a", x = mean(dat$x)))
```

```
##         1 
## 0.4450597
```

```r
mean(subset(dat, group == "a")$y)
```

```
## [1] 0.4467679
```

You'll note that this value is now different than the mean of the subset of the data because, again, it controls for the presence of $x$.

This procedure gets increasingly complicated with both the number of factor levels and the number of covariates. The *emmeans* package provides an automated way to compute marginal means:


```r
library(emmeans)

emmeans(model, specs = "group") # where specs is the variable or list of variables whose means are to be estimated
```

```
##  group    emmean         SE df  lower.CL  upper.CL
##  a     0.4450597 0.03893785 97 0.3677788 0.5223405
##  b     0.5339882 0.03893785 97 0.4567074 0.6112691
## 
## Confidence level used: 0.95
```

You'll note that the output value gives the same as using the `predict` function above, but also returns the marginal mean for group "b" while also controlling for $x$:


```r
predict(model, data.frame(group = "b", x = mean(dat$x)))
```

```
##         1 
## 0.5339882
```

and so is a handy wrapper for complex models.

Coupled with ANOVA to test for the significance of the categorical variable, the marginal means provide key information that is otherwise lacking, namely *how* the response value changes based on the factor level. In does not, however, allow for prediction in the same way a model coefficient does.

The *emmeans* package provides additional functionality by conducting post-hoc tests of differences among the means of each factor level:


```r
emmeans(model, list(pairwise ~ group))
```

```
## $`emmeans of group`
##  group    emmean         SE df  lower.CL  upper.CL
##  a     0.4450597 0.03893785 97 0.3677788 0.5223405
##  b     0.5339882 0.03893785 97 0.4567074 0.6112691
## 
## Confidence level used: 0.95 
## 
## $`pairwise differences of group`
##  contrast    estimate         SE df t.ratio p.value
##  a - b    -0.08892852 0.05520893 97  -1.611  0.1105
```

You'll note a second output which is the pairwise contrast between the means of groups "a" and "b" with an associated significance test. 

These pairwise Tukey tests provide the final level of information, which is whether the response in each level varies significantly from the other levels.

The `coefs` function in *piecewiseSEM* adopts a two-tiered approach by first computing the significance of the categorical variable using ANOVA, and then reports the marginal means and post-hoc tests:


```r
library(piecewiseSEM)
```

```
## 
##   This is piecewiseSEM version 2.1.0
## 
## 
##   If you have used the package before, it is strongly recommended you read Section 3 of the vignette('piecewiseSEM') to familiarize yourself with the new syntax
## 
##   Questions or bugs can be addressed to <LefcheckJ@si.edu>
```

```r
coefs(model)
```

```
##   Response      Predictor Estimate Std.Error DF Crit.Value P.Value
## 1        y              x   0.0567     0.093 97     0.6094  0.5437
## 2        y          group        -         -  1     0.1957  0.1105
## 3          group[a] mean=   0.4451    0.0389 97          -       -
## 4          group[b] mean=    0.534    0.0389 97          -       -
##   Std.Estimate   
## 1       0.0613   
## 2            -   
## 3            -  a
## 4            -  a
```

In this output, the significance test from the ANOVA is reported in the row corresponding to the group effect, and below that are the marginal means for each level of the grouping factor. Finally, the results of the post-hoc test are given using letters at the end of the rows reporting the marginal means. In this case, the same letter indicates no significant difference among the group levels.

This solution provides a measure of whether the path between the exogenous categorical variable and the respones is significant, as well as parameters for each level in the form of the model-estimated marginal means.

## Exogenous Categorical Variables as Marginal Means: A Worked Example

Let's consider an example from Bowen et al. (2017). In this study, the authors were interested in how different microbiomes of the salt marsh plant *Phragmites australis* drive ecosystem functioning, and ultimately the production of aboveground biomass. In this case, they considered three microbial communities: those from a native North American lineage, from Gulf Coast lineage, and an introduced lineage. There were additional genotypes within each community type, necessitating the application of random effects to account for intraspecific variation.

We will fit a simplified version of their full path diagram, focusing only on aboveground biomass (although they also test the effect on belowground biomass).

![bowen_sem](https://raw.githubusercontent.com/jslefche/sem_book/master/img/categorical_variables_bowen_sem.png)

In this case, the variable "*Phragmites* status" corresponds to the three community types, and can't be represented using a single coefficient. Thus, the marginal-means approach is ideal to elucidate the effect of each community type on both proximate and ultimate ecosystem properties.

Let's read in the data and construct the model:


```r
bowen <- read.csv("./data/bowen.csv")

bowen <- na.omit(bowen)

library(nlme)

bowen_sem <- psem(
  lme(observed_otus ~ status, random = ~1|Genotype, data = bowen, method = "ML"),
  lme(RNA.DNA ~ status + observed_otus, random = ~1|Genotype, data = bowen, method = "ML"), 
  lme(below.C ~ observed_otus + status, random = ~1|Genotype, data = bowen, method = "ML"), 
  lme(abovebiomass_g ~ RNA.DNA + observed_otus + belowCN + status, random = ~1|Genotype, data = bowen, method = "ML"),
  data = bowen
)
```

And let's retrieve the output:


```r
summary(bowen_sem, .progressBar = F)
```

```
## 
## Structural Equation Model of bowen_sem 
## 
## Call:
##   observed_otus ~ status
##   RNA.DNA ~ status + observed_otus
##   below.C ~ observed_otus + status
##   abovebiomass_g ~ RNA.DNA + observed_otus + belowCN + status
## 
##     AIC      BIC
##  67.877   125.138
## 
## ---
## Tests of directed separation:
## 
##                   Independ.Claim Estimate Std.Error DF Crit.Value P.Value
##    observed_otus ~ belowCN + ...   3.3036    2.2934 57     1.4405  0.1552
##          RNA.DNA ~ belowCN + ...   0.0002    0.0002 56     1.3740  0.1749
##          below.C ~ belowCN + ...   0.0171    0.0063 56     2.7225  0.0086
##          below.C ~ RNA.DNA + ...  -0.5383    3.0545 56    -0.1762  0.8607
##   abovebiomass_g ~ below.C + ...  -0.0357    0.0787 54    -0.4540  0.6516
##     
##     
##     
##   **
##     
##     
## 
## Global goodness-of-fit:
## 
##   Fisher's C = 17.877 with P-value = 0.057 and on 10 degrees of freedom
## 
## ---
## Coefficients:
## 
##         Response                Predictor  Estimate Std.Error DF Crit.Value
##    observed_otus                   status         -         -  2     5.9589
##                      status[native] mean= 2258.6564  105.8178 12          -
##                  status[introduced] mean=   2535.07  131.2216 14          -
##                    status[invasive] mean= 2541.4984   56.6887 12          -
##          RNA.DNA            observed_otus         0         0 57     2.1583
##          RNA.DNA                   status         -         -  2      9.348
##                    status[invasive] mean=    0.7112    0.0118 12          -
##                  status[introduced] mean=    0.7305    0.0264 14          -
##                      status[native] mean=    0.7844    0.0216 12          -
##          below.C            observed_otus     9e-04     3e-04 57     2.6546
##          below.C                   status         -         -  2    17.1995
##                  status[introduced] mean=   42.6366    0.3673 14          -
##                    status[invasive] mean=   43.4004    0.1594 12          -
##                      status[native] mean=   44.4975    0.3056 12          -
##   abovebiomass_g                  RNA.DNA   -1.8517    1.8893 55    -0.9801
##   abovebiomass_g            observed_otus    -2e-04     2e-04 55    -0.7521
##   abovebiomass_g                  belowCN     0.005    0.0041 55     1.2026
##   abovebiomass_g                   status         -         -  2    12.9519
##                      status[native] mean=    1.7489    0.2206 12          -
##                    status[invasive] mean=    1.8807    0.1041 12          -
##                  status[introduced] mean=    2.7024     0.232 14          -
##   P.Value Std.Estimate    
##    0.0508            -    
##         -            -   a
##         -            -   a
##         -            -   a
##    0.0351        0.135   *
##    0.0093            -  **
##         -            -  a 
##         -            -  ab
##         -            -   b
##    0.0103       0.2913   *
##     2e-04            - ***
##         -            -  a 
##         -            -  a 
##         -            -   b
##    0.3313      -0.1428    
##    0.4552      -0.0905    
##    0.2343       0.1356    
##    0.0015            -  **
##         -            -  a 
##         -            -  a 
##         -            -   b
## 
##   Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05
## 
## Individual R-squared:
## 
##         Response method Marginal Conditional
##    observed_otus   none     0.10        0.19
##          RNA.DNA   none     0.31        0.81
##          below.C   none     0.26        0.32
##   abovebiomass_g   none     0.22        0.28
```

In this case, it appears that the model fits the data well enough ($P = 0.057$). The linkage between microbial community type (status) and richness is non-significant, but the other paths are significant. Examination of the marginal means indicates microbial activity (RNA/DNA) and belowground carbon are generally highest in *Phragmites* with native microbial communities based on the post-hoc tests. However, none of these properties appear to influence the ultimate production of biomass. Rather, that property appears to be entirely controlled by the plant microbiome: those with the introduced microbial community have significantly higher aboveground biomass based on the post-hoc tests after controlling for microbial activity and soil nutrients. (In the full article, they draw the same inference for belowground biomass.)

Thus, despite a multi-level categorical predictor (microbiome status), the two-step procedure of ANOVA and calculation of marginal means reveals a mechanistic understanding of the drivers of plant biomass in this species.

## Endogenous Categorical Variables

Endogenous categorical variables are far trickier, and at the moment, are not implemented in *piecewiseSEM*.

In the case of endogenous categorical variables in a piecewise framework, there are really only two solutions:

* for binary variables, set the values as 0 or 1 and model as numeric, which would yield a single coefficient.

* for ordinal variables, set the values depending on the order of the factor, e.g., small = 1 < medium = 2 < large = 3, and then model as numeric, which would yield a single coefficient.

Nominal variables (i.e., levels are not ordered) cannot be modeled at this time.  One could approach this through the application of multinomial regression. 

*lavaan* provides a robust alternative in the form of confirmatory factor analysis (see [http://lavaan.ugent.be/tutorial/cat.html](http://lavaan.ugent.be/tutorial/cat.html)).

## References
Bowen, J. L., Kearns, P. J., Byrnes, J. E., Wigginton, S., Allen, W. J., Greenwood, M., ... & Meyerson, L. A. (2017). Lineage overwhelms environmental conditions in determining rhizosphere bacterial community structure in a cosmopolitan invasive plant. Nature communications, 8(1), 433.
