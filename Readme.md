# k8s_multi_container_app

Small Express + MongoDB demo that stores emails in a form and lists them on the same page.

## Prerequisites

- Node.js
- MongoDB running on `localhost:27017` (database name: `yourDatabaseName`)

## Local run

```bash
npm install
node index.js
```

Open [http://localhost:3000](http://localhost:3000). Submit an email to persist it; stored emails load from `GET /emails`.

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

The container expects MongoDB at `localhost:27017` inside the container. For a real run, point the app at a reachable MongoDB host (or use the Kubernetes manifests below).

## Kubernetes

Two layouts are included:

**Same pod** — Node app and MongoDB as sidecars:

```bash
kubectl apply -f Both\(Container\)RunInSamePod/Both\(Container\)RunInSamePod.yaml
```

**Separate pods** — MongoDB Deployment + Service, ConfigMap, then the Node app:

```bash
kubectl apply -f Both\(Container\)RunInDiffPod/mongodb.yaml
kubectl apply -f Both\(Container\)RunInDiffPod/mongoconfig.yaml
kubectl apply -f Both\(Container\)RunInDiffPod/nodeapp.yaml
```

The Node service is a LoadBalancer on port `8080` (targets container port `3000`).
