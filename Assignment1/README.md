# Gender Gap Regression
In this project, it is aimed to evaluate the weekly earnings differences of people working in financial analyst positions in the cps-earnings data set(USA-2014) by creating regressions according to their gender, education level, age and some prejudices via Python.

For this purpose, first of all, the financial analyst position was filtered from the occupancies in the dataset, and then the hourly earnings of these employees were filtered. In the hourly earnings table, a normal distribution was obtained as shown below.

![Screenshot 2023-11-26 at 01 01 12](https://github.com/ab-dum/DataAnalysis2/assets/141356115/2635bc66-12c8-4b8b-97e8-70fd3dea3cb3)

It was observed that the average hourly salary of 253 financial analysts was $35 and the median salary was $32. It was later revealed that 107 of these 253 employees were women. 

![Screenshot 2023-11-26 at 03 26 45](https://github.com/ab-dum/DataAnalysis2/assets/141356115/2cc4aa23-61c2-48d5-a157-cba8dda158b3)

## Task 1 - Show the unconditional gender gap.
### Linear regressions
**Log-level**:
$(\ln{w})^E = \alpha + \beta \times female$

```py
reg1 = smf.ols(formula="lnw~female", data=data).fit()
reg1.summary()
```
![Screenshot 2023-11-26 at 03 42 14](https://github.com/ab-dum/DataAnalysis2/assets/141356115/b59b9d81-d465-4163-b6a3-d59c8777c8ee)

In order to find the unconditional gender gap, the log(wage)-level(female) linear regression method was used and it was determined that female financial analysts earned 12.8% less than male financial analysts.


What can female university students expect about the gender gap in pay if they choose to become financial analyst? (future)

-> We should do statistical inference

```python
reg2 = smf.ols(formula="lnw~female", data=data).fit(cov_type="HC1") 
#heteroskedastic-consistent standard errors
reg2.summary()
```
![Screenshot 2023-11-26 at 03 48 42](https://github.com/ab-dum/DataAnalysis2/assets/141356115/b684ae73-2005-470d-a403-ef0a194eba76)

- Just standard error is changed very slightly(from 0.0119 to 0.0116). Confidence interval goes from -0.356 to 0.099 which is quite wide. It can be understood that we can be 95% confident that average difference between log hourly wage earnings of female financial analysts compared to males is between -0.356 and 0.099 and this interval includes 0 that means males and females have same difference so we can not say that our coefficient (-0.1285) is statistically significant. Instead of looking at the confidence interval, we can look at the statistic of p value (z = -1.106) and its bigger than 0.05. So we can not reject the null hypothesis ($\beta=0\$)

-> in 2014 in the USA we can be 95\% confident that the average difference between hourly earnings of female financial analysts versus male was -35\% to 1\%

-> the CI *includes* zero -> we cannot rule out with 95\% confidence that their average earnings (female and male) are the same 

-> $|t|=1.1<1.96$ cannot reject $H_0$

-> it can be seen also by p-value$> 0.05$

-> the coefficient cannot be considered statistically significant at 5\% (significant at 10\%)

## Task 2 - Show how the gender gap varies with the level of education
### Gender differences in earnings – log earnings, gender and education (Bachelor, Master, PhD)

# Dummies
```python
data["ed_BA"] = (data["grade92"] == 43).astype(int)
data["ed_MA"] = (data["grade92"] == 44).astype(int)
data["ed_Phd"] = (data["grade92"] == 46).astype(int)
```

We should include $k-1$ dummies 

-> the category not represented is the **reference category**

-> coefficients on each $k-1$ dummy show average differences in $y$ compared to the reference category

```python
reg3 = smf.ols(formula="lnw~female", data=data).fit(cov_type="HC1") # unconditional
reg4 = smf.ols(formula="lnw~female + ed_MA + ed_BA", data=data).fit(cov_type="HC1") # reference is Phd
reg5 = smf.ols(formula="lnw~female + ed_MA + ed_Phd", data=data).fit(cov_type="HC1") # reference is BA
```
```python
stargazer = Stargazer([reg3, reg4, reg5])
stargazer.covariate_order(["female", "ed_MA", "ed_BA", "ed_Phd", "Intercept"])
stargazer.rename_covariates({"Intercept": "Constant"})
stargazer
```
![Screenshot 2023-11-26 at 04 00 16](https://github.com/ab-dum/DataAnalysis2/assets/141356115/3940118d-6b2a-4507-a9cb-8b79a37b9a6e)

- As you can see, the coeffient of estimate on female(lnw~female) doesnt change too much (from -0.128 to -0.134) and also R square doesn't change too much between model 2 and 3 (from 0.013 to 0.020). The reason of this result is that we are just switching to a different reference group but basically we are doing same process. -0.125 coefficient of ed_BA tells us that those with PhD degree are expected to earn on average 12.5% more than employees with an BA degree; those with MA degree are expected to earn on average 6.6% more than employees with PhD degree

- (1) $\ln(w)=\alpha + \beta female + \varepsilon$
- (2) $\ln(w)=\beta_0 + \beta_1 female + \beta_2 ed\_MA + \beta_3 ed\_BA + e$:

> [Phd degree is the comparison group] comparing female financial analysts, those with MA degree are expected to earn on average 6.6% more than employees with an Phd degree; those with BA degree 12.5% less than employees with Phd degree
- (3) $\ln(w)=\gamma_0 + \gamma_1 female + \gamma_2 ed\_MA + \gamma_3 ed\_Phd + u$: 
> [BA degree is the comparison group] comparing female financial analysts, those with BA are expected to earn on average 12.5% less than employees with an PhD degree

> estimated coefficient on *female* is smaller (-0.134) when education is included
-> in the data, women appear to be more likely to be in lower-earner BA than in higher-earner MA or PhD - but only small part









Since the size of data set is too big, it is not possible to add it here. However, clean data set (morg-2014-emp.csv) can be downladed from [here](https://osf.io/g8p9j/).
