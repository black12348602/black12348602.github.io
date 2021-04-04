---
layout: post
title: "Exploration of cardiovascular mortality series"
date: 2021-03-17
---

```
library(dplyr)
library(tidyr)
library(ggplot2)
library(astsa)
```

# Q1
## a)
```
h = 1:10
(a <-(-1.0024)*(1.0847)^(-h)*cos(0.8622*h+3.0724))
(a_ <- ARMAacf(ar=c(1.2,-0.85),ma=0,10))
```

## b)
```
h = 1:10
(b = 2^(-h)*(1+0.6*h))
(b_ <- ARMAacf(ar=c(1,-0.25),ma=0,10))
```


# Q2
```
set.seed(1003772483)
dataframe = data.frame(list("model" = "True model", "Phi"= 0.9, "Theta" = -0.9, "Sigma" = 1))
for (i in 1:3){
  ARMA11 <- arima.sim(list(order = c(1,0,1),ar = 0.9, ma = -0.9), n = 500)
  ARMA11_=arima(ARMA11,order = c(1,0,1)) 
  coef = ARMA11_$coef
  dataframe = rbind(dataframe,list("model" = i, "Phi"= coef["ar1"], "Theta"= coef["ma1"], "Sigma" = ARMA11_$sigma2))
  plot.ts(ARMA11, main = "ARMA(1,1)")
  acf2(ARMA11, main = "ACF and PACF")
}

#plot(dataframe$Phi, col=c("red", "blue", "black", "green"), xlab = "aaa")
#legend("bottomleft", legend = c("True", "1", "2", "3"), col = c("red", "blue", "black", "green"), pch =1)

#plot(dataframe$Phi)

```


```
dataf <- dataframe %>%
  pivot_longer(c(-1)) 
dataf %>%
  ggplot(aes(x=name,y=value,fill=model)) +
  geom_dotplot(binaxis='y') +
  labs(title = "parameter estimates in each model", x="parameter") +
  guides(fill = guide_legend(title = "Model"))
```
we can see that the ACF and PACF graph of these models are quite similar, besides if the parameter estimates are different, the ACF and PACF graph are different for each model.

# Q3
## a)
```
(regr = ar.ols(cmort, order=2, demean=FALSE, intercept=TRUE))
x_1 = stats::lag(cmort,-1)
x_2 = stats::lag(cmort,-2)
fish = ts.intersect(cmort, x_1, x_2, dframe=TRUE)
summary(fit <- lm(cmort~x_1+x_2, data = fish, na.action=NULL))
```
$$Xt = 11.45+0.4286X_{t-1}+0.4418X_{t-2}$$
