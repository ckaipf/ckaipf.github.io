# Enable monitoring of blocked and passthrough external traffic in Istio

By default the `destination.address` is not reported when the destination is outside of the service mesh. This can be changed by using Istio's *telemetry API*. 

To add the address to the `TCP_CLOSED_CONNECTION` metric in the mesh defaults, the following manifest can be used.

```yaml
apiVersion: telemetry.istio.io/v1
kind: Telemetry
metadata:
  name: mesh-default
  namespace: istio-system
spec:
  metrics:
  - overrides:
    - match:
        metric: TCP_CLOSED_CONNECTIONS
      tagOverrides:
        destination_address:
          value: string(destination.address)
    providers:
    - name: prometheus
```

The destination IP is then added to the metric in prometheus and can be temporarily used to monitor traffic.

To report also the destination port, add 

```yaml
destination_port:
  value: string(destination.port)
```

to your defaults.
