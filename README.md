# compdev26-gitops

Desired Kubernetes state for the CompDev2026 AKS lab. Terraform (`compdev26-infrastructure`) owns Azure resources and shared cluster services; this repository owns the application objects. No object should have both owners.

## Layout

```text
apps/guestbook/base/          Environment-neutral Deployment, Service, PDB, HTTPRoute, ServiceAccount, ConfigMap
environments/lab/guestbook/   Lab overlay: image digest, Cosmos DB identifiers, identity client ID, hostname, parent Gateway
bootstrap/projects/           Argo CD AppProject: allowed repo, destination namespace and resource kinds
bootstrap/applications/       Argo CD Application with automated sync, prune and self-heal
```

The overlay contains only non-secret identifiers. The Cosmos endpoint and the managed-identity client ID come from the `workloads/guestbook` Terraform outputs. Never commit secrets, keys, connection strings, or kubeconfig files.

## Deployment

Argo CD syncs `environments/lab/guestbook` from `main` automatically, prunes objects removed from Git, and reverts manual changes to live objects. Change the cluster by merging to `main`, not with `kubectl`.

The `AppProject` and `Application` in `bootstrap/` are not synced by Argo CD itself. After changing them, apply them once, project first:

```powershell
kubectl apply -f bootstrap/projects/guestbook.yaml
kubectl apply -f bootstrap/applications/guestbook.yaml
```

The `guestbook` namespace is created by Terraform (`platform/`). Preview what Argo CD will apply with `kubectl kustomize environments/lab/guestbook`.

## Update the image

Take the digest from the `compdev26-guestbook` workflow summary and set it in `environments/lab/guestbook/kustomization.yaml`. Images must be pinned by digest, never by tag.

## Validation

The `validate` workflow renders the overlay, requires digest-pinned images, rejects committed secrets, and shows the rendered difference on pull requests. Schema validation of Kubernetes and Gateway API resources is not added yet.
