# gitops-manifests

Repositório **GitOps** (kustomize) que o **ArgoCD** observa. É a *fonte da verdade* do que
roda no cluster `kind` do lab — o ArgoCD reconcilia o cluster com este repo.

```
.
├── argocd-app/   # API FastAPI (repo de source: victor-ln/argocd-app)
│   └── kustomization.yaml  ← o CI do argocd-app dá `kustomize edit set image` aqui
└── podinfo/      # app de demo do OpenBao (cofre) — manifestos vindos do lab de infra
```

## Quem mexe em quê

- **`argocd-app/`** — o CI do repo `argocd-app` (job `bump`) atualiza a **tag da imagem**
  no `kustomization.yaml` a cada release. Humano normalmente não edita a tag à mão.
- **`podinfo/`** — editado por humano (é a demo do OpenBao; não tem CI de bump).

## Como o ArgoCD consome

Dois `Application` (definidos no repo de infra `k8s-manifests`, em `argocd/`) apontam para
cá, cada um num `path`:

| Application      | path         | namespace   |
| ---------------- | ------------ | ----------- |
| `argocd-app`     | `argocd-app` | `argocd-app`|
| `podinfo-gitops` | `podinfo`    | `prod-apps` |

> Nenhum overlay declara o recurso `Namespace`: o ArgoCD cria via `CreateNamespace=true`
> (evita dois Apps disputando o mesmo objeto Namespace).

Validar um overlay localmente: `kustomize build argocd-app` / `kustomize build podinfo`.
