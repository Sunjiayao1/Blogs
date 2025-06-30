# All about the Login-Service-Rust-k8s-Demo

## minikube configuration

### Configure Docker to Use Minikube’s Docker Daemon
```shell
eval $(minikube docker-env)
```
This command is used to set your local Docker client to communicate with the Docker daemon inside the Minikube VM. This sets environment variables like this:
```shell
$ minikube docker-env
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.93.100:2376"
export DOCKER_CERT_PATH="/home/user/.minikube/certs"
export MINIKUBE_ACTIVE_DOCKERD="minikube"
```
After running `eval` command, the shell will point to minikube's docker-demon. Any Docker images you build will be stored inside the Minikube VM.
This is essential for ensuring that your Kubernetes cluster can locate the image during deployment.
Alternatively, you can load the image into the local Docker registry and pull it inside the cluster.

### Enable Ingress Addon
There are several ways to create ingress nginx controller, for minikube is `minikube addons enable ingress`. This creates related resources (services, pods, etc.) in the `ingress-nginx` namespace. The default type of `ingress-nginx-controller` created by minikube is `NodePort`, You can change to `LoadBalancer` type using:
```shell
kubectl patch svc ingress-nginx-controller -n ingress-nginx -p '{"spec": {"type": "LoadBalancer"}}'`.
```
you can verify the result with:
```shell
kubectl get svc -n ingress-nginx
```
```shell
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.102.191.210   <pending>    80:30520/TCP,443:30884/TCP   14h
```
The `EXTERNAL-IP` of a LoadBalancer(LB) service is `<pending>` by default, because minikube runs in a local environment without access to a real cloud load balancer provider. When `minikube tunnel` is enabled, minikube simulates a cloud-like LB by creating a local proxy that forwards traffic. However, because minikube is using the Docker driver, services cannot bind to the host’s external network interface. As a result, Minikube falls back to using `127.0.0.1` as the external IP. In a real cloud environment (e.g: AWS or GCP), the LoadBalancer service would be assigned a proper external IP automatically.

### Expose High Ports for Local Access
To access your service from the host machine:
```shell
minikube service ingress-nginx-controller -n ingress-nginx --url
```
When using the Docker driver, Minikube cannot expose services on low ports like 80 or 443 directly on the host. The above command sets up a temporary port-forwarding from a random high port(e.g. 59239) to the ingress controller inside the cluster.

## How to Access the Application
You can access the application using 
```shell
curl -vk  https://rust.local:<high port>/healthz
```
If you omit the port (i.e. use the default https://rust.local/healthz), you may encounter a 502 Bad Gateway.

### Why the High Port is Required

The high port (e.g: 59320) is dynamically allocated by `minikube service ... --url` and forward traffic from your host machine -> minikube high port → to your Ingress Controller → to your Rust app.

### Why port 443 doesn't work by default
By convention, HTTPS traffc use port 443. However, on most systems, ports between 0 - 1023 are privileged ports that require root access. Since the Ingress Controller runs inside a container without root privileges, it cannot bind directly to port 443.

Minikube solves this by exposing the ingress controller on a high-numbered port (e.g., 59320) that does not require elevated permissions.

## How Requests Are Routed

```shell
Brower/curl (https://rust.local:<high port>)
   ↓
minikube port forwarding (127.0.0.1:<high port> -> ingress-nginx-controller:443 )
   ↓
ingress-nginx-controller(TLS termination)
   ↓
Ingress rule match (Host: rust.local, Path: /)
   ↓
forward to Service: login-service-cluster:80
   ↓
Service Selector find matching pod
   ↓
Forward to Pod (e.g. 10.244.0.118:8081)
   ↓
Rust app receives GET /healthz request
   ↓
return HTTP 200
```

## Debug Tips:
Here are some useful commands for debugging:

1. Check Ingress Controller Logs 
```shell
kubectl logs -l app.kubernetes.io/name=ingress-nginx -n ingress-nginx
```

2. Check Ingress Rules

```shell
kubectl describe ingress login-service-ingress -n login-service
```

3. Check Service Endpoints and Pods
```shell
kubectl get endpoints login-service-cluster -n login-service -o yaml
kubectl get pods -n login-service -o wide
```

4. Check Pods Logs
```shell
kubectl logs login-service-api-787bf9c9b-j78ct -n login-service
```

5. Check Connection from Ingress Controller to Pod
```shell
kubectl exec -it <nginx-ingress-pod-name> -n ingress-nginx -- sh
curl -v http://login-service-cluster.login-service.svc.cluster.local:8081/healthz
```

6. Check Connection inside Pod
```shell
kubectl exec -it login-service-api-787bf9c9b-stj2h -n login-service -- sh
sudo apt-get update && apt-get install curl
curl -v http://127.0.0.1:8081/healthz
```
