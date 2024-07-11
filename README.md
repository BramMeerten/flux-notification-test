## About
Contains some kubernetes resource files to setup flux and test the notification controller.
Flux watches the `resources/my-app` directory and deploys those k8s files. Alerts will be sent via http to a web server.

You can view the events via:
```bash
kubectl logs -f -n listener listener-<id>
```

## Installation
### Install flux controllers
```bash
kubectl apply -f resources/flux-system/gotk-components.yaml
```

`got-components.yaml` was generated via flux (you don't need to run this anymore):
```bash
brew install fluxcd/tap/flux
flux check --pre
flux bootstrap github \
  --owner=BramMeerten \
  --repository=flux-notification-test \
  --branch=main \
  --path=./resources/ \
  --personal
git pull

```

### Listener that will receive flux notification events
This will start a web server that listens on port 8456 and logs the incoming requests (and echos them back).
```bash
kubectl apply -f resources/listener/Listener.yaml
```

### Flux resources
This will create a `Provider` and `Alert`, it will watch the `Kustomization` resource inside the `flux-system` namespace and send events via a webhook to the listener service.
```bash
kubectl apply -f resources/flux/Notifications.yaml
```

This will create a `GitRepositories` and `Kustomization` that watches this repo and will reconciliation the k8s files inside `resources/my-app`.
After this you should see a namespace `my-app` with a Deployment called 'my-app'.
```bash
kubectl apply -f resources/flux/GitRepositories.yaml
kubectl apply -f resources/flux/Kustomization.yaml
```

