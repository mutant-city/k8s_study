### Kubernetes meaning
* Kubernetes (“koo-burr-NET-eez”) is the no-doubt-mangled conventional pronunciation of a Greek word, κυβερνήτης, meaning “helmsman” or “pilot.” 
* https://www.geekwire.com/2016/ever-come-kooky-kubernetes-name-heptio/

### Cluster enumeration and CLI:
* `kubectly get pods`
    * checks default namespace
* `kubectl get namespace`
    * get list of all namespaces
* `kubectl get pods -n <namespace>`
* `kubectl get pods --all-namespaces`
    * get all pods that exist
    
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
    * all

* useful flags
    * --all-namespaces
    * -n <namespace>

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


###  K8's objects
* Pod, Deployment, Service,  ConfigMap, Job/CronJob, NetworkPolicies, PersistentVolume, PersistentVolumeClaim

* Pods:
    * Runs a single set of containers
    * Good for one-off dev purposes
    * Rarely used directly in production
    * No state monitoring, no replicas, etc.
    * In kubernetes Pods are the smallest deployable units. 
    * Every time we create a kubernetes object like Deployments, replica-sets, statefulsets, daemonsets it creates pod.

* ReplicaSets:
    * maintains a replicated number of pods
    * References:
        * https://thenewstack.io/kubernetes-deployments-work/
 
* Deployment:
    * NOTE: in yaml file-> apiVersion: apps/v1
    * .spec.replicas=number of pods
    * .spec.template=template pod descriptor which defines pods to be created
    * .spec.selector=deploymnet will manage all pods whose lables match this selector
    * combines/wrapper around ReplicaSets and Pods and also the ability to deploy
    * will create pods and replicasets
    * when update number of specs, Deployment object sends that down to the replica set
    * when update the actual spec, i.e. change containers or parameters or something
        * Deployment creates a new replica set with size zero 
        * then  Then, the size of that replica set is progressively increased, while decreasing the size of the other replica set.
    * has readiness probes
        * A readiness probe is a test that we add to a container specification. It’s a binary test, that can only say “IT WORKS” or “IT DOESN’T,” and will get executed at regular intervals. (By default, every 10 seconds.)
        * Kubernetes uses the result of that test to know if the container and the pod that it’s a part of are ready to receive traffic. When we roll out a new version, Kubernetes will wait for the new pod to mark itself as “ready” before moving on to the next one.
        * If a pod never reaches the ready state because the readiness probe keeps failing, Kubernetes will never move on to the next. The deployment stops, and our application keeps running with the old version until we address the issue.
        * If there is no readiness probe, then the container is considered as ready, as long as it could be started. So make sure that you define a readiness probe if you want to leverage that feature!
    * Rolling between replica sets
        * see: https://semaphoreci.com/blog/kubernetes-deployment
    * MaxUnavailable: Setting MaxUnavailable to 0 means, “do not shutdown any old pod before a new one is up and ready to serve traffic“.
    * MaxSurge: Setting MaxSurge to 100% means, “immediately start all the new pods“, implying that we have enough spare capacity on our cluster, and that we want to go as fast as possible.
    * The default values for both parameters(MaxSurge and MaxUnavailable) are 25%, meaning that when updating a deployment of size 100, 25 new pods are immediately created, while 25 old pods are shutdown
    * Higher level functionality, Creates pods as part of the deployment, with additional functionality
        * you can see the pods it creates with `kubctl get pods` and see the deployment with `kubectl get deployments`
    * Runs a set of identical pods
    * Monitors the state of each pod, updating as necessary
    * Under the hood creates ReplicaSet
        *  ReplicaSet constantly monitors the list of running pods and makes sure the running 
        number of pods matching a certain specification always matches the desired number.
        * When using a Deployment, the actual pods are created and managed by the Deployment’s ReplicaSets, not by the Deployment directly
    * Deployment->ReplicaSet->Pods
    * Good for dev
    * Good for production
    * You can rollout and rollback your changes using deployment
    * Monitors the state of each pod
    * Best suitable for production
    * Supports rolling updates
    * In spec file at least one of the .spec.selector.matchLabels.{} must match one .spec.template.metadata.labels.{}: 
        * ```
          spec:
            selector:
                matchLabels:
                    app: nginx
      ````
        * ```
      spec:
          template:
            metadata:
              labels:
                app: nginx
      ```
        * `selector` does not match template `labels`
    * References: 
        * https://semaphoreci.com/blog/kubernetes-deployment
        * https://thenewstack.io/kubernetes-deployments-work/

* Services
    *  act as the load balancers for Kubernetes traffic, internal and external.
    *  uses selectors
    *  create deployment via cli: `kubectl expose deployment web --port=80`
    *  The service will have its own internal IP address (denoted by the name ClusterIP)
    *  connections to this IP address on port 80 will be load-balanced across all the pods of this deployment, matching the service’s selector..
    *  When we edit the deployment and trigger a rolling update, a new replica set is created. This replica set will create pods, whose labels will include (among others) run=web. As such, these pods will receive connections automatically.
    *  This means that during a rollout, the deployment doesn’t reconfigure or inform the load balancer that pods are started and stopped. It happens automatically through the selector of the service associated to the load balancer.
    *  `kubectl get endpoints <service name`
         * this is the connection info from service to pods
         * can use it to make sure connectivity
        
    * service types
        * clusterIP : exposes the service INSIDE the cluster 
        * NodePort: exposes the service OUTSIDE the cluster on a node port
        * LoadBalancer: provision a load balancer EXTERNAL to the app(i.e. an AWS or AZURE load balancer)
        * ExernalName: Maps the service to an external URL, such a database outside of the cluster, or an API

* NetworkPolicies
    * Like Security Groups from AWS but for K8's
    * By default(with no network policy) all pods are allowed to communicate with any other pod and reach out to any available IP
    * As soon as a NetworkPolicy applies to a pod(i.e. valid pod selectors)
        * the default will be to blacklist everything
        * the NetworkPolicy of allowed options will be a whitelist
    * Note: numerous k8's cluters don't support NetworkPolicies by default(may need to setup canal or cilium)
    * Spec:
        * The pod selector applies this NetPol to pods and locks them down/blacklist everything from them
        * the policyTypes/ingress/egress opens up per those specifications
         ```
        apiVersion: networking.k8s.io/v1
        kind: NetworkPolicy
        metadata:
          name: my-network-policy
        spec:
          podSelector:  # APPLIES THIS POLICY TO PODS
            matchLabels:
              app: secure-app
          policyTypes:   # WHITELIST
          - Ingress
          - Egress
          ingress:        # WHITELIST
          - from:
            - podSelector:
                matchLabels:
                  allow-access: "true"
            ports:
            - protocol: TCP
              port: 80
          egress:        # WHITELIST
          - to:
            - podSelector:
                matchLabels:
                  allow-access: "true"
            ports:
            - protocol: TCP
              port: 80
        ```
* Jobs/CronJobs
    * used to reliably execute a workload until it completes
    * as opposed to pods which continue to run constantly
    * will creeate one or more pods which will enter completed state after running the job
    * part of batch api, 
        * apiVersion: batch/v1
    * backoffLimit: 4 => how many times job will attempt to run if it fails
    * restartPolicy: Never (never attemp to resart)
    * restartPolicy: OnFailure (automatically restart if fails, but if complets wont be restarted)
    * CronJob-run job workload periodically, according to a schedule
        * .spec.schedule: "*/1 * * * *"
        * `kubectl get cronjobs`
        * will allow you to see the scheduling info for the cronjob
        
* PersistentVolume
    * TBD
    
* PersistentVolumeClaim
    * TBD

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
    


### State management
* k8s tries to make the status match the spec
* spec 
    * desired state of that object
* status
    * the current state of the object

### Vocab
* Node: The actual server hardware that a k8's daemon is running on
* Pod: A collection of containers that serve a purpose that run on a Node


### Networking
* Each Pod is assigned a unique IP address for each address family. 
* Every container in a Pod shares the network namespace, including the IP address and network ports. 
* Inside a Pod (and only then), the containers that belong to the Pod can communicate with one another using localhost.


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


### Monitoring
* check resources
    * CPU and Memory
    * ```
        kubectl top pods
        kubectl top pods -n <namespace, without will use default namesapce>
        kubectl top pod <podname>
        kubectl top nodes
      ```
      
* Kubernetes Monitoring app
```
git clone https://github.com/linuxacademy/metrics-server
kubectl apply -f ~/metrics-server/deploy/1.8+/
kubectl get --raw /apis/metrics.k8s.io/
```




-o 
	* yaml
	* name



* commands:
    * create
    * start,stop,delete
    * edit
        * opens up in a text editor to edit
    * apply
        * updates the object from the spec
    * view logs
        * `kubectl logs <podname>`
    * execute command inside a pod
        * exec command with -- and a space
        * `kubectl exec <podname> -- ls /etc/config`
    * exec 
        * run a command on the container
    * get
        * gets a big picture view
    * describe
        * gets detailed information about specific object

namespaces
    * without a namespace, 
        * the object will be in the default namespace
        * with a namespace, will not see the object in the basic `kubectl get pods`!!!
        * will have to add the namespace in the `get` call AND ALL calls for that object
            * without the namespace specified k8's will act like the object doesn't exist
    * can add a namespace in the `metadata` section in the spec.yaml file
        * see `my-pod-namespace.yaml`
    * uses: 
        * organizing appplications running the same cluster
        * multiple teams 
    Commands:
        create a namespace:
        * kubectl create ns <namespace name>
        specify a namespace in a get:
        * kubectl get pods -n my-ns
 
 
 Config Map
    * k/v store of configuration data that you need to pass into pods/containers
    * can pass in two ways
        1. via env section of spec file
        2. via a volume mount
            * if passed in as volume mount, each TOP LEVEL key will be added as a FILE in your mount point
            * so if your mount point is `/etc/config` and you mount a config map there will be one file per TOP LEVEL KEY
 
 
 SecurityContext
    * defines privilege and access control settings for pods/containers for a pid
    * can give a container special operating system privileges
    * `securityContext` defined as an element in the `spec` root element 

 Resource Requirements
    * k8's allows us to specifiy the resource requirments of a container in the pod spec
    * memory/cpu 
    * defined in terms of:
        * resource requests 
            * the amount of resources necessary to run a container
            * pod will only run on a node that has enough resources to run it
            * K8s's typically kills container if not enough resources as defined in resources are available 
            * k8s kills a container if the container passes its defined limit, i.e. uses to many resources
        * limits
            * maximum value for the resource usage of a container
        * specified under the `container` root key 

    * notation is here
        * https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#meaning-of-memory
    * cpu limits are percentage of a cpu core. 1000 millicpus = 1 cpu core. 
        * so 250m is 1 quarter of a cpu core.

    * doing a describe on a container will also show this.

  Secrets:
    * can create a secret file
    * can set it in spec file with apply
  
  Service account:
    * can allow certain service accounts to use a certain pod
    * service accounts allows the pod to access the kubernetes API



### Labels
* under the metadata parent node: .metadata.labels
* cli flag = --show-labels
* or there's a labels section in describe
* useable by selectors via other objects or CLI to identify objects

### Annotations
* used to store custom metadata about objects
* not intended to be identifying and not usable by selection
* just attach somekind of custom data 

### Selectors
* provide us a way to select a list of objects based on their lable
* used by objects to grab other objects and apply functionality
* used by cli to obtain objects
* in objects .spec.selector.matchLabels













