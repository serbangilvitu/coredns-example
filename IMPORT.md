# Import chart
https://artifacthub.io/packages/helm/coredns/coredns

```
helm repo add coredns https://coredns.github.io/helm
helm pull coredns/coredns --version 1.14.0
tar -xvf coredns-1.14.0.tgz
rm *.tgz
```

# Install chart
```
helm install coredns ./coredns -f ./coredns/values.yaml
```