## Table of contents
```toc
```

## Task description

There is an alert monitor `metricsforge_ktp_input_data_points_rate`
which triggered alert 845 times last year. The alert is triggered when on a single KTP there is more than 10 000 requests in one minute. The alert is based on `ktp_input_data_points_rate` metric.
Unfortunately, the metrics is wrapped inside a log, and stored (as any other log entrance) in S3 storage. The very rule for that alarm is parsing the blob of logs.

#### Slack discussion
[Slack](https://sumologic.slack.com/archives/C0LKC7QQZ/p1644945718427209?thread_ts=1644943739.404109&cid=C0LKC7QQZ "Follow link")

#### Playbook
[Playbook](https://github.com/Sanyaku/playbooks/wiki/metricsforge_ktp_input_data_points_rate) for dealing with an alert


#### How to get data for that alert?

```bash
_sourceCategory=metricsforge_metrics_reporter ktp_input_data_points_rate
```


#### What is the idea for fixing it?
First of all, change the metric to be pushed to our metrics ingest pipeline. But we have to reduce cardinality of the metrics being queried in alarm monitor. We want to use [[Dictionary#^b39067|transformation rule]] to delete a `node` dimension of that monitor 


#### How to define alert to be present on all deployments?
Alek Lewandowski should know. In the `Entities` there are alerts across all deployments (maybe they used content management repo?)


#### Meeting notes 

**Michal Matusiak**

What is the real threshold? (i.e. degradation of performance) 
What is the number of alerts triggered with 40k threshold?
Do we know the real safe threshold? 
Move that metrics to Sumo, to extend dogfooding.

Automatisation of ingest scaling: [[Dictionary#^967a86|automatic ingest scaling]]

**Szymon**

What are the factors influencing max throughput on kafka?
- size of metric definition (metric's metadata)
- ephemerality (how long-lived is a specific metric => is it in the Metrics Forge cache?)
- Kafka curent load
- how many datapoints there is per batch?
- how many rules there is defined?  



## [[Investigation]] of adjusting thresholds

## Summary of analysis
W ramach analizy wyszukalem wszystkie ktpsy, ktorych rate przekroczyl 20k, w dowolnym momencie w przeciagu ostatniego miesiaca. Lista tych ktpsow znajduje sie ponizej:
- 0000000000054B70-metrics_raw_data-234
- 0000000000054B70-metrics_raw_data-93
- 0000000000482E67-metrics_raw_data-26
- 0000000000482E67-metrics_raw_data-54
- 0000000000520B73-metrics_raw_data-27
- 0000000000520B73-metrics_raw_data-4
- 0000000000BC1E2B-metrics_raw_data-0
- 0000000000BC2213-metrics_raw_data-0
- 0000000000BC3627-metrics_raw_data-1
- 0000000000BC3627-metrics_raw_data-14
- 0000000000BC3627-metrics_raw_data-17
- 0000000000BC3627-metrics_raw_data-18
- 0000000000BC3627-metrics_raw_data-26
- 0000000000BC3627-metrics_raw_data-32
- 0000000000BC3627-metrics_raw_data-56
- 0000000000BC3627-metrics_raw_data-6

Stworzylem [dashboard](https://us1data.long.sumologic.net/ui/#/dashboardv2/in0XYDk7Ano1qEWgAcpIuR1sBr7A2oGlFpCK4AYm0HABcVb5f5zOhkN6UnAh?variables=ktp:0000000000520B73-metrics_raw_data-27), ktory pozwalal na porownanie jak rate wplywal na wielkosc laga na danym ktpsie.

**Query to get lag for specific ktp**
```bash

_sourceCategory=metricsforge LagMonitor Logging lag for | parse "lag=*." as lag | parse "KTP=*," as ktp | where ktp = "0000000000054B70-metrics_raw_data-234" | timeslice 1m | avg(lag) by _timeslice

```

Dla kazdego z powyzszych ktpsow przeanalizowalem czy podwyzszenie thresholdu na alert na rate niesie ze soba ryzyko.

Okazuje sie ze ktpsy z reguly bardzo dobrze reagowaly na spike'i w racie, a rate powyzej 30k, nawet trwajacy dluzej niz kilka minut nie powodowywal duzego wzrostu laga (ktory utrzymywal sie na poziomie <= 10k, co przy throughpucie rzedu 30k/sek nie stanowi problemu). 


[Dashboard](https://us1data.long.sumologic.net/ui/#/dashboardv2/in0XYDk7Ano1qEWgAcpIuR1sBr7A2oGlFpCK4AYm0HABcVb5f5zOhkN6UnAh?variables=ktp:0000000000520B73-metrics_raw_data-27)


## Current status

#### Our goal
Have an alert which tells us whether to apply an _source_ or not.
When to apply _ingest scaling_

#### Flaky alert
If this alert could tell us about the necessity of applying onesource, then we should adjust the threshold and leave it as **actionable**.
If not, we should turn it off.

We don't have an information about lag (delay) per ktp on metrics forge


[Ingest scaling](https://docs.google.com/document/d/1RiU_hzQl1PeLA_Qpo0KXc1i8cjyQBdlAfsX0O8YJLBY/edit#)

