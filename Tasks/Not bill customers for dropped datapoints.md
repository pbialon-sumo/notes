### Notes

niezakommitowanie offsetu, jezeli wytniemy calego batcha?


`groupedWithin` -> empty groups will not be emitted if no elements were received from upstream.

`batch.record.topic` - nazwa klienta?



```scala
PastFutureLimitPointsDropper()

private val pastLimit = MetricsCommonProperties.IngestPointsTooFarInPastLimit.get(customerId)
private val futureLimit = MetricsCommonProperties.IngestPointsTooFarInFutureLimit.get(customerId)
```


### Current status
1. Explore how each of the customers is affected by dropping old datapoints in terms of billing.
2. Talk with `#talk-collection` channel on slack on the future actions. Add Szymon for that


### Sidenotes
1. Team collection can do for us some small stuff (what can give us quick-wins)
2. If not (1): take over part of receiver and do the necessary fixes. How?
	Spit it to :
	- logics (in our team)
	- quota & authorization & authentication (in team collection)
3. Ad (2) => Szymon works on the splitting the receiver.

