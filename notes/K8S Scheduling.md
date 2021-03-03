**`K8S Scheduling:`**


For every newly created pod or other unscheduled pods, kube-scheduler selects an optimal node for them to run on.


* every container in pods has different requirements for resources and every pod also has different requirements. Therefore, existing nodes need to be filtered according to the specific scheduling requirements.
* Schelling decisions are based 
    * resource requirement
    * hardware/software/policy constraints 
    * affinity and anti-affinity spec
    * Taints/Tolerations
    * data locality *
    * inter-workload interference and deadlines

Node selection two step process 

1. Filtering
2. Scoring

In a cluster, Nodes that meet the scheduling requirements for a Pod are called feasible nodes.

scheduler scores all feasible nodes, and select highest score node to run a pod, scheduler then notifies the API server about this decision in a process called binding.

If there is more than one node with equal scores, kube-scheduler selects one of these at random.

**`Taint & Tolerations`**

* is a property of Nodes that repel pods from Nodes
* Tolerations are applied to pods, and allow (but do not require) the pods to schedule onto nodes with matching taints.
* `NoSchedule` → future pods will not be placed & existing are continue to run
* `PreferNoSchedule` → soft version of NoSchedule, future pods will try to avoid this Node but can still run here if any other Nodes are not available
* `NoExecute`  → No future pods, existing pods are stopped & moved (evicted) to other nodes. Only exception to this Pods that have toleration to NoExecute taint
* use-case:
    * troubleshooting a Node without affecting application running in pod 
    * *`Dedicated Nodes`*: reserve certain node for a pod 
    * *`Nodes with Special Hardware`*: keep other pods in your cluster away from node
    * *`Taint based Evictions`*
        * The NoExecute taint effect, mentioned above, affects pods that are already running on the node as follows
            * pods that do not tolerate the taint are evicted immediately
            * pods that tolerate the taint without specifying tolerationSeconds in their toleration specification remain bound forever
            * pods that tolerate the taint with a specified tolerationSeconds remain bound for the specified amount of time
* node controller automatically taints a Node when certain conditions are true
    * like node is in not ready/unkown status, out-of-disk, memory-pressure, network-unavailable, unschedulable ,  kubelet uninitialized
    * In case a node is to be evicted, the node controller or the kubelet adds relevant taints with NoExecute effect.

**`Assigning Pods to Nodes`**

* You can constrain a Pod to only be able to run on particular Node(s), or to prefer to run on particular nodes.

*`nodeSelector`* is a field of PodSpec. It specifies a map of key-value pairs. For the pod to be eligible to run on a node, the node must have each of the indicated key-value pairs as labels

To label a node, 
> kubectl label nodes <node_name> key=value


*`Affinity & Anti-Affinitiy`*

* nodeSelector provides a very simple way to constrain pods to nodes with particular labels. The affinity/anti-affinity feature, greatly expands the types of constraints you can express.

* property of pods that Attracts them to a set of nodes (either as a preference or a hard requirement)

    *  requiredDuringSchedulingIgnoredDuringExecution
    * preferredDuringSchedulingIgnoredDuringExecution

1. `Node affinity:`

    * Node affinity is conceptually similar to nodeSelector -- it allows you to constrain which nodes your pod is eligible to be scheduled on, based on labels on the node.


2. `Inter-pod affinity and anti-affinity:`

    *   Inter-pod affinity and anti-affinity allow you to constrain which nodes your pod is eligible to be scheduled based on labels on pods that are already running on the node rather than based on labels on nodes. 
    * Always co-located in the same node 