## About
Contains some kubernetes resource files to setup flux and test the notification controller.
Flux watches the `resources/my-app` directory and deploys those k8s files. Alerts will be sent via http to a web server.

You can view the events via:
```bash
kubectl logs -f -n listener deployment/listener
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
kubectl apply -f resources/flux/GitRepositories.yaml && \
kubectl apply -f resources/flux/Kustomization.yaml
```

### Example Alerts
<details>
<summary>Click to expand <b>Info alert</b></summary>

```json
{
  "name": "echo-server",
  "hostname": "listener-5b78d57446-ww4k8",
  "pid": 1,
  "level": 30,
  "host": {
    "hostname": "listener.listener.svc.cluster.local",
    "ip": "::ffff:10.42.0.140",
    "ips": []
  },
  "http": {
    "method": "POST",
    "baseUrl": "",
    "originalUrl": "/",
    "protocol": "http"
  },
  "request": {
    "params": {},
    "query": {},
    "cookies": {},
    "body": {
      "involvedObject": {
        "kind": "Kustomization",
        "namespace": "flux-system",
        "name": "my-app-kustomization",
        "uid": "351d7dcc-a14b-4a58-b32b-c2e47a0d751d",
        "apiVersion": "kustomize.toolkit.fluxcd.io/v1",
        "resourceVersion": "19516"
      },
      "severity": "info",
      "timestamp": "2024-07-12T08:56:37Z",
      "message": "Namespace/my-app created\nDeployment/my-app/my-app created",
      "reason": "Progressing",
      "metadata": {
        "revision": "main@sha1:07a684d59d1bc9a58e0c0d07ed480812b2b9a29e"
      },
      "reportingController": "kustomize-controller",
      "reportingInstance": "kustomize-controller-6bc5d5b96-j9lw6"
    },
    "headers": {
      "host": "listener.listener.svc.cluster.local:8456",
      "user-agent": "Go-http-client/1.1",
      "content-length": "544",
      "content-type": "application/json",
      "gotk-component": "kustomize-controller",
      "accept-encoding": "gzip"
    }
  },
  "environment": {
    "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
    "HOSTNAME": "listener-5b78d57446-ww4k8",
    "PORT": "80",
    "LISTENER_PORT_8456_TCP": "tcp://10.43.156.207:8456",
    "KUBERNETES_PORT_443_TCP_PROTO": "tcp",
    "KUBERNETES_PORT_443_TCP_ADDR": "10.43.0.1",
    "LISTENER_SERVICE_HOST": "10.43.156.207",
    "LISTENER_PORT_8456_TCP_PORT": "8456",
    "KUBERNETES_SERVICE_HOST": "10.43.0.1",
    "KUBERNETES_SERVICE_PORT_HTTPS": "443",
    "LISTENER_SERVICE_PORT": "8456",
    "LISTENER_PORT": "tcp://10.43.156.207:8456",
    "LISTENER_PORT_8456_TCP_PROTO": "tcp",
    "LISTENER_PORT_8456_TCP_ADDR": "10.43.156.207",
    "KUBERNETES_SERVICE_PORT": "443",
    "KUBERNETES_PORT_443_TCP": "tcp://10.43.0.1:443",
    "KUBERNETES_PORT_443_TCP_PORT": "443",
    "KUBERNETES_PORT": "tcp://10.43.0.1:443",
    "NODE_VERSION": "20.11.0",
    "YARN_VERSION": "1.22.19",
    "HOME": "/root"
  },
  "msg": "Fri, 12 Jul 2024 08:56:37 GMT | [POST] - http://listener.listener.svc.cluster.local:8456/",
  "time": "2024-07-12T08:56:37.078Z",
  "v": 0
}
```
</details>

<details>
<summary>Click to expand <b>Error alert</b></summary>

```json
{
  "name": "echo-server",
  "hostname": "listener-5b78d57446-ww4k8",
  "pid": 1,
  "level": 30,
  "host": {
    "hostname": "listener.listener.svc.cluster.local",
    "ip": "::ffff:10.42.0.140",
    "ips": []
  },
  "http": {
    "method": "POST",
    "baseUrl": "",
    "originalUrl": "/",
    "protocol": "http"
  },
  "request": {
    "params": {},
    "query": {},
    "cookies": {},
    "body": {
      "involvedObject": {
        "kind": "Kustomization",
        "namespace": "flux-system",
        "name": "my-app-kustomization",
        "uid": "351d7dcc-a14b-4a58-b32b-c2e47a0d751d",
        "apiVersion": "kustomize.toolkit.fluxcd.io/v1",
        "resourceVersion": "19365"
      },
      "severity": "error",
      "timestamp": "2024-07-12T08:54:33Z",
      "message": "kustomization path not found: stat /tmp/kustomization-644156964/cluster-resources/my-app: no such file or directory",
      "reason": "ArtifactFailed",
      "metadata": {
        "revision": "main@sha1:07a684d59d1bc9a58e0c0d07ed480812b2b9a29e"
      },
      "reportingController": "kustomize-controller",
      "reportingInstance": "kustomize-controller-6bc5d5b96-j9lw6"
    },
    "headers": {
      "host": "listener.listener.svc.cluster.local:8456",
      "user-agent": "Go-http-client/1.1",
      "content-length": "605",
      "content-type": "application/json",
      "gotk-component": "kustomize-controller",
      "accept-encoding": "gzip"
    }
  },
  "environment": {
    "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
    "HOSTNAME": "listener-5b78d57446-ww4k8",
    "PORT": "80",
    "LISTENER_PORT_8456_TCP": "tcp://10.43.156.207:8456",
    "KUBERNETES_PORT_443_TCP_PROTO": "tcp",
    "KUBERNETES_PORT_443_TCP_ADDR": "10.43.0.1",
    "LISTENER_SERVICE_HOST": "10.43.156.207",
    "LISTENER_PORT_8456_TCP_PORT": "8456",
    "KUBERNETES_SERVICE_HOST": "10.43.0.1",
    "KUBERNETES_SERVICE_PORT_HTTPS": "443",
    "LISTENER_SERVICE_PORT": "8456",
    "LISTENER_PORT": "tcp://10.43.156.207:8456",
    "LISTENER_PORT_8456_TCP_PROTO": "tcp",
    "LISTENER_PORT_8456_TCP_ADDR": "10.43.156.207",
    "KUBERNETES_SERVICE_PORT": "443",
    "KUBERNETES_PORT_443_TCP": "tcp://10.43.0.1:443",
    "KUBERNETES_PORT_443_TCP_PORT": "443",
    "KUBERNETES_PORT": "tcp://10.43.0.1:443",
    "NODE_VERSION": "20.11.0",
    "YARN_VERSION": "1.22.19",
    "HOME": "/root"
  },
  "msg": "Fri, 12 Jul 2024 08:54:33 GMT | [POST] - http://listener.listener.svc.cluster.local:8456/",
  "time": "2024-07-12T08:54:33.793Z",
  "v": 0
}
```
</details>
