[Github](https://github.com/Sanyaku/sumologic/pull/107469)

## Manual Override Parititioner

`ManualParititioningResult`:
	- mapa
	- excludowane node'y (?)
	- tokeny przypinowane do nieistniejacych node'ow
	- 
#### Partition (ok)
1. createMapping()
	1. prepare assignment from properties
	2. clear assignment out off wrong tokens  
2. ()
3. try to do fallback
#### Split



### Uwagi
1. `endsWith(-1)?` zamiast komentarza dac jakas zmienna/funkcje

handleBadManualPinnings jako strategia?
2. W `partition` mamy if(fallbackAssignment.nonEmpty) i tylko wtedy logujemy. Ale jezeli mnie ma node'a konczacego sie na "-1" to tez mamy blad, ale nie ma logowania?
3. Po co ten split, czym sie rozni od partition? Czemu w split nie robimy fallback assignment?



MonitorAssignment (ile monitorow na ktp) -> dynamic propsy (max monitor per ktp). Wklada monitory na ktp. 

W monitorach mozna zdefiniowac dowolne query. Nie wiemy ile metryk uzyjemy


Poczytac o terraformie. 

Jak obslugiwac partitioning?


### Zyski vs straty
**TODO**