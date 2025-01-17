## Fitting linear mixed-effects models

The `lmm` function is similar to the `lmer` function in the
[lme4](http://cran.R-project.org/package=lme4) package for
[R](http://www.R-project.org).  The first two arguments for in the `R`
version are `formula` and `data`.  The principle method for the
`Julia` version takes these arguments.

### A model fit to the `Dyestuff` data from the `lme4` package

The simplest example of a mixed-effects model that we use in the
[lme4 package for R](https://github.com/lme4/lme4) is a model fit to
the `Dyestuff` data.

```R
> str(Dyestuff)
'data.frame':	30 obs. of  2 variables:
 $ Batch: Factor w/ 6 levels "A","B","C","D",..: 1 1 1 1 1 2 2 2 2 2 ...
 $ Yield: num  1545 1440 1440 1520 1580 ...
> (fm1 <- lmer(Yield ~ 1|Batch, Dyestuff, REML=FALSE))
Linear mixed model fit by maximum likelihood ['lmerMod']
Formula: Yield ~ 1 | Batch
   Data: Dyestuff

      AIC       BIC    logLik  deviance
 333.3271  337.5307 -163.6635  327.3271

Random effects:
 Groups   Name        Variance Std.Dev.
 Batch    (Intercept) 1388     37.26
 Residual             2451     49.51
 Number of obs: 30, groups: Batch, 6

Fixed effects:
            Estimate Std. Error t value
(Intercept)  1527.50      17.69   86.33
```

These `Dyestuff` data are available through `RCall`
```julia
julia> using DataFrames,MixedModels, RCall

julia> @rimport lme4
WARNING: RCall.jl Loading required package: Matrix

julia> ds = rcopy(lme4.Dyestuff)
30x2 DataFrames.DataFrame
| Row | Batch | Yield  |
|-----|-------|--------|
| 1   | "A"   | 1545.0 |
| 2   | "A"   | 1440.0 |
| 3   | "A"   | 1440.0 |
| 4   | "A"   | 1520.0 |
| 5   | "A"   | 1580.0 |
| 6   | "B"   | 1540.0 |
| 7   | "B"   | 1555.0 |
| 8   | "B"   | 1490.0 |
| 9   | "B"   | 1560.0 |
| 10  | "B"   | 1495.0 |
| 11  | "C"   | 1595.0 |
| 12  | "C"   | 1550.0 |
| 13  | "C"   | 1605.0 |
| 14  | "C"   | 1510.0 |
| 15  | "C"   | 1560.0 |
| 16  | "D"   | 1445.0 |
| 17  | "D"   | 1440.0 |
| 18  | "D"   | 1595.0 |
| 19  | "D"   | 1465.0 |
| 20  | "D"   | 1545.0 |
| 21  | "E"   | 1595.0 |
| 22  | "E"   | 1630.0 |
| 23  | "E"   | 1515.0 |
| 24  | "E"   | 1635.0 |
| 25  | "E"   | 1625.0 |
| 26  | "F"   | 1520.0 |
| 27  | "F"   | 1455.0 |
| 28  | "F"   | 1450.0 |
| 29  | "F"   | 1480.0 |
| 30  | "F"   | 1445.0 |
```

`lmm` defaults to maximum likelihood estimation whereas `lmer` in `R`
defaults to REML estimation.

```julia
julia> m = fit!(lmm(Yield ~ 1 + (1|Batch),ds))
Linear mixed model fit by maximum likelihood
 logLik: -163.663530, deviance: 327.327060, AIC: 333.327060, BIC: 337.530652

Variance components:
           Variance  Std.Dev.
 Batch    1388.3332 37.260344
 Residual 2451.2500 49.510100
 Number of obs: 30; levels of grouping factors: 6

  Fixed-effects parameters:
             Estimate Std.Error z value
(Intercept)    1527.5   17.6946  86.326
```

In general the model should be fit through an explicit call to the `fit!`
function, which may take a second argument indicating a verbose fit.

```julia
julia> gc(); @time fit!(lmm(Yield ~ 1 + (1|Batch), ds),true);
f_1: 327.76702, [1.0]
f_2: 331.03619, [1.75]
f_3: 330.64583, [0.25]
f_4: 327.69511, [0.97619]
f_5: 327.56631, [0.928569]
f_6: 327.3826, [0.833327]
f_7: 327.35315, [0.807188]
f_8: 327.34663, [0.799688]
f_9: 327.341, [0.792188]
f_10: 327.33253, [0.777188]
f_11: 327.32733, [0.747188]
f_12: 327.32862, [0.739688]
f_13: 327.32706, [0.752777]
f_14: 327.32707, [0.753527]
f_15: 327.32706, [0.752584]
f_16: 327.32706, [0.752509]
f_17: 327.32706, [0.752591]
f_18: 327.32706, [0.752581]
FTOL_REACHED
  0.001722 seconds (2.00 k allocations: 82.828 KB)
```

The numeric representation of the model has type
```julia
julia> typeof(m)
MixedModels.LinearMixedModel{Float64}
```
Those familiar with the `lme4` package for `R` will see the usual
suspects.
```julia
julia> fixef(m)  # estimates of the fixed-effects parameters
1-element Array{Float64,1}:
 1527.5

julia> show(coef(m))  # another name for fixef
[1527.5]

julia> ranef(m)
1-element Array{Any,1}:
 1x6 Array{Float64,2}:
 -16.6282  0.369516  26.9747  -21.8014  53.5798  -42.4943

julia> ranef(m,true)  # on the u scale
1-element Array{Any,1}:
 1x6 Array{Float64,2}:
 -22.0949  0.490999  35.8429  -28.9689  71.1948  -56.4649

julia> deviance(m)
327.3270598811394
```

It may be more accurate to refer to this as the `objective`, which is
`-2loglikelihood(m)`, without the correction for the null deviance.
(It is not clear how the null deviance should be defined for these models.)

```julia
julia> objective(m)
327.3270598811394
```

## A more substantial example

Fitting a model to the `Dyestuff` data is trivial.  The `InstEval`
data in the `lme4` package is more of a challenge in that there are
nearly 75,000 evaluations by 2972 students on a total of 1128
instructors.

```julia
julia> inst = rcopy(lme4.InstEval);

julia> head(inst)
6x7 DataFrames.DataFrame
| Row | s   | d      | studage | lectage | service | dept | y |
|-----|-----|--------|---------|---------|---------|------|---|
| 1   | "1" | "1002" | "2"     | "2"     | "0"     | "2"  | 5 |
| 2   | "1" | "1050" | "2"     | "1"     | "1"     | "6"  | 2 |
| 3   | "1" | "1582" | "2"     | "2"     | "0"     | "2"  | 5 |
| 4   | "1" | "2050" | "2"     | "2"     | "1"     | "3"  | 3 |
| 5   | "2" | "115"  | "2"     | "1"     | "0"     | "5"  | 2 |
| 6   | "2" | "756"  | "2"     | "1"     | "0"     | "5"  | 4 |

julia> m2 = fit!(lmm(y ~ 1 + dept*service + (1|s) + (1|d), inst))
Linear mixed model fit by maximum likelihood
 logLik: -118792.776708, deviance: 237585.553415, AIC: 237647.553415, BIC: 237932.876339

Variance components:
            Variance   Std.Dev.  
 s        0.105417973 0.32468134
 d        0.258416365 0.50834670
 Residual 1.384727773 1.17674457
 Number of obs: 73421; levels of grouping factors: 2972, 1128

  Fixed-effects parameters:
                           Estimate Std.Error   z value
(Intercept)                 3.22961  0.064053   50.4209
dept - 5                   0.129536  0.101294   1.27882
dept - 10                 -0.176751 0.0881352  -2.00545
dept - 12                 0.0517102 0.0817523  0.632522
dept - 6                  0.0347319  0.085621  0.405647
dept - 7                    0.14594 0.0997984   1.46235
dept - 4                   0.151689 0.0816897   1.85689
dept - 8                   0.104206  0.118751  0.877517
dept - 9                  0.0440401 0.0962985  0.457329
dept - 14                 0.0517546 0.0986029  0.524879
dept - 1                  0.0466719  0.101942  0.457828
dept - 3                  0.0563461 0.0977925   0.57618
dept - 11                 0.0596536  0.100233  0.595151
dept - 2                 0.00556281  0.110867 0.0501757
service - 1                0.252025 0.0686507   3.67112
dept - 5 & service - 1    -0.180757  0.123179  -1.46744
dept - 10 & service - 1   0.0186492  0.110017  0.169512
dept - 12 & service - 1   -0.282269 0.0792937  -3.55979
dept - 6 & service - 1    -0.494464 0.0790278  -6.25683
dept - 7 & service - 1    -0.392054  0.110313  -3.55403
dept - 4 & service - 1    -0.278547 0.0823727  -3.38154
dept - 8 & service - 1    -0.189526  0.111449  -1.70056
dept - 9 & service - 1    -0.499868 0.0885423  -5.64553
dept - 14 & service - 1   -0.497162 0.0917162  -5.42065
dept - 1 & service - 1     -0.24042 0.0982071   -2.4481
dept - 3 & service - 1    -0.223013 0.0890548  -2.50422
dept - 11 & service - 1   -0.516997 0.0809077  -6.38997
dept - 2 & service - 1    -0.384773  0.091843  -4.18946


julia> gc(); @time fit!(lmm(y ~ 1 + dept*service + (1|s) + (1|d), inst));
  2.203323 seconds (20.09 M allocations: 450.600 MB, 1.34% gc time)
```

Models with vector-valued random effects can be fit
```julia
julia> slp = rcopy(lme4.sleepstudy)
180x3 DataFrames.DataFrame
| Row | Reaction | Days | Subject |
|-----|----------|------|---------|
| 1   | 249.56   | 0.0  | "308"   |
| 2   | 258.705  | 1.0  | "308"   |
| 3   | 250.801  | 2.0  | "308"   |
| 4   | 321.44   | 3.0  | "308"   |
| 5   | 356.852  | 4.0  | "308"   |
| 6   | 414.69   | 5.0  | "308"   |
| 7   | 382.204  | 6.0  | "308"   |
| 8   | 290.149  | 7.0  | "308"   |
| 9   | 430.585  | 8.0  | "308"   |
| 10  | 466.353  | 9.0  | "308"   |
| 11  | 222.734  | 0.0  | "309"   |
⋮
| 169 | 350.781  | 8.0  | "371"   |
| 170 | 369.469  | 9.0  | "371"   |
| 171 | 269.412  | 0.0  | "372"   |
| 172 | 273.474  | 1.0  | "372"   |
| 173 | 297.597  | 2.0  | "372"   |
| 174 | 310.632  | 3.0  | "372"   |
| 175 | 287.173  | 4.0  | "372"   |
| 176 | 329.608  | 5.0  | "372"   |
| 177 | 334.482  | 6.0  | "372"   |
| 178 | 343.22   | 7.0  | "372"   |
| 179 | 369.142  | 8.0  | "372"   |
| 180 | 364.124  | 9.0  | "372"   |

julia> fm3 = fit!(lmm(Reaction ~ 1 + Days + (1+Days|Subject), slp))
Linear mixed model fit by maximum likelihood
 logLik: -875.969672, deviance: 1751.939344, AIC: 1763.939344, BIC: 1783.097086

Variance components:
           Variance  Std.Dev.   Corr.
 Subject  565.51066 23.780468
           32.68212  5.716828  0.08
 Residual 654.94145 25.591824
 Number of obs: 180; levels of grouping factors: 18

  Fixed-effects parameters:
             Estimate Std.Error z value
(Intercept)   251.405   6.63226 37.9064
Days          10.4673   1.50224 6.96781
```
