# magazon-gitops

GitOps state for the Magazon production deployment.

## Layout

- `applications/microshop-prod.yaml`: Argo CD `Application` for production.
- `applications/ollama.yaml`: Argo CD `Application` for the Ollama runtime.
- `bootstrap/magazon-root.yaml`: root Argo CD `Application` that manages the apps in `applications/`.
- `environments/prod/values.yaml`: production Helm values consumed by Argo CD.
- `infrastructure/ollama/`: raw Kubernetes manifests for Ollama, including the PVC used to persist downloaded models.

The Helm chart stays in the application repository:

```text
https://github.com/pascariucosmin93/magazon.git
```

Argo CD uses multiple sources: the chart comes from `magazon`, while values come from this repository.

## Required setup

1. Keep `config.PUBLIC_BASE_URL` in `environments/prod/values.yaml` equal to the public URL users open in the browser. It can stay on the current IP while there is no Cloudflare domain.
2. Keep `config.OLLAMA_BASE_URL` pointed at the internal Ollama service. Current value: `http://ollama.ollama.svc.cluster.local:11434`.
3. Pre-create `microshop-secret` in the `microshop` namespace or install External Secrets/Sealed Secrets.
4. Configure Argo CD repository access for `magazon` and `magazon-gitops` if the repos are private.
5. Apply `bootstrap/magazon-root.yaml` into the `argocd` namespace.

Ollama stores models under `/root/.ollama`. The manifests in `infrastructure/ollama/`
mount that path from the `ollama-data` PVC, so models persist across pod restarts and
re-scheduling.

## CI access

The production workflow in `magazon` updates `applications/microshop-prod.yaml` and `environments/prod/values.yaml` after publishing images.
Create a GitHub secret in the `magazon` repository:

```text
GITOPS_REPO_TOKEN
```

It must have write access to `pascariucosmin93/magazon-gitops`.
