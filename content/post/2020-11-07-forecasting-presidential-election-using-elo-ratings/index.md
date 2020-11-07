---
title: Forecasting presidential election using Elo Ratings
author: Quang Nguyen
date: '2020-11-07'
slug: forecasting-presidential-election-using-elo-ratings
categories:
  - elections
  - predictive models
  - sports statistics
tags:
  - statistics
  - projects
---

The elo rating system is a method developed by [Arpad Elo](https://en.wikipedia.org/wiki/Arpad_Elo) as a way to rank players in a zero sum game. Elo has been used in various contexts, particularly notable in chess and more recently, competitive online games. A really nice attribute about Elo is that a difference in scores can be translated to a probability, through the formula:    

$$E_A = \frac{1}{1 + 10^{R_A - R_B}}$$

where EA is the expected score of player A with rank $$R_A$$ playing against player $$B$$ of rank $$R_B$$   
This is equivalent to applying the logistic function to a rescaled version of the difference in rankings 

$$logistic\left(\frac{\log(10)}{400} (R_A - R_B)\right)$$

This ensures that the expected values for players A and B are bounded between 0 and 1 and hence acts as probabilities.  

In an elo model, after each match, the rankings are updated as:  

$$R_{A_{new}} = R_A + K * (\text{outcome} - E_A)$$   

where K is the K-factor controlling the relative gains and losses of elo points. 

The US presidential election is the perfect place to test the elo theory since there is usually 2 candidates who campaigns for a long period of time with lots of polling data. The winner is determined through the electoral college, which follows a winner-take-all system. This further makes the US presidential election closer to a sports match.  

In this instance, we decided to use the elo system to model the probability of winning an election. We take the weekly average of polls to act as "games" that then culminate in a showdown on November 3rd. After each weekly poll, the candidate gains or lose elo based on the elo model presented above. Some **major caveats:**  
* First, this is obviously not considering any fundamental of election forecasting, such as the inclusion of demographic and economical variables as well as state-by-state correlation.   
* Second, this model does not incorporate any outside knowledge other than the way FiveThirtyEight weights the polls (we use average polling data from the site).    
* Third, the model really preferences candidates who are polling well consistently, and rewards them with higher probability of winning. We think this is a pretty good model considering that close polling reduce the gains in elo and narrows the probability of winning, and sudden losses in polling numbers also contribute towards a candidate's poor showing. However, this is very non-traditional and not really an apt approach towards proper modeling of an election.   


Considering those aspects, we further considered some modifications to the model. 
* First, we seeded initial elo based on election results from the 2016 election. using 1000 elo as a baseline, we subtract or add points to candidate's starting elo based on how they (or the candidate belonging to their party) performed previously. We call this our priors.  
* Second, we incorporate margin of victory into our elo update calculation by modifying the K-factor. This is a simple linear adjustment $$K_{eff} = K * mov$$ where $$mov$$ is the margin of victory. This means that the "winning" candidate will gain points as a proportion of K that is equal to their margin of polling victory. As such, we increase values of K significantly to compensate for the low gains in elo when margins can be 1 percent.  
* Finally, we perform elo modeling at the state level, then use those probabilities to randomly draw a number of simulations based on a Bernoulli process. We calculate the final probability of winning as the number of simulations where a focal candidate (in this case Biden) gains 270 or more Electoral Votes.   

All modeling was done in Julia. Side note: Pluto.jl is an amazing notebook system and probably going to be the way to go when it comes to the next generation of data notebooks (Sorry Jupyter). You can access the *interactive* Pluto.jl notebook (with the same text above) [at this link](https://mybinder.org/v2/gh/fonsp/pluto-on-binder/master?urlpath=pluto/open?url=https%253A%252F%252Fgithub.com%252Fqpmnguyen%252Felo_presidente%252Fblob%252Fmaster%252Fanalysis.jl%253Fraw%253Dtrue).  
