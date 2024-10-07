# LifeArc virtual work experience: Analysis of MAP immunofluorescence data for the neuronal differentiation optimisation experiment

<!--StartFragment-->

LifeArc virtual work experience: Analysis of MAP2 immunofluorescence data for the neuronal differentiation optimisation experiment 

**1. Summary of the analysis task** 

Given an input table listing the proportion of cells expressing a neuronal marker MAP2 for each combination of treatments (a dose of NGN2 virus +/- NT3 supplement; 3 replicates per condition), select the treatment combination that yields the highest proportion of MAP2-expressing cells (i.e., neurons). 

We will use data visualisations and a formal statistical analysis by logistic regression to decide on the best treatment combination. **2. Getting started** 

Welcome to the R notebook that will guide you through your data analysis task. Whether you are an R aficionado or it is the first time using it, we hope that you will find this analysis feasible, enjoyable and educational. 

For those new to R notebooks, please note how the code is organised in “chunks” surrounded by free text. Please execute the code in each chunk of the code in turn by pressing on the green arrow at its top right corner. The output (either text or graphics) will appear immediately underneath the chunk. 

In some chunks, there will be bits of code that you will need to modify. Don’t worry if you’ve never used R before - we’ll give you detailed explanations beforehand. 

We’ll start by installing several external packages (also known as libraries) that we will need for this analysis. This should only be run once - after this we recommend that you comment out these lines by adding the “#” sign to each of them. 

_#install.packages("effects")_ 

_#install.packages("dplyr")_ 

_#install.packages("ggplot2")_ 

Now that these libraries are installed, we need to load them. This will need to be done every time. 

**library**(dplyr) 

**library**(ggplot2) 

**library**(effects) 

**3. Load the data** 

The input data table contains the proportion of MAP2-positive cells in each well of our multiplexed experiment (in the “_fractMAP2”_ column). The experimental conditions in each well - the concentrations of the NGN2 and control viruses (in MOI), and the amount of the NT3 supplement - are recorded in the _NGN2\_MOI, CONTROL\_MOI_ and _NT3\_ng_ columns, respectively. 

Let’s load our dataset using the _read.table()_ function. 

**Replace the path in quotes with the path to the input data file on your machine** 

Note that “<-” is a **variable assignment** operator in R, so the table will be saved in a new variable called _data_. data <- read.table("Input data.txt", sep='\t', header=TRUE) 

Let’s print out the data as a table to have a look. 

data 

**SampleID** \<int> 

**NGN2\_MOI** \<dbl> 

**Control\_MOI** \<dbl> 

**NT3\_ng** \<int> 

**fractMAP2** \<dbl> 

1-10 of 24 rows 

1 0 5 0 0.002 2 0 5 0 0.001 3 0 5 0 0.001 4 0 5 10 0.003 5 0 5 10 0.002 6 0 5 10 0.004 7 2 0 0 0.021 8 2 0 0 0.018 9 2 0 0 0.026 

10 2 0 10 0.001 Previous **1** 2 3 Next 

As you can see, this experimental design tests four doses of the NGN2 virus (0, 2, 5 and 10 MOI), using 5 MOI of the empty virus vector as a negative control in the wells where no NGN2 virus was added. In addition, for each dose of the NGN2 virus, the effects of the NT3 supplement were tested at 0 and 10 ng/mL. There are eight experimental conditions, with each condition replicated over three wells, giving 24 samples overall (one 24-well plate). 

Note that our experimental design is not really appropriate to assess the effect of the empty vector control itself on the outcome (_can you see why?_), so we will disregard the _CONTROL\_MOI_ column in our analysis. 

**4. Represent treatment doses as categorical data** 

Currently, our treatment doses are represented as numbers. However, we don’t necessarily expect the doses to show a consistent trend with respect to the outcome (i.e., it is possible that both “too much” and “not enough” of the virus will impair differentiation efficiency). It may therefore more appropriate in our setting to treat each dose as a separate “**category**”, in the same way as we would typically encode colours or genders. (Note that categorical encoding does not mean that the underlying data must be inherently discrete!). 

To tell R that our treatment doses should be considered as categorical data, we need to transform the corresponding columns from the numerical into the so-called **factor** format. We will define the “**levels**” of each factor such that the dose of 0 will be considered as a “control”, with the following levels corresponding to increasing doses. This will help us in data visualisation and regression analysis further down below. 

Note how the “$” operator in R allows us to access specific columns in a table by their names. 

data$NGN2\_MOI <- factor(data$NGN2\_MOI, levels=c(0,2,5,10)) 

data$NT3\_ng <- factor(data$NT3\_ng, levels=c(0,10)) 

We will print just the top bit of the modified table now using the _head()_ function. 

head(data) 

**SampleID** \<int> 

**NGN2\_MOI** \<fct> 

**Control\_MOI** \<dbl> 

**NT3\_ng** \<fct> 

**fractMAP2** \<dbl> 

1 1 0 5 0 0.002 2 2 0 5 0 0.001 3 3 0 5 0 0.001 4 4 0 5 10 0.003 5 5 0 5 10 0.002 6 6 0 5 10 0.004 6 rows 

**Question:** Can you see what has changed in the output? 

Answer: The format of NGN2\_MOI and NT3\_ng columns is now showing as “\<fctr>” rather than the numeric formats “\<dbl>” or “\<int>”. **5. Plot the data** 

Let’s now generate a bar plot the data. We will construct the plot such that the heights of the bars show the mean fraction of MAP2-positive cells for each treatment condition, and the error bars showing standard deviations across the three replicates. 

To do this, we will first generate a summary table with **conditions** in rows (8 in total), and the **means** and **standard deviations** across replicates in columns. We will call the columns _meanFractMAP2_ and _sdFractMAP2_, respectively. 

We will use the _group\_by()_ and _summarise()_ functions from the _dplyr_ package as below. 

The column name(s) supplied to _group\_by()_ tell this function that the rows in which the values across all of these columns are identical should be considered a single group. In our case, a single group will be all rows with the same doses of the NGN2 virus and NT3 supplement - in other words, all replicates of the same treatment condition. 

_summarise()_ will create a table with one row per group, where each column will correspond to some summary function applied to each group (in our case, the mean and standard deviation (_sd_) of the fractMAP column). 

_%>%_ is a pipe operator used in some R packages, which works by “forwarding” the data from the left side of the operator to a function on the right side of it. This avoids the need to assign intermediate results of multiple operations to separate variables and makes the code tidier and more readable. 

sumdata <- data %>% 

group\_by(NGN2\_MOI, NT3\_ng) %>% 

summarise(meanFractMAP2=mean(fractMAP2), sdFractMAP2 = sd(fractMAP2)) 

\## \`summarise()\` has grouped output by 'NGN2\_MOI'. You can override using the 

\## \`.groups\` argument. 

We can now visualise the data from the summary table as a barplot. We will use the powerful _ggplot2_ package for this. You don’t need to understand each parameter in the code below, but note how the plot is “assembled” from a “**sum**” of different function calls: 

_ggplot()_ defines what table should be used as **input** (in our case, _sumdata_) and which of its columns should be used for the plotting and how. Here we will ask it to plot the fraction of MAP2+ cells (_meanFractMAP2_) along the **y** axis, plot the bars for different concentrations of the NGN2 virus (_NGN2\_MOI_) along the **x** axis, and use different **fill colour** for the conditions with and without NT3 (_NT3\_ng_). 

_geom\_bar()_ determines that these data should be plotted as a **barplot** (i.e., with the **y** axis reflecting the height of the bars). The _position\_dodge()_ call inside this function determines that the two barplots for the same virus concentration (with and without NT3, respectively) should be plotted next to each other (rather than stacked). 

_geom\_errorbar()_ determines how the **error bars** should be plotted. Here we define that the bottom and the top boundaries of the error bars should correspond to the _(mean+/-standard deviation)_ for each condition, respectively. 

ggplot(sumdata, aes(x=NGN2\_MOI, y=meanFractMAP2, fill=NT3\_ng)) + 

geom\_bar(position=position\_dodge(), stat="identity", colour='black') + 

geom\_errorbar(aes(ymin=meanFractMAP2-sdFractMAP2, ymax=meanFractMAP2+sdFractMAP2), 

width=0.2, position=position\_dodge(0.9)) 

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXe8yaw-zfCj7SaRiLSgSi7J6wtkX_B0h2HuSQBrl2kmQOg-_GgT-OOg9wBHWbsaj148wfnpEtL18cEQGqr-KhEDmsSGVqGYKNl9L7XcZ498qOpj-ebQTA3Z6j-b68JB4XLB7ok0ldz4k-xsXSRa2-9PyF2O?key=-1ZWPahGhCrEAwSTS-_Puw)

Let’s now try replotting the data by averaging across NT3-treated and untreated wells. This way we will only have **four bars** in the bar plot, one per each NGN2 virus dose. 

First, we will create a different summary table for this, where we will group the data differently. 

**Can you have a go at the correct parameter for this _group\_by()_ function in this case?** 

sumdataNGN2 <- data %>% 

group\_by(NGN2\_MOI) %>% 

summarise(meanFracMAP2=mean(fractMAP2), sdFracMAP2 = sd(fractMAP2)) 

The rest of the code is ready to go (note that we’ve removed the _fill_ parameter from _ggplot()_ as with just one parameter to plot - NGN2 virus dose - we no longer need to use the fill colour to represent the second condition). 

ggplot(sumdataNGN2, aes(x=NGN2\_MOI, y=meanFracMAP2)) + 

geom\_bar(stat="identity") + 

geom\_errorbar(aes(ymin=meanFracMAP2-sdFracMAP2, ymax=meanFracMAP2+sdFracMAP2)) 

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXehzLfN70cpj1bmdeSyzM-_ool7Dmk22Pv8Q9Ly9KGrjAWvyiPpWeXjwFKCp0lOtn1uzVPax1AzSSiq0pp8Lmd72EhShcEjOOiSkr6oV09YgQeh0DKdZ-_4eP0FoWTTRZxHIYlB2B6plDK2XuzhXCAvwuop?key=-1ZWPahGhCrEAwSTS-_Puw)

Let’s now do the same by averaging across all NGN2 doses, such that we only have **two bars** in the bar plot, one per each NT3 dose (treated and untreated). 

**Please insert the correct grouping into _group\_by()_ for this visualisation.** 

sumdataNGN2 <- data %>% 

group\_by(NT3\_ng) %>% 

summarise(meanFracMAP2=mean(fractMAP2), sdFracMAP2 = sd(fractMAP2)) 

ggplot(sumdataNGN2, aes(x=NT3\_ng, y=meanFracMAP2)) + 

geom\_bar(stat="identity") + 

geom\_errorbar(aes(ymin=meanFracMAP2-sdFracMAP2, ymax=meanFracMAP2+sdFracMAP2), width=0.2, 

position=position\_dodge(0.9)) 

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXfCXw6gbLg8u0XCWjtHCzm2wiaFyVtRH5jDsktNg1ADSn7Ze8OcHVIvxzi4s3Ywl2JxjS9mUNygG6F95KOYNZ1Jzo_qkkYO1Zhiq33_n8Li4DpWeIVZatvKsdUGm5pUidhX1VfyM7x-wIEvykg76h5Nn__T?key=-1ZWPahGhCrEAwSTS-_Puw)

**Question**: Look at the error bars in the plot above. What do you notice and how would you interpret this result? 

Answer: The error bars are huge, so NT3 treatment explains only a relatively small part of the variation in MAP2 positivity across conditions. **6\*. Formally test the effects of treatment doses on differentiation efficiency** 

This part is a bit more advanced than what we have been doing so far. If getting to this point took you quite a bit of time already, you may consider skipping this section and heading straight to section 7. 

From the plots above, we can probably have some idea already of what combination of treatment doses has worked best, and which treatment had a larger effect. However, we still need to test this formally. 

Importantly, the outcomes that we are comparing are given by **proportions** (of MAP2-positive cells), and proportions are not distributed normally. Therefore, we shouldn’t use standard significance tests such as the T test or ANOVA for this analysis. 

In addition, we have many conditions, so rather than doing lots of pairwise comparisons (and accumulating multiple testing errors), we are going to tie them all together into a regression model. 

When the outcome is given by a proportion, one suitable regression framework is **logistic regression**. Formally, logistic regression is a type of generalised linear models of the form: 

**f(y) = k x + k x + … + b** 

**1 1 2 2** 

**y** is a binary outcome variable (such as “pass”/“fail” or “heads”/“tails”); 

**f(y)** is typically the log-odds of success: _f(y)_ = _logit(y) = log(y/1-y)_; 

**x** , **x** , … are explanatory (predictor) variables; 

**1 2** 

**k** , **k** , … are the regression coefficients to be determined by the regression analysis; 

**1 1** 

**b** is the intercept term that affects the outcome uniformly and does not depend on the predictor variables. 

Regression analysis can be used to assess which (if any) explanatory variables (in our case, the two treatments we have varied) affect the chances of a positive outcome (in our case, the fraction of MAP2-expressing cells generated in the differentiation experiment). 

Note: The explanatory variables can be either continuous or categorical (as in our case, as we are using treatment doses as categorical variables for reasons explained above). 

Also note that in our case instead of single outcomes for each observation (which would be: “Is a given cell MAP2-positive?”), we will use the **fraction of MAP2-positive cells per well** as the outcome variable. 

One key additional piece of information that we need for this analysis is the **total number of cells scanned in each well**. 

_The input file you are given doesn’t provide this information, so you had to check with the colleague who has performed this experiment. She told you that the data are based on roughly_ **_10,000 cells_** _scanned in each well_**_._** 

We will now use R to fit our logistic regression model using the _glm()_ function, which has the following key parameters: _data -_ our input table as the _data_ parameter 

_family_ - the type of the generalised regression model (for logistic regression, it should be set to “binomial”) 

the model _formula_ in the form _y \~ x1 + … + xN_. 

The measurements of both the outcome variable **y** and all explanatory variables **x** , …, **x** should be provided in the input data table under the 

**1 N**

corresponding column names. 

For example, if we are assessing the relationship between the chance of getting heads in a coin flip and the temperature and humidity in the room, the model formula could be: _isHeads \~ temp + humidity_, assuming the input data table has the relevant information encoded in the columns named “isHeads”, “temp” and “humidity”. 

**Please think of the outcome variable and explanatory variables in our model and input them into the formula below.** 

mod = glm(fractMAP2\~NGN2\_MOI+NT3\_ng, family=binomial(link="logit"), data=data, 

weights = rep(10000,nrow(data))) 

Note how we have entered the number of cells tested in each row as the _weights_ parameter. In our case, they are the same for each well, so we are just repeating the same number for each row, but they could also be different. 

Also note that in this model we are assuming that the effects of virus concentration and NT3 treatment are fully independent. If we aren’t sure about it, it may make sense to test their **interaction effects** as well. If you are interested, you can read up on how to do this in R very straightforwardly - for example, here: https\://www\.theanalysisfactor.com/generalized-linear-models-glm-r-part4/. Feel free to try this analysis in a separate chunk below. 

Let’s now inspect the summary of this model generated by R. 

summary(mod) 

\## 

\## Call: 

\## glm(formula = fractMAP2 \~ NGN2\_MOI + NT3\_ng, family = binomial(link = "logit"), 

\## data = data, weights = rep(10000, nrow(data))) 

\## 

\## Deviance Residuals: 

\## Min 1Q Median 3Q Max 

\## -18.2885 -2.1286 0.5841 2.5317 8.0978 

\## 

\## Coefficients: 

\## Estimate Std. Error z value Pr(>|z|) 

\## (Intercept) -6.30098 0.08862 -71.10 <2e-16 \*\*\* 

\## NGN2\_MOI2 2.12408 0.09307 22.82 <2e-16 \*\*\* 

\## NGN2\_MOI5 3.97231 0.08883 44.72 <2e-16 \*\*\* 

\## NGN2\_MOI10 3.11278 0.08992 34.62 <2e-16 \*\*\* 

\## NT3\_ng10 0.31290 0.02084 15.02 <2e-16 \*\*\* 

\## --- 

\## Signif. codes: 0 '\*\*\*' 0.001 '\*\*' 0.01 '\*' 0.05 '.' 0.1 ' ' 1 

\## 

\## (Dispersion parameter for binomial family taken to be 1) 

\## 

\## Null deviance: 10204.34 on 23 degrees of freedom 

\## Residual deviance: 712.97 on 19 degrees of freedom 

\## AIC: 890.95 

\## 

\## Number of Fisher Scoring iterations: 5 

This command prints a few things, but we will focus on the following: 

The **values** of each regression coefficient given in the _Estimate_ column. 

For our purposes, it is sufficient to know that the sign of the regression coefficient corresponds to whether inreasing the value of the predictor results in either increasing (+) or decreasing (-) the chances of success (in our case, on MAP2 expression). 

Also, the higher is the absolute value of the coefficient, the stronger is the effect of the predictor on the chances of success. 

If you’d like to learn more about interpreting logistic regression coefficients, you can check, for example, this page for details: https\://stats.oarc.ucla.edu/r/dae/logit-regression/. 

The **significance** of the coefficients’ difference from zero (with a coefficient of zero corresponding to the predictor not affecting the outcome). The p-values for this significance are given in the _Pr(>|z|)_ column. 

You have also probably noticed that the names of explanatory variables listed in the summary have changed a little from what we have provided originally. This is because for categorical variables defined this way, R is testing **the effect of each level separately** compared with the first one. This way, each treatment level becomes a separate independent explanatory variable, which R names by adding the treatment level (in our case, concentration) to the original variable name. 

If you are interested in more details about regression analysis with categorical predictor variables, you can refer, e.g., to this page: http\://www\.sthda.com/english/articles/40-regression-analysis/163-regression-with-categorical-variables-dummy-coding-essentials-in r/. 

We can now **plot the predictions** of this model by using just one line of code in R. 

plot(allEffects(mod)) 

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXcgDFZ19L1dDQPxdyuEPTHOVGGY_tV0jGDcCfxzs9zuxr4xLQUBRC2pmiZL_MpBH8K0nZhYXXFaUwoUVQZuoHVHzsBmpUmqCD4DUOcL2mp_GA7jfUx8LXfL6v8h9_VunG1d-93aBeUBhBguCHaRuJ2LYwT4?key=-1ZWPahGhCrEAwSTS-_Puw)

**Questions:** 

\- What treatments/doses have a **significant** effect on the outcome compared with the no-treatment control? 

Answer: All treatments and doses analysed. 

\- What treatment/dose has the **strongest positve** effect on the outcome compared with the control? 

Answer: NGN2 virus transduction at 5 MOI. 

Finally, as you may remember, the regression coefficients and their significance are reported with respect to the no-treatment controls. In the case of NGN2 virus, they clearly show that using the virus works better than not doing it. But is the **difference between the various doses of the virus** significant? 

The most straightforward way to test this is to swap the control dose of the virus against which we are testing from 0 MOI to the best-working dose: 5 MOI (that is currently level 3). Do other doses doses perform _significantly_ worse? 

We will use the _relevel()_ function to do this, rerun regression analysis, and then return the levels back to the original order. **Please insert the model formula into the glm function (it didn’t change from the analyses above)** 

levels(data$NGN2\_MOI) _# the original level order_ 

\## \[1] "0" "2" "5" "10" 

data$NGN2\_MOI = relevel(data$NGN2\_MOI, ref=3) 

levels(data$NGN2\_MOI) _# the new level order_ 

\## \[1] "5" "0" "2" "10" 

mod5MOI = glm(fractMAP2\~NGN2\_MOI+NT3\_ng, family=binomial(link="logit"), data=data, 

weights = rep(10000,nrow(data))) 

summary(mod5MOI) 

\## 

\## Call: 

\## glm(formula = fractMAP2 \~ NGN2\_MOI + NT3\_ng, family = binomial(link = "logit"), 

\## data = data, weights = rep(10000, nrow(data))) 

\## 

\## Deviance Residuals: 

\## Min 1Q Median 3Q Max 

\## -18.2885 -2.1286 0.5841 2.5317 8.0978 

\## 

\## Coefficients: 

\## Estimate Std. Error z value Pr(>|z|) 

\## (Intercept) -2.32867 0.01782 -130.66 <2e-16 \*\*\* 

\## NGN2\_MOI0 -3.97231 0.08883 -44.72 <2e-16 \*\*\* 

\## NGN2\_MOI2 -1.84823 0.03365 -54.92 <2e-16 \*\*\* 

\## NGN2\_MOI10 -0.85953 0.02360 -36.42 <2e-16 \*\*\* 

\## NT3\_ng10 0.31290 0.02084 15.02 <2e-16 \*\*\* 

\## --- 

\## Signif. codes: 0 '\*\*\*' 0.001 '\*\*' 0.01 '\*' 0.05 '.' 0.1 ' ' 1 

\## 

\## (Dispersion parameter for binomial family taken to be 1) 

\## 

\## Null deviance: 10204.34 on 23 degrees of freedom 

\## Residual deviance: 712.97 on 19 degrees of freedom 

\## AIC: 890.95 

\## 

\## Number of Fisher Scoring iterations: 5 

_# Restore the original order - note that relevel() won't work as it will put 5 before 2_ 

data$NGN2\_MOI = factor(data$NGN2\_MOI, levels = c("0", "2", "5", "10")) 

levels(data$NGN2\_MOI) _# back to how they were originally_ 

\## \[1] "0" "2" "5" "10" 

**Question:** Did 5 MOI of the NGN2 virus perform significantly better than 2 and 10 MOI? 

Answer: Yes, it did. 

**7. Outlier removal** 

Let’s have another look at the very first plot. Did you notice that one treatment condition has a surprisingly large standard deviation (almost equal to the mean). 

**Question**: Can you see what condition it is? 

Answer: NGN2 - 2 MOI; NT3 - 10 ng/mL. 

If you locate this condition in the experimental table, you’ll see that it’s likely to do with one of the three replicates for this condition being a “dropout”, in which either the differentiation or the immunostaining hasn’t really worked for some technical reasons. 

Let’s **remove this dropout replicate** and rerun the analysis. For this, we will locate the row in the original table corresponding to this replicate and create a new data table with this row filtered out. 

**Enter the correct row instead of ROW\_NUMBER below.** 

dataFilt = data\[-10,] 

We can now rerun the analysis. 

First, let’s **regenerate the summary table** and **replot the data**. You can see that the error bar for the affected condition is now much smaller. 

sumdataFilt <- dataFilt %>% 

group\_by(NGN2\_MOI, NT3\_ng) %>% 

summarise(meanFractMAP2=mean(fractMAP2), sdFractMAP2 = sd(fractMAP2)) 

\## \`summarise()\` has grouped output by 'NGN2\_MOI'. You can override using the 

\## \`.groups\` argument. 

ggplot(sumdataFilt, aes(x=NGN2\_MOI, y=meanFractMAP2, fill=NT3\_ng)) + 

geom\_bar(position=position\_dodge(), stat="identity", colour='black') + 

geom\_errorbar(aes(ymin=meanFractMAP2-sdFractMAP2, ymax=meanFractMAP2+sdFractMAP2), 

width=0.2,position=position\_dodge(0.9)) 

![](https://lh7-rt.googleusercontent.com/docsz/AD_4nXdDzwuoleD0Ba2vQPHY7H_m6qPhz3nLt6GDrYbt025x6q8QI0cXW2Wvhxjzqo1WruuP4qVOkErQ8a5PK5GHHU2AUmVGw_et-gIigVqmkNd2m1GtdZpdkv7peFL0uYtW4oh341zZldsRbsga56WRgBh-sj4F?key=-1ZWPahGhCrEAwSTS-_Puw)

If you skipped section 6, skip the remainder of this section and proceed to section 8. 

Let’s **rerun the logistic regression model** and compare the results with those run on unfiltered data. 

**Enter the model formula in the glm() function call below.** 

modFilt = glm(fractMAP2\~NGN2\_MOI+NT3\_ng, family=binomial(link="logit"), data=dataFilt, 

weights = rep(10000,nrow(dataFilt))) 

message("With the outlier:") 

\## With the outlier: 

summary(mod) 

\## 

\## Call: 

\## glm(formula = fractMAP2 \~ NGN2\_MOI + NT3\_ng, family = binomial(link = "logit"), 

\## data = data, weights = rep(10000, nrow(data))) 

\## 

\## Deviance Residuals: 

\## Min 1Q Median 3Q Max 

\## -18.2885 -2.1286 0.5841 2.5317 8.0978 

\## 

\## Coefficients: 

\## Estimate Std. Error z value Pr(>|z|) 

\## (Intercept) -6.30098 0.08862 -71.10 <2e-16 \*\*\* 

\## NGN2\_MOI2 2.12408 0.09307 22.82 <2e-16 \*\*\* 

\## NGN2\_MOI5 3.97231 0.08883 44.72 <2e-16 \*\*\* 

\## NGN2\_MOI10 3.11278 0.08992 34.62 <2e-16 \*\*\* 

\## NT3\_ng10 0.31290 0.02084 15.02 <2e-16 \*\*\* 

\## --- 

\## Signif. codes: 0 '\*\*\*' 0.001 '\*\*' 0.01 '\*' 0.05 '.' 0.1 ' ' 1 

\## 

\## (Dispersion parameter for binomial family taken to be 1) 

\## 

\## Null deviance: 10204.34 on 23 degrees of freedom 

\## Residual deviance: 712.97 on 19 degrees of freedom 

\## AIC: 890.95 

\## 

\## Number of Fisher Scoring iterations: 5 

message("Without the outlier:") 

\## Without the outlier: 

summary(modFilt) 

\## 

\## Call: 

\## glm(formula = fractMAP2 \~ NGN2\_MOI + NT3\_ng, family = binomial(link = "logit"), 

\## data = dataFilt, weights = rep(10000, nrow(dataFilt))) 

\## 

\## Deviance Residuals: 

\## Min 1Q Median 3Q Max 

\## -9.1498 -2.2314 -0.1182 2.1493 7.0284 

\## 

\## Coefficients: 

\## Estimate Std. Error z value Pr(>|z|) 

\## (Intercept) -6.32706 0.08866 -71.36 <2e-16 \*\*\* 

\## NGN2\_MOI2 2.33672 0.09316 25.08 <2e-16 \*\*\* 

\## NGN2\_MOI5 3.97305 0.08883 44.73 <2e-16 \*\*\* 

\## NGN2\_MOI10 3.11310 0.08992 34.62 <2e-16 \*\*\* 

\## NT3\_ng10 0.35765 0.02088 17.13 <2e-16 \*\*\* 

\## --- 

\## Signif. codes: 0 '\*\*\*' 0.001 '\*\*' 0.01 '\*' 0.05 '.' 0.1 ' ' 1 

\## 

\## (Dispersion parameter for binomial family taken to be 1) 

\## 

\## Null deviance: 9415.18 on 22 degrees of freedom 

\## Residual deviance: 331.88 on 18 degrees of freedom 

\## AIC: 505.7 

\## 

\## Number of Fisher Scoring iterations: 4 

And let’s now do the same using 5 MOI of NGN2 virus as reference. 

**Enter the model formula in the glm() function call below.** 

levels(dataFilt$NGN2\_MOI) _# the original level order_ 

\## \[1] "0" "2" "5" "10" 

dataFilt$NGN2\_MOI = relevel(dataFilt$NGN2\_MOI, ref=3) 

levels(dataFilt$NGN2\_MOI) _# the new level order_ 

\## \[1] "5" "0" "2" "10" 

modFilt5MOI = glm(fractMAP2\~NGN2\_MOI+NT3\_ng, family=binomial(link="logit"), data=dataFilt, 

weights = rep(10000,nrow(dataFilt))) 

message("With the outlier") 

\## With the outlier 

summary(mod5MOI) 

\## 

\## Call: 

\## glm(formula = fractMAP2 \~ NGN2\_MOI + NT3\_ng, family = binomial(link = "logit"), 

\## data = data, weights = rep(10000, nrow(data))) 

\## 

\## Deviance Residuals: 

\## Min 1Q Median 3Q Max 

\## -18.2885 -2.1286 0.5841 2.5317 8.0978 

\## 

\## Coefficients: 

\## Estimate Std. Error z value Pr(>|z|) 

\## (Intercept) -2.32867 0.01782 -130.66 <2e-16 \*\*\* 

\## NGN2\_MOI0 -3.97231 0.08883 -44.72 <2e-16 \*\*\* 

\## NGN2\_MOI2 -1.84823 0.03365 -54.92 <2e-16 \*\*\* 

\## NGN2\_MOI10 -0.85953 0.02360 -36.42 <2e-16 \*\*\* 

\## NT3\_ng10 0.31290 0.02084 15.02 <2e-16 \*\*\* 

\## --- 

\## Signif. codes: 0 '\*\*\*' 0.001 '\*\*' 0.01 '\*' 0.05 '.' 0.1 ' ' 1 

\## 

\## (Dispersion parameter for binomial family taken to be 1) 

\## 

\## Null deviance: 10204.34 on 23 degrees of freedom 

\## Residual deviance: 712.97 on 19 degrees of freedom 

\## AIC: 890.95 

\## 

\## Number of Fisher Scoring iterations: 5 

message("Without the outlier") 

\## Without the outlier 

summary(modFilt5MOI) 

\## 

\## Call: 

\## glm(formula = fractMAP2 \~ NGN2\_MOI + NT3\_ng, family = binomial(link = "logit"), 

\## data = dataFilt, weights = rep(10000, nrow(dataFilt))) 

\## 

\## Deviance Residuals: 

\## Min 1Q Median 3Q Max 

\## -9.1498 -2.2314 -0.1182 2.1493 7.0284 

\## 

\## Coefficients: 

\## Estimate Std. Error z value Pr(>|z|) 

\## (Intercept) -2.35401 0.01796 -131.06 <2e-16 \*\*\* 

\## NGN2\_MOI0 -3.97305 0.08883 -44.73 <2e-16 \*\*\* 

\## NGN2\_MOI2 -1.63633 0.03389 -48.29 <2e-16 \*\*\* 

\## NGN2\_MOI10 -0.85995 0.02360 -36.43 <2e-16 \*\*\* 

\## NT3\_ng10 0.35765 0.02088 17.13 <2e-16 \*\*\* 

\## --- 

\## Signif. codes: 0 '\*\*\*' 0.001 '\*\*' 0.01 '\*' 0.05 '.' 0.1 ' ' 1 

\## 

\## (Dispersion parameter for binomial family taken to be 1) 

\## 

\## Null deviance: 9415.18 on 22 degrees of freedom 

\## Residual deviance: 331.88 on 18 degrees of freedom 

\## AIC: 505.7 

\## 

\## Number of Fisher Scoring iterations: 4 

_# Restore the original order - note that relevel() won't work as it will put 5 before 2_ 

dataFilt$NGN2\_MOI = factor(dataFilt$NGN2\_MOI, levels = c("0", "2", "5", "10")) 

levels(dataFilt$NGN2\_MOI) _# back to how they were originally_ 

\## \[1] "0" "2" "5" "10" 

**Questions:** 

\- What regression coefficients have changed? 

Answer: All of them, but as expected, the biggest change is for the 2 MOI treatment. 

\- Has the apparent optimal experimental condition changed after removing the outlier (and if so, how)? 

Answer: No, it remains the same**.** 

**8. Final conclusions** 

And we are done. It’s time for your verdict! 

**Question:** Based on the analyses above, what combination of treatment doses do you recommend to use in the differentiation protocol? 

Answer: I recommend using 5 MOI of NGN2 virus and 10 ng/mL NT3 supplement. 

**9. Reporting the results** 

We will now save this notebook in a human-readable format (in our case, as an HTML page). For R notebooks, this process is called “knitting” it, and so we will click the “Knit” button at the top of this window to generate our HTML report. 

**Important notes**: 

The first chunk of code (where we were installing new packages) should be commented out (by prepending each line with ‘#’) for the knitting to work. 

If you skipped the logistic regression analysis parts, you also need to comment out (or delete) code chunks 12-15, 18 and 19. You can locate them quickly in the dropdown menu on the bottom right of this window. 

Also note that Rstudio will likely ask you to download additional packages if it is your first time knitting a notebook into an HTML report. 

\


<!--EndFragment-->

