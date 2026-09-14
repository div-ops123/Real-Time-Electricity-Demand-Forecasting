# My RNN engineering-judgment

Baseline metrics:
mae	rmse	training_time_s	inference_latency_ms
model				
Persistence (lag-1)	7865.516528	10795.675682	0.0	0.000099
Seasonal naive (lag-7)	8885.072453	12223.675602	0.0	0.000114


Vanilla rnn:
	mae	rmse	training_time_s	inference_latency_ms
sequence_length				
1	7760.540527	10122.441406	3.258786	1.066827
7	19860.886719	43534.234375	3.518814	2.223256
14	269685.406250	336429.000000	4.256513	3.198894
30	306027.218750	371054.718750	7.748473	5.957185
60	263662.218750	330736.875000	13.204792	12.850017

LSTM:
	mae	rmse	training_time_s	inference_latency_ms
sequence_length				
1	9698.666016	11805.098633	2.409633	0.931547
7	8992.845703	11346.113281	2.348991	1.140558
14	9061.880859	11385.147461	2.791400	1.847200
30	9095.818359	11441.597656	2.126941	1.811726
60	9215.600586	11641.742188	2.512273	1.588128

GRU:
	mae	rmse	training_time_s	inference_latency_ms
sequence_length				
1	8906.745117	10950.436523	3.089812	2.565780
7	8473.120117	10614.452148	2.796977	1.394810
14	8331.573242	10421.300781	2.189217	0.934122
30	8322.538086	10420.489258	1.735753	1.035174
60	8428.742188	10580.535156	3.339650	1.572851

Transformer:
	mae	rmse	training_time_s	inference_latency_ms
sequence_length				
1	7516.348633	9807.522461	6.653051	2.551296
7	6575.718262	8551.777344	9.235628	3.075652
14	6325.072266	8458.743164	9.860706	4.038359
30	6783.849609	8927.573242	9.805944	3.678054
60	6921.336914	9088.635742	8.676324	2.256606

---

dataset is small
dataset is synthetic i think, becos the pattern is too obvious.
so maybe data made the performance of each model bias. n the speed not to be trusted.
I would not experiment with more features becos the dataset is small on it's own, also becos i have seen the main idea behind the models. improving accurcy by 0.001 is not my goal.

Baseline(t-1):
me: peformance is not bad on the mae, rmse.

so i can tell that sequence_length=1 is what the model should train with.

but training and inference time across rnn models is not that different. maybe beos of datasize.

Vanilla rnn:
> **What happens to the RNN as the amount of historical information increases?**
me: everything increases. error, and training time and inference time.
for sequence_length=1, the mae, and rmse is lower than lstm, gru. but transformer pass it small. bt not significant. so vanilla rnn is leading in terms of accuracy.
performances better here could be becos t=1 no long sequence so that problem with vanilla rnn(vanishing gradient) can't come up.
failure analysis: but if the question was to use longer number of days to predict then this model is bad for that.


LSTM:
> **Does the LSTM actually solve the problem we observed with the vanilla RNN?**
me: yes. the errors for longer sequence is lower very lower than vanilla rnn.
for sequence_length=1, the mae, and rmse is higher than vanilla rnn.
but for sequence_length=7 and above. the mae, and rmse is lower than vanilla rnn.
> Does LSTM benefit more from longer historical windows?
failure mode: the error across all sequence is in same range. so having long term memory is not the problem with this dataset.


GRU:
> **Does GRU give me most of LSTM's performance with less complexity?**
me: a little. not much. the difference in their mae, and rmse like less than 1000.
vanilla rnn still has the lowest error.
> Is that tiny accuracy improvement worth the additional complexity?
me: depending on what the business wants to use it for. i'll compare the cost. n if the business says evena tiny improvement earns/saves them money then gru is prefered over lstm.


Transformer:
me: has the lowest training error.
> **Does attention actually give us a meaningful advantage for this forecasting problem?**
me: no meaningful advantage.



> **How does each architecture behave as the temporal distance increases?**
me: honesly... same. each architecture behavior as distane increases doesn't change much.
only vanlla rnn that performace is super bad as distance increase. and that is becos of it's difficulty to remember long sequence.

> **New electricity-demand data arrives once per day. We need to generate a prediction immediately. The system has limited compute and memory.**
me: from the eda i saw that there is a clear pattern in the demand. and taught it is weekly pattern, but after running baseline i found that yesterday'sprice=today's price gives lower error than weekly. and it's error for t=1 is comparable to other ml models error.
So good thing the business question ispredict once per day. matches the pattern i saw.
t being 1 means i do not need to remember to much past pattern.
so i am evaluting based on sequence_length=1 only.
compute and memory is limited so we need less heavy model. that cancels out lstm, and transformer.
left with baseline, gru and vanilla rnn.
i would cancel out gru since this dataset don't benefit from long sequence.
so baseline and vanilla rnn.
i would pick baseline because that is the simplest solution. and the diff with vanilla rnn is quite small, so since the compute and memory is limited baseline works suprisingly well.

> **"For this production scenario, I would deploy baseline(lag-1) because it's performance is comparible to vanilla rnn. so no significant benefit of using a ml model."**
