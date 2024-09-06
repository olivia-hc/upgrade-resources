[Home](home.md) &nbsp; [Meta-Analysis](meta-analysis.md) &nbsp; [Behavioural](behavioural.md) &nbsp; [EEG](eeg.md) 

[Back to Sample Size Calculations](sampleSize.md) 
# Sample Data Code
*R Code for Creating Simulated Sample Data With Binary Independent and Dependent Variables*

```R
# Package names
packages <- c("ggplot2",  "tidyverse", "lme4", "lmerTest", "sjPlot","GGally","faux","afex","magrittr", "dplyr", "psych","tidyr","pROC")

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
library(sjPlot)
library(GGally)    
library(faux)
library(afex)
library(magrittr)
library(dplyr)
library(psych)
library(tidyr)

#set seed
set.seed(250923)

alldata <- read.csv("C:/Users/oh298/OneDrive - University of Exeter/PhD/Behavioural Study/sampleSize_simulations/alldata.csv")

# Calculate mean of Permissible for each combination of PBH and PDE
alldata %>%
  +     group_by(Prohibited.act, PDE) %>%
  +     summarise(mean_Permissible = mean(Permissible, na.rm = TRUE))

MG<- glmer(Permissible ~ PDE*Prohibited.act +  (1+PDE*Prohibited.act|ID) + (1+PDE*Prohibited.act|Dilemma),  data=alldata, family = binomial(link = "logit"),control = glmerControl(optimizer = "bobyqa"))
summary(MG)

stim_n = 272
sub_n = 200
##GENERATE RANDOM EFFECTS##
#SUBJECT RANDOM EFFECTS#
#### extract parameter estimates from pilot data ####
subj_ranefsd <- round(attr(VarCorr(MG)$ID, "stddev"),4)
subj_ranefcorrPilot_w.Int <- round(attr(VarCorr(MG)$ID, "corr"), 1)

### correlations
## add in new variable & remove interaction for simplicity 
#remove interaction 
subj_ranefcorrPilot <- subj_ranefcorrPilot_w.Int[-4,-4]

#add column for agent 
agentCol <- c(0.4,0.1,0.1)
subj_ranefcorr <- cbind(subj_ranefcorrPilot, agent = agentCol)

#add row for agent 
agentRow <- c(0.4,0.1,0.1,1)
subj_ranefcorr <- rbind(subj_ranefcorr, agent = agentRow)

### sd  
sub_sd <- subj_ranefsd["(Intercept)"]
sub_PDE_sd <- subj_ranefsd["PDE2"]
sub_PBH_sd <- subj_ranefsd["Prohibited.act2"]
sub_agent_sd <- mean(sub_PDE_sd, sub_PBH_sd)


#ITEM RANDOM EFFECTS#
#### extract parameter estimates from pilot data ####
item_ranefsd <- round(attr(VarCorr(MG)$Dilemma, "stddev"),4)
item_ranefcorrPilot_w.Int <- round(attr(VarCorr(MG)$Dilemma, "corr"), 1)

### correlations
## add in new variable & remove interaction for simplicity 
#remove interaction 
item_ranefcorrPilot <- item_ranefcorrPilot_w.Int[-4,-4]

#add column for agent 
agentColi <- c(0.6,-0.6,0.3)
item_ranefcorr <- cbind(item_ranefcorrPilot, agent = agentColi)

#add row for agent 
agentRowi <- c(0.6,-0.6,0.3,1)
item_ranefcorr <- rbind(item_ranefcorr, agent = agentRowi)

### sd  
stim_sd <- item_ranefsd["(Intercept)"]
stim_PDE_sd <- item_ranefsd["PDE2"]
stim_PBH_sd <- item_ranefsd["Prohibited.act2"]
stim_agent_sd <- mean(item_PDE_sd, item_PBH_sd)

#stim N based on parameters (PDE,PBH) for ONE agent
stimN_noNo = 30
stimN_noYes = 62
stimN_yesNo = 20
stimN_yesYes = 24

#HUMAN#
#PDE: NO, PBH: NO
stim_noNoHuman <- tibble(
  stim_id = 1:stimN_noNo,
  PDE_version = "No",
  PBH_version = "No",
  agent_version = "Human",
)
print(stim_noNoHuman)

#PDE: NO, PBH: YES
stim_noYesHuman <- tibble(
  stim_id = 1:stimN_noYes,
  PDE_version = "No",
  PBH_version = "Yes",
  agent_version = "Human",
)
print(stim_noYesHuman)
#PDE: YES, PBH: NO
stim_yesNoHuman <- tibble(
  stim_id = 1:stimN_yesNo,
  PDE_version = "Yes",
  PBH_version = "No",
  agent_version = "Human",
)
print(stim_yesNoHuman)
#PDE: YES, PBH: YES
stim_yesYesHuman <- tibble(
  stim_id = 1:stimN_yesYes,
  PDE_version = "Yes",
  PBH_version = "Yes",
  agent_version = "Human",
)
print(stim_yesYesHuman)
#AI#
#PDE: NO, PBH: NO
stim_noNoAI <- tibble(
  stim_id = 1:stimN_noNo,
  PDE_version = "No",
  PBH_version = "No",
  agent_version = "AI",
)
print(stim_noNoAI)
#PDE: NO, PBH: YES
stim_noYesAI <- tibble(
  stim_id = 1:stimN_noYes,
  PDE_version = "No",
  PBH_version = "Yes",
  agent_version = "AI",
)
print(stim_noYesAI)
#PDE: YES, PBH: NO
stim_yesNoAI <- tibble(
  stim_id = 1:stimN_yesNo,
  PDE_version = "Yes",
  PBH_version = "No",
  agent_version = "AI",
)
print(stim_yesNoAI)
#PDE: YES, PBH: YES
stim_yesYesAI <- tibble(
  stim_id = 1:stimN_yesYes,
  PDE_version = "Yes",
  PBH_version = "Yes",
  agent_version = "AI",
)
print(stim_yesYesAI)


##GENERATE FIXED EFFECTS##
#### extract parameter estimates from pilot data ####
betaPilot_w.Int <- round(summary(MG)$coefficients[,1],4)
sigma_e <- round(attr(VarCorr(MG), "sc"), 2)

### add in new variable & remove interaction for simplicity 
#remove interaction 
betaPilot <- betaPilot_w.Int[names(betaPilot_w.Int) != "PDE2:Prohibited.act2"]

#add agent with a small effect size
beta <- c(betaPilot, "agent" = 1)

## assemble variance-covariance matrix for subjects:
sub <- faux::rnorm_multi(
  n = sub_n, 
  vars = 4, 
  r = subj_ranefcorr,
  mu = 0, # means of random intercepts and slopes are always 0
  sd = c(sub_sd, sub_PDE_sd, sub_PBH_sd, sub_agent_sd),
  varnames = c("sub_i", "sub_PDE_slope", "sub_PBH_slope", "sub_agent_slope")
) %>%
  mutate(
    sub_id = 1:sub_n
  )

stim <- bind_rows(stim_noNoHuman,stim_noYesHuman,stim_yesNoHuman,stim_yesYesHuman,stim_noNoAI,stim_noYesAI,stim_yesNoAI,stim_yesYesAI
) %>% 
  mutate(faux::rnorm_multi(
    n = stim_n, 
    vars = 4, 
    r = item_ranefcorr, 
    mu = 0, # means of random intercepts and slopes are always 0
    sd = c(stim_sd, stim_PDE_sd, stim_PBH_sd, stim_agent_sd),
    varnames = c("stim_i", "stim_PDE_slope", "stim_PBH_slope", "stim_agent_slope")
  )
  )
## add numeric contrast codes 
stim <- stim %>%
  mutate(
    PDE_version_numeric = ifelse(PDE_version == "Yes", 1, 0),
    PBH_version_numeric = ifelse(PBH_version == "Yes", 1, 0),
    agent_version_numeric = ifelse(agent_version == "Human", 1, 0),
    stimID = 1:nrow
  )

stim <- bind_rows(stim_noNoHuman,stim_noYesHuman,stim_yesNoHuman,stim_yesYesHuman,stim_noNoAI,stim_noYesAI,stim_yesNoAI,stim_yesYesAI
) %>% 
  mutate(faux::rnorm_multi(
    n = stim_n, 
    vars = 4, 
    r = item_ranefcorr, 
    mu = 0, # means of random intercepts and slopes are always 0
    sd = c(stim_sd, stim_PDE_sd, stim_PBH_sd, stim_agent_sd),
    varnames = c("stim_i", "stim_PDE_slope", "stim_PBH_slope", "stim_agent_slope")
  )
  )
## add numeric contrast codes 
stim <- stim %>%
  mutate(
    PDE2 = ifelse(PDE_version == "Yes", 1, 0),
    PBH2 = ifelse(PBH_version == "Yes", 1, 0),
    agentAI = ifelse(agent_version == "AI", 1, 0),
    stimID = 1:nrow(stim)
  )

##merge stimulus and subject random effects

subWstim <- data.frame()

for (row in 1:nrow(sub)) {

        # Create a copy of `stim` and add identifiers for row and column
        stim_copy <- stim %>%
          mutate(
            sub_id = row,   # Adding row index
          )
        
        # Append the current `stim_copy` data frame to `subWstim`
        subWstim <- rbind(subWstim, stim_copy)
}

subXstim <- left_join(subWstim,sub, by = "sub_id")

subXstim$PDE_version = recode(subXstim$PDE_version, "No" = 0, "Yes" = 1)
subXstim$PBH_version = recode(subXstim$PBH_version, "No" = 0, "Yes" = 1)
subXstim$agent_version = recode(subXstim$agent_version, "Human" = 0, "AI" = 1)
  
## calculate the linear predictor (logit)
simulatedData <- subXstim %>%
  mutate(
    logit = beta[['PDE2']] *PDE_version +
      beta[['Prohibited.act2']] *PBH_version + 
      beta[['agent']] *agent_version + 
      sub_i + 
      sub_PDE_slope *PDE_version +
      sub_PBH_slope *PBH_version +
      sub_agent_slope *agent_version +
      stim_i + 
      stim_PDE_slope *PDE_version +
      stim_PBH_slope *PBH_version +
      stim_agent_slope *agent_version
  )

simulatedData$prob <- plogis(simulatedData$logit)
simulatedData$Permissibility <- rbinom(nrow(simulatedData), size=1, prob= simulatedData$prob)
view(simulatedData)

#####OPEN SAMPLE SIZE CALCULATION CODE#####
```
