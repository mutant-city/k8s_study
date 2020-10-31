## CLI
* k8's has an elegant and predictable cli

### Cluster enumeration and CLI:
*  
```
kubectl get <item>

kubectl get <item> <item name>

kubectl describe <item> <item name>

kubectl edit <item> <item name>

kubectl delete <item> <item name>  
```
* be sure when typing these to use the correct namespace or will get error even if object exists

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
        * can enter into namespaces
    * -o 
        * wide - get wider output for pods: a Pod IP addy, Node, and more.
        * yaml - get yaml of the object
        * name - get name of the object
        
    * --show-labels
    * -l
        * can filter via labels with this, see below
        * note this is a filter language, see below

* using labels/selectors: -l
    * --show-labels
    * describe has labels section as well
    * -l flag
        * `kubectl get <x> -l <label key value pair>`
        * `kubectl get pods -l app=my-app`
        * `kubectl get pods -l app!=my-app`
        * Select numerous values per label(set based): `kubectl get pods -l 'environment in (development,production)`        
        * Select numerous keys and values for labels(k/v or set based): `kubectl get pods -l app=my-app,environment=production`


* `kubectl describe pod <pod name> -n <namespace> `
* `kubectl get nodes`

* `kubectl get nodes $node_name`
* `kubectl get nodes $node_name -o yaml`
* `kubectl describe node $node_name`
* `kubectl create ns <namespace>`


* list all object resources available in cluster
    * `kubectl api-resources -o name`
    * can enumerate cluster capabilities
    * i.e. pods, services, etc. 
    
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

* Run a command in a Container
    * `kubectl exec <pod name> -- <bash command here>`
    * `kubectl exec <pod name> -- curl <secure pod cluster ip address>`
    
* get a shell to a container
    * `kubectl exec -it task-pv-pod -- /bin/bash`
    
* To inspect a nodes resources:
    * `kubectl describe node node-name | grep Allocatable -B 7 -A 6`
    * note: this just filters the large output of describe

* To get more info about things 
    * `kubectl get <x> -o yaml`
    * `kubectl describe <x>`
    

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
* view logs
    * `kubectl logs <podname>`
    
* Get the yaml spec from the running in memory k8s cluster
    * `kubectl get pod <pod> -n <namespace> -o yaml --export`
    * only gets the actual yaml defined spec of the pod, ignores the status and other metadata   
    
* `kubectl set image <>` change a deployment
    * -r  or --record flag
    * will store the changes so that can rollback if needed

*  can create deployment via cli: `kubectl expose deployment web --port=80`
    * also done via .yaml spec file

* get external service connectivity
    * `kubectl get svc`
    
* get internal service connectivity
    *this is the connection info from service to pods 
    * `kubectl get endpoints <service name>`
     * can use it to make sure connectivity is working
     
     
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
        
### Rollout new deployments
* `kubectl rollout history deployment/rolling-deployment`
* `kubectl rollout history deployment/rolling-deployment --revision=2`
* `kubectl rollout undo deployment/rolling-deployment`
* `kubectl rollout undo deployment/rolling-deployment --to-revision=1`

`
`