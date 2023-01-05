## this is an experiment of deploying app of apps pattern with argoCD

The entrypoint argocd app will deploy 2 argocd apps in order of syncwave annotation stated in each app.

## Prerequisites

### Custom health check

`argocd-cm` ConfigMap needs to be updated with custom health check logic as [stated here] (https://argo-cd.readthedocs.io/en/stable/operator-manual/health/#argocd-app)


```
kubectl patch cm/argocd-cm -n argocd --type=merge \
-p='{"data":{"resource.customizations.health.argoproj.io_Application":"hs = {}\nhs.status = \"Progressing\"\nhs.message = \"\"\nif obj.status ~= nil then\n  if obj.status.health ~= nil then\n    hs.status = obj.status.health.status\n    if obj.status.health.message ~= nil then\n      hs.message = obj.status.health.message\n    end\n  end\nend\nreturn hs\n"}}'
```

Use argoCD UI to create app-of-apps or `kubectl apply` the manifest:

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: arc-app-of-apps
  namespace: argocd
spec:
  source:
    path: argocd/applications
    repoURL: 'https://github.com/squoc/arc-argocd-deployment'
    targetRevision: main
  destination:
    namespace: argocd
    server: 'https://kubernetes.default.svc'
  project: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    retry:
      limit: 5
      backoff:
        duration: 5s
        maxDuration: 3m0s
        factor: 2
    syncOptions:
      - CreateNamespace=true
```