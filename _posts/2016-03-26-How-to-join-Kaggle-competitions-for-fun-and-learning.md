---
layout: post
title: "How to join Kaggle competitions for learning and fun"
author: long_nguyen
modified:
excerpt: "How to join Kaggle competitions for learning and fun"
tags: []
---
Kaggle is probably a well-known place for data science competitions with more than a half of million users. It bridges the gap between data scientists who, need data to play with and companies, which require top quality solutions for theirs business problems.

In my opinion, it is good to join Kaggle competitions for learning and fun. Firstly, you surely can learn a lot from these competitions. Each competition typically has more than 2000 participants; therefore, you definitely learn from this crowd. You can learn from the forum, scripts, and especially from the winning solutions. Secondly, it is fun since this is a place, where you can make new friends. You may form a team for the competitions and meet up for discussions.

The next question is how a person can effectively join Kaggle competitions? Here, I'd like to share some tips from my experience so that you can save your time and learn new knowledge in an effective way. Below are my basic steps when I join a competition.

1. **Structure your folders:** It is good to practice a standard folder structure when colloberating with your partners. You will need to organize our folders and don't want to spend so much time just for changing folder paths.
	-	.\input: &nbsp;&nbsp;&nbsp;&nbsp;	Folder for input data, feature data
	- .\R:&nbsp;&nbsp;&nbsp;&nbsp;	Folder for R source code
	- .\python:&nbsp;&nbsp;&nbsp;&nbsp;	Folder for python code
	- .\submission:&nbsp;&nbsp;&nbsp;&nbsp; Folder for submissions
	- .\stacking:&nbsp;&nbsp;&nbsp;&nbsp;	Folder for stacking
	- .\model:&nbsp;&nbsp;&nbsp;&nbsp;	Folder for saving trained models
	- .\figure:&nbsp;&nbsp;&nbsp;&nbsp; 	Folder for saving plots, figures
	
	Moreover, you may need to use *GitLab* for versioning and reproducing your code.
2. **Understand the dataset:** After creating your folder structure, it is the time to download the competition's data. It is worthy to find answers of the following questions:
	- Does the dataset contain missing values? Should missing values be replace by NA, median, -1, or -999? Typically, for a non-negative and numerical attribute/column, replacing missing values by -999 will work well.
	- How many ordinal columns are there in the dataset? Ordinal columns can be replaced by numbers. However, please consider magnitude gaps between ordinal values.
	- How many category/string column are there in the dataset? How many unique values does each category column have?
	- For each numerical column, what are its statistics, e.g, min, 1st quartile, median, mean, 3rd quartile, max. Histograms of each column should be plotted for further investigation.
	- What are the competition's problem? Classification or regression? Evaluation metric? Is the dataset imbalance?
3. **Check the forum:** Once you understand the dataset and competition's problem, it is time to read the forum for a quick catch-up on what participants have done.
	- Note down information on data quality discussions, which models work well.
	- Find good starting scripts to save your data pre-processing time.
4. **Test a variety of models:** You may have a set of machine learning models (which you have used for previous competitions) and want to test whether they can work well for this dataset. Below are some models for testing perfomance. Please take note that simple model will contribute more for later stacking step.
	- Logistic regession
	- Ridge, Lasso, ElasticNet, and SGD classifiers
	- Support Vector Machine (SVM)
	- Random forest (RF)
	- Extra Tree Classifier (ETC)
	- Gradient Boosting Machine (GBM)
	- Xgboost (XGB)
5. **Fine-tune the best model** After testing a variety of models, it is a good time to fine-tune the best model for satisfying the temptation of getting a higher rank. :D

	There are both GridSearch (python) and caret (R) for hyper-parameter tunning. However, these built-in functions are very expensive in terms of computing resource. My approach is tunning each parameter one by one. I usually vary one parameter and keep the rest unchange for finding the best value of the changing paramter. The process is repeated for all parameters.
6. **Feature selection:** Next, we need to remove redundent features and select the core set of features. It helps to reduce the complexity of the model while retaining comperative perfomance based on the full feature set. Moreover, it also helps to significantly reduce the numbers of *new* features that you will create in the next step.
	It is very convenient that RF, ETC, GBM, and XGB have built-in feature selection method, which is based on *coverage* of attribures. You can call the important_feature() function easily.
7. **Feature engineering:** People say *feature engineering* is the winning key. It is essenstial to check important articles about feature engineering in my reading list [link](../reading_list/). To be updated
8. **Stacking:** To be up
