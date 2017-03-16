---
layout: post
title: "SAS snippet code"
author: long_nguyen
modified:
excerpt: "SAS snippet code"
tags: []
---
Collection of small and re-usable source code for SAS
## Count number of rows/observation in a table
```SAS
/*define a macro function*/
%macro obsct (inpdat);
data _null_;
put nobs=;
stop;
set &inpdat nobs=nobs;
run;
%mend;
/*call the macro function*/
%obsct (Your_table); 
```
