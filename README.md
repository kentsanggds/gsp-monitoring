# GSP monitoring base

base monitoring configuration for all re-managed kubernetes infra

## Monitoring kubernetes cluster contents

- FluentD cloudwatch
  - Ships logs to AWS cloudwatch

- Grafana

- Kube state metrics
  - monitors API events in kubernetes

- Prometheus-operator
  - Manages Prometheus pods:

    - Prometheus
    - Prometheus Node exporter

## Development using an AWS stack via kops

This should only be necessary if you need to deploy your cluster onto AWS in order to test a chart that uses AWS resources such as cloudwatch.

### Dependencies

- aws-vault
  - `brew cask install aws-vault`

- kubectl
  - `brew install kubectl`

- kops
  - `brew cask install kops`

### kops setup

Create an s3 bucket to hold the kops state:

```
bucket_name=<your kops state bucket name>
aws-vault exec <your AWS profile name> -- aws s3api create-bucket --bucket ${bucket_name} --region <AWS region> --create-bucket-configuration LocationConstraint=<AWS region>
```

Setup your environment:

```
export KOPS_CLUSTER_NAME=kens-k8s.dev.gds-reliability.engineering
export KOPS_STATE_STORE=s3://kens-test-kops-state-store
```

Create the kops cluster:

```
aws-vault exec <your AWS profile name> -- kops create cluster \
--node-count=<number of nodes, eg 1> \
--node-size=<instance size, eg t2.medium> \
--zones=<AWS region and zone, eg eu-west-2a> \
--name=${KOPS_CLUSTER_NAME}
```

Preview and edit any changes:

```
aws-vault exec <your AWS profile name> -- kops edit cluster --name ${KOPS_CLUSTER_NAME}
```

Once you are happy with the generated Chart to deploy the changes:

```
aws-vault exec <your AWS profile name> -- kops update cluster --name ${KOPS_CLUSTER_NAME} --yes
```

After a few minutes the cluster should be running, you can check this with:

```
aws-vault exec <your AWS profile name> -- kops validate cluster
```

Once it is valid you can deploy the KubeYAML:

```
helm template monitoring --output-dir monitoring/output --name monitoring-system
aws-vault exec <your AWS profile name> -- kubectl apply -R -f monitoring/output/
```

You can also deploy the [kubernetes dashboard](https://github.com/kubernetes/dashboard) to your cluster:

```
aws-vault exec <your AWS profile name> -- kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

In order to access the web apps on the cluster:

```
aws-vault exec <your AWS profile name> -- kubectl proxy
aws-vault exec <your AWS profile name> -- kubectl port-forward service/monitoring-system-grafana 8080:80
aws-vault exec <your AWS profile name> -- kubectl port-forward service/monitoring-system-promethe-prometheus 9090:9090
```

Kubernetes dashboard: http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/overview?namespace=default

To get access the kubernetes dashboard: 

```
Basic auth login, using Admin as the username:
aws-vault exec <your AWS profile name> -- kops get secrets kube --type secret -oplaintext

Access token:
aws-vault exec <your AWS profile name> -- kops get secrets admin --type secret -oplaintext
```

Grafana: http://localhost:8080

Prometheus: http://localhost:9090/graph

### Tearing down your cluster

As your cluster will have been deployed to AWS it's important to remove your cluster after development is complete to minimize costs:

```
aws-vault exec <your AWS profile name> -- kops delete cluster --name ${KOPS_CLUSTER_NAME}
```

## Local development with minikube

Note - You will need to disable VPN in order for minikube to work

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
