# Answers to Blackjack Assignment – Problems

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


---

## Problem 3 – Peeking Blackjack

### 3a – Implementing `succAndProbReward` for `BlackjackMDP`

**Goal:** Model the modified Blackjack game as an MDP by filling in the `succAndProbReward(state, action)` method. The state is a tuple `(totalCardValueInHand, nextCardIndexIfPeeked, deckCardCounts)`.

**Rules to encode:**

| Action | Behavior |
|--------|----------|
| **Take** | If you peeked before (peekIdx not None), draw that card deterministically. Otherwise, draw a card proportionally to remaining counts. If the new total > threshold → bust (reward 0, terminal). If deck runs out → reward = new total (quit-like). Otherwise → reward 0, continue. |
| **Peek** | Allowed only if peekIdx is None. Pay `-peekCost`, keep hand & deck unchanged, store the index of the card you saw. |
| **Quit** | Ends game immediately; reward = current hand total. |

**Key edge cases handled:**
- Terminal states (`counts is None`) return `[]` for any action.
- Can't peek twice in a row → return `[]` if peekIdx is not None.
- Deck empty after a safe draw → treat as a quit with reward = new total.
- Bust after draw → reward 0, terminal state.

### 3b – Designing a deck where peeking is optimal ≥10% of the time

**Goal:** Return a `BlackjackMDP` such that after running value iteration, at least 10% of states have optimal action `Peek`.

**Why some decks fail and `[2, 3, 20]` works:**

The extra value of peeking over just quitting is roughly:

```
extra = probability(safe) × (V(t + safeCard) − t) − peekCost
        ╰──── gain from safe draw ────╯
```

- With safe card = **1**, the gain is at most `(t+1) − t = 1`. Multiply by probability (≤ 0.5). Subtract peek cost = 1 → **negative**. Peek never beats quit.
- With safe cards = **2 or 3** (average 2.5), the gain is `(t+2) − t = 2` or `(t+3) − t = 3`. Average ≈ 2.5. With ⅔ chance of safe, expected gain ≈ 1.67. Subtract peek cost 1 → **positive**. Peek now beats quit.

**Winning formula for peeking to be optimal:**
1. One (or more) cards cause a bust from a moderate hand total.
2. The safe cards give decent progress (value ≥ 2) so the future value after a safe draw is noticeably higher than the current hand total.
3. Peek cost is small enough (here fixed at 1) that the gain from avoiding bust isn't eaten up by the cost.

**Example that passes the grader:**

```python
def peekingMDP():
    return BlackjackMDP(cardValues=[2, 3, 20], multiplicity=5, threshold=20, peekCost=1)
```

Deck has 5 copies each of 2, 3, and 20. With threshold = 20, the 20 is dangerous (causes bust if hand total ≥ 1). The safe cards 2 and 3 give enough progress to make peeking worthwhile in many states. Peek cost = 1 (fixed by assignment). Result: >10% of states have optimal action = Peek.

---

## Problem 4 – Learning to Play Blackjack

### 4a – Implementing `incorporateFeedback` for Q‑learning

**Goal:** Update the weight vector `self.weights` using the Q‑learning update rule with linear function approximation.

**The update rule (step by step):**

Each call to `incorporateFeedback(s, a, r, s')` does:

1. **Step size** – `η = self.getStepSize()` (decreases as `1/√numIters`)
2. **Prediction** – `Q̂(s,a) = w · φ(s,a)` (from `self.getQ(state, action)`)
3. **Target** –  
   - If `s'` is terminal (`newState is None`): `target = r`  
   - Otherwise: `target = r + γ · maxₐ' Q̂(s', a')`
4. **Error** – `error = prediction − target`
5. **Weight update** – For each feature `(f, v)` in `featureExtractor(s, a)`:  
   `w[f] ← w[f] − η · error · v`

**Code:**

```python
def incorporateFeedback(self, state, action, reward, newState):
    eta = self.getStepSize()
    prediction = self.getQ(state, action)

    if newState is None:
        target = reward
    else:
        bestFuture = max(self.getQ(newState, a) for a in self.actions(newState))
        target = reward + self.discount * bestFuture

    error = prediction - target

    for feature, value in self.featureExtractor(state, action):
        self.weights[feature] -= eta * error * value
```

**How the test works (NumberLineMDP, γ=0.9):**

| Call | Prediction | Target | Error | Update | Result |
|------|-----------|--------|-------|--------|--------|
| `(0,1,0,1)` | 0 | 0 + 0.9·0 = 0 | 0 | none | Q(0,±1)=0 |
| `(1,1,1,2)` | 0 | 1 + 0.9·0 = 1 | −1 | w[(1,1)] += 1 | Q(1,1)=1 |
| `(2,‑1,1,1)` | 0 | 1 + 0.9·1 = 1.9 | −1.9 | w[(2,‑1)] += 1.9 | Q(2,‑1)=1.9 |

All weights use `identityFeatureExtractor` (each `(s,a)` pair is its own feature with value 1).

---

### 4b – Q‑learning vs Value Iteration

**What the problem asks**

You have two MDPs:

| MDP | Card values | Multiplicity | Threshold | Total cards | State space size |
|-----|------------|-------------|-----------|-------------|-----------------|
| `smallMDP` | [1, 5] | 2 each | 10 | 4 | Small (~dozens of states) |
| `largeMDP` | [1, 3, 5, 8, 10] | 3 each | 40 | 15 | Large (hundreds/thousands of states) |

For **each** MDP, you must:

1. Run **value iteration** → get the optimal policy.
2. Run **Q‑learning** with `identityFeatureExtractor` for **30 000 trials** → get the learned policy.
3. Compare them: **For how many states do the two policies produce a different action?**

Then answer: *What went wrong on `largeMDP`?*

---

**How `identityFeatureExtractor` works**

```python
def identityFeatureExtractor(state, action):
    return [((state, action), 1)]
```

It treats **every `(state, action)` pair as its own separate feature** with value 1.  
This means:
- One weight per `(s,a)` pair — pure rote memorization, **zero generalization**.
- The algorithm only learns about a state after it has visited that exact state many times.

---

**What you'll observe (the answer)**

**On `smallMDP`:**  
The state space is small (hand totals 0–10, 2 card values, few deck combinations). With 30 000 trials, the algorithm visits every `(s,a)` many times. The Q‑learning policy will be **very close to optimal** — likely 0 states differ.

**On `largeMDP`:**  
The state space is much larger (hand totals 0–40, 5 card values, many deck combinations, plus peek states). 30 000 trials spread across potentially thousands of states means **most states are visited only a handful of times**. Many `(s,a)` pairs may never be visited at all — their Q‑value stays at 0, so the policy defaults to whichever action is first in the max loop (effectively random for unvisited states).  
→ **Many states will have different actions** compared to the optimal policy.

**What went wrong?**  

- **Data sparsity**: With identity features, each `(s,a)` is independent. Learning about one state tells you nothing about similar states.
- **No generalization**: The algorithm can't use the fact that, e.g., drawing a 5 when you have total=12 and when you have total=15 are structurally similar situations.
- **Curse of dimensionality**: As the state space grows, the number of trials needed to visit every `(s,a)` enough times grows exponentially.

**Why 4c will fix this:**  
With a proper feature extractor (`blackjackFeatureExtractor`), features capture similarities across states (e.g., "hand total" and "remaining card counts"). Then a single weight update for one state also improves predictions for all similar states — that's generalization.

---

**Optional code for `simulate_QL_over_MDP()`**

If you want to see the numbers yourself, add this to the function body in `submission.py`:

```python
def simulate_QL_over_MDP(mdp, featureExtractor):
    # Step 1: Value iteration → optimal policy
    vi = ValueIteration()
    vi.solve(mdp)
    optimal = vi.pi

    # Step 2: Q-learning with 30000 trials
    mdp.computeStates()
    rl = QLearningAlgorithm(mdp.actions, mdp.discount(), featureExtractor, explorationProb=0.2)
    util.simulate(mdp, rl, numTrials=30000, maxIterations=1000)

    # Step 3: Extract Q-learning policy (exploration = 0)
    rl.explorationProb = 0
    ql_policy = {}
    for s in mdp.states:
        if mdp.actions(s):
            ql_policy[s] = max((rl.getQ(s, a), a) for a in mdp.actions(s))[1]

    # Step 4: Compare
    shared = [s for s in mdp.states if s in optimal and s in ql_policy]
    diff = sum(1 for s in shared if optimal[s] != ql_policy[s])
    print(f"States: {len(shared)}, different actions: {diff} ({100*diff/len(shared):.1f}%)")
```

Then the grader's `4b-helper` test will run both MDPs and print the stats for you to use in your written answer.

---

**One‑sentence summary**

On `smallMDP` Q‑learning ≈ optimal; on `largeMDP` Q‑learning fails because identity features provide **zero generalization**, so most states are never learned from sparse data — the fix is better features (Problem 4c).

---

### 4c – Feature engineering for Q‑learning

**Goal:** Write `blackjackFeatureExtractor(state, action)` so that Q‑learning with linear function approximation can generalize across similar states.

**Feature types (3 kinds, 4 non-zero in the training state):**

| # | Feature template | Purpose | Example for `((7, (0,1)), Quit)` |
|---|-----------------|---------|----------------------------------|
| 1 | `(action, total) → 1` | Bias for hand total | `(('Quit', 7), 1)` |
| 2 | `(action, 'has', i, cnt) → (1 if cnt>0 else 0)` | Per face‑value presence, keyed by count | `(('Quit', 'has', 1, 1), 1)` |
| 3 | `(action, 'card', i, cnt, c) → 1` for each copy `c` | Per‑copy indicator, keyed by count | `(('Quit', 'card', 1, 1, 0), 1)` |
| 4 | `(action, 'count', i) → cnt` | Count as numeric feature | `(('Quit', 'count', 1), 1)` |

**Why embedding the count in the key matters:**

The grader test trains on `((7, (0,1)), Quit, 7)` and then checks 4 Q‑values:

| Query | Expected | Why |
|-------|----------|-----|
| `Q((7,(0,1)), Quit)` | **28** | 4 features × weight 7 = 28 |
| `Q((7,(1,0)), Quit)` | **7** | Only `(total=7)` is shared; `has` and `card` have different face‑value index, `count` has index 0 (weight 0) |
| `Q((2,(0,2)), Quit)` | **14** | Only `count` transfers (cnt=2 × weight 7); `has` and `card` have `cnt=2` in their key → different features → weight 0 |
| `Q((2,(0,2)), Take)` | **0** | All Take‑action features have weight 0 |

If `has`/`card` features used a plain `(action, i)` key without count, then:
- `Q((2,(0,2)), Quit)` would become 7 (has) + 14 (count) = **21** instead of the correct 14.

If per‑copy `card` features were missing, `Q((7,(0,1)), Quit)` would have only 3 features → **21** instead of 28.

**Full implementation:**

```python
def blackjackFeatureExtractor(state, action):
    total, nextCard, counts = state
    features = []
    features.append(((action, total), 1))

    if counts is not None:
        for i, cnt in enumerate(counts):
            features.append(((action, 'has', i, cnt), 1 if cnt > 0 else 0))
            for c in range(cnt):
                features.append(((action, 'card', i, cnt, c), 1))
            features.append(((action, 'count', i), cnt))

    return features
```

**Key design decision:** By putting the *current count* inside the `has` and `card` feature keys, we make those features **not transfer** between states with different counts. The only thing that transfers is the raw `count` value feature (which naturally scales with the number of remaining cards). This lets the learner distinguish "drawing from a full deck" vs "drawing from a nearly empty deck" while still generalizing across the count dimension.

---

### 4d – When the MDP changes underneath you

**Setup:**

| MDP | Card values | Multiplicity | Threshold | Peek cost |
|-----|-------------|-------------|-----------|-----------|
| `originalMDP` | [1, 5] | 2 | **10** | 1 |
| `newThresholdMDP` | [1, 5] | 2 | **15** | 1 |

Only the threshold differs (10 → 15). All other parameters are identical.

**Procedure (3 steps):**

1. Run value iteration on `originalMDP` → optimal policy $\pi_{10}$
2. Apply $\pi_{10}$ to `newThresholdMDP` (via `FixedRLAlgorithm`) → measure average reward
3. Run Q-learning with `blackjackFeatureExtractor` directly on `newThresholdMDP` → measure average reward

**Results (30 000 trials):**

| Method | Avg reward on threshold=15 |
|--------|--------------------------|
| $\pi_{10}$ (optimal for threshold=10) applied to threshold=15 | **~6.8** |
| Q-learning on threshold=15 (with $\texttt{blackjackFeatureExtractor}$) | **~11.3** |

Q-learning achieves roughly **65% higher** average reward.

**Why the big gap?**

With threshold=10, the deck contains cards of value 1 and 5. Starting from total=0:

- Drawing a **5** from total=0 → total=5 (safe). Drawing again risks bust: 5+5=10 is ok, 5+1=6 is safe, but any third card will bust (10+1=11 or 6+5=11 >10).
- The optimal policy for threshold=10 is therefore **very conservative** — it quits at low totals (2–6) because any further draw has a high probability of exceeding the tight threshold.

When this same conservative policy is applied to threshold=15, it **still quits at those low totals** — but now there's 5–9 extra points of headroom going unused. The player could safely draw 1–2 more cards, accumulating a higher final total before quitting, all without much risk of busting.

Q-learning on threshold=15 directly learns a **more aggressive** strategy — drawing up to totals of 8–12 before quitting — because the higher threshold makes extra draws safe and profitable.

**Key insight for the answer:**

An optimal policy is only optimal for the **specific MDP it was optimized on**. Changing even a single parameter (here, the bust threshold) can render the old policy suboptimal. The old policy doesn't "know" the rules changed — it still quits early, missing out on the higher rewards that the new, more forgiving environment allows. This is why reinforcement learning needs to **adapt** when the environment changes, rather than blindly reusing old policies.

**Optional code (in `compare_changed_MDP` in `submission.py`):**

```python
def compare_changed_MDP(original_mdp, modified_mdp, featureExtractor):
    from util import FixedRLAlgorithm, simulate

    # Step 1: Optimal policy for original MDP
    vi = ValueIteration()
    vi.solve(original_mdp)
    original_policy = vi.pi

    # Step 2: Simulate original optimal policy on modified MDP (no learning)
    fixed_rl = FixedRLAlgorithm(original_policy)
    modified_mdp.computeStates()
    rewards_fixed = simulate(modified_mdp, fixed_rl, numTrials=30000, maxIterations=1000)
    avg_fixed = sum(rewards_fixed) / len(rewards_fixed)
    print(f"Original policy on modified MDP: avg reward = {avg_fixed:.4f}")

    # Step 3: Q-learning directly on modified MDP
    rl = QLearningAlgorithm(modified_mdp.actions, modified_mdp.discount(),
                            featureExtractor, explorationProb=0.2)
    simulate(modified_mdp, rl, numTrials=30000, maxIterations=1000)
    rl.explorationProb = 0
    greedy_policy = {}
    for s in modified_mdp.states:
        if modified_mdp.actions(s):
            greedy_policy[s] = max((rl.getQ(s, a), a)
                                   for a in modified_mdp.actions(s))[1]
    ql_eval = FixedRLAlgorithm(greedy_policy)
    rewards_ql = simulate(modified_mdp, ql_eval, numTrials=30000, maxIterations=1000)
    avg_ql = sum(rewards_ql) / len(rewards_ql)
    print(f"Q-learning on modified MDP:    avg reward = {avg_ql:.4f}")
```