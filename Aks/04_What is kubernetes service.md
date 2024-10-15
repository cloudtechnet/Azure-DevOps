# What is kubernetes service?

A **Kubernetes Service** is an abstraction that defines a logical set of pods and a policy by which to access them. It provides a stable endpoint for a set of pods that are dynamically created or destroyed, ensuring that other applications can reliably communicate with these pods.

Since pods are ephemeral and can be rescheduled or replaced, their IP addresses can change frequently. A Kubernetes Service solves this problem by giving a consistent IP and DNS name for accessing these pods. Kubernetes Services can also handle load balancing across multiple pods.

### Key Types of Kubernetes Services:
1. **ClusterIP (default)**: Exposes the service only within the cluster. This is the default service type.
2. **NodePort**: Exposes the service on each node's IP at a static port. You can access the service from outside the cluster via `<NodeIP>:<NodePort>`.
3. **LoadBalancer**: Exposes the service externally using a cloud provider's load balancer (for clusters running in cloud environments like AWS, GCP, Azure).
4. **ExternalName**: Maps the service to a DNS name outside the cluster, allowing access to external services.

### Example of a Kubernetes Service

Let’s say you have an application running in a Kubernetes cluster with multiple pods (e.g., a web application with a front end and back end). You want to expose the back-end pods so that the front-end pods can communicate with them reliably.

#### Example YAML for a ClusterIP Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  labels:
    app: backend
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80         # The port on which the service is exposed
      targetPort: 8080  # The port on the pod
  type: ClusterIP       # Internal service within the cluster
```

In this example:
- The **selector** field defines which pods this service will route traffic to. In this case, it routes to pods with the label `app: backend`.
- The **port** (80) is the port where the service will listen, and the **targetPort** (8080) is the port on the backend pod.
- The **type: ClusterIP** makes this service accessible only within the cluster.

#### Steps to Create and Manage Kubernetes Services

##### 1. Create a Service
You can apply a service YAML file to create it in the cluster:

```bash
kubectl apply -f backend-service.yaml
```

##### 2. Check Services in the Cluster
To see all the services in your Kubernetes cluster:

```bash
kubectl get services
```

Example output:

```
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP    10d
backend-service  ClusterIP   10.96.0.100      <none>        80/TCP     1m
```

In this output:
- **NAME**: The name of the service.
- **TYPE**: The type of the service (`ClusterIP`, `NodePort`, `LoadBalancer`).
- **CLUSTER-IP**: The internal IP address assigned to the service.
- **EXTERNAL-IP**: If using `LoadBalancer`, this field will have the external IP.
- **PORT(S)**: The ports exposed by the service.

##### 3. Get Detailed Information about a Service

You can use the `describe` command to get more detailed information about a specific service:

```bash
kubectl describe service backend-service
```

This provides detailed information, including the endpoints (i.e., IPs of the pods backing the service), labels, and events.

#### Example of a NodePort Service

Let’s say you want to expose the backend pods externally, so you create a `NodePort` service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30007  # Static port assigned to access the service externally
  type: NodePort
```

- The **nodePort** field defines the port on which the service will be available on each node.
- This will expose the backend application outside the cluster on any of the node IPs on port `30007`.

You can now access the service from outside the cluster using `<NodeIP>:30007`.

##### 4. Deleting a Service

To delete a service, you can use the `kubectl delete` command:

```bash
kubectl delete service backend-service
```

### Conclusion

A Kubernetes Service is a critical abstraction that ensures reliable communication between components in a Kubernetes cluster. It allows you to expose pods internally or externally and provides a stable IP/DNS for access, even as the underlying pods change dynamically.