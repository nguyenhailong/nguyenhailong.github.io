---
layout: post
title: "SAS snippet code"
author: long_nguyen
modified:
excerpt: "Collection of small and re-usable source code for SAS"
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
## Import data from .CSV file
```SAS
data your_table;  
infile 'your_table.csv' delimiter = ','  firstobs=2;  
length col1 $16. col2 $20. col3 $20.; 
input   col1 $   col2 $   col3 $   col4   col5   col6 ; 
run; 
```
## Print data summary
```SAS
PROC MEANS DATA=your_table N Q1 Median Q3 QRANGE Mean std Min Max MAXDEC=3;
	title 'Summary of your_table';
	VAR var1 var2 var3;
RUN;
```
