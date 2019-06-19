### MySQL helm chart on Kubernetes
This README describes how to install MySQL on a kubernetes cluster, based on the Bitnami Helm chart. 

# Install helm
Make sure you have helm installed.

For Mac with Homebrew, run the following:

``bash
brew install kubernetes-helm
```

For Windows with Chocolatey

```bash
choco install kubernetes-helm
```

For linux with snap:

```bash
sudo snap install helm --classic
```

Or simply run a script:

```bash
curl -L https://git.io/get_helm.sh | bash
```

# Setup tiller
Tiller is the server-side component for helm on the Kubernetes cluster. Although it will be removed in the future versions of helm, it is still needed for most Charts out there.

Setup an RBAC yaml file for tiller:

```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

Apply the yaml file:

```bash
kubectl apply -f initialize_helm_rbac.yaml
```

