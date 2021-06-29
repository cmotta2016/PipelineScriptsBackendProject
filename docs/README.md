## Using Chart

Add repo:  
helm repo add wae-consumer https://cmotta2016.github.io/PipelineScriptsBackendProject  

Create values.yaml with:  
```
environment: "qa"

ingress:
  enabled: true

autoscaling:
  enabled: true
```

And them install chart:  
```
helm install wae-consumer -n wae-consumer-qa wae-consumer/wae-consumer -f values.yaml
```

Or set values at install step:  
```
helm install wae-consumer -n wae-consumer-qa --set autoscaling.enabled=true --set ingress.enabled=true
```
