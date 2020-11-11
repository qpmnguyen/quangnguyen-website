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
[[Source Code](https://github.com/qpmnguyen/elo_presidente)]  

The elo rating system is a method developed by [Arpad Elo](https://en.wikipedia.org/wiki/Arpad_Elo) in order to rank players by ability in a zero sum game. Elo has been used in various contexts, most famously the way to rank chess players through the UCSF and FIDE systems. Elo has been used in modern contexts as well, from Tinder to World of Warcraft. A really nice attribute about Elo is that a difference in scores can be translated to a probability, through the formula:    

$$E_A = \frac{1}{1 + 10^{R_A - R_B}}$$

where $$E_A$$ is the expected score of player A with rank $$R_A$$ playing against player $$B$$ of rank $$R_B$$   
This is equivalent to applying the logistic function to a rescaled version of the difference in rankings   
$$logistic\left(\frac{\log(10)}{400} (R_A - R_B)\right)$$

The above is the most common formulation of Elo. Through the logistic function, the expected values for players A and B are bounded between 0 and 1 and hence can act as probababilities.  

In an elo model, after each match, the rankings are updated as:  
$$ R_A* = R_A + K * (outcome - E_A) $$  

where K is the K-factor controlling the relative gains and losses of elo points. 

The US presidential election is a surprisingly appropriate context to use elo. In this election, two candidates compete for the "title" of the President of the United States. Polling data is collected regularly, which can act as "matches" where the two candidates can test their "mettle" before the final election day. Furthermore, the structure of the election, a winner-take-all Electoral College system, makes it even closer to being a sports match.     

In this instance, elo system is applied to model the probability of winning an election. Weekly average polling results are used as proxy for "games". After each weekly poll, the candidate gains or lose elo based on whether they win or lose (polling average > 50%) and how much did they win (or lose) by. Some **major caveats and assumptions:**  

* First, this model is for fun, and does not consider variables such as demographics, economics, recent news, etc., which are considered part of fundamentals forecasting. Since the model ingests polling averages from FiveThirtyEight, it does account for quality of polling and other associated factors.  

* Second, the model does not account for state-by-state correlation.  

* Third, the model rewards consistent polling, and rewards them with higher probability of winning. This is a reasonable assumption given how consistent polls are conducted. Any dip in the polls right before the election will be captured in the elo gains/losses as elo decreases the ranking of a player significantly more if he/she loses to a lower ranked player (a leading candidate loosing at the polls against a trailing candidate). However, this is very non-traditional and not really an apt approach towards proper modeling of an election.     

Considering those aspects, we further considered some modifications to the model. 

* First, we seeded initial elo based on election results from the 2016 election. using 1000 elo as a baseline, we subtract or add points to candidate's starting elo based on how they (or the candidate belonging to their party) performed previously. We call this our priors.    

* Second, we incorporate margin of victory into our elo update calculation by modifying the K-factor. This is a simple linear adjustment $$K_{eff} = K * mov$$ where $$mov$$ is the margin of victory. This means that the "winning" candidate will gain points as a proportion of K that is equal to their margin of polling victory. As such, we increase values of K significantly to compensate for the low gains in elo when margins can be 1 percent.    

* Finally, we perform elo modeling at the state level, then use those probabilities to randomly draw a number of simulations based on a Bernoulli process. We calculate the final probability of winning as the number of simulations where a focal candidate (in this case Biden) gains 270 or more Electoral Votes.    

At K = 50, our model predicts that Joe Biden will win the election in ~75% of simulated scenarios. Interested users can fiddle with the K-factor and the number of simulations in the interactive Pluto.jl notebook [at this link](https://mybinder.org/v2/gh/fonsp/pluto-on-binder/master?urlpath=pluto/open?url=https%253A%252F%252Fgithub.com%252Fqpmnguyen%252Felo_presidente%252Fblob%252Fmaster%252Fanalysis.jl%253Fraw%253Dtrue). The notebook also includes visuals of elo progression across time for both candidates for each state, a histogram of electoral votes across all simulation conditions, and the "probability" of winning per state.  



