# Manifest Review Checklist

Before applying a manifest, check:

- [ ] Correct `apiVersion` for the resource kind (`apps/v1` for Deployment/ReplicaSet/StatefulSet/DaemonSet).
- [ ] Correct `kind`.
- [ ] Resource name is meaningful and unique in the namespace.
- [ ] Namespace is correct.
- [ ] Labels in `spec.selector.matchLabels` exactly match the Pod template labels.
- [ ] Container names are correct and unique.
- [ ] Image name and tag exist and are trusted.
- [ ] `containerPort` spelling and application port are correct.
- [ ] Service selector matches the target Pod labels.
- [ ] `port` and `targetPort` are correct.
- [ ] CPU/memory requests and limits are suitable.
- [ ] ConfigMap/Secret names and keys exist.
- [ ] Secrets contain no real credentials in Git.
- [ ] PVC references an available StorageClass/claim.
- [ ] Probes test the real application health endpoint.
- [ ] Scheduling constraints do not make the Pod impossible to schedule.
- [ ] RBAC grants only the permissions required.
- [ ] Dependencies (Ingress controller, CSI driver, Metrics Server, CNI policy support) are installed.
- [ ] `kubectl apply --dry-run=server -f file.yaml` succeeds.
- [ ] `kubectl diff -f file.yaml` has been reviewed.
