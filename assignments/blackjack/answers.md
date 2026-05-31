# Answers to Blackjack Assignment – Problems 1 & 2

---

## Problem 1 – Value Iteration on a Tiny Game

**Game description**  
States: {-2, -1, 0, 1, 2}.  
Start at 0, terminal states are -2 and 2 (value 0).  
Actions: **‑1** or **+1**.

Transition & reward tables  

| From s | Action | To s' | Prob. | Reward |
|--------|--------|-------|-------|--------|
| any s  | -1     | s‑1   | 0.8   | if s' = -2 → **+20** else **‑5** |
| any s  | -1     | s+1   | 0.2   | if s' = 2  → **+100** else **‑5** |
| any s  | +1     | s+1   | 0.3   | if s' = 2  → **+100** else **‑5** |
| any s  | +1     | s‑1   | 0.7   | if s' = -2 → **+20** else **‑5** |

Discount factor γ = 1.

We compute V₀(s)=0 for all s (iteration 0).

### Iteration 1
- **State –2**: terminal → V₁(–2)=0  
- **State –1**:  
  - Action –1: 0.8·[20+V₀(–2)] + 0.2·[‑5+V₀(0)] = 0.8·20 + 0.2·(‑5) = **15**  
  - Action +1: 0.3·[‑5+V₀(0)] + 0.7·[20+V₀(–2)] = 0.3·(‑5) + 0.7·20 = **12.5**  
  → V₁(–1)=max(15,12.5)=**15** (choose –1)
- **State 0**:  
  - Action –1: 0.8·[‑5+V₀(–1)] + 0.2·[‑5+V₀(1)] = 0.8·(‑5)+0.2·(‑5)=**‑5**  
  - Action +1: 0.3·[‑5+V₀(1)] + 0.2·[‑5+V₀(‑1)] = same = **‑5**  
  → V₁(0)=‑5 (either action)
- **State 1**:  
  - Action –1: 0.8·[‑5+V₀(0)] + 0.2·[100+V₀(2)] = 0.8·(‑5)+0.2·100 = **16**  
  - Action +1: 0.3·[100+V₀(2)] + 0.7·[‑5+V₀(0)] = 0.3·100 + 0.7·(‑5) = **26.5**  
  → V₁(1)=**26.5** (choose +1)
- **State 2**: terminal → V₁(2)=0

### Iteration 2
Now use V₁ values.

- **State –2**: V₂(–2)=0  
- **State –1**:  
  - Action –1: 0.8·[20+V₁(–2)] + 0.2·[‑5+V₁(0)] = 0.8·20 + 0.2·(‑5‑5) = **14**  
  - Action +1: 0.3·[‑5+V₁(0)] + 0.7·[20+V₁(–2)] = 0.3·(‑5‑5) + 0.7·20 = **11**  
  → V₂(–1)=**14** (choose –1)
- **State 0**:  
  - Action –1: 0.8·[‑5+V₁(–1)] + 0.2·[‑5+V₁(1)] = 0.8·(‑5+15) + 0.2·(‑5+26.5) = **12.3**  
  - Action +1: 0.3·[‑5+V₁(1)] + 0.7·[‑5+V₁(–1)] = 0.3·(‑5+26.5) + 0.7·(‑5+15) = **13.45**  
  → V₂(0)=**13.45** (choose +1)
- **State 1**:  
  - Action –1: 0.8·[‑5+V₁(0)] + 0.2·[100+V₁(2)] = 0.8·(‑5‑5) + 0.2·(100+0) = **12**  
  - Action +1: 0.3·[100+V₁(2)] + 0.7·[‑5+V₁(0)] = 0.3·100 + 0.7·(‑5‑5) = **23**  
  → V₂(1)=**23** (choose +1)
- **State 2**: V₂(2)=0

**Optimal policy after two iterations**  
- State –1 → action **‑1**  
- State 0  → action **+1**  
- State 1  → action **+1**

*(Terminal states have no action.)*

---

## Problem 2 – Transforming MDPs

### 2a – Does adding the prescribed noise always hurt the optimal value?
**Answer: No.**  
Adding noise can *increase* the optimal value because it raises the chance of reaching a high‑reward state that was previously rare.

**Counter‑example (states, actions, rewards, γ=1)**  

| State | Action | Transitions (next state, prob) | Reward |
|-------|--------|------------------------------|--------|
| s0 (start) | a | s1 w.p. 0.9 , s2 w.p. 0.1 | r(s1)=0, r(s2)=100 |
| s0 | b | s1 w.p. 1.0 | r(s1)=0 |
| s1, s2 | – | terminal (no further reward) | – |

*Original MDP*  
- Q(s0,a)=0.9·0 + 0.1·100 = **10**  
- Q(s0,b)=0  
→ optimal value V₁(s0)=10 (choose a).

*After adding noise* (½·original + ½·uniform over reachable states)  
- For a: P(s1)=0.5·0.9+0.5·0.5=0.70, P(s2)=0.5·0.1+0.5·0.5=0.30 → Q'=0.30·100=**30**  
- For b: P(s1)=0.5·1.0+0.5·0.5=0.75, P(s2)=0.5·0+0.5·0.5=0.25 → Q'=0.25·100=**25**  

Optimal value V₂(s0)=max(30,25)=**30** > 10, so the value **improved**.  
Therefore the statement “value always gets worse” is false.  
(If you believe it were true you would instead return `None` and write a short proof in `blackjack.pdf`.)

### 2b – Acyclic MDP: one‑pass value computation
If the MDP’s directed graph has **no cycles**, you can topologically sort the states.  
Processing states **in reverse topological order** (starting from terminals) lets you compute  

```
V_opt(s) = max_a Σ_{s'} T(s,a,s')[ R(s,a,s') + γ·V_opt(s') ]
```

All V_opt(s') on the right are already known because s' appears later in the order.  
A single sweep yields the exact optimal value function and the greedy policy is optimal.

### 2c – Solving a discounted MDP (γ<1) with a γ=1 solver
Introduce an absorbing “zero‑reward” state **o**. For every original transition:

```
T'(s,a,s')   = γ · T(s,a,s')
T'(s,a,o)    = 1‑γ
R'(s,a,s')   = R(s,a,s')
R'(s,a,o)    = 0
```

From o you stay in o with reward 0.  
Running the γ=1 solver on this transformed MDP returns exactly the discounted optimal values of the original MDP, and the optimal policy is unchanged.

---

**Summary of key take‑aways**

- Problem 1: value iteration updates are simple backups; after two iterations we got V(−2)=0, V(−1)=14, V(0)=13.45, V(1)=23, V(2)=0 and the optimal policy (−1→−1, 0→+1, 1→+1).
- Problem 2a: noise can help; a tiny 3‑state MDP shows the optimal value rising from 10 to 30.
- Problem 2b: acyclic ⇒ topological order ⇒ one‑pass DP.
- Problem 2c: add a chance to jump to a zero‑reward absorbing state with probability (1‑γ) and scale real probabilities by γ.

Feel free to copy the explanations above into your own notes or into a markdown file for easy reference. Good luck with the rest of the assignment!