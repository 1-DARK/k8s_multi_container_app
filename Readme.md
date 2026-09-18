# K8s_multi_container_app

Express + MongoDB demo. Submit an email on the home page; it is stored in MongoDB and listed on the same page.

Use this repo in three ways:

1. Run locally with Node.js
2. Run with Docker
3. Deploy to Kubernetes (three layouts)

## Prerequisites

| How you run it | What you need |
| --- | --- |
| Local | Node.js, MongoDB on `localhost:27017` |
| Docker | Docker |
| Kubernetes | `kubectl` and a cluster (Minikube, kind, Docker Desktop, or a cloud cluster) |

MongoDB database name: `yourDatabaseName`.

## Project layout

```
index.js, index.html, Dockerfile   # app source
Both(Container)RunInSamePod/       # Node + Mongo in one pod
Both(Container)RunInDiffPod/       # Node and Mongo in separate pods
PersistentVolume/                  # same as DiffPod, Mongo data on a hostPath volume
```

## Local run

Start MongoDB on port `27017`, then:

```bash
npm install
node index.js
```

Open [http://localhost:3000](http://localhost:3000). Submit an email to save it. Stored emails also load from `GET /emails`.

The local app connects to `mongodb://localhost:27017/yourDatabaseName`.

| Route | Method | Description |
| --- | --- | --- |
| `/` | GET | Form + email list |
| `/add-email` | POST | Save an email |
| `/emails` | GET | JSON list of emails |
| `/exit` | GET | Stops the process (demo only) |

## Docker

```bash
docker build -t db-demo-app .
docker run -p 3000:3000 db-demo-app
```

This image still talks to MongoDB at `localhost:27017` inside the container. For a working setup, run MongoDB as a second container/network, or use the Kubernetes manifests below.

## Kubernetes

Pick **one** layout. Do not apply all three at the same time (they reuse names like `node-app` and `service-mongodb`).

The Node service is a LoadBalancer on port **8080** (container port `3000`).

### 1. Same pod (sidecars)

Node app and MongoDB share a pod. Mongo is reachable as `localhost` from the app container.

```bash
kubectl apply -f "Both(Container)RunInSamePod/Both(Container)RunInSamePod.yaml"
```

Image: `philippaul/node-mongo-db:02` (2 replicas).

### 2. Separate pods

MongoDB Deployment + Service, ConfigMap (`MONGO_HOST` / `MONGO_PORT`), then the Node app.

```bash
kubectl apply -f "Both(Container)RunInDiffPod/mongodb.yaml"
kubectl apply -f "Both(Container)RunInDiffPod/mongoconfig.yaml"
kubectl apply -f "Both(Container)RunInDiffPod/nodeapp.yaml"
```

The app uses `service-mongodb:27017` via ConfigMap. Image: `philippaul/node-mongo-db:04`.

Mongo data is **ephemeral**: deleting the Mongo pod loses stored emails.

### 3. Separate pods with PersistentVolume

Same networking as layout 2, plus a hostPath PV so Mongo keeps data under `/data/` on the node.

```bash
kubectl apply -f PersistentVolume/hostpv.yaml
kubectl apply -f PersistentVolume/hostpvc.yaml
kubectl apply -f PersistentVolume/mongodb.yaml
kubectl apply -f PersistentVolume/mongoconfig.yaml
kubectl apply -f PersistentVolume/nodeapp.yaml
```

Mongo mounts the claim `hostpvc` at `/data/db`.

**Note:** `hostPath` is for single-node testing. It will not move with the pod on a multi-node cluster. Use a real storage class in production.

### Access the app

```bash
kubectl get pods
kubectl get svc service-node-app
```

- Cloud / Docker Desktop LoadBalancer: open the `EXTERNAL-IP` on port `8080`.
- Minikube:

  ```bash
  minikube service service-node-app
  ```

- No LoadBalancer: port-forward

  ```bash
  kubectl port-forward svc/service-node-app 8080:8080
  ```

  Then open [http://localhost:8080](http://localhost:8080).

### Check Mongo persistence (layout 3)

Submit an email, then restart the Mongo pod and confirm the list is still there:

```bash
kubectl delete pod -l app=mongo-app
```

### Tear down

Same pod:

```bash
kubectl delete -f "Both(Container)RunInSamePod/Both(Container)RunInSamePod.yaml"
```

Separate pods:

```bash
kubectl delete -f "Both(Container)RunInDiffPod/"
```

PersistentVolume:

```bash
kubectl delete -f PersistentVolume/
```
