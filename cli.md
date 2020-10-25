### Cluster enumeration and CLI:
*  
```
kubectl get <item>

kubectl get <item> <item name>

kubectl describe <item> <item name>

kubectl edit <item> <item name>

kubectl delete <item> <item name>  
```
* Where item=
    * deployments
    * pods
    * nodes
    * services
    * pv (PersistentVolume)
    * pvc (PersistentVolumeClain)
    * rs (ReplicaSets)
    * events
    * namespaces
    * jobs
    * cronjobs
    * networkpolicies
    * endpoints
    * all

* useful flags
    * --all-namespaces
    * -n <namespace>
    * -o 
        * wide - get wider output for pods: a Pod IP addy, Node, and more.
        * yaml - get yaml of the object
        * name - get name of the object
        
    * --show-labels
    * -l
        * can filter via labels with this, see below

* `kubectl describe pod <pod name> -n <namespace> `
* `kubectl get nodes`
* `kubectl api-resources -o name`
* `kubectl get nodes $node_name`
* `kubectl get nodes $node_name -o yaml`
* `kubectl describe node $node_name`


* service
    * `kubectl get endpoints <service name`
        * this is the connection info from service to pods
        * can use it to make sure connectivity


* can run images directly from cli
    * ` kubectl run web --image=nginx`

* Run a spec file:
    * `kubectl apply -f <filename>`

* Get wider output for pods: a Pod IP addy, Node, and more.
    * `kubectl get pods -o wide`
    * `kubectl get pod <podname> -o wide`

* Run a command in a Container in 
    * `kubectl exec <pod name> -- curl <secure pod cluster ip address>`

* To inspect a nodes resources:
    * `kubectl describe node node-name | grep Allocatable -B 7 -A 6`
    * note: this just filters the large output of describe

* To get more info about things 
    * `kubectl get <x> -o yaml`
    * `kubectl describe <x>`
    
* using labels/selectors
    * --show-labels
    * describe has labels section as well
    * -l flag
        * `kubectl get <x> -l <label key value pair>`
        * `kubectl get pods -l app=my-app`
        * `kubectl get pods -l app!=my-app`
        * Select numerous values per label(set based): `kubectl get pods -l 'environment in (development,production)`        
        * Select numerous keys and values for labels(k/v or set based): `kubectl get pods -l app=my-app,environment=production`

* edit running objcts
    * can in place edit running objects
    * `kubectl edit <x> <object name>`
    * `kubectl edit deployment <deployment name>`

* check resources
    * CPU and Memory
    * ```
        kubectl top pods
        kubectl top pods -n <namespace, without will use default namesapce>
        kubectl top pod <podname>
        kubectl top nodes
      ```
### Running objects
* commands:
    * create
    * start,stop,delete
    * edit
        * opens up in a text editor to edit the object spec
    * apply 
        * via file: -f <filename>
        * updates the object from the spec
    * view logs
        * `kubectl logs <podname>`
    * execute command inside a pod
        * exec command with -- and a space
        * `kubectl exec <podname> -- ls /etc/config`
    * exec 
        * run a command on the container
    * get
        * gets a big picture view of all of the objects in that class
    * describe
        * gets detailed information about specific object   
        
### Troubleshooting/Debugging
* `kubectl get X` 
    * see the status field in the `get` command statements
* `kubectl logs`
* Removing a pod from the scope of the ReplicationController comes in handy
when you want to perform actions on a specific pod. For example, you might 
have a bug that causes your pod to start behaving badly after a specific amount 
of time or a specific event.

### Fixing pods by editing the in memory cluster k8's spec
* kubectl edit
* can directly edit the definition itself
* when save the file, will automatically/edit & update the pod 
* note: can't edit certain fields once a pod is running
    * for example: liveness probes
    * have to delete and recreate the pod
* can get the spec from the in memory k8s cluster
    * `kubectl get pod <pod> -n <namespace> -o yaml --export`
    * only gets the actual spec of the pod, ignores the status and other metadata
    

###  Logging
* Everything a containerized application writes to stdout and stderr is handled and redirected somewhere by a container engine.  
* ensure log rotation on the node so that space doesn't fill up 
* ```
    kubectl logs <pod name> # pod logs
    kubectl logs <pod name> -c <container name> # specific container logs
    ```
* For crashed containers
    * `kubectl logs --previous <pod name> -c <container name>`
* Exceptions:
    * The kubelet and container runtime, for example Docker, do not run in containers, the run on the node and log to the node.
        * On machines with systemd, the kubelet and container runtime write to journald. If systemd is not present, they write to .log files in the /var/log directory. System components inside containers always write to the /var/log directory, bypassing the default logging mechanism.
        * logrotate node logs as well
* https://kubernetes.io/docs/concepts/cluster-administration/logging/

