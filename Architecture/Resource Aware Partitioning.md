## Introduction

==TODO==
There is a [design doc](https://docs.google.com/document/d/1aYMXJOh-wp46GI577l7gIeRAHy45gSbIAN02tTMHjTU/edit) for RAP.

## Autoscaling


## Partitioning


## Known problems

This part of code reuses some structures from Log's Partitioning, which is a bit different from ours. Therefore it's:
- quite complicated and bloated.
- not well tested

Furthermore it's a _critical part of a system_. If partitioning failed everything would fail and stop working.

```ad-warning
title: Do not count on Deadman

If there is a bug in the partitioning algorithm (e.g. some KTPs are left unassigned to any node) then deadman will not call!
Beware that deadman observes only KTPs that are assigned.
```


## Ideas for the future

```ad-hint
title: Improvements for the KTPRebalancer

Check if there is any sense in global rebalancing before performing it. There are two situations when it is pointless to do it:
- traffic is really low and inbalance is not causing any troubles
- after changes there is no significant improvement
```

```ad-hint
title: Normalization of partitioning

In Alert's Partitioning we use Sumo's metrics to get current rates.
However in Metrics Forge Partitioning (MFP) we pull the metrics from the cluster nodes. It makes sense to change the source of the metrics in the MFP to Sumo.

**Possible obstacles**
- firewall (blocking queries to Sumo on some org)
- client generated from OpenAPI has some problems on production (throwed random exceptions), while working OK on local environment (Wojtek's experience)

```



----


`resourceUsageJobs`

Lider w metricsforge odpytuje node'y o throughputy na ich ktpsach.

[https://github.com/Sanyaku/sumologic/blob/1bb6c8995733685b99dc8685c8c305222a00bc8f/metricsforge/src/main/scala/com/sumologic/metricsforge/MetricsForgeProperties.scala#L200](https://github.com/Sanyaku/sumologic/blob/1bb6c8995733685b99dc8685c8c305222a00bc8f/metricsforge/src/main/scala/com/sumologic/metricsforge/MetricsForgeProperties.scala#L200)

System do autoskalowania ktpsow (zwieksza liczbe partycji, ale nie dostawia node'ow => trzeba to zrobic recznie).


### Autoscaling

Strategia skalowania ktpsow.
`ktpScalingStrategyPicker`

Metricsforge obserwuje ktpsy z ktorych czyta i do ktorych czyta (troche sredni design - moze osobny mikroserwis) i decyduje czy trzeba je skalowac.


`metricsKTPPartitioner` - [https://github.com/Sanyaku/sumologic/blob/3d3c07e7e766ad23ec040203df0b7f46a51c2a50/metricsforge/src/main/scala/com/sumologic/metricsforge/PartitioningAssignerBeans.scala#L183](https://github.com/Sanyaku/sumologic/blob/3d3c07e7e766ad23ec040203df0b7f46a51c2a50/metricsforge/src/main/scala/com/sumologic/metricsforge/PartitioningAssignerBeans.scala#L183)


Mutatory requestujace zmiany do partitioningu.
Jest `EmergencyKTPUnassigner` ktory probuje 



**resourceUsageCache**

W Metrics Forge wyciagamy dane o rate (dps/sec) odpytujac poszczegolne node'y o metryki.
W Alertach wyciagamy dane o rate z Sumo.

=> warto ustandaryzowac

**Problemy**





----
Lag na metrykach (p99 1-2min) wplywa na alerty ktore maja krotkie okno (np. alerty na okno 2 minutowe dla Acqui).
Ale wiekszosc monitorow ma okna 10-15 minutowe. 


----


Autoscaler vs Rebalancing
Dzialaja "w miare niezaleznie".
KTPAutoscaler => jakie obciazenie ma ktps
Rebalancing => jakie obciazenie ma node

KTPAutoscaler zwieksza liczbe partycji na topicu, zeby mozna bylo je "latwiej" rozrzucic pomiedzy node'y

Jak pchniemy za duzo na wszystkie node'y na klastrze (rownomiernie) to rebalancing stwierdzi ze wszystko jest OK.


#### Korzystania z metryk z Sumo w MetricsForge
MetricsForge odpytuje node'y o metryki. Moznaby pobierac je z Sumo jak to robi Alert.