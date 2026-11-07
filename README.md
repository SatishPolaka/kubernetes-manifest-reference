# Kubernetes Manifest Reference

A practical Kubernetes YAML/manifest reference built from hands-on notes. It is organised for GitHub so you can revisit each resource, understand why each field exists, and reuse safe starting examples.

> **Scope:** This repository covers the commonly used built-in Kubernetes resources and their important fields. Kubernetes has many APIs and optional fields; no single guide can list every field of every API object. Use the official API reference for exhaustive, version-specific schemas.

## Repository structure

```text
kubernetes-manifest-reference/
├── 01-workloads/
│   ├── pod.yaml
│   ├── deployment.yaml
│   ├── replicaset.yaml
│   ├── statefulset.yaml
│   └── daemonset.yaml
├── 02-networking/
│   ├── service-clusterip.yaml
│   ├── service-nodeport.yaml
│   ├── service-loadbalancer.yaml
│   ├── ingress.yaml
│   └── networkpolicy.yaml
├── 03-configuration/
│   ├── configmap.yaml
│   ├── secret.yaml
│   └── deployment-config-secret-env.yaml
├── 04-storage/
│   ├── persistentvolume.yaml
│   ├── persistentvolumeclaim.yaml
│   └── storageclass-ebs-example.yaml
├── 05-scheduling/
│   ├── node-affinity.yaml
│   ├── pod-affinity.yaml
│   ├── pod-anti-affinity.yaml
│   └── taints-tolerations.yaml
├── 06-health-and-scaling/
│   ├── probes.yaml
│   ├── hpa.yaml
│   └── poddisruptionbudget.yaml
├── 07-batch/
│   ├── job.yaml
│   └── cronjob.yaml
├── 08-namespace-and-security/
│   ├── namespace.yaml
│   ├── resourcequota.yaml
│   ├── limitrange.yaml
│   ├── serviceaccount.yaml
│   └── rbac-role-rolebinding.yaml
└── docs/
    ├── manifest-field-guide.md
    └── commands-and-corrections.md
```

## How to use

Each YAML manifest includes inline comments beside important fields, explaining what the field/value is for and why it is present. The longer field guide provides additional context and caveats.

1. Read the explanation in `docs/manifest-field-guide.md`.
2. Open the relevant YAML file and change names, image tags, labels, ports, resource values and storage settings for your environment.
3. Validate before applying:
   ```bash
   kubectl apply --dry-run=client -f path/to/file.yaml
   kubectl diff -f path/to/file.yaml
   ```
4. Apply:
   ```bash
   kubectl apply -f path/to/file.yaml
   ```
5. Verify:
   ```bash
   kubectl get all
   kubectl describe <resource-type> <resource-name>
   kubectl get events --sort-by=.metadata.creationTimestamp
   ```
6. Remove only resources you intentionally created:
   ```bash
   kubectl delete -f path/to/file.yaml
   ```

## Important safety notes

- These examples are learning templates, not production-ready manifests without environment-specific review.
- Replace example image tags with maintained, trusted, preferably immutable image tags or digests.
- Do not commit real passwords, tokens, private keys, kubeconfigs or cloud credentials to Git.
- Kubernetes Secret values are base64-encoded in the API representation; base64 is **not encryption**. Enable encryption at rest, restrict RBAC, and consider an external secret manager.
- The EBS StorageClass example requires an AWS cluster with the EBS CSI driver and suitable IAM permissions. It will not work on Minikube without the matching storage provisioner.
- Ingress requires an Ingress controller. Creating an Ingress object alone does not install a controller or create an AWS load balancer.
- HPA resource metrics require Metrics Server (or another supported metrics API).
- NetworkPolicy enforcement depends on a CNI plugin that supports NetworkPolicy.
- `kubectl apply` is the normal declarative workflow. `kubectl create` is also valid, but repeated `create` calls do not update an existing object.

## Main terminology

Files such as `deployment.yaml`, `service.yaml`, `configmap.yaml` and `ingress.yaml` are called **Kubernetes manifests** or **Kubernetes resource manifests**. YAML is the file format; the `kind` identifies the Kubernetes resource.

## API version check

The correct command is:

```bash
kubectl api-versions
kubectl api-resources
kubectl explain deployment
kubectl explain deployment.spec
kubectl explain deployment.spec.template.spec.containers
```

`kubectl api-version` is not the correct command. API versions lists API groups/versions; `api-resources` lists resource kinds and their short names.


## Production-readiness guidance

The manifests include production-oriented patterns such as resource requests/limits, health probes,
least-privilege ServiceAccounts, security contexts, rollout controls, persistent storage examples,
and RBAC. They are **production-oriented templates**, not a guarantee that they are ready to apply
unchanged to every cluster.

Before using them in production:

1. Replace the example namespace, hostnames, image references, storage classes, ports and resource sizing.
2. Pin container images to immutable digests after validating the exact image. A tag such as `v1` can be changed in a registry.
3. Confirm the application actually supports the configured non-root user and read-only root filesystem. Add writable `emptyDir` mounts for required temporary paths if needed.
4. Use a real secret manager or sealed/external secret workflow. Never commit actual credentials.
5. Confirm the cluster has the required CSI driver, Ingress controller, metrics API and NetworkPolicy-capable CNI.
6. Test manifests in a staging cluster, run server-side dry-run, review `kubectl diff`, and validate rollback and disaster-recovery procedures.
7. Set resource values, probe endpoints, disruption budgets and replica counts based on load tests and service-level objectives.

The `satishpolaka/cars:v1` image is retained from the practical notes. Verify its availability,
contents, security scan results and immutable digest before production use.
