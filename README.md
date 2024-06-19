## Setup LGTM
On kubernetes cluster
```bash
# Add Helm repository
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Deploy Loki
kubectl create namespace loki
helm install loki grafana/loki-stack --namespace=loki

# Deploy Grafana
kubectl create namespace grafana
helm install grafana grafana/grafana --namespace=grafana
kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
kubectl port-forward --namespace grafana svc/grafana 3000:80

# Deploy Tempo
kubectl create namespace tempo
helm install tempo grafana/tempo --namespace=tempo

# Deploy Mimir
kubectl create namespace mimir
helm install mimir grafana/mimir-distributed --namespace=mimir
```


## Troubleshoot
- PVC pending state
    - Set default dynamic storage class:
        ```bash
        kubectl patch storageclass csi-rbd-sc -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
        ```

- mimir-nginx CrashLoopBackOff
    - Edit mimir-nginx configmap and match the resolver endpoint
        ```bash
        # Edit configmap
        kubectl edit cm mimir-nginx -n mimir

        # Match resolver endpoint
        apiVersion: v1
        data:
            nginx.conf: |
                ...
                http {
                    ...
                    resolver coredns.kube-system.svc.cluter.simple.;
                }
        ```
- Datastore: Unable to connect with Loki
    - Edit loki-stack values.yml
        ```bash
        # get values.yml
        helm show values grafana/loki-stack > loki.yaml

        # use the correct image tag
        loki:
          ...
          image:
            repository: grafana/loki
            tag: 2.9.3

        # run helm upgrade
        helm upgrade --install  loki grafana/loki-stack -f loki.yaml -n loki
        ```