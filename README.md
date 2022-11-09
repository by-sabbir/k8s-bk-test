# K8s Manifests

This monorepo contains the kubernetes objects needed to deploy [bk-rest-app](https://github.com/by-sabbir/bk-rest-app)

## Set Aliases

```bash
alias k=kubectl
alias kp="kubectl get pods"
alias kd="kubectl describe"
alias ks="kubectl get services"
```

### Step 1: Matrics Server

This will allow us to get the resource consumption from cli

```bash
k apply -f metrics-server/metrics-server.yaml
```

### Step 2: Deploy Postgres in NodePort svc

- Create PV and PVC
  `k apply -f postgres/postgres-pvc-pv.yaml`

- Create ConfigMap for Postgres service
  - Update the `data` parameter at `postgres/postgres-config.yaml` then apply
  `k apply -f postgres/postgres-config.yaml`

- Deploy Postgres
  `k apply -f postgres/postgres-deployment.yaml`
  - watch the process with `kp --watch`

- Expose Postgres within Nodes
  `k apply -f postgres/postgres-service.yaml`
  - verify with `ks`

### Step 3: Configure External DNS (CloudFlare)

-