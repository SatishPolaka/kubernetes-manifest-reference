# Command Notes and Corrections

This file preserves the command patterns from the practical notes and corrects common typos or unsafe assumptions.

## API discovery

```bash
kubectl api-versions
kubectl api-resources
kubectl api-resources | grep -i service
kubectl api-resources | grep -i pod
```

The command is `api-versions` (plural), not `api-version`. Searching for `"service/pod"` as one phrase is not useful because Service and Pod are separate resource kinds.

## Minikube installation (Amazon Linux-style example)

```bash
sudo yum install -y docker
sudo systemctl enable --now docker

curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm -f minikube-linux-amd64

minikube start --driver=docker
```

Use `--force` only when you understand and accept the warning that prompted it; it should not be the default. Ensure your user has permission to access Docker, or run Minikube with an appropriately configured account.

## kubectl installation (Linux x86_64)

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo install kubectl /usr/local/bin/kubectl
rm -f kubectl
kubectl version --client
```

## kOps notes

kOps creates and manages Kubernetes clusters on supported infrastructure. It is not the same thing as Minikube. The cluster name, S3 state store, AWS region/zones, instance types, Kubernetes version and IAM permissions must all match the environment.

```bash
aws sts get-caller-identity
aws s3 ls
export KOPS_STATE_STORE=s3://YOUR-UNIQUE-KOPS-STATE-BUCKET
kops get clusters
```

Do not publish state-store names if they expose internal account/environment details. Check current kOps documentation before copying installation URLs or flags because release options change.

## Imperative vs declarative

```bash
# Imperative: create a Pod directly
kubectl run web --image=nginx

# Declarative: define desired state in YAML and apply it
kubectl apply -f 01-workloads/pod.yaml
```

Declarative manifests are generally easier to review, version-control and reuse. `kubectl create -f` creates a resource and errors if it already exists; `kubectl apply -f` manages the declared configuration over time.

## Pod commands

```bash
kubectl get pods
kubectl get pods -o wide
kubectl describe pod POD_NAME
kubectl logs POD_NAME
kubectl exec -it POD_NAME -- sh
kubectl delete pod POD_NAME
kubectl delete pods --all
```

`kubectl delete pod dec-pod pod1 pod2` is valid syntax for deleting multiple named Pods.

## Labels and selectors

```bash
kubectl label pod POD_NAME app=web env=dev
kubectl get pods --show-labels
kubectl get pods -l env=prod
kubectl get pods -l 'env!=prod'
kubectl get pods -l 'env in (prod,dev)'
kubectl label pod POD_NAME env-
```

`env-` removes the label key `env`. `env=!prod` is not the usual valid not-equal selector syntax; use `env!=prod`.

## Deployments

```bash
kubectl get deployments
kubectl get deploy -o wide
kubectl describe deployment DEPLOYMENT_NAME
kubectl scale deployment DEPLOYMENT_NAME --replicas=5
kubectl set image deployment/DEPLOYMENT_NAME CONTAINER_NAME=nginx:stable
kubectl rollout status deployment/DEPLOYMENT_NAME
kubectl rollout history deployment/DEPLOYMENT_NAME
kubectl rollout undo deployment/DEPLOYMENT_NAME
kubectl rollout restart deployment/DEPLOYMENT_NAME
kubectl logs deployment/DEPLOYMENT_NAME
```

The container name in `kubectl set image` must match the actual container name in the Pod template. `--record` is deprecated/removed in modern kubectl workflows; use change-cause annotations or Git history instead.

## HPA

```bash
kubectl get hpa
kubectl describe hpa HPA_NAME
```

HPA requires a metrics API. CPU utilisation percentages are calculated relative to CPU requests, so define requests on containers.

## ResourceQuota

A quota can reject creation/scaling once the namespace reaches its hard limit. After scaling, inspect:

```bash
kubectl describe resourcequota QUOTA_NAME -n NAMESPACE
kubectl get events -n NAMESPACE --sort-by=.metadata.creationTimestamp
```

## Ingress

Applying an AWS Load Balancer Controller manifest is not a generic Ingress setup. The controller also needs AWS IAM permissions/IRSA or an appropriate role, cluster/network prerequisites, and controller configuration. Use the official AWS Load Balancer Controller installation guide for the exact cluster version and deployment method.


## Production image and security checks

```bash
# Inspect the image before promoting it.
docker pull satishpolaka/cars:v1
docker image inspect satishpolaka/cars:v1

# Scan the image with an approved scanner (example: Trivy).
trivy image satishpolaka/cars:v1

# Validate against the live cluster API without creating resources.
kubectl apply --dry-run=server -f manifest.yaml

# Review the changes before applying.
kubectl diff -f manifest.yaml
```

For production, prefer an immutable image digest (`image@sha256:...`) after validating the
image. The image tag from the practical notes is kept for continuity, not as a production
security recommendation.
