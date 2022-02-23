
### Local deployment

#### How to run it?
```
> dev lsh
[lsh] run-script run-metrics.lsh
# restart metricsforge with -g flag
```

#### Send test data to service
Two ways:
- run `gen` in lsh
- try http endpoint, where you can send single data point using curl
 
#### Debugger
1. Run your code on local deployment.
2. Attach a debugger via *attach to process* option in intellij