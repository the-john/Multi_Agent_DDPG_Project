# Multi Agent Project Report

One thing that really caught me off-guard was the instability of multi-agent system.  I was warned about it, but man … it took a lot of hyperparameter tweaks to get something that finally was able to meet the requirements of what was deemed as “learned”.

## The Learning Algorithm

If we’d like our agents to become truly intelligent, they must be able to communicate with — and learn from — other agents. Multi-agent reinforcement learning has many real-world applications, ranging from self-driving cars to warehouse management.

Not only does one agent have their set of possible actions, and the other agent their set of possible actions; but together they have a shared set of actions.  (So three Q-Tables).  When using the Markove Game Framework, State transitions are Markovian, just like an MDP (Markov Decision Process).  Remember, Mazrkovian is when the next State only depends on the present State and the Actions taken in this State.  With multi-agents, the transition from one State to the next depends on the “joint action” of the agents.

There are two approaches here; train all the agents independently without considering other agents (any agent considers other agents as being part of the environment), or the meta-agent approach.

The Meta-Agent approach:
Takes into account the existence of multiple agents.  A single policy is learned for all of the agents
-	The policy takes as input the present state of the environment
-	It returns the action of EACH agent in the form of a single joint action vector
-	Typically a single reward function given the environment state and the action vector returns a "global" reward

The joint Action Space increases exponentially with the number of agents.
Because agents can only see locally, each agent will have a different observation of the environment state.  So this approach only works when EVERY agent knows EVERYTHING about the environment

Cooperative Environment = Agents attempt a group task and cooperate to do so
Competitive Environment = Agents are just concerned with maximizing their own rewards
 
The way the rewards are defined makes an environment competitive or cooperative.  Mixed Cooperative-Competitive Environments are when in many environments, the agents need to show a mixture of cooperative and competitive behaviors.

I chose the Multi-Agent Deep Deterministic Policy Gradient (MA-DDPG) for this project because I was impressed on how well DDPG worked in my last project.  It made a lot of sense to me.  A DDPG is an off policy, Actor-Critic algorithm that uses the condept of Target networks.  The input of the active network is the current state, while the output of the active network is the action chosen from a continuouse action space.

A framework of centralizing training with decentralized execution is used here.  What this implies is that some extra information is used to ease training ... BUT, that information is not used during the testing time.  And this is where the Actor-Critic algorithm kicks in.  During training, the critic for each agent uses extra information (like states observed and actions taken by all the other agents).  Each agent has its own actor that has access to only its agents’ observations and actions.  During execution time, only the Actors are present - and all observations and actions are used.  A learning critic for each agent allows us to use a different reward structure for each … which means that the final algorithm can be used in all cooperative, competitive, and mixed scenarios.  You can see this in the diagram below.

![Actor_Critic_PIC.png](https://github.com/the-john/Multi_Agent_DDPG_Project/blob/master/Actor_Critic_PIC.png)

I used a replay buffer to keep past experiences and randomly chose from that buffer to influence my next choice of action.  To keep things simple, both agents are sharing the same experiences.  I did used the same noise methodology as I used in my last DDPG project, OUNoise.  OUNoise is based off of the Ornstein-Uhlenbeck process is both a Gaussian and Markov process that is temporarily homogeneous.  Over time, this process tends to drift towards its long-term mean.

My hyperparameters where the tough thing though.  I just could not get my model to train.  I started out using hyperparameters from my original DDPG project, and they where WAY too high.  Specifically, the number of nodes in my Fully Connected (FC) model layers.  My original DDPG project had 400 nodes in the first layer followed with another 300 nodes in the second layer.  Initially I backed these way off to 128 and 64 accordingly, with no luck.  I then doubled that amount to 256 and 128 and slowly began to see some generalization.  Another set of hyperparameters that I played a lot with was the learning timestep interval and number of learning passes.  I ended up going online to look at other peoples code to see what they did and emulated that.  However, I still don't have a good feel for how these hyperparameters truly impact the overall training and need to spend more time with this.    

## Plot of Rewards

There are two lines in the plot below.  The Blue line shows what the scores where during training.  The Yellow line shows the average score over 100 episodes.

We are considered "trained" when we can average a score of 0.5 for every Agent over 100 episodes.  And that happened at episode 1581.

This time I didn't bother to train a little bit longer to ensure that my saved weights will give me a good chance of getting a score of 0.5 or better in the future.  I had a REALLY hard time getting to 0.5, and it took a LONG time.  So, I'm going to take my win and move on.

![MA_DDPG Plot.png](https://github.com/the-john/Multi_Agent_DDPG_Project/blob/master/MA_DDPG%20Plot.png)
 

## Ideas for Future Work

The first thing I think that I would do is spend a LOT more time tweaking the hyperparameters.  It seemed to take forever to get this thing trained.  The repeatability of this outcome is questionable.

I struggled to get my model to converge when I tried to implement batch normalization in the model.  I eventually abandoned it for the sake of expediency.  I think I would spend a LOT more time trying to get batch normalization to work.

One last key thing that can be done to help increase performance would be to try and prioritize the replay of the experience.  Currently I’m selecting them randomly instead of forcing the use of either rare or successful past experiences.  My hope is things would improve if I got rid of the randomness.  

