---
title: Create R models (SQL and R deep dive)| Microsoft Docs
ms.prod: sql
ms.technology: machine-learning

ms.date: 04/15/2018  
ms.topic: tutorial
author: HeidiSteen
ms.author: heidist
manager: cgronlun
---
# Create R models (SQL and R deep dive)
[!INCLUDE[appliesto-ss-xxxx-xxxx-xxx-md-winonly](../../includes/appliesto-ss-xxxx-xxxx-xxx-md-winonly.md)]

This article is part of the Data Science Deep Dive tutorial, on how to use [RevoScaleR](https://docs.microsoft.com/machine-learning-server/r-reference/revoscaler/revoscaler) with SQL Server.

Now that you have enriched the training data, it's time to analyze the data using linear regression. Linear models are an important tool in the world of predictive analytics, and the **RevoScaleR** package in [!INCLUDE[rsql_productname](../../includes/rsql-productname-md.md)] includes a high-performance, scalable algorithm.

## Create a linear regression model

In this step, you create a simple linear model that estimates the credit card balance for the customers, using as independent variables the values in the *gender* and *creditLine* columns.
  
To do this, use the [rxLinMod](https://docs.microsoft.com/machine-learning-server/r-reference/revoscaler/rxlinmod) function, which supports remote compute contexts.
  
1. Create an R variable to store the completed model, and call **rxLinMod**, passing an appropriate formula.
  
    ```R
    linModObj <- rxLinMod(balance ~ gender + creditLine,  data = sqlFraudDS)
    ```
  
2. To view a summary of the results, call the standard R `summary` function on the model object.
  
     ```R
     summary(linModObj)
     ```

You might think it peculiar that a plain R function like `summary` would work here, since in the previous step, you set the compute context to the server. However, even when the **rxLinMod** function uses the remote compute context to create the model, it also returns an object that contains the model to your local workstation, and stores it in the shared directory.

Therefore, you can run standard R commands against the model just as if it had been created using the "local" context.

**Results**

```
Linear Regression Results for: balance ~ gender + creditLineData: sqlFraudDS (RxSqlServerData Data Source)
Dependent variable(s): balance
Total independent variables: 4 (Including number dropped: 1)
Number of valid observations: 10000
Number of missing observations: 0
Coefficients: (1 not defined because of singularities)

Estimate Std. Error t value Pr(>|t|) (Intercept)
3253.575 71.194 45.700 2.22e-16
gender=Male -88.813 78.360 -1.133 0.257
gender=Female Dropped Dropped Dropped Dropped
creditLine 95.379 3.862 24.694 2.22e-16
Signif. codes: 0  0.001  0.01 ???*??? 0.05 ???.??? 0.1 ??? ??? 1

Residual standard error: 3812 on 9997 degrees of freedom
Multiple R-squared: 0.05765
Adjusted R-squared: 0.05746
F-statistic: 305.8 on 2 and 9997 DF, p-value: < 2.2e-16
Condition number: 1.0184
```

## Create a logistic regression model

Next, you create a logistic regression model that indicates whether a particular customer is a fraud risk. You'll use the **RevoScaleR** [rxLogit](https://docs.microsoft.com/machine-learning-server/r-reference/revoscaler/rxlogit) function, which supports fitting of logistic regression models in remote compute contexts.

1.  Keep the compute context as is. You???ll also continue to use the same data source as well.

2.  Call the **rxLogit** function and pass the formula needed to define the model.

    ```R
    logitObj <- rxLogit(fraudRisk ~ state + gender + cardholder + balance + numTrans + numIntlTrans + creditLine, data = sqlFraudDS, dropFirst = TRUE)
    ```
  
    Because it is a large model, containing 60 independent variables, including three dummy variables that are dropped, you might have to wait some time for the compute context to return the object.
    
    The reason the model is so large is that, in R and in the **RevoScaleR** package, every level of a categorical factor variable is automatically treated as a separate dummy variable.
  
3.  To view a summary of the returned model, call the R `summary` function.
  
    ```R
    summary(logitObj)
    ```
  
**Partial results**

```
Logistic Regression Results for: fraudRisk ~ state + gender + cardholder + balance + numTrans + numIntlTrans + creditLine
Data: sqlFraudDS (RxSqlServerData Data Source)
Dependent variable(s): fraudRisk
Total independent variables: 60 (Including number dropped: 3)
Number of valid observations: 10000 -2

LogLikelihood: 2032.8699 (Residual deviance on 9943 degrees of freedom)

Coefficients:
Estimate Std. Error z value Pr(>|z|)     (Intercept)
-8.627e+00  1.319e+00  -6.538 6.22e-11
state=AK                Dropped    Dropped Dropped  Dropped
state=AL             -1.043e+00  1.383e+00  -0.754   0.4511

(other states omitted)

gender=Male             Dropped    Dropped Dropped  Dropped
gender=Female         7.226e-01  1.217e-01   5.936 2.92e-09
cardholder=Principal    Dropped    Dropped Dropped  Dropped
cardholder=Secondary  5.635e-01  3.403e-01   1.656   0.0977
balance               3.962e-04  1.564e-05  25.335 2.22e-16
numTrans              4.950e-02  2.202e-03  22.477 2.22e-16
numIntlTrans          3.414e-02  5.318e-03   6.420 1.36e-10
creditLine            1.042e-01  4.705e-03  22.153 2.22e-16

Signif. codes:  0 ???\*\*\*??? 0.001 ???\*\*??? 0.01 ???\*??? 0.05 ???.??? 0.1 ??? ??? 1
Condition number of final variance-covariance matrix: 3997.308
Number of iterations: 15
```

## Next step

[Score new data](../../advanced-analytics/tutorials/deepdive-score-new-data.md)

## Previous step

[Visualize SQL Server data using R](../../advanced-analytics/tutorials/deepdive-visualize-sql-server-data-using-r.md)
