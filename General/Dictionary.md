# Shortcuts

**[[Metrics in Memory|MIM]]** - metrics in memory

**KTP** - kafka topic partition ^eac209

**NFR** - non-functional requirement

**SCR** - system change request

**RDS** - relacyjne bazy danych w AWS

**RAP** - [[Resource Aware Partitioning]] - (resource w znaczeniu memory / CPU) partitioning ktory rozrzuca “cos” po node’ach wedlug pewnej metryki zuzycia (np. CPU)

**I1S** - (i-one-s) postmortem (action outage item  - do zrobienia w max 2 tyg), wieksze rzeczy do zrobienia po outage’ach 

**GA** - general availability, mozliwosc podejmowania nowych customerow, bez koniecznosci specjalnego przygotowania infrastruktury pod nich 

**KDQ** - kafka data queue -> wewnetrzny mechanizm do czytania danych z kafki, nie wszystkie serwisy go uzywaja

**RWC** - biuro w Redwood City

**SME** - subject matter expert 



# Concepts


## Metrics namespace

**Efemerical metrics**- krotko zyjace metryki

**Carbon** - metrics format, eg. `name=cpu host=panels12 16923392 76.3`

**Metrics name** - jeden z wymiarow metryki. Najwazniejszy wymiar identyfikujacy grupe metryk opisujacy

**Metryka** - Dane ktore sa wysylane przez klientow, maja wiele wymiarow. Jednym z tych wymiarow jest **metrics name** 

**Metric definition** - metadane metryki 

## Others

**Pinowanie node’ow** - przypiecie w konfiguracji ktpsow do node’ow

**Kwantyzacja** - przy duzym timerange korzystamy raczej z rollupow niz z pojedynczych date’a pointow

==TODO: poprawic dwie ponizsze definicje==
**Ingest scaling** - receiver dostaje batch (od source'a) -> nie routuje tylko po source_id, ale tez po jakims hashu metryki albo dimentionie. Na razie jest dosyc manualny. Definiujemy ile KTPsow na ingest scaling a ile na default.

**Automatic ingest scaling** - na etapie receivera mamy customowy kod, ktory slabo reaguje na lagi i powoduje data lossy. Mamy reczne konfiguracje, ale chcemy automat.

**Rightsizing** - optymalizacja ilosci instancji vs zuzycia zasobow 

**Shards per customer** - w [[Metrics in Memory]]  mamy shardowanie metryk (po jakims kluczu) miedzy dostepnymi node’ami; tworzymy je o pelnych godzinach, ale chcemy to zmienic; tworzac sharda trzeba pushowac metadane do rdsow, poindeksowac rzeczy itd;


**Rozsmarowane tworzenie shardow** - nietworzenie shardow o pelnych godzinach

**Aggregated baseline** - (logs to metric - parsujemy logi zeby zagregowac to do metryk, trace to metrics) - zrodlo inne niz metrics receivery

**Backfilling** - monitor alertowy po starcie musi przeczytac czesc danych z przeszlosci z Metrics Query, zeby miec kontekst do alarmow, ktore sa ustawione np. na ostatnie 30 minut

**Kafka switch** -  przepiecie sie na nowy kluster kafki w przypadku np. awarii klastra, robimy to stopniowo, step by step

**Single source** - standardowy proces dodania nowego customera polegal na podpieciu roznych source'ow danych (AWS cloud watch, collector, HTTP receiver endpoint) do naszego pipeline'u; niektorzy klienci maja tylko jeden source; problem w takim przypadku polega na prawidlowym poroutowaniu / poshardowaniu danych przy wysylaniu ich dalej do kafki 

**Source** - to logiczna jednostka ktora wysyla dane. Sourcem moze byc:
- collector po stronie customera
- cloud collector
- endpoint http

**E2E test** - odpala sie na jenkinsie co jakis czas (dziala na efemeral deployment)

**Transformation rule** -  ^b39067


# Ecosystem quants

**Content repo** - dashboard as a code (terraform)

**Monitory metrykowe** - monitor jest to system, ktory wykonuje definicje alertu i w razie jej spelnienia wysyla notyfikacje (trigger / alarm ktory dzwoni customerowi). Trigger moze nastapic w kilku trybach.

**Cloud collector** - collector na pierwszym poczatku flow w diagramie ([Metrics Architecture](https://docs.google.com/document/d/1VwB0_XJz5hgh3U-h0YuR7skzYDJhdrDH9bcCltgx8yU/edit#)) ktory pulluje metryki z Cloud Watcha  i wysyla je dalej do kafki. Obsluguje rowniez requesty http od customerow z lista metryk.

**[[Deadman]]** - kod obserwujacy nasze serwisy; obserwuje offsety na kafce, czy ida do przodu


# Systems / organizations

**Signal FX** - konkurencyjna firma, przejeta przez Splunka

**Zendesk** - system do zglaszania zadan z zewnatrz

**ZAIDAN** - wewnetrzny wrapper na kubernetesa

**Lsh** - shell do local developmentu (setupuje odpowiednie assembly lokalnie)

**Dsh** - shell do developmentu w chmurze

**Boss** - odpowiednik Ansible’a, narzedzie do konfiguracji maszyn

**props** - a dsh command to update properties, which are stored in dynamodb (mainly)

**pingdom** -  our external monitoring service [Alert List](https://wiki.kumoroku.com/confluence/display/OPS/Pingdom+Alerts+List)




# Random

Tworzenie nowych assembly jest przekomplikowane.













