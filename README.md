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

- We need to get CloudFlare API KEY and Email
- Update the `env` at `dns/external-dns.yaml` then
  `k apply -f dns/external-dns.yaml`
  - verify with `kp --watch`

### Step 4: Deploy the `bk-app`
- Update the ConfigMap for postgres and application specific env vars at `bk-rest-app/bk-app-config.yaml`
  `k apply -f bk-rest-app/bk-app-config.yaml`
  - verify with `k get cm`
- Create docker registry secret
  - Here I have used my [private registry](registry.sabbir.dev), so the `dockerconfigjson` was omitted. You can use any public registry or private ones.
    - Generate and create the docker-registry secret with imperative command
      `k create secret docker-registry registry.sabbir.dev --docker-username=<username> --docker-password=<password> --docker-server=registry.sabbir.dev --dry-run=client -o yaml`
    - Replace `<username>` and `<password>` with the target registry credential.
- Deploy App 
  `k apply -f bk-rest-app/bk-app-deployment.yaml`
  verify with- `kp --watch`

- Expose application globally
  - Edit the `externalIPs` and `annotations` then run-
  `k apply -f bk-rest-app/bk-svc.yaml`