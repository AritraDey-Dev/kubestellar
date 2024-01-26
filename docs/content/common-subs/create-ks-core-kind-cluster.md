<!--create-ks-core-kind-cluster-start-->
create the ks-core kind cluster
```shell
KUBECONFIG=~/.kube/config kind create cluster --name ks-core --config - <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 443
    hostPort: {{ config.ks_kind_port_num }}
    protocol: TCP
EOF
```

Be sure to create an ingress control with SSL passthrough to **ks-core**. This is a special requirement for Kind that allows access to the KubeStellar core running on **ks-core**.
```shell
KUBECONFIG=~/.kube/config kubectl \
  create -f https://raw.githubusercontent.com/kubestellar/kubestellar/release-0.14/example/kind-nginx-ingress-with-SSL-passthrough.yaml
```
**Wait about 20 seconds** and then check if the ingress control is ready on **ks-core**:
```shell
sleep 20

KUBECONFIG=~/.kube/config kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```
<!--create-ks-core-kind-cluster-end-->
