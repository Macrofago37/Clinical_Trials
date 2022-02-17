---
title: "Re-analysis of Lombard et al. 2016"
author: "Luiz Felipe Martucci"
date: "2/16/2022"
output:  
  html_document:
       keep_md: true
---



Reanálise dos dos dados do paper:Preventing Weight Gain in Women in Rural Communities: A Cluster Randomised Controlled Trial. Lombard et al. (2016) DOI: https://doi.org/10.1371/journal.pmed.1001941

Data available at:https://figshare.com/ndownloader/files/2600235

Packages:


Trazendo a base de dados:


```r
#download.file("https://figshare.com/ndownloader/files/2600235", "Data.dta")
Data <- foreign::read.dta("Data.dta")
```
Objetivo:
Intent to treat: simple, low-intensity, self-management lifestyle intervention (HeLP-her) can prevent weight gain in young to middle-aged women

Primary outcome: weight gain at 1 year.
Baseline and within-group differences over time were assessed using paired Student’s t tests for continuous variables 
The effects of the intervention on study outcomes (between group differences) at 1 yr were analysed using linear regression with the variable of interest at 1 yr as the outcome variable, adjusted for baseline values, and obtained robust standard errors to adjust for the clustering effect of town in the regression models (the Huber/White/sandwich estimate of variance)

In the second step, analyses were performed on data that included values imputed using linear regression, multiple imputation with boot strapping.


### Calculating sample size:

```r
power.t.test(power=.8,
             delta=1,
             sd=3.5,
             type="two.sample")
```

```
## 
##      Two-sample t test power calculation 
## 
##               n = 193.2626
##           delta = 1
##              sd = 3.5
##       sig.level = 0.05
##           power = 0.8
##     alternative = two.sided
## 
## NOTE: n is number in *each* group
```
The sample size estimated on the study was:

```r
power.t.test(n=196,
             delta=1, #1kg de diferença de peso
             sd=3.5, # hypotesized based on annual weight gain in young Australian rural women
             type="two.sample")
```

```
## 
##      Two-sample t test power calculation 
## 
##               n = 196
##           delta = 1
##              sd = 3.5
##       sig.level = 0.05
##           power = 0.8055163
##     alternative = two.sided
## 
## NOTE: n is number in *each* group
```
Power= 0.8055163. Therefore the obtained result was very similar to the one used in the study

### Cluster sample size
However, the study was based on 40 clusters, with variance inflaction factor (VIF) of 1.28 and intracluster correlation(p) of 0.02
As proposed by Ukoumunne et al. (2002, https://www.jstor.org/stable/3650354) the n by clusters can be calculated by:
$$
VIF=1+(n-1)\rho\
$$
Therefore:
$$
n=\frac{(1.28-1)}{0.02}+1
$$
Giving n=15 for each cluster. For 40 clusters, a sample size of 600 is required.

### Summary of the intervention impact:

```r
Data <- Data %>% mutate(weight_var= wgt12- weight_base)

Data %>%
  group_by(group) %>%
  summarise(n=n(),
            Mean=mean(weight_var, na.rm=T),
            SE=(function(...){
  sqrt(var(..., na.rm=T)/sum(!is.na(...)))
})(weight_var)) %>% 
  kable() %>% 
  kable_styling(bootstrap_options = "striped", 
                full_width = T, 
                font_size = 12)
```

<table class="table table-striped" style="font-size: 12px; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> group </th>
   <th style="text-align:right;"> n </th>
   <th style="text-align:right;"> Mean </th>
   <th style="text-align:right;"> SE </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Control </td>
   <td style="text-align:right;"> 291 </td>
   <td style="text-align:right;"> 0.4360515 </td>
   <td style="text-align:right;"> 0.2708768 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Intervention </td>
   <td style="text-align:right;"> 340 </td>
   <td style="text-align:right;"> -0.4818533 </td>
   <td style="text-align:right;"> 0.2607643 </td>
  </tr>
</tbody>
</table>

There was no difference in weight gain inside groups:

```r
(sapply(c("Control", "Intervention"), function(x){
  Data %>% filter(group==x) %>% t.test(.$weight_var, data=.)
}))[3,]#Extract p.value
```

```
## $Control
## [1] 0.1088061
## 
## $Intervention
## [1] 0.06576863
```

```r
# Map Alternative
# map(.x=c("Control", "Intervention"), .f = function(x){
#  Data %>% filter(group==x) %>% t.test(.$weight_var, data=.) %>% .$p.value
# })
```
However, the groups mean are significantly different 

```r
Data %>% t.test(.$weight_var~ .$group, data=.)
```

```
## 
## 	Welch Two Sample t-test
## 
## data:  .$weight_var by .$group
## t = 2.4413, df = 485.96, p-value = 0.01499
## alternative hypothesis: true difference in means between group Control and group Intervention is not equal to 0
## 95 percent confidence interval:
##  0.1791281 1.6566815
## sample estimates:
##      mean in group Control mean in group Intervention 
##                  0.4360515                 -0.4818533
```


### Adjusted analysis


```r
model <- Data %>% 
  lm(weight_var~ group + BMI_base, data=.)

model %>% summary()
```

```
## 
## Call:
## lm(formula = weight_var ~ group + BMI_base, data = .)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -28.236  -1.853   0.161   2.062  14.021 
## 
## Coefficients:
##                   Estimate Std. Error t value Pr(>|t|)   
## (Intercept)        2.25348    0.86050   2.619   0.0091 **
## groupIntervention -0.89027    0.37624  -2.366   0.0184 * 
## BMI_base          -0.06489    0.02915  -2.226   0.0264 * 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 4.157 on 487 degrees of freedom
##   (141 observations deleted due to missingness)
## Multiple R-squared:  0.02197,	Adjusted R-squared:  0.01795 
## F-statistic:  5.47 on 2 and 487 DF,  p-value: 0.004474
```

```r
confint(model, level=.95)
```

```
##                        2.5 %       97.5 %
## (Intercept)        0.5627300  3.944223037
## groupIntervention -1.6295177 -0.151013289
## BMI_base          -0.1221605 -0.007625789
```
Portanto, a análise do artigo é reproduzível, com efeito estimado da intervenção (ajustado pelo IMC inicial) de 0.89



