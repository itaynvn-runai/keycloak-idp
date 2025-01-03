# keycloak-idp
## description
- installs keycloak
- creates realm/users/groups/clients
- integrates the SAML client with a self-hosted Run:AI ctrl plane

## instructions

**1) create keycloak namespace**
```bash
kubectl create namespace keycloak
```

**2) duplicate TLS secret** (requires `yq` utility)
```bash
kubectl -n runai get secret runai-cluster-domain-tls-secret -o yaml | \
yq eval '
  .metadata.namespace = "keycloak" |
  del(.metadata.creationTimestamp) |
  del(.metadata.resourceVersion) |
  del(.metadata.selfLink) |
  del(.metadata.uid)
' - | \
kubectl apply -f -
```

**2) create configmap for keycloak realm data:**
```bash
kubectl -n keycloak create configmap keycloak-realm-data \
--from-file realm.json
```

**3) create secret for runai values**
```bash
kubectl -n keycloak create secret generic runai-ctrl-plane-data \
--from-literal=RUNAI_CTRL_PLANE_URL=placeholder \
--from-literal=RUNAI_ADMIN_USERNAME=placeholder \
--from-literal=RUNAI_ADMIN_PASSWORD=placeholder
```

**4) install keycloak**
```bash
helm install keycloak bitnami/keycloak \
-n keycloak \
-f keycloak_helm_values.yaml \
--debug
```
before moving to next step, verify
- installation is complete
- pods are in running state
- keycloak URL is accessible

**5) apply the post install job**
```bash
kubectl -n keycloak apply -f job.yaml
```
this job creates realm/users/groups/clients, then integrates the SAML client with the self-hosted Run:AI ctrl plane.