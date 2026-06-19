# gitops-manifests

Repositório **GitOps** (kustomize) que o **ArgoCD** observa. É a fonte da verdade
da POC multi-cluster: ArgoCD roda no cluster `poc-prod`, mas reconcilia workloads
nos clusters `poc-prod` e `poc-dev`.

```
.
├── argocd/apps/
│   ├── poc-argo-app-dev.yaml
│   └── poc-argo-app-prod.yaml
└── projects/poc/argo-app/
    ├── base/
    └── overlays/
        ├── dev/   # cluster poc-dev, tag sha-<7>
        └── prod/  # cluster poc-prod, tag vX.Y.Z
```

## Quem mexe em quê

- **`projects/poc/argo-app/overlays/dev`** — bump automático do CI em branch de
  validação, usando tag `sha-<7>`.
- **`projects/poc/argo-app/overlays/prod`** — bump automático da promoção para
  produção, usando tag `vX.Y.Z`.
- **`argocd/apps`** — App-of-Apps: duas `Application`, uma para cada cluster.

## Como o ArgoCD consome

O `root-poc-apps` (definido em `k8s-manifests/bootstrap/argocd/app-root.yaml`) aponta
para `argocd/apps` e cria as Applications abaixo:

| Application      | path         | namespace   |
| ---------------- | ------------ | ----------- |
| `poc-argo-app-dev`  | `projects/poc/argo-app/overlays/dev`  | `poc` no cluster `poc-dev` |
| `poc-argo-app-prod` | `projects/poc/argo-app/overlays/prod` | `poc` no cluster `poc-prod` |

Os dois overlays usam `ExternalSecret` com o mesmo `ClusterSecretStore`
`openbao-cluster-store`. No cluster `poc-dev`, esse store aponta para o OpenBao
exposto pelo cluster `poc-prod`.

Validar localmente:

```sh
kustomize build projects/poc/argo-app/overlays/dev
kustomize build projects/poc/argo-app/overlays/prod
```
