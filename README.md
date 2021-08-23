# Consul Transparent Proxies on OpenShift

## Prerequisites

- Consul CLI
- OpenShift (using Code Ready Containers)
  - RedHat account
  - Installer
  - CRC pull file
- Helm

### Code Ready Containers

- <https://access.redhat.com/documentation/en-us/red_hat_codeready_containers/1.30/html/getting_started_guide/installation_gsg#macos>
- <https://learn.hashicorp.com/tutorials/consul/kubernetes-openshift-red-hat>

## OpenShift installation

```shell
crc setup
crc start
# add "oc" command to the environment
eval $(crc oc-env)
# add "podman" command to the environment
eval $(crc podman-env)
# You can copy this from the CRC app menu in the top bar on macOS
oc login -u kubeadmin -p <redacted> https://api.crc.testing:6443
```

### Get credentials from CLI

```shell
oc console --credentials
```

## Consul installation

### Create new project

```shell
oc new-project consul
kubectl cluster-info
```

### Add Consul Helm chart

```shell
helm repo add hashicorp https://helm.releases.hashicorp.com
helm search repo hashicorp/consul
helm repo update
```

### Deploy consul

```shell
helm install -f config.yaml consul hashicorp/consul --wait
watch kubectl get pods
```

### Accessing Consul

This needs to be running as long as you want to access Consul

```shell
kubectl port-forward consul-server-0 8500:8500
```

Consul UI: <http://localhost:8500/ui/>

```shell
export CONSUL_HTTP_ADDR=http://127.0.0.1:8500
consul members
```

## Deploy server & client

```shell
# name: static-server
kubectl apply -f server.yaml
kubectl apply -f client.yaml
```

Youn can deploy additional services with:

```shell
# name: static-server-2
kubectl apply -f server2.yaml
# name: static-server-3
kubectl apply -f server3.yaml
```

### Test

```shell
kubectl exec deploy/static-client -it /bin/sh
curl -s http://static-server/
# output should be (with quotes): "hello world"

# see envoy listeners
curl -s http://127.0.0.1:19000/listeners

# see envoy config
curl -s http://127.0.0.1:19000/config_dump
```

## Resources

- <https://access.redhat.com/documentation/en-us/red_hat_codeready_containers/1.30/html/getting_started_guide/installation_gsg#macos>
- <https://learn.hashicorp.com/tutorials/consul/kubernetes-openshift-red-hat>
- <https://www.consul.io/docs/k8s/installation/install>
- <https://www.consul.io/docs/k8s/helm>
- <https://www.consul.io/docs/k8s/connect>
- <https://www.consul.io/docs/connect/transparent-proxy#>
- <https://learn.hashicorp.com/collections/consul/developer-mesh>
- <https://www.hashicorp.com/blog/transparent-proxy-on-consul-service-mesh>
- <https://learn.hashicorp.com/tutorials/consul/kubernetes-secure-agents>
- <https://learn.hashicorp.com/tutorials/consul/service-mesh-application-secure-networking>
- <https://console-openshift-console.apps-crc.testing/topology/ns/consul?view=graph>
- <https://crc.dev/crc/#setting-up-remote-server_gsg>

