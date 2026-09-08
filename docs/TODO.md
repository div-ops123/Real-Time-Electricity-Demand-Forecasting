
data source: https://www.kaggle.com/datasets/aramacus/electricity-demand-in-victoria-australia

---

This is the task my teacher gave me(He is talking to me):

Your goal today is **not** "build an electricity forecasting model."

Your goal is:

> **Use a real forecasting problem to understand the strengths, failure modes, and trade-offs of RNNs, LSTMs, GRUs, and Transformers, then make an engineering decision about which architecture you would deploy.**

# Your RNN Engineering Judgment Project

## Project

**Real-World Electricity Demand Forecasting**

### Business problem

A power company wants to forecast **tomorrow's electricity demand** using historical electricity demand and information available up to the current day.

You have daily data from 2015–2020.

Your job is to determine:

> **What sequence model should we use for this forecasting system?**

You are going to investigate:

**Vanilla RNN vs LSTM vs GRU vs Transformer**

---

# 1. First Principle — Before touching the data

Write this down before you start.

### Your prediction

Answer these **before training anything**:

> **Why do I think an RNN should work for electricity demand forecasting?**

Then predict:

### Hypothesis 1 — RNN

> I expect a vanilla RNN to ...

### Hypothesis 2 — LSTM

> I expect LSTM to ...

### Hypothesis 3 — GRU

> I expect GRU to ...

### Hypothesis 4 — Transformer

> I expect a Transformer to ...

**Do not change these predictions after seeing the results.**

You'll come back later and check whether reality agreed with you.

---

# 2. Engineering Question

This is the **main question** your entire project must answer:

> ### "For this electricity-demand forecasting problem, under different sequence lengths and system constraints, which architecture would I actually choose: RNN, LSTM, GRU, or Transformer, and why?"

You are NOT trying to prove:

> "Transformer is best."

You are trying to discover:

> **When does each architecture make sense?**

---

# 3. Establish the simplest possible baseline

Before neural networks.

Your first model should be:

### Naive forecasting

This gives you your baseline.

Then calculate:

* MAE
* RMSE

Your neural networks must beat this baseline to justify their complexity.

### Engineering question

> **Does the neural network actually provide enough improvement over a stupidly simple baseline to justify its complexity?**

---

# 4. Prepare the forecasting problem

Use:

### Target

```text
demand
```

Predict:

```text
demand(t+1)
```

using historical information up to:

```text
t
```

Start with **only demand**.

Don't introduce all the other features yet.

---

## Create multiple sequence lengths

This is critical.

Create:

```text
7 days  → predict next day
14 days → predict next day
30 days → predict next day
60 days → predict next day
```

For example, with a 7-day window:

```text
Day 1 ─┐
Day 2  │
Day 3  │
Day 4  ├──► Model ──► Day 8 demand
Day 5  │
Day 6  │
Day 7 ─┘
```

With a 60-day window:

```text
Day 1
Day 2
...
Day 59
Day 60
       ↓
     Model
       ↓
Day 61
```

This is how you **experimentally create different levels of temporal dependency**.

---

# 5. Experiment 1 — Vanilla RNN

Train a simple vanilla RNN.

Keep everything else controlled:

* same dataset
* same train/validation/test split
* same target
* same sequence lengths
* comparable model size where practical
* same evaluation metrics

Test:

```text
7 days
14 days
30 days
60 days
```

Record:

| Sequence length | MAE | RMSE | Training time | Inference latency |
| --------------: | --: | ---: | ------------: | ----------------: |
|               7 |     |      |               |                   |
|              14 |     |      |               |                   |
|              30 |     |      |               |                   |
|              60 |     |      |               |                   |

### Your engineering question

> **What happens to the RNN as the amount of historical information increases?**

Don't just look at accuracy.

Look for:

* degradation
* training instability
* slower training
* inability to benefit from longer history

---

# 6. Experiment 2 — LSTM

Now replace:

```text
RNN
```

with:

```text
LSTM
```

Everything else stays as similar as possible.

Again:

```text
7 days
14 days
30 days
60 days
```

Record the same metrics.

### Engineering question

> **Does the LSTM actually solve the problem we observed with the vanilla RNN?**

You should specifically look for:

> Does LSTM benefit more from longer historical windows?

If RNN performs well at 7 days but deteriorates at 60 days while LSTM remains stable, you have experimentally observed the reason gated recurrence exists.

---

# 7. Experiment 3 — GRU

Now:

```text
GRU
```

Run the same experiment.

Record:

* MAE
* RMSE
* parameter count
* training time
* inference latency

### Engineering question

> **Does GRU give me most of LSTM's performance with less complexity?**

This is your first genuine architecture-selection decision.

Suppose you discover:

```text
LSTM   RMSE = 20     100k parameters
GRU    RMSE = 20.5    75k parameters
```

You now have something to reason about.

Is that tiny accuracy improvement worth the additional complexity?

Maybe.

Maybe not.

---

# 8. Experiment 4 — Transformer

Now introduce a Transformer.

Use the same forecasting task.

Again test:

```text
7
14
30
60
```

Measure:

* MAE
* RMSE
* parameter count
* training time
* inference latency
* memory usage if practical

### Engineering question

> **Does attention actually give us a meaningful advantage for this forecasting problem?**

You're testing the hypothesis:

> Longer-range information should be easier to access because the Transformer does not have to pass information through a recurrent chain.

---

# 9. Experiment 5 — The important one: Long history

Now make the sequence longer.

If your dataset and implementation allow it, test something like:

```text
7 days
14 days
30 days
60 days
90 days
```

You are specifically investigating:

> **How does each architecture behave as the temporal distance increases?**

Create a graph:

```text
Sequence Length
       →
Model performance
```

You want to see the behavior of:

```text
RNN
LSTM
GRU
Transformer
```

This experiment connects your implementation directly to:

**recurrence → gradient flow → long-term dependencies → LSTM/GRU → attention**

---

# 10. Experiment 6 — Add contextual features

Only **after** understanding the univariate problem should you add:

* minimum temperature
* maximum temperature
* solar exposure
* rainfall
* school day
* holiday
* etc.

Now your model gets:

```text
Past demand
+
Weather
+
Calendar information
       ↓
     Model
       ↓
Tomorrow's demand
```

Compare against your demand-only models.

### Engineering question

> **Is the model failing because it cannot remember enough history, or because demand alone doesn't contain enough information?**

This distinction is important.

A better architecture cannot magically recover information that isn't present in the input.

---

# 11. Experiment 7 — Simulate the production constraint

Now bring in the actual system-design problem.

Imagine:

> **New electricity-demand data arrives once per day. We need to generate a prediction immediately. The system has limited compute and memory.**

Ask:

### RNN/LSTM/GRU

Can I maintain:

```text
hidden state
```

and update it when new data arrives?

Conceptually:

```text
Yesterday's hidden state
          +
Today's observation
          ↓
Today's hidden state
          ↓
Tomorrow's prediction
```

### Transformer

What happens if I need to provide the model with a growing historical context?

Now reason about:

* latency
* memory
* sequence length
* compute
* implementation complexity

---

# 12. Your final engineering decision

At the end, you must choose **one architecture**.

Write:

> **"For this production scenario, I would deploy ______ because..."**

Your decision must consider:

### 1. Accuracy

Does it actually forecast better?

### 2. Long-term dependency

Does performance remain good as history increases?

### 3. Latency

How quickly can it produce a prediction?

### 4. Memory

How much state/context must it maintain?

### 5. Compute

How expensive is training/inference?

### 6. Complexity

How difficult is it to operate and maintain?

### 7. Data size

Does your dataset justify a more complex model?

### 8. Streaming requirement

Can the model naturally process new observations as they arrive?

---

# 13. Your final comparison table

By the end, you should have something like:

|                        | RNN | LSTM | GRU | Transformer |
| ---------------------- | --: | ---: | --: | ----------: |
| Best RMSE              |     |      |     |             |
| MAE                    |     |      |     |             |
| Parameters             |     |      |     |             |
| Training time          |     |      |     |             |
| Inference latency      |     |      |     |             |
| Short sequences        |     |      |     |             |
| Long sequences         |     |      |     |             |
| Streaming suitability  |     |      |     |             |
| Memory characteristics |     |      |     |             |
| Complexity             |     |      |     |             |
| Your verdict           |     |      |     |             |

The table isn't the final answer.

**Your reasoning behind the table is the final answer.**

---

# 14. Your failure analysis

This is mandatory.

For every model that performs poorly, don't write:

> "Model performed badly."

Ask:

### RNN

> **Why?**

Possible explanation:

```text
Longer sequence
      ↓
Repeated recurrent transformations
      ↓
Difficult gradient flow
      ↓
Difficulty learning long-term dependencies
```

### LSTM

If it performs poorly:

> Was the problem actually long-term memory?

Maybe the problem is:

* insufficient data
* poor features
* bad hyperparameters
* overfitting
* demand patterns aren't predictable from historical demand

### Transformer

If it performs poorly:

> **Was the Transformer actually justified for this dataset?**

This is an extremely important engineering question.

A more sophisticated architecture can perform worse when:

* the dataset is small
* the problem is simple
* the model is overparameterized
* training isn't sufficient

---

# 15. What you should NOT do(IMPORTANT)

Don't turn this into:

```text
Kaggle competition
        ↓
hyperparameter tuning
        ↓
0.001 better RMSE
        ↓
celebrate
```

That would miss your learning goal.

Your goal is **architecture understanding + engineering judgment**.

So don't spend 3 hours trying:

```text
learning rate = 0.001
learning rate = 0.0009
learning rate = 0.0008
...
```

Keep the models reasonably small and controlled.

---

# Your project in one sentence

If someone asks what you're doing, you should be able to say:

> **I'm using electricity-demand forecasting as a real-world sequence problem to experimentally investigate when recurrence is sufficient, when LSTM/GRU's gated state helps, and when a Transformer's attention is worth its additional complexity.**

---

# Your order of work

Do **not** jump around.

Follow exactly this order:

```text
1. Make predictions before learning
          ↓
2. Understand the dataset
          ↓
3. Create chronological train/validation/test split
          ↓
4. Build naive baseline
          ↓
5. Build 7/14/30/60-day forecasting datasets
          ↓
6. Train vanilla RNN
          ↓
7. Analyze RNN failure
          ↓
8. Train LSTM
          ↓
9. Compare RNN vs LSTM
          ↓
10. Train GRU
          ↓
11. Compare LSTM vs GRU
          ↓
12. Train Transformer
          ↓
13. Compare all four
          ↓
14. Add contextual features
          ↓
15. Simulate streaming inference
          ↓
16. Make deployment decision
          ↓
17. Write what your experiments taught you
```

## The real thing you're trying to walk away with

Not:

> "I know what an LSTM is."

But:

> **"I understand the problem that recurrence creates, I can see its failure experimentally, I understand why gating helps, I understand what attention changes, and I can choose an architecture based on workload constraints rather than popularity."**

**That is your RNN engineering-judgment project.**
