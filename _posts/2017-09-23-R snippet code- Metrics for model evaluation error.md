---
layout: post
title: "R snippet code- Metrics for model evaluation error"
author: long_nguyen
modified:
excerpt: "R snippet code- Metrics for model evaluation error"
tags: []
---
Collection of small and re-usable source code for R

Feel free to use it if you find it helpful.
## Library
```R
library(ROCR)
```

## Gain chart
```R
gain.chart <- function(score, y) {
  ## Shuffle predicition
  set.seed(0)
  rand <- sample(1:length(y))
  score <- score[rand]
  y <- y[rand]
  
  pred <- prediction(score, y)
  perf <- performance(pred, measure = "tpr", x.measure = "rpp")
  plot.data <- data.frame(xvals=unlist(perf@x.values), yvals=unlist(perf@y.values))
  theme_update(plot.title = element_text(hjust = 0.5))
  print(ggplot(plot.data, aes(x=xvals, ymin=0, ymax=yvals)) +
    geom_ribbon(alpha=0.2) + geom_line(aes(y=yvals)) + scale_x_continuous(breaks = seq(0, 1, by = 0.2)) +
    ggtitle("Gain chart") + xlab('% of Population') + ylab('% of active POS'))
  results.sortedByProb<-order(score,decreasing=TRUE)
  test_y.sorted <-y[results.sortedByProb]
  for(k in 1:10){
    cat('Gain ratio at ',k*10,'% is: ',round(sum(100*test_y.sorted[1:(length(test_y.sorted)*k/10)])/sum(test_y.sorted),digits=2),'% \n')
  }
  for(k in 1:10){
    cat('# Responders at ',k*10,'% is: ',round(sum(test_y.sorted[(length(y)*(k-1)/10+1):(length(y)*k/10)])),'\n')    
  }
}
```

## Lift chart
```R
lift.chart <- function(score, y) {
  pred <- prediction(score, y)
  perf <- performance(pred, measure = "lift", x.measure = "rpp")
  plot(perf, main="Lift curve", col = 'green')
}
```
## ROC chart
```R
roc.chart <- function(score, y) {
  pred <- prediction(score, y)
  perf <- performance(pred, measure = "tpr", x.measure = "fpr")
  auc <- performance(pred, measure = "auc")
  auc <- auc@y.values[[1]]*100
  plot.data <- data.frame(xvals=unlist(perf@x.values), yvals=unlist(perf@y.values))
  theme_update(plot.title = element_text(hjust = 0.5))
  print(ggplot(plot.data, aes(x=xvals, ymin=0, ymax=yvals)) +
    geom_ribbon(alpha=0.2) + geom_line(aes(y=yvals)) + scale_x_continuous(breaks = seq(0, 1, by = 0.2)) +
    ggtitle(paste0("ROC Curve with AUC=", round(auc,digits = 2),"%")) + xlab('FPR') + ylab('TPR'))
  return(auc)
}
```

## KS
```R
KS <- function(score, y){
  perf <- performance(prediction(score, y), "tpr", "fpr")
  ks <- max(attr(perf, "y.values")[[1]] - (attr(perf, "x.values")[[1]]))
  return(ks)
}
```
