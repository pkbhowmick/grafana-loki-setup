### Vector Installation

```bash
helm repo add vector https://helm.vector.dev
helm repo update
helm install vector vector/vector -n vector --create-namespace --values=sample-values.yaml
```

Note: In values file, loki's service address should be added in `.values.customConfig` section & 
configure `.values.role` to run it in different mode.

#### Todo
Configure level in pod logs in kubernetes. 
