---
title: "Coefficients"
author: "Jon Lefcheck"
date: "January 15, 2021"
output: html_document
---

# Coefficients

## Unstandardized and Standardized Coefficients

Path (or regression) coefficients are the inferential engine behind structural equation modeling, and by extension all of linear regression. They relate changes in the dependent variable $y$ to changes in the independent variable $x$, and thus act as a measure of association. In fact, you may recall from the chapter on global estimation that, under some circumstances, path coefficients can be expressed as (partial) correlations, a unitless measure of association that makes them excellent for comparisons. They also allow us to generate predictions for new values of $x$ and are therefore useful in testing and extrapolating model results.

We will consider two kinds of regression coefficients: unstandardized (or raw) coefficients, and standardized coefficients.

Unstandardized coefficients are the default values returned by all statistical programs. In short, they reflect the expected (linear) change in the response with each unit change in the predictor. For a coefficient value $\beta = 0.5$, for example, a 1 unit change in $x$ is, on average, an 0.5 unit change in $y$.

In models with more than one independent variable (e.g., $x1$, $x2$, etc), the coefficient reflects the expected change in $y$ *given* the other variables in the model. This implies that the effect of one particular variable controls for the presence of other variables, generally by holding them constant at their mean. This is why such coefficients are referred to as *partial* regression coefficients, because they reflect the independent (or partial) contributions of any particular variable.

As an aside: one tricky aspect to interpretation involves transformations. When the log-transformation is applied, for example, the relationships between the variable are no longer linear. This means that we have to change our interpretation slightly. When $y$ is log-transformed, the coefficient $\beta$ is interpreted as a 1 unit change in $x$ leads to a $(exp(\beta) - 1) \times 100%$ change in $y$. Oppositely, when the independent variable $x$ is log-transformed, $\beta$ is interpreted as a 1% change in $x$ leads to a $\beta$ change in $y$. Finally, when both are transformed, both are expressed in percentages: a 1% change in $x$ leads to a $(exp(\beta) - 1) \times 100%$ change in $y$. Transformations often confound interpretation, so it is worth mentioning.

In contrast to raw coefficients, standardized coefficients are expressed in equivalent units, regardless of the original measurements. Often these are in units of standard deviations of the mean (scale standardization) but, as we shall see shortly, there are other possibilities. The goal of standardization is to increase _comparability_. In other words, the magnitude of standardized coefficients can be directly compared to make inferences about the relative strength of relationships.

In SEM, it is often advised to report both unstandardized and standardized coefficients, because they present different and mutually exclusive information. Unstandardized coefficients contain information about both the variance *and* the mean, and thus are essential for prediction. Along these lines, they are also useful for comparing across models fit to the same variables, but using different sets of data. Because the most common form of standardization involves scaling by the sample standard deviations, data derived from different sources (i.e., different datasets) have different sample variances and their standardized coefficients are not immediately comparable.

Unstandardized coefficients also reflect the phenomenon of interest in straightforward language. Imagine telling someone that 1 standard deviation change in nutrient input levels would result in a 6 standard deviation change in water quality. That might seem impressive until it becomes clear that the size of the dataset has reduced the sample variance, and the absolute relationship reveals only a very tiny change in water quality with each unit change in nutrient levels. Not so impressive anymore.

Standardized effects, on the other hand, are useful for comparing the relative magnitude of change associated with different paths in the same model (i.e., using data drawn from the same population). Care should be taken *not* to interpret these relationships as the 'proportion of variance explained'--for example, a larger standardized coefficient does not explain more variance in the response than a smaller standardized coefficient--but rather in terms of relative influence on the mean of the response.

By extension, standardization is necessary to compare indirect or compound effects among different sets of paths in the same model: for example, comparing direct vs. indirect pathways in a partial mediation model. This is because those pathways can and often are measured in very different units, and their relative magnitudes might simply reflect their measurement units rather than any stronger or weaker effects.

In contrast, comparing the strength of indirect or compound effects across the same set of variables in different models *requires* unstandardized coefficients, due to the issue of different sample variances raised above. Comparing the same path across different models using standardized coefficients would require a demonstration that the sample variances are not significantly different (or alternately, that the entire population has been sampled).

Thus, both standardized and unstandardized coefficients have their place in structural equation modeling. Let's now explore some of the different forms of standardization, and how they can be achieved.

## Scale Standardization

The most typical implementation of standardization is placing the coefficients in units of standard deviations of the mean. This is accomplished by scaling the coefficient $\beta$ by the ratio of the standard deviation of $x$ over the standard deviation of $y$:

  $$b = \beta*\left( \frac{sd_x}{sd_y} \right)$$

This coefficient has the following interpretation: for a 1 standard deviation change in $x$, we expect a $b$ unit standard deviation change in $y$.

This standardization can also be achieved by *Z*-transforming the raw data, in which case $b$ is already the (partial) correlation between $x$ and $y$.

Both *lavaan* and *piecewiseSEM* return scale-standardized coefficients. *lavaan* requires a different set of functions or arguments, while *piecewiseSEM* will do it by default using the functions `coefs`. `coefs` has the added benefit in that it can be called on any model object, and thus has applications outside of structural equation modeling.

Let's run an example:


```r
library(lavaan)
library(piecewiseSEM)

set.seed(6)

data <- data.frame(y = runif(100), x = runif(100))

xy_model <- lm(y ~ x, data = data)

# perform manual standardization
beta <- summary(xy_model)$coefficients[2, 1]

(beta_std <- beta * (sd(data$x)/sd(data$y)))
```

```
## [1] 0.09456659
```

For this example, we recover a standardized coefficient of $b = 0.095$, suggesting that for a 1 standard deviation change in $x$, there is expectde to be a 0.095 standard deviation change in $y$.


```r
# now retrieve with piecewiseSEM
coefs(xy_model)$Std.Estimate
```

```
## [1] 0.0946
```

We get the same estimate from *piecewiseSEM* using `coefs`.


```r
# and with lavaan
xy_formula <- 'y ~ x'

xy_sem <- sem(xy_formula, data)

standardizedsolution(xy_sem)$est.std[1]
```

```
## [1] 0.09456659
```
And the same for *lavaan*, demonstrating that these packages use the same scaling procedure under the hood.

## Range Standardization

An alternative to scale standardization is *relevant range* standardization. This approach scales the coefficients over some relevant range. Typically this is the full range of the data, in which case $\beta$ can be standardized as follows:

  $$b = \beta * \frac{max(x) - min(x)}{max(y) - min(y)}$$

The interpretation for the coefficient would then be the expected proportional shift in $y$ along its range given a full shift along the range of $x$.

At first, this might seem like a strange form of standardization, but it has some powerful applications. For example, consider a binary predictor: 0 or 1. In such a case, the relevant range-standardized coefficient is the expected shift in $y$ given the transition from one state (0) to another (1). Or consider a management target such as decreasing nutrient runoff by 10%. Would reducing fertilizer application by 10% of its range yield a similar reduction in runoff? Such expressions are necessarily the currency of applied questions.

Perhaps the best application of relevant ranges is in comparing coefficients within a model: rather than dealing in somewhat esoteric quantities of standard deviations, relevant range standardization simply asks which variable causes a greater shift in $y$ along its range. This is a much more digestable concept to most scientists. It may even provide a more fair comparison across the same paths fit to different datasets, if the ranges are roughly similar and/or encompassed in the others. Restricting the range may be a useful solution for comparing coefficients across models fit to different data, as long as the range doesn't extend beyond that observed in any particular dataset.

Note that the decomposition of (partial) correlations as shown in the chapter on global estimation is not possible with relevant ranges, so range standardization is not recommended if the objective to compute indirect or total effects.

For a worked example, we have now entered fully into the realm of *piecewiseSEM*--it does not appear as if *lavaan* has integrated this functionality a of yet. Let's attempt to scale the results by hand, then compare to the output from `coefs` with the argument `standardize = "range"`:


```r
#by hand
(beta_rr <- beta * (max(data$x) - min(data$x))/(max(data$y) - min(data$y)))
```

```
## [1] 0.09806703
```

```r
coefs(xy_model, standardize = "range")$Std.Estimate
```

```
## [1] 0.0981
```

In both cases, we obtain a $b = 0.098$ suggesting that a full shift in $x$ along its range would only result in a predicted shift of about 10% along the range of $y$.

Both scale and relevant range-standardization only apply when the response is normally-distributed. If not, we must make some assumptions in order to obtain standardized coefficients. Let's start with binomial responses, which are the trickiest case.

## Binomial Response Models

Binomial responses are those that are binary (0, 1) such as success or failure, or present vs. absent. What is unique about them is that they do not have a linear relationship with a predictor $x$. Instead, they are best modeled using a sigmoidal curve. To demonstrate, let's generate some data, fit a binary model, and plot the predicted relationship:


```r
set.seed(44)

x <- rnorm(20)

x <- x[order(x)]

y <- c(rbinom(10, 1, 0.8), rbinom(10, 1, 0.2))

glm_model <- glm(y ~ x, data = data.frame(x = x, y = y), "binomial")

xpred <- seq(min(x), max(x), 0.01)

ypred <- predict(glm_model, list(x = xpred), type = "response")

plot(x, y)

lines(xpred, ypred)
```

<img src="04-Coefficients_files/figure-html/unnamed-chunk-5-1.png" width="672" />

Clearly these data are not linear, and modeling them as such would ignore the underlying data-generating process. Instead, as you can see, we fit them to a binomial distribution using a generalized linear model (GLM). 

GLMs consist of three parts: (1) the random component, or the expected values of the response based on their underlying distribution, (2) the systematic component that represents the linear combination of predictors, and (3) the link function, which links the expected values of the response (random component) to the linear combination of predictors (systematic component) via a transformation. 

Basically, the link functions take something inherently non-linear and attempts to linearize it. This can be shown by plotting the predictions on the link-scale:


```r
ypred_link <- predict(glm_model, list(x = xpred), type = "link")

plot(xpred, ypred_link)
```

<img src="04-Coefficients_files/figure-html/unnamed-chunk-6-1.png" width="672" />

Note how the line is no longer sigmoidal, but straight!

For binomial responses, there are two kinds of link functions: logit and probit. We'll focus on the logit link for now because it's more common. With this link, the coefficients are in units of logits or the *log odds ratio*, which reflect the log of the probability of observing an outcome (1) relative to the probability of not observing it (0). 

Often these coefficients are reverted to just the odds ratio by taking the exponent, which yields the proportional change in the probablity observing one outcome (1) with a unit change change in the predictor.

Say, for example, we have a coefficient $\beta = -0.12$. A 1 unit change in $x$ would result in $exp(-0.12) = 0.88 \times 100%$ or 88% reduction in the odds of observing the outcome $y$.

The problem is that (log) odds ratios themselves are not comparable across models. Further, it's not immediately clear how they might be standardized since the coefficient is reported on the link (linear) scale, while the only variance we can compute is from the raw data, which is on the non-linear scale. Thus, we need to find some sway to obtain estimates of variance on the same linearized scale as the coefficient.

### Latent Theoretic Approach

One approach is to consider that for every value of $x$, there is an underlying probability distribution of observing a 0 or a 1 for $y$. The mean of these distributions is where a particular outcome is *most* likely. Let's say at low values of $x$ we observe $y = 0$, at at high values of $x$ we observe $y = 1$. If we order $x$, the mean probabilities give rise to a linear increase in observing $y = 1$ with increasing $x$. Here is an illustration of this phenomenon (from Long 1997):

![](https://raw.githubusercontent.com/jslefche/sem_book/master/img/coefficients_latent_propensity.png)

This linear but latent (i.e., unobserved) variable, which we call $y^*$, is therefore related to the observed values of $x$ through a vector of linear coefficients $\beta$ as in any other linear model:

  $$y^*_{i} = x_{i}\beta + \epsilon_{i}$$

Generally, the linear $y^*$ is related to the non-linear $y$ via a cutpoint, which is generally $\tau = 0.5$ where any value of $x$ where $y^*$>0.5 is equivalent to $y$ = 1, and any value of $x$ where $y^*$<0.5 is equivalent to $y$ = 0.

The problem is we can never observe this linear underlying or *latent propensity* and so we must approximate its error variance. In a later chapter on Latent Variable Modeling, we often fixed their error variance to 1. In this case, there are theoretically-derived error variances depending on the distribution and the link function: for the probit link, the error variance $\epsilon = 1$, while for the logit link, $\epsilon = \pi^2/3$, both for the binomial distribution.

Regardless of the type of standardization, we need to know about the range or variance of the response. With our knowledge of $y^*_{i}$ and the theoretical error variances, we have all the information needed to compute the variance on the link (linear) scale.

The variance in $y^*$ is the sum of the variance of the linear (link-transformed) predictions *plus* the theoretical error variance. For a logit link, then:

  $$\sigma_{y^*_{i}}^2 = \sigma_{x\beta}^2 + \pi^2/3$$
  
The square-root of this quantity gives the standard deviation of $SD_{y^*}$ on the linear scale for use in scale standardization, or alternately, the range of $y^*$ to use in relevant range standardization.

### Observation-Empirical Approach

There is an alternate method which relies on the proportion of variance explained, or $R^2$. Here, we can express the $R^2$ as the ratio of the variance of the predicted values (on the linear scale) over the variance of the observed values (on the non-linear scale):

  $$R^2 = \frac{\sigma_{\hat{y}}^2}{\sigma_{y}^2}$$

We can compute $R$, which is the correlation between the non-linear observed and predicted values on the non-linear scale that, when taken to the power of 2, yields $R^2$. If we also know the variance of the predicted values on the non-linear scale $y$, we have all the information to solve for $\sigma_{\hat{y}}$, whose square-root is the standard deviation of $y$ that we can use in the calculation of the standardized coefficient.

This method, called the *observation-empirical approach*, does not require the acknowledgement of any latent variables or theoretical error variances, but does require an acceptance of this is a valid measurement of $R^2$ (which some consider it not, as GLM estimation is based on deviance, not variance, and thus this statistic is not equivalent). It also does not provide a measure of the range of $y$ although we can assume, againbased on sampling theory, that $6 * \sigma_{y}$ encompasses the full range of $y$. 

Let's revisit our earlier GLM example and construct standardized coefficients:

```r
# get beta from model
beta <- summary(glm_model)$coefficients[2, 1]

preds <- predict(glm_model, type = "link") # linear predictions

# latent theoretic
sd.ystar <- sqrt(var(preds) + (pi^2)/3) # for default logit-link

beta_lt <- beta * sd(x)/sd.ystar

# observation empirical
R2 <- cor(y, predict(glm_model, type = "response"))^2 # non-linear predictions

sd.yhat <- sqrt(var(preds)/R2)

beta_oe <- beta * sd(x)/sd.yhat

# obtain using `coefs`
coefs(glm_model, standardize.type = "latent.linear"); beta_lt
```

```
##   Response Predictor Estimate Std.Error DF Crit.Value P.Value Std.Estimate  
## 1        y         x  -2.0975    0.9664 18    -2.1703    0.03      -0.8122 *
```

```
## [1] -0.8121808
```

```r
coefs(glm_model, standardize.type = "Menard.OE"); beta_oe
```

```
##   Response Predictor Estimate Std.Error DF Crit.Value P.Value Std.Estimate  
## 1        y         x  -2.0975    0.9664 18    -2.1703    0.03      -0.6566 *
```

```
## [1] -0.6565602
```

We see that both approaches produce valid coefficients and they are the same as those returned by the `coefs` function in *piecewiseSEM* (with the appropriate argument).

You'll note that the observation-empirical approach yields a smaller coefficient than the latent-theoretic. This is because the former approach is influenced by the fact that it is based on the relationship between a linear approximation (predictions) of a non-linear variable (raw values), introducing a loss of information. The *latent theoretic approach* also suffers from a loss of information from use of a distribution-specific but theoretically-derived error variance for binomial distrbutions, which may or may not approach the true error variance (which is unknowable). Either way, both kinds of standardization are not without their drawbacks, but both provide potentially useful information in being able to compare linear and now *linearized* standardized coefficients for logistic regression.

## Scaling to Other Non-Normal Distributions

In many ways, logistic regression is the most demanding case for standardization because we are actually modeling a latent (unmeasurable) property whose variance can't be known (but as explained above, can be estimated). For other distributions, we are modeling the actual values: this means that only the observation-empirical approach can be used, which simplifies the procedure greatly as we don't need to worry about any theoretical error variances. Let's work through an example using the Poisson distribution for count data. 

The default link function for Poisson models is the log-link, which for our purposes means that we are simply modeling the log of the response. Therefore, we will assume that a generalized linear model fit to a Poisson distribution is approximately the same as a general linear model of the log-transformed response fit to a normal distribution (see papers by Ives and others on this topic). This approximate equivalency will become clear in a moment.

First, let's create some example Poisson-distributed data:


```r
set.seed(100)

count_data <- data.frame(y = rpois(100, 10))

count_data$x <- count_data$y * runif(100, 0, 5)
```

Now let's fit an LM with the log-transformed response and see what kind of coefficient we recover:


```r
lm_model <- lm(log(y) ~ x, count_data)

coefs(lm_model)$Std.Estimate
```

```
## [1] 0.5346
```

As you may recall from the second rule of path coefficients, the standardized coefficient from a simple linear regression is actually the bivariate correlation between the two:


```r
with(count_data, cor(x, log(y)))
```

```
## [1] 0.5345506
```

And we see in this example that $b_x = r = 0.54$.

Now let's re-fit this model to $y$ using GLM to a Poisson distribution with the default log-link:


```r
glm_model2 <- glm(y ~ x, family = poisson(link = "log"), count_data)

coef(glm_model2)[2]
```

```
##          x 
## 0.01204693
```

Here, $\beta_x = 0.012$ which is a bit different from the linear model. That is because the unstandardized coefficient is reported on the link scale.

Let's repeat our observation-empirical procedure, first by getting the $R^2$ which is the squared correlation between the raw vs. fitted values, then by getting the variance of the raw observations:


```r
R2 <- cor(count_data$y, predict(glm_model2, type = "response"))^2 # non-linear predictions

sd.yhat <- sqrt(var(predict(glm_model2, type = "link"))/R2)

coef(glm_model2)[2] * sd(count_data$x)/sd.yhat
```

```
##         x 
## 0.5695438
```
This value of $b_x = 0.57$ from the GLM is remarkably close to the standardized coefficient obtained from the linear model, which was $b_x = 0.54$ and also the correlation between $x$ and $log(y)$. The differences arise from the fact that for the linear model we have considered the error on $log(y)$ whereas in the GLM, we have only considered the systematic but not the random component. They are generally small enough to be negligible if the data are truly Poisson-distributed. Therefore, we feel comfortable enough reporting the observation-empirical values from the GLM, noting again that they have a slight downward bias due to not incorporating the random component.

Its important to note that by virtue of considering only the the total variance of the fitted values produced by the model, we can extend these methods to hierarchical, mixed, and other models where variance is partitioned or modeled.

For distributions other than Poisson and negative binomial, the procedure becomes trickier. For example, here we assume the variance of the response equals the mean. However, other classes--such as the quasi-distributions--estimate an additional parameter $\phi$ to explain how the variance changes with the mean. As we are interested in quantifying this variance, it is not yet clear how to derive meaningful approximations of $sd_y$ from such distributions. However, we hope to make significant progress on this front in the coming months, so this functionality may be incorporated into the *piecewiseSEM* package soon.

## References

Grace, J. B., Johnson, D. J., Lefcheck, J. S., & Byrnes, J. E. (2018). Quantifying relative importance: computing standardized effects in models with binary outcomes. Ecosphere, 9(6), e02283.

Scott Long, J. (1997). Regression models for categorical and limited dependent variables. Advanced quantitative techniques in the social sciences, 7.
