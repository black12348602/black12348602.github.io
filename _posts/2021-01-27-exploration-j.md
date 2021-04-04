---
layout: post
title: "Exploration of Johnson & Johnson data"
date: 2014-04-30
---

## a)
```
library(astsa)
trend = time(jj) - 1970 # helps to 'center' time
Q = factor(cycle(jj) ) # make (Q)uarter factors
reg = lm(log(jj)~0 + trend + Q, na.action=NULL) # no intercept
summary(reg) # view the results 
```
$$\hat{x}_t = 0.167172 t + 1.052793 Q_1(t) + 1.080916 Q_2(t) + 1.151024 Q_3(t) + 0.882266 Q_4(t)$$
Since the p-value are less than 0.05, so the estimated parameters and the full model are very significant.

## b)
$$E(x_{t+1}-x_t) = E(\beta(t+1)-\beta t) = E(\beta) = \beta$$
The estimated average annual increase 0.167172 in the logged earnings per share.

## c)
If the other estimators remain the same,
$$Q_3: E(x_t) = E(\beta t + \alpha_3) = \beta t + \alpha_3$$  
$$Q_4: E(x_t) = E(\beta t + \alpha_4) = \beta t + \alpha_4$$  
$$Q_3-Q_4 = \alpha_3 - \alpha_4 = 1.151024 - 0.882266 = 0.268758$$  
The average logged earnings rate is decrease from the third quarter to the fourth quarter.
It decreased by (1.151024-0.882266)/1.151024 = 23.35%

## d)
```
reg.new = lm(log(jj)~ + trend + Q, na.action=NULL) # have intercept
summary(reg.new) #View the results
```
Compare to the model a, we add the intercept and the $Q_1$ is been taken, $\alpha_2$ and $\alpha_3$ become less significant since the p-value is greater than before.

## e)
```
xt=log(jj) #data
plot(xt, main = 'data and the fitted value', ylab = 'log earning') #data plots
fxt = fitted(reg) #fitted value
lines(fxt, col = 4) #fitted value plots
res = xt - fxt # define residuals = data - fitted values
plot(res, main = 'residuals', ylab = 'residuals') #residual plot
```

It is not the white noise since based on the residual plot, we can see that the mean is not constant since the pattern has obvious fluctuations, so the model didn't fit the data very well. 
