### One of the most pressing research questions in the field of AI of our time is in my opinion learning from observation of the real-world data.
 How come 17-year-olds can drive just after 20 hours of tuition, yet self-driving cars elude even the smartest tech companies? 

To approach this question I have tried to analyze its challenges one by one. Below I will focus on three examples where learning from observation will be of use: 
1. **Learning how to play tic-tac-toe**
2. **Learning how to play variants of chess**
3. **Learning how to drive a car**

Without much hassle, if we were given a tree of allowed steps in the games of tic-tac-toe and chess we would be able to use Reinforcement Learning, but this would not yield any interesting results for the car problem. So I will try to approach the first two problems in a bit atypical manner. I assume that during training, the agent is allowed to 'sit in the passenger seat', meaning they will have access to a large number of trajectories in the environment. This is significantly different from sitting behind the wheel, as the agent is not able to see what exactly his options for placing an 'X', moving a knight, or turning the steering wheel are, and what the immediate consequences of his actions will be. The passenger seat, or offline, training is relatively cheap, as we do not pose any hazard to the environment, and if need be, we can let the agent watch thousands of tic-tac-toe or chess matches, and use large amounts of sensor data from modern cars and driver.

It is asking the agent to move a piece or drive a car somewhere, which can be dangerous for the model and his surroundings.

Furthermore, I will assume that will try to compare how different levels of explaining the environment to the agent influence his behavior. It is unreasonable to assume we will be able to build a perfect model of the environment and give it as a start for the self-driving car problem. We can approximate it, i.e. define what different signs of the road mean, on which side of the road to drive, etc., but we will not be able to explain which drivers on the road are drunk, which are in a hurry, which have suspiciously slow reaction times in a traffic jam.  Any one of these situations can be dangerous when a model-based algorithm encounters an anomaly. 

1. # Learning the rules of tic-tac-toe.
I will therefore construct a model-free approach for tic-tac-toe I will put the tightest constraints on the attanable information for offline training: I will assume the agent can only access 'snapshots' of the board. Not even singular moves, but a set of 3x3 pictures like:

```
..X   X.O
.O.   OX.
..X   O.X
```
and from them construct the graph of allowed moves of the game. The code for this is available in `tic-tac-toe.ipynb`.

To make this example more useful, I would like to add self-supervised learning to the mix here to aid in capturing different modalities of moves for more complicated games. These difficulties do not occur in tic-tac-toe, but in the harder examples, for instance, I would like to capture offline the different moves of a bishop, knight, or a rook, to capture better 'close' states of the board, before applying the persistence homology algorithm.


2. # Learning the rules of an unknown variant of chess. 
The next step is to try to apply the same SSL + PH algorithm to a more complicated game like chess. I am not sure if the broken symmetry of the players here (i.e. white always moves first) will allow me to use the H_1 trick efficiently, but this is the next thing on my to-do list. 

What would be more interesting for the chess example is investigating how a model-based algorithm can be integrated with the geometrical observations technique developed in 1.

Namely, let's assume we have the tree of allowed moves for the game of chess. The tree is of course quite big, but we can probably store it. Now let us assume that we observe anomalies in the moves of the other players at our tournament (i.e. in the phase of offline learning, or even when the agent is observing his opponent's move). For example, let us assume that the game has suddenly been changed and now a knight moves only two tiles N/S + two tiles E/W, instead of the classical 1+2 or 2+1 move. Not adhering to the rules of the graph can be immediately detected: the opponent has used an edge that didn't exist in the graph, or our understanding of the game, but the research question here is: how many of such irregular moves do we have to see to be able to infer from observation how the agent is supposed to move his knights, and how the winning strategy has changed. 

This has, in my opinion, significant implications for the interesting driving case: how long do we have to observe another driver on the road to be able to determine that they are not adhering to the rules of the road, that their loss functions if optimal control is not what we expected, and most importantly how the agent is supposed to act in the presence of this anomaly to not break the 'changed rules of the road' and avoid potential crashes with the odd driver.

3. # Learning the parameters of the utility and loss functions of an agent.
I will not focus here on the computer vision aspect of the challenge, but rather treat it as a more continuous example of the changed chess game. For instance, let us consider an adapted example from Dr. Kalise's Optimization class at Imperial: 

Consider a boat moving in $\mathbb{R}^2$, from the starting point $x(0) = y(0) = 0$ to the endpoint $x(1) = 1, y(1) = 0$ with the model: 
$$ \bar{x}' = f(\bar{x}(t), u(t)) = \begin{pmatrix} V cos(u) \\ V sin(u) \end{pmatrix}$$

 and let us assume the driver wants to minimize the time spent around the apex of the corner (low values of y), which can be expressed by the loss function as $$J = \int_0^1 y dt $$. One can show that a trajectory satisfying the Pontryagin Minimum Principle will satisfy: $tan(u(t)) = At + b $ for some real values A, B. 

The research question I would like to work on next is: 
Given a sample trajectory: X[t] how quickly can we infer the parameters of his loss function (i.e. the parameter C) and establish that the loss function of this agent is different from our prior assumption (i.e. there might be another term like $||u||_{L_1}, or a variety of other terms of $x(t), u(t)$ in J), how quickly can we model these terms, and act upon it in the presence of uncertainty i.e. optimize our strategy for taking the corner, in case our result also depends on the trajectory of the observed car. 
