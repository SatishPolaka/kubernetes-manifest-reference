# Kubernetes Manifest Field Guide

This guide explains the common fields that appear repeatedly across Kubernetes manifests, followed by resource-specific explanations.

## 1. Common manifest structure

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: default
  labels:
    app: web
  annotations:
    description: "Example application"
spec:
  # Resource-specific desired state goes here
```

| Field | Why it is present |
|---|---|
| `apiVersion` | Selects the Kubernetes API group and version used to interpret the object. For example, core resources use `v1`; Deployments use `apps/v1`. |
| `kind` | Declares the resource type, such as `Pod`, `Service`, `Deployment` or `ConfigMap`. |
| `metadata` | Holds identifying and descriptive information about the object. |
| `metadata.name` | Required object name, unique within its namespace for that resource type. |
| `metadata.namespace` | Namespace in which a namespaced resource is created. Omit it to use the current namespace. |
| `metadata.labels` | Key/value metadata used for selection, grouping and organisation. Selectors commonly match labels. |
| `metadata.annotations` | Non-selector metadata for tools, controllers and human-readable information. |
| `spec` | Desired configuration for the resource. Its exact fields depend on `kind`. |
| `status` | Observed state written by Kubernetes/controllers. Normally do not include `status` in manifests you apply. |

## 2. Pod and container fields

| Field | Why it is present |
|---|---|
| `spec.containers` | List of application containers in the Pod. At least one is required. |
| `containers[].name` | Unique container name within the Pod. |
| `containers[].image` | Container image and tag/digest to run. |
| `containers[].imagePullPolicy` | Controls image pulling: `Always`, `IfNotPresent` or `Never`. |
| `containers[].command` | Overrides the image ENTRYPOINT. |
| `containers[].args` | Supplies arguments to the entrypoint. |
| `containers[].workingDir` | Working directory inside the container. |
| `containers[].ports` | Documents container ports; it does not itself publish the port outside the Pod. |
| `ports[].containerPort` | Port on which the process listens inside the container. Spelling is `containerPort`, not `caontainerPort`. |
| `ports[].protocol` | `TCP` by default; can be `UDP` or `SCTP` where supported. |
| `env` | Defines individual environment variables. |
| `env[].name` | Environment variable name. |
| `env[].value` | Literal value (avoid putting secrets directly here). |
| `env[].valueFrom` | Gets a value from a ConfigMap key, Secret key, field or resource value. |
| `envFrom` | Imports all eligible keys from a ConfigMap or Secret as environment variables. |
| `resources.requests` | Minimum resources used by the scheduler when placing the Pod. |
| `resources.limits` | Maximum CPU/memory allowed by container runtime enforcement. |
| `volumeMounts` | Mounts a Pod volume into a container filesystem. |
| `volumeMounts[].mountPath` | Directory/file path inside the container where the volume appears. |
| `volumeMounts[].readOnly` | Mounts the volume read-only when true. |
| `volumes` | Declares storage/config sources available to containers in the Pod. |
| `imagePullSecrets` | References namespace-local registry credentials for private images. |
| `serviceAccountName` | Selects the ServiceAccount identity used by the Pod. |
| `restartPolicy` | Pod restart behaviour: `Always`, `OnFailure` or `Never` (Job/CronJob commonly use the latter two). |
| `nodeSelector` | Simple node-label constraint. |
| `affinity` | Advanced node and Pod placement rules. |
| `tolerations` | Allows a Pod to be scheduled onto nodes with matching taints; it does not force placement. |
| `securityContext` | Configures user/group IDs, privilege settings, capabilities and filesystem security. |
| `livenessProbe` | Detects a container that should be restarted. |
| `readinessProbe` | Determines whether a container should receive Service traffic. |
| `startupProbe` | Gives slow-starting applications time to start before liveness/readiness checks take effect. |

### Environment variable sources

- `env[].value`: one literal value.
- `env[].valueFrom.configMapKeyRef`: one key from a ConfigMap.
- `env[].valueFrom.secretKeyRef`: one key from a Secret.
- `envFrom[].configMapRef`: imports all ConfigMap keys.
- `envFrom[].secretRef`: imports all Secret keys.
- `env[].valueFrom.fieldRef`: exposes Pod metadata such as `metadata.name` or `status.podIP`.

## 3. Workload controllers

### Pod
A Pod is the smallest deployable Kubernetes unit. It wraps one or more tightly coupled containers and shared networking/storage. Directly created Pods are not automatically replaced if deleted.

### ReplicaSet
Maintains a requested number of matching Pod replicas. Usually let a Deployment manage ReplicaSets rather than creating ReplicaSets directly.

### Deployment
Manages stateless application rollouts and rollbacks. `replicas` is the desired Pod count; `selector` identifies Pods managed by the Deployment; `template` is the Pod template used to create them. The selector must match the template labels.

### StatefulSet
Manages applications that need stable Pod identities and/or stable persistent storage. Common fields include `serviceName`, `replicas`, `selector`, `template` and `volumeClaimTemplates`. Usually pair it with a headless Service for stable network identities.

### DaemonSet
Ensures a Pod runs on each eligible node (or each node matching scheduling constraints). Common uses include log collectors, monitoring agents and node-level networking components.

## 4. Service and networking

### Service
Provides a stable virtual endpoint for a changing set of Pods. `selector` selects backend Pods by labels. `port` is the Service port; `targetPort` is the destination port on Pods. `type` is commonly `ClusterIP`, `NodePort` or `LoadBalancer`.

- `ClusterIP`: internal cluster access (default).
- `NodePort`: exposes a port on each node; default allocation range is commonly 30000–32767.
- `LoadBalancer`: asks a supported cloud/infrastructure integration to provision an external load balancer. Cloud behaviour and annotations are provider-specific.
- `ExternalName`: DNS alias to an external hostname; it does not proxy traffic.

`nodePort` is optional for a LoadBalancer Service and normally unnecessary unless you specifically need to choose it.

### Ingress
Defines HTTP/HTTPS routing rules to Services. Requires an Ingress controller. `ingressClassName` selects the controller class; `rules` define host/path routing; `tls` references a TLS Secret.

### NetworkPolicy
Controls allowed ingress and/or egress traffic for selected Pods. Policies are additive. Enforcement requires a NetworkPolicy-capable CNI.

## 5. Configuration and credentials

### ConfigMap
Stores non-confidential configuration. It can be consumed as environment variables or mounted files. Do not store passwords or tokens in a ConfigMap.

### Secret
Stores confidential values such as passwords, tokens or TLS material. Secret data is base64-encoded in normal YAML/API output, not automatically encrypted. Protect it with encryption at rest and RBAC. Prefer a secret manager for production credentials.

## 6. Storage

### PersistentVolume (PV)
Represents storage provisioned in the cluster, either statically or dynamically. It is cluster-scoped.

### PersistentVolumeClaim (PVC)
A namespaced request for storage. It asks for capacity, access mode and optionally a StorageClass. Pods reference a PVC; they should not normally refer directly to a cloud disk.

### StorageClass
Describes how dynamic storage should be provisioned. `provisioner` and `parameters` are driver-specific. `reclaimPolicy` controls what happens to dynamically provisioned PVs after claim release. `volumeBindingMode: WaitForFirstConsumer` delays provisioning/binding until scheduling context is known.

Access modes such as `ReadWriteOnce`, `ReadOnlyMany`, `ReadWriteMany` and `ReadWriteOncePod` have driver-specific support and semantics. `ReadWriteOnce` does not mean that two Pods on the same node can never mount the volume; check the storage driver's behaviour.

## 7. Scheduling

- **Node affinity:** places Pods according to node labels.
- **Pod affinity:** places Pods near matching Pods within a topology domain.
- **Pod anti-affinity:** avoids placing Pods near matching Pods.
- **Taint:** node-side restriction.
- **Toleration:** Pod-side permission to tolerate a matching taint. A toleration does not guarantee the Pod will land on that node.
- **topologyKey:** defines the topology domain, commonly `kubernetes.io/hostname` for node-level placement.

`requiredDuringSchedulingIgnoredDuringExecution` is a hard rule during scheduling. `preferredDuringSchedulingIgnoredDuringExecution` is a weighted preference.

## 8. Health, scaling and disruption

- **Liveness:** restart a container when it is unhealthy.
- **Readiness:** remove a Pod from Service endpoints while it is not ready.
- **Startup:** protect slow-starting containers while startup is in progress.
- **HPA:** changes replica count based on metrics. Requires a metrics API; CPU utilisation targets need CPU requests set on containers.
- **PDB:** limits voluntary disruptions, such as node drain, by specifying `minAvailable` or `maxUnavailable`. It does not prevent involuntary failures.

Probe timing fields:
- `initialDelaySeconds`: delay before probing begins (startup probe changes the interaction with other probes).
- `periodSeconds`: interval between probes.
- `timeoutSeconds`: probe timeout.
- `successThreshold`: consecutive successes needed (must be 1 for liveness/startup).
- `failureThreshold`: consecutive failures before the probe is considered failed.
- `httpGet`, `tcpSocket`, `exec` and (where supported) `grpc`: probe mechanisms.

## 9. Batch

- **Job:** runs one or more Pods until the required number of successful completions is reached. `completions` is the number of successful tasks required; `parallelism` is the maximum number running at once.
- **CronJob:** creates Jobs on a cron schedule. `schedule` uses five-field cron syntax. `timeZone` is available in supported Kubernetes versions. `suspend: true` stops future scheduling but does not stop Jobs already started.

## 10. Namespace governance and RBAC

- **Namespace:** logical scope for namespaced resources.
- **ResourceQuota:** limits aggregate resource consumption/counts within a namespace.
- **LimitRange:** supplies default requests/limits and/or enforces per-container/Pod bounds.
- **ServiceAccount:** identity available to Pods.
- **Role:** namespaced permissions.
- **RoleBinding:** grants a Role (or ClusterRole) to subjects within a namespace.
- **ClusterRole / ClusterRoleBinding:** cluster-scoped permissions/bindings. Follow least privilege.

## 11. Useful discovery and validation commands

```bash
kubectl api-versions
kubectl api-resources
kubectl explain pod
kubectl explain pod.spec
kubectl explain deployment.spec.template.spec
kubectl explain service.spec
kubectl explain ingress.spec
kubectl get <resource> -o yaml
kubectl apply --dry-run=server -f manifest.yaml
kubectl diff -f manifest.yaml
```

`kubectl explain` is especially useful because it shows fields supported by the API schema of the cluster you are actually using.
