---
title: 'Portfolio Optimization and Daily Fantasy Sports'
date: 2024-12-30
permalink: /posts/daily-fantasy-sports/
---

# Portfolio Optimization and Daily Fantasy Sports

#### Technologies and topics used: SQL, R, Julia, Optimization, Statistics, Selenium, Web Scraping

## Introduction

I started grad school during COVID, and since I had a lot of free time as a result, I wanted a project that would keep my data and numerical methods skills sharp. Now, I went to Alabama for undergrad, and if there's one thing we love in Alabama, it's sports, and I thought something in that domain would be fun and interesting. I decided to implement this [paper](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3393127) by Martin Haugh and Raval Singal, which presents a quantitative strategy for betting on daily fantasy sports (DFS).

The core idea of DFS is that for a given sport, you build a "team" of players within a salary limit, who are awarded fantasy points based off of some system the website sets. You pay an entry fee per lineup, and if you finish in the top X%, you get a payout. These payout structures vary, but the one we're interested in is called a "top heavy" payout structure, which is simply one where e.g. 1st place gets a bigger payout than 2nd, and so on down to the lowest scoring finisher. In these contests, you can typically enter many lineups, often up to 150, and costs can be as low as 5 cents per entry.

In this structure, each lineup can be thought of like an option (as in, from finance). If the lineup scores higher than opponents, you get a payout, which may be higher than your entry fee (premium). If your lineup does terrible relative to opponents, you lose your option premium. The goal is to create a portfolio of lineups with the highest summed payout. The problem is that, more-so than finance, your payout depends on what lineups your opponents pick (e.g. if everybody knows beforehand what the top scoring fantasy lineup is, then everyone ties and no-one profits). This is what Haugh and Singal address in their paper.

## Let's see the math

Ok, let's restate the DFS description from the introduction, with some notation. First, assume there are D payoff rungs and number all of them as $$d \in {1:D}$$. Write the payoff for each rung as $$R_d$$, and assume these payoffs are earned if your rank is in $$(r_{d-1}, r_d]$$. The same payoff can be applied to multiple ranks.

$$R_1 > R_2 > ... > R_D > R_{D+1} := 0$$

Now, write our portfolio of $$N$$ lineups as $$\textbf{W} := \{w_i \}_{i=1}^N$$, all $$O$$ opponent lineups as $$\textbf{W}_o$$. These vectors have an entry for every possible player (out of $$P$$ total players) on every possible team (out of $$T$$ teams) for a given contest, and have an entry of either 1 or 0 if a player is in a given lineup. The realized points each player scores is written as $$\delta$$.  Finally, wirte the order statistic of all the fantasy scores (minus lineup $$i$$) for the contest as $$G_i^{(r_d)}$$. For $$r_1$$, $$G_i^{(r_1)} = N +O - 1$$,  We then want to maximize the probability our portfolio has a positive total playoff.

$$\max_{\textbf{W}_N} \sum_{i=1}^N \sum_{d = 1}^D (R_d - R_{d+1}) P(w_i^T \delta > G_i^{(r_d)})$$ 

In other words, we want to maximize the sum of expected value of the $$\textbf{marginal gain}$$ of obtaining a rung  $$R_d$$, across all rungs across all lineups. 

### How do we solve this?

To illustrate this, let's take the case where $$N = 1$$.

$$\max_{\textbf{w}} \sum_{d = 1}^D (R_d - R_{d+1}) P(w^T \delta > G_i^{(r_d)})$$ 

Let's define $$Y_{\textbf{w}}^d$$ with $$Y_{\textbf{w}}^d = w^T\delta  -  G_i^{(r_d)}$$. Let's also assume this is normally distributed. In the following, $$\sigma_{\delta,G_i^{(r_d)}}$$ is a $$P$$ x $$1$$ vector where each element is equal to the covariance of player $$p$$'s score and $$G_i^{(r_d)}$$.

$$\mu_{Y_{\textbf{w}}^d} = w^T E(\delta) - E(G_i^{(r_d)})$$
$$\sigma_{Y_{\textbf{w}}^d}^2 = w^T var(\delta) w + \sigma_{G_i^{(r_d)}}^2 - 2w^T \sigma_{\delta,G_i^{(r_d)}}$$.

Then, we can approximate the optimization problem for the single lineup case as 

$$\max_{\textbf{w}} \sum_{d = 1}^D (R_d - R_{d+1}) (1 - \Phi(\frac{\mu_{Y_{\textbf{w}}^d} }{\sigma_{Y_{\textbf{w}}^d}}))$$ 

With some assumptions, we can turn this into a tractable problem. First, we assume that $$\mu_{Y_{\textbf{w}}^d} < 0$$ for all possible $$w$$. This doesn't stretch the imagination, because if the assumption wasn't reasonable then it would be way easier to make money using a normal integer program. Further, assume the covariance of player $$p$$'s score and $$G_i^{(r_d)}$$ is the same value for all ranks $$d$$. This makes our lives easier, but Haugh and Singal also provide a proof if you're curious. Then, we have to maximize the variance of $$Y_{\textbf{w}}^d$$ to maximize expected payoff of each rung. 

All of this leads directly to a proposed algorithm in the single contest case.

1. Form a grid of penalty parameters $$\lambda \in \Lambda = (0,0.01,...0.19,0.20)$$
2. For each $$\lambda$$, solve $$w_{\lambda} = argmax_{w} ( w' \mu_{\delta} + \lambda (w^T var(\delta) w - 2w^T \sigma_{\delta,G_i^{(r_d)}}))$$
3. Figure out which $$\lambda^* = \argmax_{\lambda} \sum_{d = 1}^D (R_d - R_{d+1}) P(w_{\lambda}^T \delta > G_i^{(r_d)})$$
4. Return $$w_{\lambda^*}$$

### How do we extend to the case of multiple lineups?

This optimization problem gets a lot harder if you want to apply it to the case of multiple lineups. Luckily, if we're willing to assume the portfolio reward function (let's call it $$R(\textbf{W})$$) is monotone submodular (adding a lineup to a smaller portfolio of size $$N_1$$ increases $$R(\textbf{W})$$ at least as much to a larger set of portfolios of size $$N_2 > N_1$$) then we can trivially adapt the above algorithm to the multi-lineup case to approximate the optimal result (see [this result](https://link.springer.com/article/10.1007/BF01588971)).

$$R(W) = \sum_{w}^W \sum_{d = 1}^D (R_d - R_{d+1}) P(w^T \delta > G_i^{(r_d)})$$

1. Set $$\textbf{W}^* = \empty$$
2. For each $$i$$ in $$1:N$$:
  1. Form a grid of penalty parameters $$\lambda \in \Lambda = (0,0.01,...0.19,0.20)$$
  2. For each $$\lambda$$, solve $$w_{\lambda} = argmax_{w} ( w' \mu_{\delta} + \lambda (w^T var(\delta) w - 2w^T \sigma_{\delta,G_i^{(r_d)}}))$$
  3. Figure out which $$\lambda^* = \argmax_{\lambda} R(\textbf{W}^* \cup w_{\lambda})$$
  4. set $$\textbf{W}^* =\textbf{W}^* \cup w_{\lambda^*}$$

## Numerical implementation

Ok, neat math, but this project is about implementation.

### How do we optimize this?

This objective function is a binary quadratic program, so any BQP solver will do. Gurobi is fast, but not licensed for commercial use, so unless you pay 10 grand, you should use a slower one and go drink a cup of coffee while you wait.

### Are there constraints?

Yes, and that's also the richest area to improve the implementation of this. One thing you can do to affect outcomes is limit the number of players than can be repeated (e.g. no lineup can share more than X players wiht another lineup). This is an especially important lineup in sports like hockey, where DFS contests happen daily instead of weekly, and may only include 1 or 2 games, when you'd want the allowed overlap to be very high, or as many as 12, when it should be lower.

You also need to enforce the normal lineup constraints--at least 3 teams, X many players from each position, and the salary cap.

### What about stacking?

Absolutely stack your lineups, if you have expertise in that. The idea here is to promote within-team correlation (e.g. in football WRs and QBs are highly correlated, since QBs earn points when WRs earn points, or in hockey you may want to include at least one full line in every lineup). This is a great area to put your domain knowledge to work. 

### Where do we get $$E(\delta)$$?

Obviously, these are unobservable quantities. So to get these, we have to cheat a little. I, and Haugh and Singal, scrape $$E(\delta) = \hat{\delta}$$ from an off-the-shelf forecast websites. For many sports, Daily Fantasy Nerd is easy to scrape. If you can get the data for it, you could probably do better by fitting your own forecasts in Stan or PyMC3. 

### Where do we get $$var(\delta)$$?
To compute this, I take the historical fantasy sports scored for each game, and assemble a correlation matrix for the positions in a match (for example, first string goalie for the home team vs third string winger for the away team). Using this "game correlation matrix," I then query each player's historical points standard deviation, and use that to compute the point covariance matrix $$\hat{\Sigma_{\delta}}$$ for each game in a contest. For players without histories, I use the average for their position.

 If you were rolling your own forecasts in Stan, you could probably get a better approximation of the true $$var(\delta)$$.

### Where do we get $$G_i^{(r_d)}$$?

This is a great question--after all, the performance of this strategy is completely dependent on the opponent portfolios, so this is central to the method. To forecast this, I scrape forecasts of player ownership percentages (how many contestants will draft a given fantasy player for a given DFS contest). These vary contest to contest, so Draftkings and Fanduel and Yahoo will have different ownership distributions for the same games. I then use this to simulate a portfolio of lineups that roughly matches this distribution and the size of the contest.

Martin and Singal go a step further, and start to develop their own forecasts from historical percent ownership and realized percent ownership data, using some cool Bayesian methods. You need to manually check the ownership percentages on Fanduel for that though, and as this was a learning project for me, that didn't pass the CBA.

These ownership percentages are hard to find--you definitely need to pay for them. RotoWire and RotoGrinders are pretty cheap though, and they're eaily scraped.

Once you have your simulated lineups, simulate point vectors $$\delta_i \sim N(\hat{\delta},\hat{\Sigma_{\delta}})$$ from your points distribution and use that to calculate the distribution of order statistics for a DFS contest.


## Data work

I implemented this for baseball (MLB) and hockey (NHL). The webscraping was done with Selenium and R. After scraping, data was stored in an SQLite db. I also prepped the covariance matrices, and simulated the opponent lineups in R. For the actual optimization, I used Julia--it doesn't much matter, since all the optimization engines work in lower level languages anyway, but Julia's fun to play around with. There is a manual part to this process--for example, it's in Fanduel's TOS not to scrape their website, so lineups have to be uploaded manually. The rest of this process can be completely automated though, and run via cron (or you could set these up as batch jobs on AWS).










> Written with [StackEdit](https://stackedit.io/).