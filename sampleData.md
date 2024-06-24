[Back to Sample Size Calculations](sampleSize.md) &nbsp; [Home](home.md) &nbsp; [Meta-Analysis](meta-analysis.md) &nbsp; [Behavioural](behavioural.md) &nbsp; [EEG](eeg.md) 
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
library(pROC)

options(digits = 10)

#install pilot data (for purpose of reproducibility: swap this with directory of pilot data on OSF)
df <- read.csv("~/Library/Documents/example folder/pilot_data.csv")

#run GLMM on pilot data:
MG<- glmer(Permissible ~ PDE*Prohibited.act +  (1+PDE*Prohibited.act|ID) + (1+PDE*Prohibited.act|Dilemma),  data=df, family = binomial(link = "logit"),control = glmerControl(optimizer = "bobyqa"))
summary(MG)
#Groups  Name                 Std.Dev. Corr                
#ID      (Intercept)          1.14030                      
#PDE2                 0.96576   0.413              
#Prohibited.act2      1.66273   0.417 -0.080       
#PDE2:Prohibited.act2 0.83688   0.275 -0.384 -0.078
#Dilemma (Intercept)          1.03962                      
#PDE2                 1.26028  -0.579              
#Prohibited.act2      1.06800   0.587  0.199       
#PDE2:Prohibited.act2 1.23052   0.569 -0.264  0.353

#get rndm effects from GLMM
randomeffects <- ranef(MG)

#set seed
set.seed(250923)
#general 
sub_n = 200
stim_n = 268
dilemma_n = 33 

#####RANDOM EFFECTS#####


####SUBJECTS####

###intercepts###
sub_i <- sample((randomeffects$ID$`(Intercept)`),sub_n)
print(sub_i)

###slopes###
#slopes for random effects (PDE & PBH) from pilot data output:
sub_PDE_slope <- sample((randomeffects$ID$PDE2),sub_n)
print(sub_PDE_slope)
sub_PBH_slope <- sample((randomeffects$ID$Prohibited.act2),sub_n)
print(sub_PBH_slope)

#random slopes for novel variable estimated w. pilot:
sub_agent_slope <- sample(((sub_PBH_slope + sub_PDE_slope) / 2),sub_n)
print(sub_agent_slope)


##interaction slopes##
#interaction slopes for random effects (PDE & PBH) from pilot data output:
sub_PDExPBH_slope <- sample((randomeffects$ID$PDE2:Prohibited.act2),sub_n)

#random interaction slopes for novel variable estimated w. pilot:
#sampled slopes for agent & create new variable w. sampled PDEXPBH slope 
#w.PDE
sample_agent <- sample(sub_agent_slope, size = sub_n)
sample_PDExPBH <- sub_PDExPBH_slope
#w.PBH
sample_PDExPBH2 <- sample(sub_PDExPBH_slope, size = sub_n)
sample_agent2 <- sample(sub_agent_slope, size = sub_n)

#Generate random sampled slopes for two-way interactions w. agent:
sub_PDExagent_slope <- (sample_PDExPBH + sample_agent) / 2
sub_PBHxagent_slope <- (sample_PDExPBH2 + sample_agent2) / 2

#Generate random sampled slopes for three-way interaction:
sub_PDExPBHxagent_slope <- (sub_PDExPBH_slope + sub_PDExagent_slope + sub_PBHxagent_slope) / 3


####STIMULI####
#includes replace=TRUE as novel data stim_n > pilot stim_n

###intercepts###
stim_i <- sample((randomeffects$Dilemma$`(Intercept)`),(dilemma_n),replace=TRUE)
print(stim_i)

###slopes###
#slopes for random effects (PDE & PBH) from pilot data output:
stim_PDE_slope <- sample((randomeffects$Dilemma$PDE2),(dilemma_n),replace=TRUE)
print(stim_PDE_slope)
stim_PBH_slope <- sample((randomeffects$Dilemma$Prohibited.act2),(dilemma_n),replace=TRUE)
print(stim_PBH_slope)

#random slopes for novel variable estimated w. pilot:
stim_agent_slope <- sample(((stim_PBH_slope + stim_PDE_slope) / 2),(dilemma_n),replace=TRUE)
print(stim_agent_slope)


##interaction slopes##
#interaction slopes for random effects (PDE & PBH) from pilot data output:
stim_PDExPBH_slope <- sample((randomeffects$Dilemma$`PDE2:Prohibited.act2`),dilemma_n, replace=TRUE)
print(stim_PDExPBH_slope)

#random interaction slopes for novel variable estimated w. pilot:
#sampled slopes for agent & create new variable w. sampled PDEXPBH slope 
#w.PDE
sampleS_agent <- sample(stim_agent_slope, size = dilemma_n, replace = TRUE)
sampleS_PDExPBH <- stim_PDExPBH_slope
#w.PBH
sampleS_PDExPBH2 <- sample(stim_PDExPBH_slope, size = dilemma_n, replace = TRUE)
sampleS_agent2 <- sample(stim_agent_slope, size = dilemma_n, replace = TRUE)

#Generate random sampled slopes for two-way interactions w. agent:
stim_PDExagent_slope <- (sampleS_PDExPBH + sampleS_agent) / 2
stim_PBHxagent_slope <- (sampleS_PDExPBH2 + sampleS_agent2) / 2

#Generate random sampled slopes for three-way interaction:
stim_PDExPBHxagent_slope <- (stim_PDExPBH_slope + stim_PDExagent_slope + stim_PBHxagent_slope) / 3


#####TRIALS#####
####SUBJECTS####
sub <- tibble(
  sub_id = 1:sub_n,
  sub_i = sub_i,
  sub_PDE_slope = sub_PDE_slope,
  sub_PBH_slope = sub_PBH_slope,
  sub_agent_slope = sub_agent_slope,
  sub_PDExPBH_slope = sub_PDExPBH_slope,
  sub_PDExagent_slope = sub_PDExagent_slope,
  sub_PBHxagent_slope = sub_PBHxagent_slope,
  sub_PDEXPBHxagent_slope = sub_PDExPBHxagent_slope
  
)
print(sub)


####STIMULI####
###estimates for stimuli via cross-over of variable levels### 
#probability of "yes" DV responses based on each level combo from pilot - assuming no effect of agent
df$interaction <- interaction(df$PDE, df$Prohibited.act)
counts <- table(df$interaction, df$Permissible)
total <- table(df$interaction)
print(df$interaction)
print(counts)
print(total)
# Calculate proportion of "Yes" responses for each level combo
noNoProb <- 4552 / 7476
print(noNoProb)
yesNoProb <- 4735 / 5340
print(yesNoProb)
noYesProb <- 4378 / 14952
print(noYesProb)
yesYesProb <- 3887 / 6408
print(yesYesProb)

#scaling factor (putting estimate exactly as GLMM output will not equal estimate value in novel output as REs are removed, scaled up to compensate)
#estimate of pilot interaction (as interaction estimate corresponds to YES:YES levels)
unscaled_yesYesEstimate <- 0.4154014
sf <- yesYesProb / unscaled_yesYesEstimate
print(sf)
yesYesEstimate <- sf

noNoEstimate <- noNoProb * sf
yesNoEstimate <- yesNoProb * sf
noYesEstimate <- noYesProb * sf


###number of stimuli per level for ONE agent###
stimN_noNo = 15
stimN_noYes = 31
stimN_yesNo = 10
stimN_yesYes = 12


##Create a vector to assign group numbers to each stim_id for each level##
noNo_group_assignment <- rep(1:stimN_noNo, times = stimN_noNo)
print(noNo_group_assignment)
noYes_group_assignment <- rep((stimN_noNo+1):(stimN_noNo+stimN_noYes), times = stimN_noYes)
print(noYes_group_assignment)
yesNo_group_assignment <- rep(((stimN_noNo+stimN_noYes)+1):(stimN_noNo+stimN_noYes+stimN_yesNo), times = stimN_yesNo)
print(yesNo_group_assignment)
yesYes_group_assignment <- rep(((stimN_noNo+stimN_noYes+stimN_yesNo)+1):(stimN_noNo+stimN_noYes+stimN_yesNo+stimN_yesYes), times = stimN_yesYes)
print(yesYes_group_assignment)


###HUMAN STIMULI###
#PDE: NO, PBH: NO
stim_noNoHuman <- tibble(
  stim_id = 1:stimN_noNo,
  PDE_version = "No",
  PBH_version = "No",
  agent_version = "Human",
  dilemmaID = c(noNo_group_assignment),
  PDExPBH = noNoEstimate
)
print(stim_noNoHuman)

#PDE: NO, PBH: YES
stim_noYesHuman <- tibble(
  stim_id = 1:stimN_noYes,
  PDE_version = "No",
  PBH_version = "Yes",
  agent_version = "Human",
  dilemmaID = c(noYes_group_assignment),
  PDExPBH = noYesEstimate
)
print(stim_noYesHuman)
#PDE: YES, PBH: NO
stim_yesNoHuman <- tibble(
  stim_id = 1:stimN_yesNo,
  PDE_version = "Yes",
  PBH_version = "No",
  agent_version = "Human",
  dilemmaID = yesNo_group_assignment,
  PDExPBH = yesNoEstimate
)
print(stim_yesNoHuman)
#PDE: YES, PBH: YES
stim_yesYesHuman <- tibble(
  stim_id = 1:stimN_yesYes,
  PDE_version = "Yes",
  PBH_version = "Yes",
  agent_version = "Human",
  dilemmaID = yesYes_group_assignment,
  PDExPBH = yesYesEstimate
)
print(stim_yesYesHuman)

###AI STIMULI###
#PDE: NO, PBH: NO
stim_noNoAI <- tibble(
  stim_id = 1:stimN_noNo,
  PDE_version = "No",
  PBH_version = "No",
  agent_version = "AI",
  dilemmaID = c(noNo_group_assignment),
  PDExPBH = noNoEstimate
)
print(stim_noNoAI)
#PDE: NO, PBH: YES
stim_noYesAI <- tibble(
  stim_id = 1:stimN_noYes,
  PDE_version = "No",
  PBH_version = "Yes",
  agent_version = "AI",
  dilemmaID = c(noYes_group_assignment),
  PDExPBH = noYesEstimate
)
print(stim_noYesAI)
#PDE: YES, PBH: NO
stim_yesNoAI <- tibble(
  stim_id = 1:stimN_yesNo,
  PDE_version = "Yes",
  PBH_version = "No",
  agent_version = "AI",
  dilemmaID = yesNo_group_assignment,
  PDExPBH = yesNoEstimate
)
print(stim_yesNoAI)
#PDE: YES, PBH: YES
stim_yesYesAI <- tibble(
  stim_id = 1:stimN_yesYes,
  PDE_version = "Yes",
  PBH_version = "Yes",
  agent_version = "AI",
  dilemmaID = yesYes_group_assignment,
  PDExPBH = yesYesEstimate
)
print(stim_yesYesAI)

###bind human + AI stimuli###
stim <- bind_rows(stim_noNoHuman,stim_noYesHuman,stim_yesNoHuman,stim_yesYesHuman,stim_noNoAI,stim_noYesAI,stim_yesNoAI,stim_yesYesAI)

###create dilemma random effects###
dilemmaRE <- tibble(
  dilemmaID = 1:dilemma_n,
  stim_i = stim_i,
  stim_PDE_slope = stim_PDE_slope,
  stim_PBH_slope = stim_PBH_slope,
  stim_agent_slope = stim_agent_slope,
  stim_PDExPBH_slope = stim_PDExPBH_slope,
  stim_PDExagent_slope = stim_PDExagent_slope,
  stim_PBHxagent_slope = stim_PBHxagent_slope,
  stim_PDEXPBHxagent_slope = stim_PDExPBHxagent_slope
  ) 
print(dilemmaRE)

##add dilemma ID to dilemmaRE data frame##
dilemmaRE <- dilemmaRE %>%
  mutate(dilemmaID= 1:nrow(dilemmaRE))
print(dilemmaRE)

##add dilemmaRE to stim using dilemmaID as mutual column##
stim <- stim %>%
  left_join(dilemmaRE, by = "dilemmaID")
print(stim)

#####CREATE TRIALS FOR EACH PARTICIPANT#####
###create data frame###
##data frame for each participant##
participant_data <- lapply(1:sub_n, function(i) {
  sub_data <- stim
  sub_data$sub_id <- i  # Add the participant's ID number
  return(sub_data)
})

##bind all the participant data frames together##
allData <- bind_rows(participant_data)

###add random effects to data frame###
all_data <- allData %>%
  left_join(sub, by = "sub_id") # includes the intercept, slope, and condition for each subject
print(all_data)

#####FIXED EFFECTS#####
###fixed effects based on pilot data for PDE & PBH### 
#PDE
PDE_yes = 2.1917278 * sf
#PBH (is -2.0254895 but add minus to iterative code)
PBH_yes = 2.0254895 * sf
###agent set to small effect (log(1.5)))###
agent_yes = 0.42 * sf

###interactions###
##from pilot##
PDExPBH = 0.4154014 * sf
##with agent - set to 0##
PDExagent = 0
PBHxagent = 0
PBHXPDExagent = 0

#####ERROR#####
###extract error from pilot data###
# Raw residuals
raw_residuals <- residuals(MG, type = "response")
print(raw_residuals)
error_sd <- sd(raw_residuals)
print(error_sd)

#####THRESHOLD FOR YES RESPONSES#####
###find threshold in pilot data###
##extract predicted probabilities from pilot##
predicted_probabilities <- predict(MG, type = "response")
print(predicted_probabilities)

##create ROC curve from pilot data##
#compute ROC curve
roc_curve <- roc(df$Permissible, predicted_probabilities)

#plot ROC curve
plot(roc_curve)

##use ROC curve to determine threshold characterising responses in pilot data##
# Find the optimal threshold
optimal_threshold <- coords(roc_curve, "best", ret = "threshold")
print(optimal_threshold)


#####CREATE SAMPLE DATA VIA ITERATIONS OF CODE BELOW FOR EACH ROW IN ALL_DATA#####
sampleData <- all_data %>%
  mutate(
    #effect-code subject condition and stimulus version
    PDE_version.e = recode(PDE_version, "No" = -PDE_yes, "Yes" = PDE_yes),
    PBH_version.e = recode(PBH_version, "No" = PBH_yes, "Yes" = -PBH_yes),
    agent_version.e = recode(agent_version, "AI" = -agent_yes, "Human" = agent_yes),
    # calculate trial-specific effects by adding overall effects and slopes
    PDE_eff = PDE_version.e * sub_PDE_slope * stim_PDE_slope,
    PBH_eff = PBH_version.e * sub_PBH_slope *stim_PBH_slope,
    agent_eff = agent_version.e * sub_agent_slope *stim_agent_slope,
    # calculate error term (normally distributed residual with SD set above)
    err = rnorm(nrow(.), 0, error_sd),
    # calculate DV from intercepts, effects, and error 
    dv = (sub_i + stim_i + 
          PDE_eff +
            PBH_eff +
            agent_eff + err),
    #convert to binary DV using calculated threshold
    Permissibility = ifelse(dv > optimal_threshold, 1, 0))
  
###check worked as expected###
#visual check
print(sampleData)
view(sampleData)


#test analysis w.o RE as faster, just to check general direction
sampleTest <- glm(Permissibility~PDE_version*PBH_version*agent_version,family=binomial,data=sampleData)
summary(sampleTest)

#####OPEN SAMPLE SIZE CALCULATION CODE#####
'''
