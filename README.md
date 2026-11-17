# Student Management on Kubernetes

A sample student roster served by Nginx, packaged with Docker, and deployed to a local Kubernetes cluster using Minikube. Version 1 displayed three students. Version 2 added Jordan Patel to the roster and was rolled out to five replicas.

## Project files

| File | Purpose |
| --- | --- |
| `index.html` | Version 2 of the student roster |
| `Dockerfile` | Packages the roster into an Nginx image |
| `deployment.yaml` | Records the original lab configuration with Version 1 and three replicas |
| `service.yaml` | Defines the ClusterIP Service that selects the application Pods |

## Run the current version

From the repository folder, with Docker and Minikube installed:

```bash
docker build -t student-management:v2 .
minikube start -p student-lab --driver=docker
minikube -p student-lab image load student-management:v2
```

Apply the Deployment with the V2 image and five replicas. The substitutions below apply to the manifest sent to Kubernetes and leave the original lab file on disk.

```bash
sed -e 's/student-management:v1/student-management:v2/' \
    -e 's/replicas: 3/replicas: 5/' deployment.yaml \
  | minikube -p student-lab kubectl -- apply -f -
minikube -p student-lab kubectl -- apply -f service.yaml
minikube -p student-lab kubectl -- rollout status deployment/student-management --timeout=120s
minikube -p student-lab kubectl -- port-forward service/student-management 8083:80
```

Keep port forwarding active. In another terminal, run `curl -i http://127.0.0.1:8083/` and look for `200 OK`, `Version 2`, and `Jordan Patel`.

## Lab results

| Check | Observed result |
| --- | --- |
| Local Docker test | Nginx returned `200 OK` with the Version 1 roster |
| Initial deployment | Three Pods were Running and the Deployment showed `3/3` Ready |
| Service access | The Service port-forward returned the Version 1 page |
| Self-healing | After I deleted the Pod ending in `4ff7w`, a replacement ending in `kdjnj` became Ready |
| Scaling | The Deployment went from three to five Ready replicas |
| Version 2 rollout | All five replicas were Ready and all five Pods used `student-management:v2` |

## What happened when I deleted a Pod?

The Deployment still requested three replicas, so its ReplicaSet created a replacement Pod automatically. The replacement had a new name and passed its HTTP readiness probe. The other two Pods kept running, and the Deployment returned to three Ready replicas.

## Author

Franklin Tumenta