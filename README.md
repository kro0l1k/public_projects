### One of the most pressing research questions in the field of AI of our time is, in my opinion, learning from observation of real-world data.
 How come 17-year-olds can drive just after 20 hours of tuition, yet fully self-driving cars are still a work in progress? 

To approach this question, I have tried to divide the problem of learning from observation into subproblems, which gradually increase in difficulty. I will focus on three examples where I developed some insights about the problem: 
1. **Learning how to play tic-tac-toe**
2. **Learning how to play variants of chess**
3. **Learning how to predict the behaviour of nearby cars**

In principle, we can generate trees of allowed moves for the tic-tac-toe and chess games and based on that, use Search tools to find an optimal strategy. This approach would not likely yield any new results for the car problem. Instead, I will describe below a few insights I have derived from easier examples, which I hope will be useful for the autonomous driver.

I assume that during training, the agent is allowed to 'sit in the passenger seat' for a large amount of time, meaning they will have access to a large set of trajectories, without any influence over the control variables or the players' choices. This is significantly different from sitting behind the wheel, as the agent can't see what exactly his options for placing an 'X', moving a knight, or turning the steering wheel are and what the immediate consequences of his actions will be. The passenger seat, or offline, training is relatively cheap, as we do not pose any hazard to the environment, and if need be, we can let the agent watch thousands of tic-tac-toe or chess matches, and use large amounts of sensor data from modern cars and driver.

It is asking the agent to move a piece or drive a car somewhere, which can be dangerous for the model and his surroundings.

Furthermore, I will assume that will try to compare how different levels of detail in the the environment models influence the agent's behavior. It is unreasonable to assume we will be able to build a perfect model of the street environment. We can approximate it, i.e. define what different signs of the road mean, on which side of the road to drive, etc., but we will not be able to explain which drivers on the road are drunk, which are in a hurry, which have unusual reaction times in a traffic jam.  Any of these situations can be dangerous when a rigid model-based algorithm encounters an anomaly. 

1. # Learning the rules of tic-tac-toe.
I will construct a model-free approach for tic-tac-toe - I will put the tightest constraints on the attainable information for offline training: assume the agent can only access 'snapshots' of the board. Not even singular moves, but a set of 3x3 pictures like:

```
..X   X.O
.O.   OX.
..X   O.X
```
From them, I construct the graph of allowed moves of the game. The code for this is available in `tic-tac-toe.ipynb`.

To make this example more useful, I would like to add self-supervised learning to the mix here to aid in capturing different modalities of moves for more complicated games. These difficulties do not occur in tic-tac-toe, but in the harder examples, for instance, I would like to capture offline the different moves of a bishop, knight, or a rook to capture better 'similar' states of the board, before applying the persistence homology algorithm. I also want to learn how to use additional data, such as sequences of moves, to verify the correctness of the model and, in case of inconsistencies, fix the output graph.


2. # Learning the rules of an unknown variant of chess. 
The next step is to try to apply the same SSL + PH algorithm to a more complicated game like chess. I am not sure if the lost symmetry of the players (white always moves first) will allow me to use the H_1 trick efficiently, but this is the next thing on my to-do list. 

What would be insightful here, is investigating how a model-based algorithm can be integrated with the geometrical observations technique developed in 1.

Namely, let's assume we have the tree of allowed moves for the game of chess. The tree is quite large, but we know methods to construct it, at least locally (for instance: the next 3-5 moves). Now let us assume that we observe anomalies in the moves of the other players at our tournament (i.e. in the phase of offline learning, or even 'online' when the opponent's move seems illegal in the agent's graph). For example, let us assume that the game has suddenly been changed, and now a knight moves only two tiles N/S + two tiles E/W, instead of the classical 1+2 or 2+1 move. Not adhering to the rules of the graph can be immediately detected: the opponent has used an edge that didn't exist in the graph, or our understanding of the game, but the research question here is: how many of such irregular moves do we have to see to be able to infer from observation how the agent is supposed to move his knights, and what the new strategy for winning is. 

It has, in my opinion, significant implications for the interesting driving case: how long do we have to observe another driver on the road to be able to determine that they are not adhering to the rules of the road, that their loss functions if optimal control is not what we expected, and most importantly how the agent is supposed to act in the presence of this anomaly to not break the 'changed rules of the road' and avoid potential crashes with the odd driver.

3. # Learning the parameters of the utility and loss functions of an agent.
I will not focus here on the computer vision aspect of the challenge, but rather treat it as an optimal control problem in a multi-agent setting. For instance, I will consider an adapted example from Dr. Kalise's Optimization class at Imperial, which will correspond to the behaviour of a car trying to do a $180 \degree$ turn around the point $(0.5, 0)$: 

Consider a car moving in $\mathbb{R}^2$, from the starting point $x(0) = y(0) = 0$ to the endpoint $x(1) = 1, y(1) = 0$ with the following model of its dynamics: 

$$ \bar{x}' = f(\bar{x}(t), u(t)) = \begin{bmatrix} V cos(u) \\ V sin(u) \end{bmatrix} $$

where $V > 0$ is a constant (velocity), and the control variable $u$ corresponds to its direction. Let us assume the driver wants to minimize the time spent around the apex of the corner (low values of y), which can be expressed by the loss function as $$J = C \cdot \int_0^1 y dt $$, for a fixed $C>0$. One can show that a trajectory satisfying the Pontryagin Minimum Principle (so a solution to the optimal control problem) will satisfy: $tan(u(t)) = at + b$ for some real values $a, b$. 

The research question I would like to work on next is: 
Given an observed trajectory of another car: $X[t]$ how quickly can we infer the parameters of its loss function (for instance, the parameter $C$) and establish that the loss function of this agent is different from our prior assumption (i.e. there might be another term like $||u||_{L_1}$, or a variety of other terms in $x(t), u(t)$ in the loss function $J$), how quickly can we model these terms, and act upon it in the presence of uncertainty i.e. optimize our strategy for taking the corner, in case our result also depends on the trajectory of the observed car. 
