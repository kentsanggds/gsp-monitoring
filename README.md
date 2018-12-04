# GSP monitoring base

base monitoring configuration for all re-managed kubernetes infra

## Local development with minikube

### Dependencies

- virtualbox
  https://www.virtualbox.org/wiki/Downloads
  - Latest stable platform package - OSX hosts
  - Virtualbox extension pack

- kubectl
  - `brew install kubectl`

- minikube
  - `brew cask install minikube`

### Getting something running

```
# startup minikube
minikube start

# generate the output from the templates
mkdir monitoring/output
helm template monitoring --output-dir monitoring/output --name monitoring-system

# apply the output
kubectl apply -R -f monitoring/output/

# get access to grafana via http://localhost:8080
kubectl port-forward service/monitoring-system-grafana 8080:80

# get access to prometheus via http://localhost:9090
kubectl port-forward service/monitoring-system-promethe-prometheus 9090:9090
```

### Troubleshooting

If you are having problems with `minikube` it might be worth removing `minikube` and restarting it.

```
minikube delete
rm /usr/local/bin/minikube
rm -rf ~/.minikube
minikube start
```

It's also worth making sure that you are running the latest version of virtualbox.
