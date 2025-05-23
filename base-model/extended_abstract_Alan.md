# Exploring Effects of Consumer Finance Regulation with Deep Learning

Consumer finance protection in the United States has opened a new chapter after the Trump administration has dismantled much of the Consumer Finance Protection Bureau (CFPB), the federal agency with the strongest mandate for enforcing it. 

The United States now has an evolving patchwork of state and federal consumer finance laws and rules. Lawmakers have many potential policy levers available to them when regulating consumer credit, with many potential policy goals for these rules. We leverage recent advances in deep learning for solving dynamic economic models (@Maliar2021) to test the impact of a variety of policy interventions on policy outcomes.

Our method compares multiple models, each representing a choice of public policy and social scenario. We compare the outcomes of these models as a way of testing the theoretical causal impact of policy choices. To support this method, we adapt the deep learning techniques from @Maliar2021. We define models as systems of structural equations characterizing dynamic interactions between consumers and a strategic lender. We generate objective functions based on model residuals (specifically, Bellman residuals) programmatically, applying the All-in-One (AiO) expectation operator for efficient integration, and use these loss functions to find optimal decision policies and ergodic distributions via deep learning. We extend @Maliar2021 by solving for a strategic Nash equilibrium with two optimizing agents (consumers and the lender) using an alternating optimization procedure (stochastic fictitious play).


# Background

Our project builds on prior work in heterogeneous agent modeling (@Huggett1993) of consumer credit ([Chatterjee et al., 2007](https://doi.org/10.1111/j.1468-0262.2007.00806.x), [Chatterjee et al., 2023](https://doi.org/10.3982/ECTA18771)) and agent-based modeling ([Papadopoulos, 2019](https://doi.org/10.1016/j.jedc.2019.05.002)).

We are particularly interested in the way outcomes differ for consumers that vary based on a protected attribute, such as race, immigration status, gender, or age, and in how personal data and machine learning is used in lending decisions and credit scoring ([Fuster et al., 2021](https://doi.org/10.1111/jofi.13090)). Of particular interest in consumer protection regulation is whether and how a lending policy can be minimally discriminatory while serving the business needs of the lender ([Laufer et al., 2025](https://doi.org/10.1145/3709025.3712214)). Prior work in computer science, where these issues are more frequently addressed, has identified many nuances of "fairness" through the use of structural causal models (SCM), in which the topology of structural equations is carefully analyzed to identify pathways through which protected attributes can affect outcomes ([Madras et al., 2019](https://doi.org/10.1145/3287560.3287564), [Mhasawade and Chunara, 2021](https://doi.org/10.1145/3461702.3462587)).

Typically, the computer science literature does not consider dynamic or equilibrium outcomes, though there are examples of extensions of SCM to dynamic systems with choice ([Creager et al., 2020](https://dl.acm.org/doi/10.5555/3524938.3525142)) and multi-agent systems ([Lazri et al., 2025](https://doi.org/10.48550/arXiv.2502.18534)). These frameworks resemble those in economics, but deep learning techniques such as @Maliar2021 have not been fully applied to these strategic, fairness-aware settings.

# Methodology

We build and compare models sharing common properties. Key notation is in the Appendix. We adopt the convention that lowercase variables refer to individual consumers.

At the start of period $t$, the consumer observes their state $x_t = (a_t, h_t)$ (carried over from the previous period) and the realization of the exogenous shock $z_t$ (which includes earnings). Given this information $(x_t, z_t)$, the consumer chooses an action $y_t=(c_t, d_t)$ based on the anticipated asset price $q_t$, following their policy rule $y_t = \sigma(x_t, z_t; q_t, \theta)$ (approximated by a DNN with parameters $\theta$). Simultaneously, the lender sets the asset price $q_t$ based on the consumer's state and action, following their pricing rule $q_t = \rho(x_t, y_t, z_t; \phi)$ (approximated by a DNN with parameters $\phi$). The inverse of this price, $R_t = 1/q_t$, represents the gross interest rate effectively charged to the consumer, which incorporates any risk premium determined by the lender's rule. The equilibrium is characterized by the pair of policy functions $(\sigma^*(\cdot; \theta^*), \rho^*(\cdot; \phi^*))$ such that for any given state $(x_t, z_t)$, the resulting action $y^*_t$ and price $q^*_t$ simultaneously satisfy $y^*_t = \sigma^*(x_t, z_t; q^*_t, \theta^*)$ and $q^*_t = \rho^*(x_t, y^*_t, z_t; \phi^*)$. In a stationary equilibrium, these policy functions (the rules mapping states and other inputs to actions or prices) are time-invariant, even though the specific actions and prices experienced by an individual agent evolve with their state.

There is a population of $N$ consumers and a single lender agent. We analyze the interaction in a setting with an exogenous risk-free rate $r^*$, focusing on the credit market interaction.

Each consumer enters period $t$ in state $x_t$ and encounters shocks $z_t$. The subsequent action $y_t$ and asset price $q_t$ are determined simultaneously by the equilibrium conditions $y_t = \sigma(x_t, z_t; q_t, \theta)$ and $q_t = \rho(x_t, y_t, z_t; \phi)$. Consumers derive utility from consumption $u(c_t)$. The state transitions according to $x_{t+1} = T(x_t, y_t, z_t; q_t)$, where $(y_t, q_t)$ are the equilibrium values. They anticipate future lifetime value discounted by $\beta$.

The asset transition follows the budget constraint:
$$ a_{t+1} = (1-d_t) (1/q_t) [ a_t + e(z_t) - c_t ] $$
where $a_t$ represents net financial assets (debt if $a_t<0$). If $d_t=1$ (default), next-period assets $a_{t+1}$ are set to zero. The transition for $h_{t+1}$ would typically depend on $x_t, y_t, z_t$ (e.g., improving with repayment, worsening with default or high utilization): $h_{t+1} = T_h(x_t, y_t, z_t; q_t)$.

The lender is a strategic actor choosing the optimal pricing rule $q_t = \rho(x_t, y_t, z_t; \phi)$. The lender anticipates that for any $(x_t, z_t)$, the equilibrium action $y_t$ and price $q_t$ will satisfy both $y_t = \sigma(x_t, z_t; q_t, \theta)$ and the chosen rule $q_t = \rho(x_t, y_t, z_t; \phi)$. We use a deep learning framework based on @Maliar2021 to find the strategic Nash equilibrium policy parameters $(\theta^*, \phi^*)$ via an alternating optimization scheme.

## The consumer's problem

After the state $x_t$ is known and the shock $z_t$ is realized, the consumer chooses actions $y_t = (c_t, d_t)$ to maximize expected lifetime utility. The consumer anticipates that their chosen action $y_t$ will influence the asset price $q_t$ they face via the lender's rule $q_t = \rho(x_t, y_t, z_t; \phi)$, and their optimal action must be consistent with this price, $y_t = \sigma(x_t, z_t; q_t, \theta)$. Let $q^*_t$ denote the equilibrium asset price resulting from this simultaneous interaction for the given $(x_t, z_t)$. The value function $V(x_t, z_t; q^*_t)$ satisfies the Bellman equation:
$$ V(x_t, z_t; q^*_t) = \max_{y_t=(c_t, d_t)} \left\{ u(c_t) + \beta \mathbb{E}_{z_{t+1}} [V(x_{t+1}, z_{t+1}; q^*_{t+1})] \right\} \label{eq:consumer-bellman} $$
where $(y^*_t, q^*_t)$ implicitly solve the simultaneous equations $y_t = \sigma(x_t, z_t; q_t, \theta)$ and $q_t = \rho(x_t, y_t, z_t; \phi)$. $x_{t+1} = T(x_t, y^*_t, z_t; q^*_t)$. The expectation $\mathbb{E}_{z_{t+1}}$ is over the next period's shock, which determines the next period's equilibrium rate $q^*_{t+1}$ along with $x_{t+1}$. The optimal policy function $\sigma^*$ implicitly defines the consumer side of the equilibrium.

## The lender's problem

The lender can borrow at $r^*$ and maximizes expected profit by choosing the optimal asset price $q_t$ for each state, anticipating the resulting equilibrium interaction with the consumer. The lender anticipates that for any $(x_t, z_t)$, the chosen price $q_t$ and the resulting equilibrium action $y_t$ will satisfy $y_t = \sigma(x_t, z_t; q_t, \theta)$.

Let $V_l(\mu_t)$ be the lender's value function, where $\mu_t$ is the joint distribution of states and shocks $(x_t, z_t)$. The lender chooses the state-contingent price $q_t$ to solve:
$$ V_l(\mu_t) = \max_{q_t} \left\{ \int \pi(x_t, y^*_t, z_t; q_t, r^*) d\mu_t(x_t, z_t) + \delta \mathbb{E}[V_l(\mu_{t+1})] \right\} \label{eq:lender-problem} $$
where $(y^*_t)$ is the equilibrium action resulting from the consumer policy $y_t = \sigma(x_t, z_t; q_t, \theta)$ given the chosen $q_t$, and the expectation $\mathbb{E}[V_l(\mu_{t+1})]$ is taken over the evolution of the joint distribution $\mu_{t+1}$. Computationally, finding the optimal state-contingent prices involves determining the lender's optimal policy $\rho(x_t, y_t, z_t; \phi)$. This is achieved by minimizing the Bellman residual loss associated with Eq. {eq}`eq:lender-problem`, where the integral over $\mu_t$ is approximated by averaging over the $N$ simulated agents, potentially using techniques like the All-in-One (AiO) expectation operator (@Maliar2021) for efficiency.

The *ex-post* profit $\pi$ from the lender's net position with the consumer is:
$$ \pi(x_t, y_t, z_t; q_t, r^*) = [ (1-d_t)/q_t - (1+r^*) ] (-a_t) $$where $d_t \in \{0, 1\}$ is the default decision (with $d_t=0$ assumed if $a_t \ge 0$).

The lender's pricing rule $\rho(x_t, y_t, z_t; \phi)$ must operate in concert with the consumer's policy $\sigma(x_t, z_t; q_t, \theta)$ to form the equilibrium.

*Alternative Lender Objective: Zero-Profit Equilibrium:* In some scenarios, competitive pressure forces expected profits to zero. In the static case ($\delta=0$), this implies the lender sets the asset price $q_t$ for each specific transaction profile $(x_t, y_t, z_t)$ such that the expected profit *from that transaction type* is zero. This requires the price to reflect the risk-free rate and the probability of default *next* period conditional on information available after decisions at time $t$. Since the price $q_t$ influences the future state $x_{t+1}$ (which determines $d_{t+1}$), $q_t$ itself is part of the relevant information set, leading to the following implicit equilibrium condition for the price $q_t^*$ associated with that transaction profile:
$$ q_t^*(x_t, y_t, z_t) = \frac{1 - P(d_{t+1}=1 | x_t, y_t, z_t, q_t^*)}{1+r^*} $$where $P(d_{t+1}=1 | x_t, y_t, z_t, q_t^*)$ is the probability, assessed at time $t$, that the consumer defaults at time $t+1$ given state, action, and the equilibrium price $q_t^*$. This equation defines $q_t^*$ as a fixed point for each $(x_t, y_t, z_t)$.

*Continuation Value with Aggregate States:* When $\delta > 0$, the lender's value function $V_l(\mu_t)$ in Eq. {eq}`eq:lender-problem` (where $\mu_t$ is the high-dimensional joint distribution over $(x_t, z_t)$) is intractable. Following @Maliar2021, we can approximate this in two main ways. First, as described below, we can assume the lender conditions their choice of optimal price $q_t$ on a vector $\mathcal{M}_t$ containing low-order moments of the joint distribution $\mu_t$. The approximated value function conditional on these moments is:
$$V_l(\mathcal{M}_t) \approx \max_{q_t} \left\{ \mathbb{E}_{x_t, z_t|\mathcal{M}_t}[ \pi(x_t, y^*_t, z_t; q_t, r^*) ] + \delta \mathbb{E}_{Z_t}[V_l(\mathcal{M}_{t+1})] \right\} $$
where the optimal state-contingent price $q_t$ (and the resulting equilibrium action $y^*_t$) depends implicitly on $\mathcal{M}_t$. Computationally, this optimal pricing strategy based on moments is implemented by finding the parameters $\phi$ of the lender's policy network $\rho(x_t, y_t, z_t; \phi)$, which may take $\mathcal{M}_t$ as input. The expectation $\mathbb{E}_{x_t, z_t|\mathcal{M}_t}$ denotes averaging over the micro states consistent with the aggregate moments $\mathcal{M}_t$. Alternatively, following @Maliar2021's baseline approach for heterogeneous agent models, the lender's policy network $\rho(\cdot; \phi)$ could take the specific agent's information $(x_t, y_t, z_t)$ along with the full panel state information (representing $\mu_t$, e.g., $(x_{1t}, z_{1t}, \dots, x_{Nt}, z_{Nt})$) as direct, high-dimensional input. The deep learning architecture is then tasked with performing optimal model reduction internally, learning to condense the panel information into the features necessary for pricing.

*Equilibrium Existence and Uniqueness:* The existence and uniqueness of a strategic Nash equilibrium $(\sigma(\dots; \theta^*), \rho(\dots; \phi^*))$ in this two-agent game, particularly with discrete choices and potentially non-convexities, are not guaranteed a priori. Standard fixed-point theorems may not apply directly. The full paper will discuss relevant theoretical considerations, potential sources of multiplicity, and the implications for interpreting the computationally obtained results, including whether the alternating optimization algorithm converges and to which equilibrium if multiple exist. Benchmarking against simpler cases with known properties (e.g., a toy model with a single representative borrower and quadratic utility where the equilibrium is known) will be crucial for validating the solver's accuracy.

We propose solving for this strategic Nash equilibrium computationally using an alternating optimization scheme based on deep learning, as detailed in the Appendix.

## Specific Model Variations

We explore different model configurations by varying assumptions about the lender.

### Lender Type Characteristics

Different types of lenders might be modeled by adjusting their information set or the functional form/constraints on their pricing rule $\rho(x_t, y_t, z_t; \phi)$:

*   **Payday Lender Analogue:** Might be modeled with a pricing rule $\rho(x_t, y_t, z_t; \phi)$ that places less weight on $h_t$ or $y_t$.
*   **Fintech Lender Analogue:** Typically aims to maximize profits (see below) and might incorporate alternative data sources, potentially expanding the state vector $x_t$ or influencing the shock process $z_t$ in more complex model variants.

### Lender Objective Formulation

We analyze scenarios based on different lender objectives:

*   **Short-term Profit Maximizing:** Corresponds to $\delta = 0$ in Eq. {eq}`eq:lender-problem`. The lender chooses state-contingent prices $q_t$ (implemented via the policy $\rho(\cdot; \phi)$) to maximize the expected aggregate single-period profit, typically approximated as $\frac{1}{N}\sum_{i=1}^N \pi(x_{it}, y^*_{it}, z_{it}; q^*_{it}, r^*)$, where $(y^*_{it}, q^*_{it})$ solve the simultaneous equations for each agent $i$. Computationally, this involves optimizing $\phi$.
*   **Zero-Profit Equilibrium:** Imposes zero expected profit in equilibrium. In the static case, this implies the price for each transaction profile $(x_t, y_t, z_t)$ satisfies $q_t^* = (1 - P(d_{t+1}=1 | x_t, y_t, z_t, q_t^*))/(1+r^*)$, reflecting expected next-period default risk for that profile.
*   **Dynamic Profit Maximizing (with Continuation Value):** Solves the full dynamic problem Eq. {eq}`eq:lender-problem` (or its moment-based approximation) for $\delta > 0$.

These variations allow us to study how different market structures and lender motivations interact with consumer behavior and regulatory policies. Comparing outcomes across these model variations allows us to test hypotheses regarding the impact of market structure, lender objectives, and regulatory policies on credit access, pricing, and fairness.

# Data

(This section will describe the data sources used for calibration of shock processes $z_t$, earnings $e(z_t)$, utility parameters $\beta, \gamma$, and lender parameters $\delta, r^*$. If purely simulation-based, this will be stated and justified. Potential sources include survey data (SCF), credit panel data (CCP), or supervisory data (Y-14M).)

# Key Hypotheses

(This section will list the main hypotheses regarding the differential impact of regulations on outcomes like default rates, credit access, loan pricing ($q_t$), welfare distribution (using $V(x_t)$), and fairness metrics across different consumer groups and lender types.)

# Appendix

```{list-table} Notation
:header-rows: 1
:label: notation-table

* - Symbol
  - Domain/Type
  - Explanation
* - $N$
  - $\mathbb{N}^+$
  - Number of consumers.
* - $a_t$
  - $\mathbb{R}; a_t \ge \underline{a}$
  - Individual consumer net financial assets at start of period $t$ (state).
* - $h_t$
  - $[0, 1]$
  - Individual consumer credit history score at start of period $t$ (state).
* - $x_t = (a_t, h_t)$
  - $\mathbb{R} \times [0, 1]; x_t \ge (\underline{a}, 0)$
  - Individual consumer state vector.
* - $q_t$
  - $\mathbb{R}^+$
  - Price at time $t$ of one unit of assets delivered at $t+1$. This is the inverse of the gross interest rate ($q_t = 1/R_t$), incorporating a risk premium (i.e., $q_t < 1/(1+r^*)$ if default risk exists). Set by lender rule $\rho(x_t, y_t, z_t; \phi)$.
* - $c_t$
  - $\mathbb{R}^+$
  - Individual consumption in period $t$ (action).
* - $d_t$
  - $\{0, 1\}$
  - Individual default decision in period $t$ (action).
* - $y_t = (c_t, d_t)$
  - $\mathbb{R}^+ \times \{0,1\}$
  - Individual consumer action vector.
* - $z_t$
  - (Varies)
  - Vector of individual consumer shocks realized in period $t$.
* - $e(z_t)$
  - $\mathbb{R}$
  - Earnings component derived from shocks $z_t$.
* - $\mu_t$
  - $\Delta(X \times Z)$
  - Joint distribution of consumers over endogenous states $x_t \in X$ and exogenous shocks $z_t \in Z$.
* - $u(.)$
  - Function
  - Consumer single period utility function (e.g., CRRA over $c_t$).
* - $\pi(...)$
  - Function
  - Lender single-period profit function from net position with consumer.
* - $T(...)$
  - Function
  - Individual consumer state transition function for $(a_{t+1}, h_{t+1})$.
* - $\sigma(...; \theta)$
  - Function (DNN)
  - Consumer policy rule mapping state $(x_t, z_t)$ and price $q_t$ to action $y_t$: $y_t = \sigma(x_t, z_t; q_t, \theta)$.
* - $\rho(...; \phi)$
  - Function (DNN)
  - Lender policy (pricing schedule) mapping $(x_t, y_t, z_t)$ to $q_t$.
* - $\theta$
  - $\mathbb{R}^k$
  - Parameters (weights, biases) of the consumer policy DNN $\sigma$.
* - $\phi$
  - $\mathbb{R}^m$
  - Parameters (weights, biases) of the lender policy DNN $\rho$.
* - $\beta$
  - $(0, 1)$
  - Consumer discount factor.
* - $\delta$
  - $[0, 1)$
  - Lender discount factor.
* - $r^*$
  - $\mathbb{R}^+$
  - Risk-free interest rate.
* - $\Gamma(...)$
  - Set
  - Feasible action set for consumer, potentially implicitly defined by existence of equilibrium with $\rho, \sigma$.
* - $\Xi_c, \Xi_l$
  - Function
  - Theoretical Loss functions (based on Bellman residuals).
* - $\hat{\Xi}_c, \hat{\Xi}_l$
  - $\mathbb{R}^+$
  - Empirical Loss counterparts (sample average of residuals).
* - $\mathcal{M}_t$
  - $\mathbb{R}^p$
  - Vector of moments (e.g., means, variances, covariances) of the joint distribution $\mu_t$ over $(x_t, z_t)$.
* - $V(.)$
  - Function
  - Value function (consumer $V(x_t, z_t; q^*_t)$ or lender $V(\mu_t)$).
```

## Solution Algorithm

:::{prf:algorithm} Solution procedure (Conceptual Outline - Stochastic Fictitious Play)
:label: my-algorithm

**Objective:** Find parameters $(\theta^*, \phi^*)$ for the consumer policy network $\sigma(...; \theta)$ and lender policy network $\rho(...; \phi)$ that approximate a strategic Nash equilibrium, considering the within-period simultaneity.

**Method:** We adapt the deep learning approach of @Maliar2021 using an alternating optimization scheme. A key challenge is handling the simultaneous determination of $(y_t, q_t)$ within each step.

1.  **Initialization:**
    *   Initialize DNN parameters $\theta^{(0)}, \phi^{(0)}$ for policy networks $\sigma$ and $\rho$. Initialize value function networks $V_c, V_l$.
    *   Define theoretical loss functions $\Xi_c, \Xi_l$ based on Bellman equations ({eq}`eq:consumer-bellman`, {eq}`eq:lender-problem`).
    *   Initialize experience replay buffer $\mathcal{D}$.
    *   Set optimization hyperparameters. Define a method for finding the within-period equilibrium $(y_t, q_t)$ (e.g., inner fixed-point iteration, relaxation).

2.  **Iteration Loop (Epochs $k=0, 1, \dots$):**

        1.  **Simulate:** Generate trajectories using current policies $(\sigma(\cdot; \theta^{(k)}), \rho(\cdot; \phi^{(k)}))$ for $S$ steps. For each step $t$: Given $(x_t, z_t)$, determine action $y_t$ using $y_t \approx \sigma(x_t, z_t; q_t, \theta^{(k)})$ and price $q_t$ using $q_t \approx \rho(x_t, y_t, z_t; \phi^{(k)})$, reflecting the interaction based on current policies (precise handling depends on implementation, see Note). Then calculate $x_{t+1} = T(x_t, y_t, z_t; q_t)$. Store transitions $(x_t, z_t, y_t, q_t, x_{t+1})$ in $\mathcal{D}$.
    
        2.  **Optimize Consumer Policy $\theta$ (given fixed $\phi^{(k)}$):**
            1.  For $N_c$ steps: Sample a mini-batch of $(x_j, z_j)$ states from $\mathcal{D}$. Draw required future shocks.
            2.  Compute empirical risk $\hat{\Xi}_c(\theta^{(k)}, \phi^{(k)})$ based on {eq}`eq:consumer-bellman`. This requires evaluating the Bellman residual using the consumer's policy $\sigma(\cdot; \theta^{(k)})$ and the lender's *fixed* policy $\rho(\cdot; \phi^{(k)})$ to determine the relevant $(y_j, q_j)$ for the sampled state $(x_j, z_j)$. The expectation term $\mathbb{E}[V(\dots)]$ is evaluated using the current consumer value network $V_c$, potentially employing techniques like the AiO operator (@Maliar2021) for efficiency. Use Gumbel-Softmax relaxation for $y_j$ if needed.
            3.  Calculate gradient $\nabla_\theta \hat{\Xi}_c$. Gradient calculation needs to account for the dependence of $y_j$ on $\theta$ (directly via $\sigma$).
            4.  Update $\theta$ towards $\theta^{(k+1)}$.
    
        3.  **Optimize Lender Policy $\phi$ (given fixed $\theta^{(k+1)}$):**
            1.  For $N_l$ steps: Sample a mini-batch of $(x_j, z_j)$ states from $\mathcal{D}$. Draw required future shocks.
            2.  Compute empirical risk $\hat{\Xi}_l(\theta^{(k+1)}, \phi^{(k)})$ based on {eq}`eq:lender-problem`. Requires evaluating the lender Bellman residual using the lender's policy $\rho(\cdot; \phi^{(k)})$ and the consumer's *fixed* policy $\sigma(\cdot; \theta^{(k+1)})$ to determine the relevant $(y_j, q_j)$ for the sampled state $(x_j, z_j)$. The expectation term $\mathbb{E}[V_l(\mu_{t+1})]$ is evaluated using the current lender value network $V_l$ (or its moment-based approximation $V_l(\mathcal{M}_{t+1})$), potentially using the AiO operator (@Maliar2021) for efficiency.
            3.  Calculate gradient $\nabla_\phi \hat{\Xi}_l$. Gradient calculation needs to account for the dependence of $q_j$ on $\phi$ (directly via $\rho$).
            4.  Update $\phi$ towards $\phi^{(k+1)}$.
    
        4.  **Check Convergence:** Evaluate convergence criteria.

3.  **Output:** The converged parameters $(\theta^*, \phi^*)$ approximate the equilibrium decision rules $(\sigma(...; \theta^*), \rho(...; \phi^*))$.

*Note:* A key challenge is handling the within-period interaction where $y_t$ depends on $q_t$ via $\sigma$ and $q_t$ depends on $y_t$ via $\rho$. While the description above implies using the opponent's fixed policy within each optimization step (consistent with alternating optimization / fictitious play), some implementations might explicitly solve the inner fixed-point $y_t = \sigma(x_t, z_t; q_t, \theta^{(k)})$, $q_t = \rho(x_t, y_t, z_t; \phi^{(k)})$ at each simulation/evaluation step. This adds computational cost and complexity, particularly for gradient calculation (requiring techniques like the implicit function theorem or differentiating the solver). The convergence properties and efficiency may differ between these approaches.
:::
