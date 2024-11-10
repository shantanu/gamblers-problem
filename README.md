# How to Run the Notebook

1. In your terminal:

```nasm
docker compose up
```

1. In your web browser, open `localhost:8888` 
2. Enter the token `my-token` 
3. Navigate into the `work/` folder, and open the `gamblers-problem.ipynb` notebook in the JupyterLab environment. 

[Link to Notion blog post](https://harmless-resistance-e28.notion.site/Gambler-s-Problem-1334b68d0c4080c58e2aff68cd2f9c50)

## Problem Statement

A gambler has the opportunity to make bets on the outcomes of a sequence of coin flips, with $p_h$ probability of flipping heads independently.

- If the coin is heads, he wins as many dollars as he staked on that flip. If it's tails, he loses his stake.
- The game ends when the gambler wins by reaching his goal of $100, or loses by running out of money.
- On each flip, the gabler must decide what portion of his capital to take, in integer number of dollars.

# Results

- At $p_h=0.25$, a deeper look into the policy and the convergence of the value function over iterations of Value Iteration. With the coin biased against you, make larger bets that are more likely to make you win instantly.

![ph=0.25](ph0.25.png)

- At $p_h=0.55$, The shape of the value function and the optimal policy change drastically. With the coin biased in your favor, making small bets and slowing making your way to the win becomes optimal.

![ph=0.55](ph0.55.png)

- Left plot shows optimal bet sizing for various coin flip probabilities. Right plot shows value functions for same coin flip probabilities. There are basically two forms of optimal policy: the spiky one for $p_h < 0.5$, and the flat one for $p_h>0.5$.

![parameter study](parameter-study.png)

- Indifferent case with $p_h=0.5$: This is just one of the optimal policies. In fact, making any bet size is optimal with the coin is perfectly fair.

![ph=0.5](ph0.5.png)

## Formulation to MDP:

- Undiscounted, episodic, finite MDP.
- State: gambler's capital, $s \in \{1, 2, ..., 99\}$
- Actions: stakes, $a \in \{0, 1, ..., min(s, 100-s)\}$
- Rewards: 0 on all transitions except when gambler reaches his goal of $100.
- The state-value function given the probability of winning from each state.
- A policy is a mapping from levels of capital to stakes. The optimal policy maximizes the probability of winning from each state.

## Solution via Value Iteration

Value iteration is an instance of Generalized Policy Iteration (GPI). Until convergence of the optimal policy, you sweep the state space making updates according to the Bellman Optimality Equation:

$$
v_{k+1}(s) = \max_a \sum_{s',r} p(s', r | s, a) [r + \gamma v_k(s')]
$$

In this problem's case, the sum simplifies based on how much is at stake. There are only 2 possible ${s', r}$ pairs, one for the case that the coin flips Heads, the other Tails. The action space is defined such that the agent cannot stake more than the capital available, or more than what would make the total on a heads flip equal 100. Note that the reward for any tails coin flip is always 0. The reward for a heads coin flip is 1 only in the case when $s+a = 100$

Note that for an episodic task, the discount factor $\gamma = 0$. As we increase the goal beyond 100, loops can be more easily introduced in the iteration and the discount factor helps avoid them.

$$
v_{k+1}(s) = \max_a p_h[\mathbb{1}_{s+a=100} + \gamma v_k(s+a)] + (1-p_h)\gamma v_k(s-a)
$$

After the value function has converged, we can determine a policy $\pi \approx \pi_*$ such that:

$$
 \pi(s) = \underset{a}{\operatorname{argmax}} p_h[\mathbb{1}_{s+a=100} + \gamma v(s+a)] + (1-p_h)\gamma v(s-a)
$$

The value iteration algorithm is given below in pseudocode. Note that the policy computation only happens after the value function has approximately converged, but we can think of the value function update using Bellman's Optimality Equation as doing one round of greedy action selection while doing the value update:

```
Value Iteration:
    initialize V(s) = 0 for all s in S

    while True:
        delta = 0
        for s in S:
            v = V(s)
            V(s) = updated as above
            delta = max(delta, |v-V(s|)
        if delta < threshold:
            break

    Compute a policy according to the greedy function above.

```

## Outputs

- Observe value function convergence after successive state sweeps of Value Iteration.
- Shape of optimal policy, and why it looks a certain way
- Value functions and optimal policies for different values of $p_h$

## Notes

- Selecting the optimal policy function:
    - floating-point error can mess up your `argmax`. To avoid floating point errors, do a fuzzy comparison between value estimates by re-using the same threshold for value convergence and finding all values that are within that threshold to be considered equal.
    - Tendency to pick action 0 - for values of $p_h < 0.5$, doing nothing is considered one of the "optimal" actions because the coin is biased against you and you'll lose more often than you win. So if you have some capital you might as well keep it. Since this is not interesting, I forced the agent to pick a non-zero action in these cases.
    - There are multiple optimal policies. All of them have a sawtooth/triangle shape. The way to visualize them is by choosing different tie-breaking strategies.
- When $p_h > 0.5$, the shape of the value function changes drastically. It favors the optimal policy essentially just betting 1 every time.
    - At exactly $p_h=0.5$, bet sizing doesn't matter. All bets are made equal.
    - At $p_h<0.5$, you get spikes of the value function at spots where you can quickly win, so the optimal policy is essentially trying to get you to those spikes. Optimal policies are the same for all $p_h < 0.5$.