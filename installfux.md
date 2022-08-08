## Check Flux Installation
  `flux check --pre`

## Connect To GitHub

The command below not only connects Flux to GitHub, but it creates a new repo called `flux-fleet` where you can manage all of your clusters and repos/sources from one place.

Create a new file called `git.sh`


Add the following code into the bash script
```
#!/bin/bash -e
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=fleet-gitops \
  --branch=$BRANCH \
  --path=./clusters/minikube \
  --namespace=$namespace \
  --personal
```

Run the script:
```
cloudtruth --project flux run -- ./git.sh
```

You should see an output similar to the below:
```
► applying source secret "fluxnamespace/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ committed sync manifests to "main" ("a5a833e6c72d50b88ba63b144a1dda9986dc2491")
► pushing sync manifests to "https://github.com/adminturneddevops/fleet-gitops.git"
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for Kustomization "fluxnamespace/fluxnamespace" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ helm-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
```

## Flux-fleet repo

You're going to need to perform your flux commands from the `flux-fleet` repository.

1. Clone the `flux-fleet` repository to your `localhost`
2. `cd` into the `flux-fleet` repo


## Add the repo/source to Flux
Create a new file called `flux.sh`

Add the following code into the bash script
```
#!/bin/bash -e
flux create source git nginxdeployment \
  --url=$URL \
  --branch=$BRANCH \
  --interval=30s \
  --namespace=$namespace \
  --export > ./clusters/minikube/nginxdeployment-source.yaml
```

Run the script:
```
cloudtruth --project flux run -- ./flux.sh
```

You should see the following context in the `nginxdeployment-source.yaml` config:

```
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: nginxdeployment
  namespace: fluxnamespace
spec:
  interval: 30s
  ref:
    branch: main
  url: https://github.com/adminturneddevops/DevRel-As-A-Service
```

Push the config to GitHub
```
git add -A && git commit -m "Add nginxdeployment GitRepository"
git push
```

You can now set up your deployment to deploy your app with Flux
