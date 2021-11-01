Comparing Social Mobility across Countries with the UNIDIFF Model
================

## Loading required packages

``` r
library(gnm)
library(vcdExtra)
```

## Pre-processing the Data

The survey data is read into R, selecting the variables of interest: the
five-year period, the eight-category social class variable for father
and respondent, and the weighted frequencies. The period and social
class variables are converted to factors.

``` r
allMen <- read.table(params$data, header = FALSE)[, c(4, 10, 12, 14)]
colnames(allMen) <- c("period", "origin8", "dest8", "wFreq")
allMen[,1:3] <- lapply(allMen[, 1:3], factor)
```

Following the analysis by Ganzeboom and Luijkx (in Social Mobility in
Europe, ed. R. Breen, 2004), we collapse the data by the factors
`period`, `origin8`, and `dest8`.

``` r
dataI <- as.data.frame(xtabs(wFreq ~ period + origin8 + dest8, data = allMen))
```

Since the response is a weighted frequency, the resulting counts are
non-integer. This generates multiple warnings during the model fitting,
which are suppressed in the output below. \#\# Modelling We first fit
the conditional independence model as a baseline (the model names refer
to the names used in Ganzeboom and Luijkx, 2004):

``` r
modelA <- suppressWarnings(gnm(Freq ~ -1 + period:origin8 + period:dest8,
                               family = poisson, data = dataI, verbose = FALSE))
```

Then we update this model to the unstructured UNIDIFF model, using `Exp`
to ensure the period multipliers are non-negative and the `ofInterest`
argument to mark these parameters as the only ones that should be
included in model summaries.

``` r
modelC <- update(modelA, . ~ . + Mult(Exp(period), origin8:dest8),
                 ofInterest = "[.]period")
dev <- c("C" = deviance(modelC))
df <- c("C" = df.residual(modelC))
```

``` r
dev["C"]
```

           C 
    233.2034 

``` r
df["C"]
```

      C 
    240 

The `getContrasts()` function is used to obtain simple contrasts of the
log-multipliers along with their quasi-variances:

``` r
periodContr <- getContrasts(modelC, ofInterest(modelC))
```

``` r
summary(periodContr)
```

    Model call:  gnm(formula = Freq ~ period:origin8 + period:dest8 + Mult(Exp(period),      origin8:dest8) - 1, ofInterest = "[.]period", family = poisson,      data = dataI, verbose = FALSE) 
                                               estimate         SE    quasiSE    quasiVar
        Mult(Exp(.), origin8:dest8).period1  0.00000000 0.00000000 0.04392732 0.001929609
        Mult(Exp(.), origin8:dest8).period2 -0.07768054 0.05784884 0.03809095 0.001450920
        Mult(Exp(.), origin8:dest8).period3 -0.18057676 0.05839454 0.03839825 0.001474426
        Mult(Exp(.), origin8:dest8).period4 -0.16868188 0.05524584 0.03374171 0.001138503
        Mult(Exp(.), origin8:dest8).period5 -0.31131465 0.05823650 0.03795222 0.001440371
        Mult(Exp(.), origin8:dest8).period6 -0.36722453 0.05893905 0.03894530 0.001516736
    Worst relative errors in SEs of simple contrasts (%):  -0.4 0.5 
    Worst relative errors over *all* contrasts (%):  -1.1 0.6 

We then consider two models suggested by Ganzeboom and Luijkx, with
either a linear or quadratic dependence of the multipliers on period
number.

``` r
modelD <- update(modelA, . ~ . + Mult(Const(1) + as.numeric(period),
                                      origin8:dest8),
                 ofInterest = "[.]as")
modelE <- update(modelA, . ~ . +
                 Mult(Const(1) + as.numeric(period)+ I(as.numeric(period)^2),
                      origin8:dest8),
                 ofInterest = "[.]as")
dev <- c(dev, "D" = deviance(modelD), "E" = deviance(modelE))
df <- c(df, "D" = df.residual(modelD), "E" = df.residual(modelE))
```

``` r
dev[c("D", "E")]
```

           D        E 
    236.3052 236.2995 

``` r
df[c("D", "E")]
```

      D   E 
    244 243 

Finally we propose a model in which the log-multipliers depend on period
number – note a `Const` term is unnecessary here as this would simply
scale the multipliers by a constant.

``` r
modelEa <- update(modelA, . ~ . + Mult(Exp(as.numeric(period)), origin8:dest8),
                  ofInterest = "[.]as")
dev <- c(dev, "Ea" = deviance(modelEa))
df <- c(df, "Ea" = df.residual(modelEa))
```

``` r
dev["Ea"]
```

          Ea 
    236.3904 

``` r
df["Ea"]
```

     Ea 
    244 

Having considered a selection of models, we impose some identifiability
constraints, for presentational purposes. For the UNIDIFF model, we
constrain the log-multiplier for the first period to 0:

``` r
modelC <- update(modelC, constrain = "[.]period1", constrainTo = 0)
```

For the structured UNIDIFF models, we fix the origin-destination
interaction as in `modelC` - note that this constraint slightly impairs
model fit, but enables the multipliers to be directly compared as in the
plot below.

``` r
keep <- pickCoef(modelC, "[.]origin")
modelD <- update(modelD, constrain = "[.]origin",
                 constrainTo = coef(modelC)[keep],
                 ofInterest = "[.]as")
modelEa <- update(modelEa, constrain = "[.]origin",
                  constrainTo = coef(modelC)[keep],
                  ofInterest = "[.]as")
```

A plot of the period contrasts from the UNIDIFF model and a second plot
of the corresponding scaling factors are created as follows. The points
are joined by lines in the second plot to highlight the trends in the
estimates.

``` r
par(mfrow = c(1,2), mar = c(4.5, 4.1, 3.1, 1.1), cex = 0.9, mex = 0.8,
    oma = c(0, 0, 0, 0))
plot(periodContr, xaxt = "n", xlab = "Period",
     ylab = "Log-multipliers of origin-destination association",
     main = "")
axis(1, lab = c("1970-4", "1975-9", "1980-4", "1985-9", "1990-4", "1995-9"),
     at = 1:6)
mtext("a)", side = 2, line = 2, at = 0.19, las = 2)
plot(1:6, exp(parameters(modelC)[ofInterest(modelC)]), ylim = c(0.6, 1.0),
     xlab = "Period number",
     ylab = "Multipliers of origin-destination association")
lines(1:6, 1 + coef(modelD)[ofInterest(modelD)]*1:6, lty = 2)
lines(1:6, exp(coef(modelEa)[ofInterest(modelEa)]*1:6), lty = 1)
mtext("b)", side = 2, line = 2, at = 1.03, las = 2)
```

<img src="UNIDIFF_simulated_files/figure-gfm/periodContr-1.png" title="contrasts of period multipliers" alt="contrasts of period multipliers" width="100%" />

We finally consider the Goodman-Hauser model selected by Ganzeboom and
Luijkx:

``` r
modelJ <- update(modelA, . ~ . + Diag(origin8, dest8) +
                 Mult(Const(1) + as.numeric(period), MultHomog(origin8, dest8)),
                 etastart = modelA$predictors)
dev <- c(dev, "J" = deviance(modelJ))
df <- c(df, "J" = df.residual(modelJ))
```

``` r
dev["J"]
```

           J 
    552.8534 

``` r
df["J"]
```

      J 
    278 

This time we focus on the homogenous multiplicative effects of
origin/destination class, using `getContrasts` to obtain the simple
contrasts of these parameters for summary and display:

``` r
classContr <- getContrasts(modelJ, pickCoef(modelJ, "[.]origin"))
```

``` r
summary(classContr)
```

    Model call:  gnm(formula = Freq ~ period:origin8 + period:dest8 + Diag(origin8,      dest8) + Mult(Const(1) + as.numeric(period), MultHomog(origin8,      dest8)) - 1, family = poisson, data = dataI, etastart = modelA$predictors,      verbose = FALSE) 
                                                                                         estimate         SE    quasiSE    quasiVar
        Mult(as.numeric(period) + Const(1), .)MultHomog(origin8, dest8).origin8|dest81 0.00000000 0.00000000 0.04648840 0.002161172
        Mult(as.numeric(period) + Const(1), .)MultHomog(origin8, dest8).origin8|dest82 0.06825937 0.06491878 0.04307077 0.001855091
        Mult(as.numeric(period) + Const(1), .)MultHomog(origin8, dest8).origin8|dest83 0.45231697 0.05524246 0.03602710 0.001297952
        Mult(as.numeric(period) + Const(1), .)MultHomog(origin8, dest8).origin8|dest84 0.73483908 0.05797822 0.03589153 0.001288202
        Mult(as.numeric(period) + Const(1), .)MultHomog(origin8, dest8).origin8|dest85 1.53261415 0.06815692 0.04724094 0.002231707
        Mult(as.numeric(period) + Const(1), .)MultHomog(origin8, dest8).origin8|dest86 1.23494012 0.05352805 0.02603765 0.000677959
        Mult(as.numeric(period) + Const(1), .)MultHomog(origin8, dest8).origin8|dest87 1.67086566 0.06255856 0.04062114 0.001650077
        Mult(as.numeric(period) + Const(1), .)MultHomog(origin8, dest8).origin8|dest88 2.27760147 0.09676955 0.08070731 0.006513671
    Worst relative errors in SEs of simple contrasts (%):  -4.2 7.2 
    Worst relative errors over *all* contrasts (%):  -14.6 19.3 

``` r
plot(classContr, xaxt = "n", xlab = "Social class",
     ylab = "Multiplicative effects of origin/destination class",
     main = "")
axis(1, lab = c("I", "II", "III", "IVab", "IVc", "V + VI", "VIIa", "VIIb"),
     at = 1:8)
```

<img src="UNIDIFF_simulated_files/figure-gfm/classContr-1.png" title="contrasts of class multipliers" alt="contrasts of class multipliers" width="100%" />

We evaluate the fit of this model by plotting the aggregate residuals:

``` r
mosaic(modelJ, ~origin8 + dest8,
       set_varnames =
       list(origin8 = "Origin class", dest8 = "Destination class"),
       set_labels =
       list(origin8 = c("I", "II", "III", "IVab", "IVc", "V + VI", "VIIa", "VIIb"),
       dest8 = c("I", "II", "III", "IVab", "IVc", "V + VI", "VIIa", "VIIb")),
       rot_labels = c(0, 0), offset_varnames = c(0.3, 0, 0, 1.3),
       offset_labels = c(0, 0, 0, 0.3), margins = c(1, 1, 0, 4))
```

<img src="UNIDIFF_simulated_files/figure-gfm/GHmosaic-1.png" title="aggregate residuals from Goodman-Hauser model" alt="aggregate residuals from Goodman-Hauser model" width="100%" />

## References

Ganzeboom, Harry B. G., and Ruud Luijkx. 2004. “Recent trends in
intergenerational occupational class reproduction in the Netherlands
1970-99.” In *Social Mobility in Europe*, edited by Richard Breen,
345–81. Oxford University Press.
