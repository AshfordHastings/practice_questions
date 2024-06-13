## Resource Management
### How does the Kubernetes scheduler use resourceRequests to determine which node to allocate your Pod to?

<details>
The Kubernetes scheduler uses `resourceRequests` to determine which node to allocate a Pod to by checking each node's available resources against the requests specified in the Pod's specification. `resourceRequests` are the minimum amount of CPU and memory that a Pod needs to run. The scheduler follows these steps:

1. **Filtering:** It first filters out nodes that do not have enough available resources (CPU and memory) to meet the Pod's `resourceRequests`.
2. **Scoring:** It then scores the remaining nodes based on various factors such as resource availability, node affinity, and taints/tolerations.
3. **Binding:** Finally, it selects the node with the highest score and binds the Pod to that node.
</details>

### What does it mean for a Node to be overcommitted in resource utilization?

<details>
A Node is overcommitted in resource utilization when the total amount of CPU or memory requested by all running Pods exceeds the actual physical capacity of the node. Kubernetes allows overcommitting resources because it assumes that not all Pods will use their requested resources simultaneously. Overcommitting can lead to better utilization of resources but also increases the risk of resource contention and performance degradation.
</details>

### What does a Node do if it surpasses its available CPU?

<details>
If a Node surpasses its available CPU, Kubernetes handles this by throttling the CPU usage of the Pods. The Linux kernel uses a scheduling algorithm to ensure fair distribution of CPU cycles among the containers. Pods that exceed their `cpu` limits specified in their resource requests are restricted, leading to reduced performance for those Pods, but preventing any one Pod from monopolizing the CPU resources.
</details>

### How can I know if a pod has had its CPU throttled?

<details>
You can determine if a node throttles a pod's CPU usage by checking the pod's CPU throttling metrics using tools like `kubectl` or monitoring solutions like Prometheus. Specifically, you can use:

1. **Kubectl Command:**
   ```bash
   kubectl top pod <pod-name> --namespace=<namespace>
   ```
   This command shows the current CPU usage of the pod. High CPU usage near the pod's limit indicates possible throttling.

2. **Prometheus Metrics:**
   Query the `container_cpu_cfs_throttled_seconds_total` metric, which shows the total time a pod's CPU usage has been throttled.

3. **Logs:**
   Inspect the node's kubelet logs for messages related to CPU throttling.

Monitoring these metrics and logs helps you identify if CPU throttling is occurring.
</details>

### What does a Node do if it surpasses its available memory?

<details>
If a Node surpasses its available memory, the consequences are more severe than CPU overcommitment. The operating system may start killing processes, including Kubernetes Pods, to free up memory. This is known as an Out Of Memory (OOM) condition. Kubernetes uses an eviction process to handle memory pressure, prioritizing which Pods to evict based on factors like QoS class, resource requests, and usage. Pods with lower priority (best-effort and burstable Pods) are more likely to be evicted than those with higher priority (guaranteed Pods).
</details>

### How do QoS classes affect the Kubernetes eviction process?

<details>
QoS (Quality of Service) classes affect Kubernetes eviction by determining the priority of pods during eviction when resources are constrained:

1. **Best-Effort:** Lowest priority; evicted first. Pods have no resource requests or limits.
2. **Burstable:** Medium priority; evicted second. Pods have resource requests but limits can be higher than requests.
3. **Guaranteed:** Highest priority; evicted last. Pods have equal resource requests and limits.

Pods with lower QoS classes are more likely to be evicted during resource shortages to free up resources for higher-priority pods.
</details>

### What are eviction thresholds in Kubernetes?

<details>
**Eviction Thresholds**: Kubernetes defines eviction thresholds that specify when to begin evicting pods. These thresholds can be configured and usually include soft and hard eviction thresholds.

- Soft Eviction Thresholds: These thresholds are less aggressive and provide a grace period before eviction actions are taken.
- Hard Eviction Thresholds: These thresholds trigger immediate eviction of pods without a grace period.

</details>

### What strategies can mitigate nodes from surpassing avaliable memory?

<details>

**Mitigation Strategies:** To prevent nodes from surpassing available memory, several strategies can be employed:
   - **Setting Appropriate Requests and Limits:** Ensuring that pods have appropriate resource requests and limits to match their actual usage.
   - **Resource Quotas and Limits:** Implementing resource quotas and limits at the namespace level to control overall resource consumption.
   - **Vertical Pod Autoscaling:** Using Vertical Pod Autoscaler to automatically adjust resource requests for pods based on their usage patterns.
   - **Cluster Autoscaling:** Adding more nodes to the cluster when overall resource usage exceeds a certain threshold.
</details>

### How does the cluster autoscaler work? 

<details>
The Cluster Autoscaler in Kubernetes works by dynamically adjusting the number of nodes in a cluster based on the resource demands of the workloads. Here’s a semi-succinct explanation:

#### Scaling Up

1. **Monitor Pending Pods:** The autoscaler continuously monitors the cluster for pods that cannot be scheduled due to insufficient resources.
2. **Add Nodes:** When it detects unschedulable pods, it calculates the required resources and requests the cloud provider (like Azure, AWS, or GCP) to add more nodes to the cluster.
3. **Schedule Pods:** Once the new nodes are available and ready, the pending pods are scheduled on these nodes.

#### Scaling Down

1. **Detect Idle Nodes:** The autoscaler identifies nodes that are underutilized or idle over a configurable period.
2. **Evict Pods:** It attempts to evict and reschedule the pods from these nodes onto other nodes with available capacity, respecting Pod Disruption Budgets to ensure service availability.
3. **Remove Nodes:** After the pods are safely evicted, the autoscaler requests the cloud provider to terminate the underutilized nodes, reducing the cluster size.

#### Configuration and Optimization

- **Thresholds and Limits:** You can configure minimum and maximum node counts and set thresholds for scaling actions.
- **Node Groups:** It can manage multiple node groups with different instance types and scaling policies, ensuring efficient resource allocation.

#### Integration with Other Kubernetes Components

- **Horizontal Pod Autoscaler (HPA):** Works alongside HPA, which scales the number of pod replicas based on metrics. The Cluster Autoscaler ensures there are enough nodes to accommodate the increased pod count.
- **Pod Disruption Budgets (PDBs):** Ensures that critical pods are not evicted in a way that would violate availability requirements.

#### Benefits

- **Cost Efficiency:** Optimizes resource usage and reduces costs by scaling down when resources are not needed.
- **Resource Availability:** Ensures that sufficient resources are available for workloads by scaling up when demand increases.

By automatically managing the number of nodes, the Cluster Autoscaler helps maintain a balance between resource availability and cost efficiency in a Kubernetes cluster.
</details>


### What is the purpose of priority classes in Kubernetes, and how do they influence pod scheduling and eviction?

<details>
Priority classes in Kubernetes are used to assign different levels of importance to pods. Higher priority pods are scheduled before lower priority pods and are less likely to be evicted during resource shortages. Priority classes ensure that critical workloads are given preference over less important ones, improving the overall reliability and efficiency of resource utilization in the cluster.

</details>

### How do you create a new priority class in Kubernetes, and what fields must be specified in its manifest?
<details>

To create a new priority class in Kubernetes, you define a `PriorityClass` resource in a YAML manifest. The key fields to specify are `metadata.name` (the name of the priority class), `value` (an integer representing the priority level), `globalDefault` (a boolean indicating if this should be the default priority class), and `description` (a brief explanation of the priority class). Here is an example manifest:
   ```yaml
   apiVersion: scheduling.k8s.io/v1
   kind: PriorityClass
   metadata:
      name: high-priority
   value: 1000
   globalDefault: false
   description: "This priority class is for high-priority workloads."
   ```

</details>

### What happens if multiple pods with different priority classes are competing for the same resources, and how does Kubernetes resolve this conflict?**

<details>
If multiple pods with different priority classes are competing for the same resources, Kubernetes resolves the conflict by giving preference to the pods with higher priority classes. The scheduler will attempt to place higher priority pods on nodes before considering lower priority pods. During resource shortages, higher priority pods are less likely to be evicted compared to lower priority pods. If necessary, Kubernetes will evict lower priority pods to free up resources for higher priority ones.
</details>

### What is preemption in Kubernetes, and how does it help in scheduling high-priority pods?

<details>
Preemption in Kubernetes is a mechanism that allows the scheduler to evict lower-priority pods to make room for higher-priority pods that cannot be scheduled due to resource constraints. When a high-priority pod cannot be scheduled, the scheduler looks for lower-priority pods that can be evicted to free up the necessary resources. This ensures that critical workloads with higher priority can run even when the cluster is under resource pressure.
</details>


### How can you control the preemption behavior of a pod in Kubernetes, and what are the possible values for the preemption policy?**
<details>

You can control the preemption behavior of a pod in Kubernetes by setting the `preemptionPolicy` field in the pod specification. The possible values for the `preemptionPolicy` are `PreemptLowerPriority` and `Never`. `PreemptLowerPriority` (the default) allows the pod to preempt lower-priority pods if needed, while `Never` prevents the pod from preempting any other pods, while still placing it higher in the queue. Here is an example of setting the `preemptionPolicy` in a pod manifest:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
      name: high-priority-pod
   spec:
      priorityClassName: high-priority
      preemptionPolicy: Never
      containers:
      - name: my-container
      image: my-image
   ```

</details>

### What is a use case for configuring premption policies?

<details>
An example use case is for data science workloads. A user may submit a job that they want to be prioritized above other workloads, but do not wish to discard existing work by preempting running pods. The high priority job with preemptionPolicy: Never will be scheduled ahead of other queued pods, as soon as sufficient cluster resources "naturally" become free.

</details>

### What is the difference between QoS classes and Preemption Policies in terms of resource contention and scheduling? 

<details>

#### Pod Scheduling and Eviction:
- **QoS Classes**: During resource contention, Kubernetes prefers evicting lower QoS class pods (Best-Effort first, then Burstable, and Guaranteed last) to free up resources.
- **Preemption Policy**: High-priority pods with preemptionPolicy set to PreemptLowerPriority can evict lower-priority pods to get scheduled. Pods with Never cannot preempt any other pods, regardless of their QoS class.

#### Resource Allocation:
- **QoS Classes**: Affect how resources are allocated and reserved for pods. Guaranteed pods are given full resource reservations, ensuring they are less likely to be evicted.
- **Preemption Policy**: Influences the ability of a pod to claim resources by evicting other pods, which can be particularly relevant when Guaranteed pods with high priority need to be scheduled.

#### Example Scenario
- A high-priority Guaranteed pod with PreemptLowerPriority can evict Burstable or Best-Effort pods to get scheduled.
- A Burstable pod with Never preemption policy will not preempt any other pods, even if there are lower-priority or Best-Effort pods running.

</details>

### What is the scheduler backoff in Kubernetes? How do you configure this?

<details>

Scheduler backoff in Kubernetes is a mechanism that temporarily prevents the scheduler from repeatedly trying to schedule a pod that cannot be placed due to resource constraints or other issues. This helps reduce unnecessary load on the scheduler.

The scheduler backoff goes through an exponential increase:s The backoff period doubles with each failed attempt to schedule the pod, up to a maximum limit. This exponential backoff helps to reduce the load on the scheduler and the API server.

#### How to Configure Scheduler Backoff

You can configure scheduler backoff parameters in the Kubernetes scheduler configuration file. The key parameters are:

1. **podInitialBackoffSeconds:** The initial backoff duration in seconds.
2. **podMaxBackoffSeconds:** The maximum backoff duration in seconds.

#### Example Configuration

Here’s an example of how to configure these parameters in a scheduler configuration file:

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: default-scheduler
    backoff:
      podInitialBackoffSeconds: 1
      podMaxBackoffSeconds: 10
```

In this example, the scheduler will start with a 1-second backoff and can increase the backoff duration up to a maximum of 10 seconds for unschedulable pods.
</details>

### Let's say a Horizonal Pod Autoscaler is attached to a Deployment. The first pod is alive, but when the HPA attempts to scale, the attempted deployment has an error of "unschedulable". What could be the issue?

<details>
<summary>Possible Issues for "unschedulable" Error</summary>

#### Insufficient Node Resources
- **CPU or Memory:** The cluster may not have enough available CPU or memory resources to schedule additional pods.
- **Node Capacity:** Nodes might be at full capacity, preventing new pods from being scheduled.

#### Resource Quotas
- **Namespace Quotas:** The namespace might have a Resource Quota that limits the total number of pods or the amount of CPU/memory that can be used, preventing the deployment of new pods.

#### LimitRanges
- **Resource Requests/Limits:** The pod’s resource requests or limits may not comply with the LimitRange defined in the namespace, causing the scheduler to fail to place the pod.

#### Pod Disruption Budgets (PDBs)
- **Disruption Limits:** A Pod Disruption Budget might be preventing new pods from being scheduled due to constraints on the allowed disruptions.

#### Node Affinity/Anti-Affinity
- **Affinity Rules:** The deployment might have affinity or anti-affinity rules that cannot be satisfied with the current cluster nodes.
- **Taints and Tolerations:** Nodes may have taints that the new pods do not tolerate, preventing them from being scheduled.

#### Insufficient Permissions
- **Service Account:** The service account used by the deployment might not have the necessary permissions to create new pods or interact with the required resources.

#### Cluster Autoscaler
- **Autoscaler Configuration:** If the cluster uses a Cluster Autoscaler, it might not be scaling up quickly enough to accommodate the new pods, or it might have reached its maximum limit.

#### Pending Node Issues
- **Node Not Ready:** Some nodes might be in a NotReady state due to underlying issues, reducing the available capacity for new pods.

#### Storage Constraints
- **Persistent Volumes:** If the deployment uses Persistent Volumes, there may not be enough available storage or the required storage class may not be available.

</details>

### What is a Pod Disruption Budget? 


<details>
<summary>Pod Disruption Budget (PDB)</summary>

#### Definition
A Pod Disruption Budget (PDB) is a Kubernetes resource that specifies the minimum number or percentage of replicas of a pod that must remain available during voluntary disruptions.

#### Purpose
PDBs ensure high availability and reliability of applications by controlling the impact of disruptions such as node maintenance, upgrades, or scaling operations.

#### Key Fields
- **minAvailable:** Specifies the minimum number of pods that must be available.
- **maxUnavailable:** Specifies the maximum number of pods that can be unavailable.

#### Example
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: example-pdb
  namespace: my-namespace
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-app
```
This PDB ensures that at least 2 pods with the label `app: my-app` remain available during disruptions.

</details>