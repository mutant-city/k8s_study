### Enumeration

### Daddy enumeration
* `kubectl get all -A`
    * this gets everything in every namespace
    * if this is too big, can drill down more precisely with the following commands

### Cluster enumeration and CLI:
*  
```
kubectl get <item>
kubectl get <item> <item name>
kubectl describe <item> <item name>
kubectl edit <item> <item name>
kubectl delete <item> <item name>  
```

* EX:
* `kubectl describe pod <pod name> -n <namespace> `
* `kubectl get nodes`
* `kubectl get nodes $node_name`
* `kubectl get nodes $node_name -o yaml`
* `kubectl describe node $node_name`


* view logs
    * `kubectl logs <podname>`

### Get yaml from existing objects
* `kubectl get deployments ghost --export -n ghost -o yaml > objectfile.yaml`


### Gotchas 
* be sure when typing these to use the correct namespace or will get error even if object exists


### Resources

* check resources
    * CPU and Memory
    * ```
        kubectl top pods
        kubectl top pods -n <namespace, without will use default namesapce>
        kubectl top pod <podname>
        kubectl top nodes
      ```
      
* To inspect a nodes resources:
    * `kubectl describe node node-name | grep Allocatable -B 7 -A 6`
    * note: this just filters the large output of describe


### Filtering/Increasing output
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

* To get more info about things 
    * `kubectl get <x> -o yaml`
    * `kubectl describe <x>`

* Get wider output for pods: a Pod IP addy, Node, and more.
    * `kubectl get pods -o wide`
    * `kubectl get pod <podname> -o wide`
 
* list all object resources available in cluster
    * `kubectl api-resources -o name`
    * can enumerate cluster capabilities
    * i.e. pods, services, etc.    
    
    
### Services
* get external service connectivity
    * `kubectl get svc`
    
* get internal service connectivity
    *this is the connection info from service to pods 
    * `kubectl get endpoints <service name>`
     * can use it to make sure connectivity is working
     
* `kubectl get endpoints <service name`
    * this is the connection info from service to pods
    * can use it to make sure connectivity
    