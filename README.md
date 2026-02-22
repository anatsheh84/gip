# gip — GitOps & Infrastructure Policies

A personal sandbox repository containing a mix of OpenShift application manifests, RHACM governance policies, and demo workloads used for GitOps, traffic splitting, autoscaling, and storage snapshotting exercises.

---

## Repository Structure

| Directory | Description |
|-----------|-------------|
| [`governance-policies`](./governance-policies) | RHACM governance policies for PVC storage limits |
| [`pacman-simple`](./pacman-simple) | Standard Pacman two-tier app (Node.js + MongoDB) |
| [`pacman-multi`](./pacman-multi) | Pacman with blue/green traffic splitting via OpenShift Route |
| [`php-scale`](./php-scale) | PHP app with split Route and load-generator for HPA demos |
| [`snapapp`](./snapapp) | Image-tool app using ODF/CephFS PVC; includes Ansible-ServiceNow integration stub |

---

## Directory Details

### `governance-policies` — RHACM PVC Policies

Two RHACM policy manifests for enforcing PVC storage limits across managed clusters.

| File | Policy Name | Rule | Target Namespace | ClusterSet |
|---|---|---|---|---|
| `10g-pvc-size-policy.yaml` | `pvc-max10g` | Max **10Gi** per PVC (LimitRange) | All `app*` namespaces | `remote` |
| `pvc.yaml` | `policy-pvc` (template only) | Min **1Gi**, Max **50Gi** per PVC (LimitRange) | All | — |

- `10g-pvc-size-policy.yaml` is a complete, self-contained manifest (Policy + Placement + PlacementBinding) aligned to **NIST SP 800-53 CM-2**. Deploy to the `governancepolicies` namespace on the hub cluster.
- `pvc.yaml` is a partial policy template (spec only) intended to be embedded in a parent Policy resource.

```bash
oc apply -f governance-policies/10g-pvc-size-policy.yaml
```

---

### `pacman-simple` — Standard Pacman App

A baseline two-tier Pacman deployment (Node.js frontend + MongoDB backend with PVC). Suitable for standard OpenShift deployment demos and namespace-scoped exercises.

| Manifest | Kind | Description |
|---|---|---|
| `pacman-deployment.yaml` | Deployment | Pacman frontend (`quay.io/jpacker/nodejs-pacman-app:latest`) on port 8080 |
| `pacman-service.yaml` | Service | ClusterIP, port 80 → 8080 |
| `pacman-route.yaml` | Route | HTTP ingress |
| `mongo-deployment.yaml` | Deployment | MongoDB backend (`bitnami/mongodb`) |
| `mongo-service.yaml` | Service | ClusterIP on port 27017 |
| `mongo-pvc.yaml` | PVC | 8Gi ReadWriteOnce for MongoDB data |

```bash
oc new-project pacman-app
oc apply -f pacman-simple/
oc get route pacman -n pacman-app
```

---

### `pacman-multi` — Blue/Green Traffic Splitting

An enhanced Pacman deployment designed for **blue/green traffic splitting** demos. Deploys two Pacman variants (`pacman` and `pacman-green`) and routes traffic between them using an OpenShift Route with weighted backends.

**Key differences from `pacman-simple`:**
- `pacman-deployment.yaml` — includes CPU/memory resource limits and requests
- `pacman-deployment green.yaml` — the "green" variant deployment
- `pacman-route.yaml` — configured with **50/50 round-robin split** between `pacman` and `pacman-green` services, with TLS edge termination and cookie-based session stickiness disabled
- Namespace is commented out — deploy into your current project

| Manifest | Description |
|---|---|
| `pacman-deployment.yaml` | Blue Pacman (resources: 100m–200m CPU, 256–512Mi RAM) |
| `pacman-deployment green.yaml` | Green Pacman variant |
| `pacman-service.yaml` | Service for blue Pacman |
| `pacman-service green.yaml` | Service for green Pacman |
| `pacman-route.yaml` | Round-robin Route splitting 50% blue / 50% green (HTTPS) |
| `mongo-deployment.yaml` + `mongo-service.yaml` + `mongo-pvc.yaml` | Shared MongoDB backend |

```bash
oc new-project pacman-app
oc apply -f pacman-multi/
```

---

### `php-scale` — PHP App with Traffic Splitting & HPA

A PHP application paired with a load generator, used to demonstrate **Horizontal Pod Autoscaling (HPA)** and **traffic splitting** on OpenShift (ROSA).

| Item | Description |
|---|---|
| `index.php` | PHP app — prints hostname, server IP, and current date |
| `route.yaml` | OpenShift Route with round-robin balancing; currently weighted 100% to `scale` service. Adjust weights to enable traffic splitting. |
| `stress/` | Load generator for CPU stress testing to trigger autoscaling |
| `virtualization/` | Additional manifests for OpenShift Virtualization demos |

The Route targets `split-1-scale-01.apps.rosa-fxnlv.w102.p1.openshiftapps.com` — update the `spec.host` field for your own cluster.

---

### `snapapp` — Image-Tool App with ODF Storage

A demo application (`image-app`) that uses **OpenShift Data Foundation (ODF)** CephFS persistent storage, suitable for snapshot and storage management exercises.

| Manifest | Description |
|---|---|
| `all.yaml` | Complete stack: Deployment + Service + ServiceAccount + PVC (1Gi CephFS) + Route (TLS edge) |
| `Ansible-ServiceNow/credentials.yaml` | Stub credentials manifest for Ansible Automation Platform + ServiceNow integration exercises |

- Container image: `quay.io/redhattraining/image-tool:latest`
- Storage: `ocs-storagecluster-cephfs` StorageClass (1Gi PVC, RWO), mounts at `/var/storage`
- Service: `LoadBalancer` type, port 80 → 5000
- Route: TLS edge termination

```bash
oc apply -f snapapp/all.yaml
```

> **Note:** The `ocs-storagecluster-ceph-rbd` StorageClass is commented out as an alternative — uncomment and swap if RBD is preferred over CephFS.

---

## Prerequisites

- OpenShift cluster (ROSA or on-premises)
- For `governance-policies`: RHACM hub cluster with the `remote` ClusterSet configured
- For `snapapp`: ODF operator installed with CephFS StorageClass available (`ocs-storagecluster-cephfs`)
- `oc` CLI logged in with appropriate permissions

---

## License

See [LICENSE](./LICENSE) if applicable.
