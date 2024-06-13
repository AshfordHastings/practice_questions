
## Kubectl Syntax

### How do you use -o jsonpath with kubectl?

<details>

#### Using -o jsonpath in Kubernetes:
1. Basic syntax:
   ```bash
   kubectl get <resource> -o jsonpath='{.path.to.field}'
   ```

2. Common use cases:
   - Get specific field:
     ```bash
     kubectl get pods <pod-name> -o jsonpath='{.status.podIP}'
     ```
   - List all container images:
     ```bash
     kubectl get pods -o jsonpath='{.items[*].spec.containers[*].image}'
     ```

3. Advanced features:
   - Range: `{range .items[*]}{.metadata.name}{"\n"}{end}`
   - Conditionals: `{.items[?(@.status.phase=="Running")].metadata.name}`
   - Multiple fields: `{.metadata.name}{"\t"}{.status.podIP}{"\n"}`

4. Example: Node IPs
   ```bash
   kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'
   ```
</details>

### How do you see which node a Pod is running on? 

<details>

To get the specific node a Pod is running on, you can use:
```
kubectl get pod backend-dev-6d59475fdb-9h4jn -o jsonpath='{.spec.nodeName}'
```

</details>


## Other
## With Answers

### How do you load a Docker image into minikube?
<details>

```
minikube image load my-dags:0.0.1
```
</details>