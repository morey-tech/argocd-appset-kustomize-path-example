# Argo CD ApplicationSet Kustomize-Only Example

## Stand Up Argo CD
- Open in devcontainer.
- Run commands:
```
akuity login
akuity config set organization-name=morey-tech
akuity argocd apply -f argocd/
akuity argocd cluster get-agent-manifests kind --instance-name=argocd-appset-kustomize-path-example | kubectl apply -f -
```

## Example Directories
```
/app1
    kustomization.yaml
/app2
    test.yaml
/app3
    kustomization.yaml
```

We want an Application for `app1` and `app3`, but not `app2` because it doesn't contain a `kustomization.yaml`.

## The `directories` `git` Generator
A limitation of the `directories` `git` generator because it's only aware of the directories, not files. So you can't filter based on files in the directories.

```yaml
spec:
  generators:
    - git:
        directories:
          - path: '*/kustomization.yaml'
          - exclude: true
            path: app2
```

The simplest solution, if you need to stick with the `directories` `git` generator, is to use the exclude for `app2`:

```yaml
spec:
  generators:
    - git:
        directories:
          - path: '*'
          - exclude: true
            path: app2
```

## The `files` `git` Generator
The best solution (in my opinion), is to use [the `files` `git` generator](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Git/#git-generator-files) to filter based on directories with a kustomization.yaml. The limitation here is that you can't [not `exclude` directories](https://github.com/argoproj/argo-cd/issues/13690), like in the `directories` generator.
```yaml
  generators:
    - git:
        repoURL: https://github.com/morey-tech/argocd-appset-kustomize-path-example
        revision: HEAD
        files:
          - path: '*/kustomization.yaml'
```