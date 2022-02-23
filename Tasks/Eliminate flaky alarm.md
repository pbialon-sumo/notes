### Introduction
There is an alert monitor `metricsforge_ktp_input_data_points_rate`
which triggered alert 845 times last year. The alert is triggered when on a single KTP there is more than 10 000 requests in one minute. The alert is based on `ktp_input_data_points_rate` metric.
Unfortunately, the metrics is wrapped inside a log, and stored (as any other log entrance) in S3 storage. The very rule for that alarm is parsing the blob of logs.
==**Why is that?** TODO: Cardinality??????==

**Slack discussion**
[Slack](https://sumologic.slack.com/archives/C0LKC7QQZ/p1644945718427209?thread_ts=1644943739.404109&cid=C0LKC7QQZ "Follow link")

**Playbook**
[Playbook](https://github.com/Sanyaku/playbooks/wiki/metricsforge_ktp_input_data_points_rate) for dealing with an alert

**How to get data for that alert?**

```bash
_sourceCategory=metricsforge_metrics_reporter ktp_input_data_points_rate
```


**What is the idea for fixing it?**
First of all, change the metric to be pushed to our metrics ingest pipeline. But we have to reduce cardinality of the metrics being queried in alarm monitor. We want to use [[Dictionary#^b39067|transformation rule]] to delete a `node` dimension of that monitor 


**How to define alert to be present on all deployments?**
Alek Lewandowski should know. In the `Entities` there are alerts across all deployments (maybe they used content management repo?)


==TODO: unified dogfooding => write feedback to them, about adding `transformation rule` (how useful interface and help section was)==


### Notatki (Michal Matusiak)
Jaki threshold jest problematyczny
Ile alertow by sie trigerowalo jakby bylo 10minut + 40k.
Pogadac w zespole jaki treshold na KTP jest bezpieczny. 
Przeniesc do sumo jako dogfooding. Content management repository.

Automatyzacja ingest scalingu.

---


## Investigation of adjusting thresholds

**How to get data from Sumo interface?**
```bash
_sourceCategory=metricsforge_metrics_reporter ktp_input_data_points_rate | parse "oneMinuteRate=*," as rate | toDouble(rate) | where rate > 10000 | count
```

**How to check real alarm triggered?**

(!) Rate can be > 10000 for a long period of time, and there will be only one alarm. 

`getMaxInputDataPointsRate` => transformation of records in the code, after sorting:
```json
[
	(ktp_1, 10000),
	(ktp_2, 9999),
	(ktp_1, 9888),
]
```
We return the greatest value (or _None_)

**How many metrics which contains `ktp_input_data_points_rate` do we generate?**

_Hypothesis_: for each ktp we have different meter. That's why we get them this way.



### Problem analysis (100k export limit)

In the interface there is a limit of 100k datapoints, for export / exploring.
100k dp covers ~ 8 hours of data. We need at least 7 days, best 30, or 365.

There are two ways of dealing with that problem.
##### Possible approaches
1. Anyway try to export full data; that can be accomplished in two ways:
	- use [Search Job API](https://help.sumologic.com/APIs/Search-Job-API/About-the-Search-Job-API) for sumo logic log queries
	- export one by one from the interface (very tedious, requires ~ 21 exports for a week, and ~ 100 for a month)
2. Filter out the unnecessary data (below some threshold)


In the 2nd way there is a problem of missing alarm resolvers.
How to prevent / assess the risk?

I can modify my analysis, to capture alarm resolvers.
It can be a reinforcing loop of analysis:

```python

def get_data(threshold):
	q = construct_query(threshold)
	result = q.run()
	timerange = result.get_timerange()
	return result, timerange

def get_min_alarm_resolver(data):
	return min(
		dp
		if is_resolver(dp)
		for dp in data
	)	

def main():
	threshold = 0
	timerange = 8 hours 
	
	while timerange < 30 days:
		data, timerange = get_data(threshold)
		threshold = get_min_alarm_resolver()
	
```


The second analysis showed that the filtering out the results is not an option.
There were alarm resolvers as low as 100 or even 1.

##### Another approach

**10k**
Check how many times there were rates greater than 20k, 40k and 50k in comparison to 10k.

Results of the following two queries doesn't show that we can accomplish anything with switching to `fifteenMinuteRate` instead of `oneMinuteRate`
```bash
search() | where fifteenMinuteRate > 10000 | count
search() | where oneMinuteRate > 10000 | count
```

**20k**
```bash
_sourceCategory=metricsforge_metrics_reporter ktp_input_data_points_rate | parse "oneMinuteRate=*," as oneMinuteRate | parse "ktp=* " as ktp | parse "fifteenMinuteRate=*," as fifteenMinuteRate | where oneMinuteRate > 20000 | count
```



###
Od czego zalezy max ilosc ktpsow?

