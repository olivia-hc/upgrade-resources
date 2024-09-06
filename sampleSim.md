[Home](home.md) &nbsp; [Meta-Analysis](meta-analysis.md) &nbsp; [Behavioural](behavioural.md) &nbsp; [EEG](eeg.md)

[Back to Sample Size Calculations](sampleSize.md)  
# Sample Size Calculations via Simulations 

*R Code for Conducting Sample Size Simulations for a Logistic GLMM*

```R
# Package names
packages <- c("ggplot2",  "tidyverse", "lme4", "lmerTest", "simr", "MBESS","future","future.apply","binom", "see","optimx", "afex")

# Install packages not yet installed
installed_packages <- packages %in% rownames(installed.packages())
if (any(installed_packages == FALSE)) {
  install.packages(packages[!installed_packages])
}

# Packages loading
library(ggplot2)
library(lme4)
library(lmerTest)
library(tidyverse)
library(simr)
library(MBESS)
library(future)
library(future.apply)
library(binom)
library(see)
library(afex)

df <- simulatedData
df$PBH <- df$PBH_version
df$PDE <- df$PDE_version
df$agent <- df$agent_version
# make sure everything is factor
df$PBH <- as.factor(df$PBH)
df$PDE <- as.factor(df$PDE)
df$agent <- as.factor(df$agent)
df$Permissibility <- as.factor(df$Permissibility)

#set deviation contrasts for ease of interpretation -.5 vs .5
c<-contr.treatment(2)
my.coding<-matrix(rep(1/2, 2), ncol=1)
my.simple<-c-my.coding
my.simple

#personal force .5 Yes, -.5 No
contrasts(df$PBH)<-my.simple
contrasts(df$PBH)

#intention .5 Yes, -.5 No
contrasts(df$PDE)<-my.simple
contrasts(df$PDE)

#intention .5 Yes, -.5 No
contrasts(df$agent)<-my.simple
contrasts(df$agent)

#prediction DPv2.0 .5 Yes, -.5 No 
contrasts(df$Permissibility)<-my.simple
contrasts(df$Permissibility)


#fit reduced model without by-subject random slope 
fit<- glmer(Permissibility ~ PDE*PBH*agent +  (1+PDE+PBH+agent|sub_id) + (1+PDE+PBH+agent|stimID),  data=df, family = binomial(link = "logit"),control = glmerControl(optimizer = "bobyqa"))

summary(fit)

set_theme(base = theme_classic())
plot_model(fit, terms = "PDE", type = "pred")
plot_model(fit, type = "int")

diff_optims <- allFit(fit, maxfun = 1e5) 
diff_optims_OK <- diff_optims[sapply(diff_optims, is, "glmerMod")] 
lapply(diff_optims_OK, function(x) x@optinfo$conv$lme4$messages)

allFit <- allFit(fit)
allFit <- all_fit(fit)

#increase iterations
control <- glmerControl(optimizer = "bobyqa", optCtrl = list(maxfun = 2e5))

# Fit the model with increased iterations
fitInc2 <- glmer(Permissibility ~ PDE * PBH * agent +  
               (1 + PDE + PBH + agent | sub_id) + 
               (1 + PDE + PBH + agent | dilemmaID),  
             data = df, 
             family = binomial(link = "logit"), 
             control = control)

#test the effect of interest to make sure it's getting the model correctly, this should be same as standard model  
doTest(fitInc2, fixed("PBH2", "z"))

#change effect size to OR = 1.5 (sensitivity to detect a "small" effect)

teff <- "agent2"
veff <- 1.25

fef <- fixef(fit)
fef[teff] <- veff

#grab random effects from variance-covariance matrix
vcv <- VarCorr(fit)
for (l in names(vcv)) {
  attr(vcv[[l]],"stddev") <- NULL
  attr(vcv[[l]],"correlation") <- NULL
}

#dataframe from model
sdata <- cbind(fit@frame[c("sub_id","stimID","PBH","PDE","agent")])

# basic options
nsim <- 500
alpha <- 0.05
nitems <- c(25, 50, 75, 100)

# set seed 
set.seed(123)

#set glmer options
glmerctrlist <- glmerControl(optimizer = "bobyqa")

#create the model structure
tglmer <- makeGlmer(attr(fit@frame,"formula"), 
                    family="binomial", fixef=fef, VarCorr=vcv, data=sdata)

# Create empty dataframe for power sim output
lnitem<-length(nitems)
poweroutput <- data.frame(nitem=nitems, mean=rep(NA_real_, lnitem),lower=rep(NA_real_, lnitem), upper=rep(NA_real_, lnitem), warnings=rep(NA_real_, lnitem), errors=rep(NA_real_, lnitem))

#power curve 25, 50, 75, and 100 participants (original dataset had 45 participants) with 104 scenarios

for (nitem in nitems) {
  
  print(nitem)
  
  tglx <- extend(tglmer,along="sub_id",n=nitem)
  
  plan(multisession)
  
  pstests <- future_replicate(nsim, powerSim(tglx, nsim=1,   
                                             test=fixed("agent2", "z"),
                                             fitOpts=list(control=glmerctrlist),
                                             progress = FALSE),
                              future.globals = c("powerSim","tglx","teff","glmerctrlist"),
                              simplify = FALSE)
  plan(sequential)
  
  pvals <- sapply(pstests,function(x){x$pval})
  print(round(sum(pvals<alpha)/length(pvals),2))
  sucess <- sum(pvals<alpha)
  n <- length(pvals)
  interval <- binom.confint(sucess, n, level=0.95)[c("mean", "lower", "upper")]
  mean <- (round(mean(interval$mean),digits = 2))
  lower <- (round(mean(interval$lower),digits = 2))
  upper <- (round(mean(interval$upper),digits = 2))
  print(sprintf("power is %s, 95%% CIs [%s, %s]", mean, lower, upper))
  warnings <- sapply(pstests,function(x){length(unique(x$warnings$index))})
  errors <- sapply(pstests,function(x){length(unique(x$errors$index))})
  print(sprintf("with %s warnings and %s errors", sum(warnings), sum(errors)))
  output <- data.frame(nitem = nitem, mean = mean, lower=lower, upper = upper, warnings = sum(warnings), errors = sum(errors))
  poweroutput <- poweroutput %>% 
    rows_update(output)
}
 
```
