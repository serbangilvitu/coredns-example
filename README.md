## Helm repo and value file
Add Helm repo
```
helm repo add coredns https://coredns.github.io/helm
```

Get value file
```
helm show values coredns/coredns --version 1.14.0 > values.yaml
```

## Configuring forwarder
Following configuration (in `values.yaml`) will forward requests for `example.org` to `1.1.1.1`, and requests for `wikipedia.org` to `8.8.8.8`.
This is already included in the example `values.yaml`
```
servers:
- zones:
  - zone: example.org.
  port: 53
  plugins:
  - name: errors
  # Serves a /health endpoint on :8080, required for livenessProbe
  - name: health
    configBlock: |-
      lameduck 5s
  # Serves a /ready endpoint on :8181, required for readinessProbe
  - name: ready
  - name: forward
    parameters: . 1.1.1.1:53
- zones:
  - zone: wikipedia.org.
  port: 53
  plugins:
  - name: errors
  - name: health
    configBlock: |-
      lameduck 5s
  - name: ready
  - name: forward
    parameters: . 8.8.8.8:53
```

## Install chart
```
helm upgrade -i dns-forwarder coredns/coredns --version 1.14.0 -f values.yaml
```

## Test
Start a test pod
```
kubectl run -it --rm --restart=Never --image=alpine:3.12 connect -- sh
```

You should be able to resolve `example.org` and `wikipedia.org` using the newly deployed CoreDNS
```
nslookup example.org dns-forwarder-coredns.dns.svc.cluster.local
nslookup wikipedia.org dns-forwarder-coredns.dns.svc.cluster.local
nslookup en.wikipedia.org dns-forwarder-coredns.dns.svc.cluster.local
```

## Local deployment with Docker
For a quick local deployment using Docker, create a Corefile under `/home/$USER/coredns/` and run
```
docker run -d --name coredns --restart=always --volume=/home/$USER/coredns/:/root/ -p 8053:53/udp coredns/coredns -conf /root/Corefile
```
And a query to check if it's working as expected
```
dig example.org @127.0.0.1 -p 8053
```