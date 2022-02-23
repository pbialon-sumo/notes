Metricsforge jest oparty an frameworku Spring. Framework ten opiera sie na dependency injection. Wykorzystujemy je do zdefiniowania pewnych obiektow, ktore potem sa wykorzystywane przez assembly.


Wszystko zaczyna sie na etapie `MetricsForgeProductionBeans`. 
Wsrod dependencies (`@Configuration`) jest `StreamsManagementBeans`.
W `StreamManagementBeans` mamy beana `forgeStreamApplicators` ktory tworzy pewnego aplicatora. Aplikator ten to `ForgeStream`, ktory przyjmuje metrykowe raw data i wypluwa dane do kafki. 
**Co robi `ForgeStream`?**
1. *input instrumentation*: stawia timery + loguje dane o przepustowosci itd
2. *reshuffle data*: grupuje metryki po czasie (default = max 1s, 4000 elementow; nie wiecej niz 300 output (?) time series - konfigurowalne per customer)
	process
3. prepare kafka sink



Wewnatrz `ForgeStream`:
przetwarzamy batche metryk (lista `[ktp: String, batch: MetricsBatch]` ) z kafki. `MetricsBatch` sklada sie z timeseriesow i metadanych (np. retries)

Przetwarzanie batchy:
jeden *batch*, ma w sobie:
- record (key + value z kafki a oprocz tego topic + partition number, offset)
- partitionOffset



`PastFutureLimitPointsDropper` has `filter` method which creates new batch of points, with old ones filtered out.
It is used by `SubflowBuilder`



--------

Akka streams => 8 streamow, na ktore rozrzuca 