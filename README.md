# Group Optimization Project

This paper explores the application of combinatorial optimization techniques to form optimal music artist collaborations under budget constraints. We evaluate four methods—brute force search, greedy algorithms, integer programming, and simulated annealing—to maximize group potential by analyzing artists’ skills, collaboration compatibility, and cost considerations. Using data from Spotify’s top 25 artists with the most monthly listeners, we demonstrate that simulated annealing achieves optimal results with significantly less computational expense than brute force methods, while greedy algorithms provide quick but suboptimal solutions. We compare two objective function formulations, L1 and L2 norms, to assess how they influence the composition and synergy of selected artist groups. Our findings show that the L2 formulation creates more synergistic groupings than the L1 formulation, providing better overall group selection by maximizing both individual skill contributions and collaborative potential, all while adhering to budget constraints.

## Introduction
In the world of modern music, many of the biggest artists today have achieved success as solo acts. However, an intriguing question arises when considering the potential benefits of collaboration: how can we combine some of these top artists into a group that maximizes profits, while adhering to a set budget? The objective is to find the best possible combination of artists that optimizes revenue generation by analyzing each artist’s skills, willingness to collaborate and the financial constraints they incur. By applying combinatorial optimization techniques, we can explore how strategic groupings of artists can lead to greater artistic and economic success.

## Definitions and Equations

**Given the variables:**

- `x_i`: decision variable denoting whether artist `i` is chosen  
- `c_i`: price of artist `i`; number of monthly listeners divided by 1000  
- `s_ik`: proficiency of artist `i` in skill `k`  
- `Z`: collaboration matrix, with `z_ij` = collaboration value between artist `i` and artist `j`  
- `m`: total number of artists  
- `o`: total number of skills  
- `b`: total budget available  
- `λ`: cohesion penalty strength; constant multiplier for penalty incurred by adding more artists to the group  

---

#### Objective Function (group score) – L1 norm formulation

**Maximize:**  

```math
\sum_{i=1}^{m} x_i (\sum_{k=1}^{o} |s_ik| + \sum_{j=i+1}^{m} x_j z_ij) - λ \sum_{i=1}^{m} x_i
```
#### Objective Function (group score) – L2 norm formulation

Maximize:
```math
\sum_{i=1}^{m} x_i (\sqrt(\sum_{k=1}^{o} s_ik^2) + \sum_{j=i+1}^{m} x_j z_ij) - λ * \sum_{i=1}^{m} x_i^2
```
Subject to
```math
\sum_{i=1}^{m} x_i c_i <= b
```
```math
z_ij >= x_i + x_j - 1
```
```math
\forall i, j \in {1,...,m}, i != j
```
Objective function components
Skill contribution:
L1 norm: 
```math
\sum_{i=1}^{m} x_i \sum_{k=1}^{o} |s_ik|
```
Represents the total skill value provided by selected artists.
L2 norm: 
```math
\sum_{i=1}^{m} x_i \sqrt(\sum_{k=1}^{o} s_ik^2)
```
Places a moderate premium on specialization while still valuing versatility.

Example:
Five artists with singing skills (7,7,7,7,7) → total score 15.65.
Group with specialized singing skills (10,10,10,0,0) → total score 17.32.
This controlled premium on specialization reflects the music industry's valuation of distinctive talents without excessively rewarding one-dimensional specialists. The L2 norm also provides diminishing returns when adding artists with similar skills, encouraging diversity in the optimal solution.

Collaboration value:
```math
\sum_{i=1}^{m} \sum_{j=i+1}^{m} x_i x_j z_ij
```
Captures the synergy between selected artists.

Cohesion penalty: 
```math
λ \sum_{i=1}^{m} x_i^2
```
Penalizes overly large groups through a quadratic term. The value of λ can be adjusted to control the penalty strength.


## Modeling Process
We decided to take the current 25 artists with the most monthly listeners on Spotify to model our problem. To quantify an artist's talents, we first defined 5 different types of skills, consisting of singing, dancing, rapping, composing, and producing. We assigned values to each artist on a scale of $1-10$ with 1 representing unfamiliarity of skill and 10 demonstrating a great specialty in the skill. These skill scores were generated using AI based on publicly available information about each artist's career and performances.

We also defined a collaboration matrix, evaluating the possibility of collaboration between each pairing of artists and used the upper triangle of the matrix in each of our methods to avoid double counting artist pairs. Similar to the skill value, the collaboration value is a value between $0-10$ with 0 meaning that the two artists were extremely unlikely to work together, and 10 meaning that the two artists were very likely to work together. These collaboration scores were also generated using AI to analyze factors such as musical style compatibility, past collaborations, and public statements about potential partnerships. While our data was AI-generated, this approach could easily be adjusted for real-world applications, where a company would likely have their own assessment of artists and more detailed data on collaboration potential.

In order to solve the problem of selecting the best combination of artists while considering budget constraints, collaboration likelihood, and skill distribution, we explored multiple combinatorial optimization techniques. The applied methodologies include brute force search, greedy algorithms, integer programming, and simulated annealing.

We developed two mathematical formulations of the objective function using different norm penalties: L1 and L2. The fundamental difference between these formulations lies in how penalties scale with group size. The L1 norm applies a linear penalty proportional to the number of artists, while the L2 norm (our primary approach) introduces a quadratic penalty that increases more rapidly as group size grows. Mathematically, this means the L2 formulation exhibits diminishing returns for each additional artist added, creating a natural balance point where the marginal benefit of adding another artist is exactly offset by the increased cohesion penalty. This property of L2 regularization encourages more balanced, synergistic groups by preventing the algorithm from simply selecting as many high-skill artists as the budget allows.

Implementation constraints with integer programming necessitated using only the L1 formulation for that method, which evolved into a valuable research opportunity. We implemented both formulations across our other optimization methods to provide comprehensive analysis of how mathematical modeling choices affect artist group composition and performance.


#### Brute Force Approach
A brute force approach systematically evaluates all possible artist groupings to determine the optimal combination. Given $m$ available artists, this method requires evaluating $2^m$ possible selections. While this guarantees the optimal solution, the exponential growth in search space makes it computationally infeasible for large values of $m$.

#### Greedy Algorithms
Greedy methods are computationally efficient but may result in suboptimal solutions due to their local decision-making nature. They work as incremental enhancements of the optimal solution by choosing the next best artist that improves the objective function while considering the cost constraints incurred by the chosen artist. The algorithms were programmed in Python, using helper functions to calculate the best score of each greedily chosen artist. If the group's value exceeded the best score of the selected heuristic, the artist was added to the group. The process would reach its conclusion if no new artist was picked for the group. We implemented two greedy heuristic approaches that use different choices to assign an optimal solution.

#### Maximizing Skill Contribution
Firstly, we considered creating a group of the most skilled artists that can be added with the available budget. At each step of the algorithm, the artist that adds the highest marginal increase in skill score is selected, subject to budget constraints.

#### Maximizing Efficiency
The previous greedy algorithm did not factor cost-effectiveness when the group could have a cheaper monetary value of comparative skill caliber based on the budget allocated. Hence, artists were ranked based on skill-to-cost ratio, prioritizing those that provide the highest skill contribution per unit cost.

#### Comparison of Greedy Heuristics
Comparing the two greedy algorithms, we found that maximizing the efficiency at each step produces significantly worse results than maximizing skill contribution. We believe this has to do with the inability of the second model to deal with the regularizer, as it tends to add in efficient low-cost artists during its first few iterations despite the quadratic regularizer favoring solutions with a smaller number of high-cost artists.

#### Simulated Annealing
Simulated annealing is a probabilistic optimization technique that allows exploration of the solution space while avoiding local optima. The algorithm iteratively modifies the artist selection, accepting new solutions based on the random addition or removal of an artist in the selected group. Expanding on the randomness factor, we accepted new solutions based on an energy function, which decayed over time by an exponential cooling schedule, to determine the new group. This energy function created either a random chance to choose artists that improved the optimality of the solution, or a small probability to instead decrease the optimality. As the algorithm continued to find the global maximum of the group's objective value function, the change in `temperature' of this function is slowly reduced to zero. Due to the method's random nature, we rerun the solver multiple times to ensure the convergence into a global optimal solution. This method results in an algorithmically efficient solution that is satisfactory with larger amounts of data. 

#### Integer Programming
We used Integer Programming to model the objective function and constraints for mathematical optimization using the PuLP library. While our primary approach employed an L2 norm formulation for other methods, the quadratic penalty term in the L2 formulation presented challenges for integer programming solvers, which typically handle linear constraints and objectives.
To accommodate these limitations, we reformulated our problem using an L1 norm approach for the integer programming implementation. We modeled the basic structure as a 0-1 Knapsack problem, where each artist represents an item with an associated cost. For added complexity, our inclusion of the collaboration matrix was leveraged to determine group compatibility.
While straightforward to implement in our other methods, the collaboration matrix introduced a non-linear relationship to the problem. To adhere to linear programming requirements, we introduced binary decision variables and additional constraints that ensured some artists would be selected together while others would not, based on their collaboration values. 
This modification allowed us to solve the problem using standard integer programming techniques while maintaining the essential structure of our optimization model. The necessity of this L1 reformulation for integer programming ultimately led to valuable comparative insights between L1 and L2 approaches, revealing how different penalty formulations fundamentally affect group composition and optimization outcomes


