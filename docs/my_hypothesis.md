> **Why do I think an RNN should work for electricity demand forecasting?**
me:
because the demand today might be dependent on the demand of yester of previous days.
because how electricity is consumed is more of seasonal something.

### Hypothesis 1 — RNN
me:
If the dataset timesteps is very long.
I expect vanila RNN training speed to be slow. and the accuracy to be the lowest among all. because it struggles to remember earlier information. so the maths during training causes vanishing, and exploding gradients.
But if the data is not too long. I expect vanila RNN to be on par with LSTM and GRU.

### Hypothesis 2 — LSTM
me:
I expect LSTM to perform better than vanila RNN in terms of model accuracy, because this architecture is designed to solve the vanishing gradient problem.

### Hypothesis 3 — GRU
me: i expect GRU to be on par with LSTM. but the training and inference speed is faster.

### Hypothesis 4 — Transformer
me:
I expect Transformer to have the highest model accuracy. but be the most computational and memory expensive