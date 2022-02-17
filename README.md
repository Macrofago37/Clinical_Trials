# Clinical_Trials
Analysis of Clinical Trials Data


# "Re-analysis of Lombard et al. 2016"
## Luiz Felipe Martucci

Reanálise dos dos dados do paper:Preventing Weight Gain in Women in Rural Communities: A Cluster Randomised Controlled Trial. Lombard et al. (2016) DOI: https://doi.org/10.1371/journal.pmed.1001941

Data available at:https://figshare.com/ndownloader/files/2600235

Packages:
```{r libraries, include=FALSE}
x <- c("tidyverse", "foreign", "kableExtra")
(function(x){
  sapply(x, function(x) if(!x %in% installed.packages()){
    install.packages(x, dependencies = T)
  })
  sapply(x, library, character.only=T)
})(x)

```

Trazendo a base de dados:

```{r Uploading file}
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
```{r}
power.t.test(power=.8,
             delta=1,
             sd=3.5,
             type="two.sample")
```
The sample size estimated on the study was:
```{r}
power.t.test(n=196,
             delta=1, #1kg de diferença de peso
             sd=3.5, # hypotesized based on annual weight gain in young Australian rural women
             type="two.sample")
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
```{r}
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

There was no difference in weight gain inside groups:
```{r}
(sapply(c("Control", "Intervention"), function(x){
  Data %>% filter(group==x) %>% t.test(.$weight_var, data=.)
}))[3,]#Extract p.value

# Map Alternative
# map(.x=c("Control", "Intervention"), .f = function(x){
#  Data %>% filter(group==x) %>% t.test(.$weight_var, data=.) %>% .$p.value
# })

```
However, the groups mean are significantly different 
```{r}
Data %>% t.test(.$weight_var~ .$group, data=.)
```


### Adjusted analysis

```{r}
model <- Data %>% 
  lm(weight_var~ group + BMI_base, data=.)

model %>% summary()

confint(model, level=.95)
```
Portanto, a análise do artigo é reproduzível, com efeito estimado da intervenção (ajustado pelo IMC inicial) de 0.89



