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

### Gender differences in earnings – log earnings, gender, age, and their interaction

Are patterns with age similar or different for men vs women?
- Regress separately for men and women
- Include interaction $age*gender$

$(\ln{w})^E = \beta_0 + \beta_1\times age +\beta_2\times female + \beta_3\times age \times female$

```python
reg6 = smf.ols(formula="lnw~age", data=data.query("female==1")).fit(cov_type="HC1")
reg7 = smf.ols(formula="lnw~age", data=data.query("female==0")).fit(cov_type="HC1")
reg8 = smf.ols(formula="lnw~female+age+age*female", data=data).fit(cov_type="HC1")
```
```python
stargazer = Stargazer([reg6, reg7, reg8])
stargazer.covariate_order(["female", "age", "age:female", "Intercept"])
stargazer.rename_covariates({"Intercept": "Constant", "age:female": "female x age"})
stargazer.custom_columns(["Women", "Men", "All"], [1, 1, 1])
stargazer
```
![Screenshot 2023-11-28 at 23 14 24](https://github.com/ab-dum/DataAnalysis2/assets/141356115/f6c0f76b-d3b7-4558-ba7b-4a04d747d100)

- (1): women who are one year older are expected to earn 0.5% less, on average
- (2): men who are one year older are expected to earn 2.5% more, on average
- (3): slope of log earnings - age pattern is 0.030 less positive for women, on average  -> can do **inference** about gender differences but we can not say much about the population.

> women at age 30 -> predicted log wage $= 3.504 - 30\times 0.005 \sim 3.354$

> men at age 30 -> predicted log wage $= 2.511 + 30\times 0.025 \sim 3.261$

> difference $0.093 \sim 0.993 + 30\times(-0.003)$

### Earning differences by gender as function of age
(A): $(\ln{w})^E = \beta_0 + \beta_1\times age +\beta_2\times female + \beta_3\times age \times female$

```python
# Regression for males
data_m = data.query("female==0")

pred = reg8.predict(data_m)

pred = reg8.get_prediction(data_m).summary_frame()[["mean", "mean_se"]]
pred.columns = ["fit", "fit_se"]

data_m = data_m.reset_index(drop=True).join(pred)

data_m["CIup"] = data_m["fit"] + 0.3 * data_m["fit_se"]
data_m["CIlo"] = data_m["fit"] - 0.3 * data_m["fit_se"]
```
```python
# Regression for females
data_f = data.query("female==1")

pred = reg8.predict(data_f)

pred = reg8.get_prediction(data_f).summary_frame()[["mean", "mean_se"]]
pred.columns = ["fit", "fit_se"]

data_f = data_f.reset_index(drop=True).join(pred)

data_f["CIup"] = data_f["fit"] + 0.3 * data_f["fit_se"]
data_f["CIlo"] = data_f["fit"] - 0.3 * data_f["fit_se"]
```
```python
(
    ggplot(data_m, aes(x="age", y="fit"))
    + geom_line(colour=color[0])
    + geom_line(data_m, aes(x="age", y="CIup"), colour=color[0], linetype="dashed")
    + geom_line(data_m, aes(x="age", y="CIlo"), colour=color[0], linetype="dashed")
    + geom_line(data_f, aes(x="age", y="fit"), colour=color[1])
    + geom_line(data_f, aes(x="age", y="CIup"), colour=color[1], linetype="dashed")
    + geom_line(data_f, aes(x="age", y="CIlo"), colour=color[1], linetype="dashed")
    + labs(x="Age (years)", y="ln(earnings per hour, US dollars)")
    + scale_x_continuous(expand=(0.01, 0.01), limits=(24, 65), breaks=seq(25, 65, by=5))
    + scale_y_continuous(
        expand=(0.01, 0.01), limits=(2.8, 3.8), breaks=seq(2.8, 3.8, by=0.1)
    )
    + theme_bw()
)
```
![Screenshot 2023-11-28 at 23 21 30](https://github.com/ab-dum/DataAnalysis2/assets/141356115/5f65c71b-9564-48c3-ae94-c4ecc8af9b68)

Including the interaction $female\times age$ enables us to see that the earnings difference appears to be higher for older age

-> growing gap 

Without interaction -> two parallel lines would be at constant distance

### Gender differences in earnings – log earnings, gender, 4th-order polynomial of age, and their interaction

```python
data["agesq"] = np.power(data["age"], 2) # squared
data["agecu"] = np.power(data["age"], 3) # cubic
data["agequ"] = np.power(data["age"], 4) # power 4

reg9 = smf.ols(formula="lnw~age+agesq+agecu+agequ", data=data.query("female==1")).fit(
    cov_type="HC1"
) # for women
reg10 = smf.ols(formula="lnw~age+agesq+agecu+agequ", data=data.query("female==0")).fit(
    cov_type="HC1"
) # for men
reg11 = smf.ols(
    formula="lnw ~ age + agesq + agecu + agequ + female + female*age + female*agesq + female*agecu + female*agequ",
    data=data,
).fit(cov_type="HC1")

Stargazer([reg9, reg10, reg11])
```
![Screenshot 2023-11-28 at 23 24 31](https://github.com/ab-dum/DataAnalysis2/assets/141356115/6b9eda40-ed3b-42af-a630-fe503178f72f)

$(ln{w})^E = \beta_0 + \beta_1 age + \beta_2age^2+\beta_3age^3+\beta_4age^4+\beta_5female + \beta_6female\times age + \beta_7female\times age^2 + \beta_8female\times age^3 + \beta_9 female\times age^4$

```python
# PREDICTION AND GRAPH POLYNOMIAL
# male
data_m = data.query("female==0")

pred = reg11.predict(data_m)

pred = reg11.get_prediction(data_m).summary_frame()[["mean", "mean_se"]]
pred.columns = ["fit", "fit_se"]

data_m = data_m.reset_index(drop=True).join(pred)

data_m["CIup"] = data_m["fit"] + 0.2 * data_m["fit_se"]
data_m["CIlo"] = data_m["fit"] - 0.2 * data_m["fit_se"]

# female
data_f = data.query("female==1")

pred = reg11.predict(data_f)

pred = reg11.get_prediction(data_f).summary_frame()[["mean", "mean_se"]]
pred.columns = ["fit", "fit_se"]

data_f = data_f.reset_index(drop=True).join(pred)

data_f["CIup"] = data_f["fit"] + 0.2 * data_f["fit_se"]
data_f["CIlo"] = data_f["fit"] - 0.2 * data_f["fit_se"]
```
```python
(
    ggplot(data_m, aes(x="age", y="fit"))
    + geom_line(colour=color[0])
    + geom_line(data_m, aes(x="age", y="CIup"), colour=color[0], linetype="dashed")
    + geom_line(data_m, aes(x="age", y="CIlo"), colour=color[0], linetype="dashed")
    + geom_line(data_f, aes(x="age", y="fit"), colour=color[1])
    + geom_line(data_f, aes(x="age", y="CIup"), colour=color[1], linetype="dashed")
    + geom_line(data_f, aes(x="age", y="CIlo"), colour=color[1], linetype="dashed")
    + labs(x="Age (years)", y="ln(earnings per hour, US dollars)")
    + scale_x_continuous(expand=(0.01, 0.01), limits=(24, 65), breaks=seq(25, 65, by=5))
    + scale_y_continuous(
        expand=(0.01, 0.01), limits=(2.8, 3.8), breaks=seq(2.8, 3.8, by=0.1)
    )
    + theme_bw()
)
```
![Screenshot 2023-11-28 at 23 26 20](https://github.com/ab-dum/DataAnalysis2/assets/141356115/144b183f-3d4b-40da-910d-5868370a20f6)

- At younger ages (between 25 and age 35) 2 confidence intervals overlap it comes out maybe there is not much difference in t. At age 45 the difference is nearly 30%. Its at the top %80 by age 55 and the difference decrease from age 55 to 63.

## Causal analysis
IS IT DISCRIMINATION?

In the data, relatively large average gender difference in earnings between ages 40 and 55

-> analyze it to see if members of a group (women, minorities) earn systematically less per hour than the others (men, majority), all other things being equal -> towards causality (but impossible to control for everything

```python
# FILTER DATA ACCORDING THE AGE-SELECTION of the sample we need
data = data.query("age>=40 & age<=55")
```
```python
data["white"] = (data["race"] == 1).astype(int)
data["afram"] = (data["race"] == 2).astype(int)
data["asian"] = (data["race"] == 4).astype(int)
data["hisp"] = (data["ethnic"].notna()).astype(int)
data["othernonw"] = (
    (data["white"] == 0) & (data["afram"] == 0) & (data["asian"] == 0) & (data["hisp"] == 0)
).astype(int) #other nonwhite
data["nonUSborn"] = (
    (data["prcitshp"] == "Foreign Born, US Cit By Naturalization")
    | (data["prcitshp"] == "Foreign Born, Not a US Citizen")
).astype(int)
```
```python
# Potentially endogeneous demographics
data["married"] = ((data["marital"] == 1) | (data["marital"] == 2)).astype(int)
data["divorced"] = ((data["marital"] == 3) & (data["marital"] == 5)).astype(int)
data["wirowed"] = (data["marital"] == 4).astype(int)
data["nevermar"] = (data["marital"] == 7).astype(int)

data["child0"] = (data["chldpres"] == 0).astype(int)
data["child1"] = (data["chldpres"] == 1).astype(int)
data["child2"] = (data["chldpres"] == 2).astype(int)
data["child3"] = (data["chldpres"] == 3).astype(int)
data["child4pl"] = (data["chldpres"] >= 4).astype(int)

# Work-related variables
data["fedgov"] = (data["class"] == "Government - Federal").astype(int)
data["stagov"] = (data["class"] == "Government - State").astype(int)
data["locgov"] = (data["class"] == "Government - Local").astype(int)
data["nonprof"] = (data["class"] == "Private, Nonprofit").astype(int)
data["ind2dig"] = ((pd.Categorical(data["ind02"]).codes + 1) / 100).astype(int)
data["occ2dig"] = (data["occ2012"] / 100).astype(int)
data["union"] = ((data["unionmme"] == "Yes") | (data["unioncov"] == "Yes")).astype(int)
```
```python
data["uhourssq"] = np.power(data["uhours"], 2)
data["uhourscu"] = np.power(data["uhours"], 3)
data["uhoursqu"] = np.power(data["uhours"], 4)
```

### Gender differences in earnings – regression with many covariates on a narrower sample
```python
# Extended regressions
reg12 = smf.ols(formula="lnw ~ female", data=data).fit(cov_type="HC1") # unconditional gender gap
reg13 = smf.ols(formula="lnw ~ female + age + ed_MA + ed_BA", data=data).fit(
    cov_type="HC1" # age, gender and education
)
reg14 = smf.ols(
    formula="lnw ~ female + age + afram + hisp + asian + othernonw + nonUSborn + ed_MA + ed_BA + married + divorced+ wirowed + child1 + child2 + child3 +child4pl + C(stfips) + uhours + fedgov + stagov + locgov + nonprof + union + C(ind2dig) + C(occ2dig)",
    data=data,
).fit(cov_type="HC1")
reg15 = smf.ols(
    formula="lnw ~ female + age + afram + hisp + asian + othernonw + nonUSborn + ed_MA + ed_BA + married + divorced+ wirowed + child1 + child2 + child3 +child4pl + C(stfips) + uhours + fedgov + stagov + locgov + nonprof + union + C(ind2dig) + C(occ2dig) + agesq + agecu + agequ + uhoursqu + uhourscu + uhourssq",
    data=data,
).fit(cov_type="HC1")
```
```python
stargazer = Stargazer([reg12, reg13, reg14, reg15])
stargazer.covariate_order(["female"])
stargazer.add_line("Age and education", ["", "Yes", "Yes", "Yes"])
stargazer.add_line("Family circumstances", ["", "", "Yes", "Yes"])
stargazer.add_line("Demographic background", ["", "", "Yes", "Yes"])
stargazer.add_line("Job characteristics", ["", "", "Yes", "Yes"])
stargazer.add_line("Age in polynomial", ["", "", "", "Yes"])
stargazer.add_line("Hours in polynomial", ["", "", "", "Yes"])
stargazer
```
![Screenshot 2023-11-28 at 23 29 21](https://github.com/ab-dum/DataAnalysis2/assets/141356115/b84c2d68-2f98-4767-bbf9-c6b18f3b4f69)

- Unconditional coefficient increases because we took some biases in and R square increases with the increasing regression numbers
- (1): women are expected to earn, on average, 52.8% less than men (in the data)
- (2): conditioning on age and on education, the difference is 53.7% -> differences in age and education do not contribute that much to gender difference in earnings -> check CIs and see if they overlap
- (3): including all other covariates, the estimated coefficient is 20% -> comparing people with same personal and family characteristics and job features, women are expected to earn 20% less than men
- (4): include potential nonlinearities in age and hours -> similar estimate 17%

  **We can conclude that**
- the gender gap is quite small below age 30, while largest between ages 40 and 55 
- whether due to discrimination or other reasons, gender differences tend to be smaller for younger employees 
