## K8's objects
* Pod, Deployment, Service,  ConfigMap, Job/CronJob, NetworkPolicies, PersistentVolume, PersistentVolumeClaim
* and more :)
* also called API Primitives

### Pods:
* Runs a single set of containers
* Good for one-off dev purposes
* Rarely used directly in production
* No state monitoring, no replicas, etc.
* In kubernetes Pods are the smallest deployable units. 
* Every time we create a kubernetes object like Deployments, replica-sets, statefulsets, daemonsets it creates pod.
* run commands in containers and pass args:
    * .spec.containers[].command
    * .spec.containers[].args
    
   ```
    containers:
      - name: myapp-container
        image: busybox
        command: ['echo']
        args: ['This is my custom argument']
    
    ```
* open ports to cluster on the container
    * by default ports only available to other containers in the Pod
    * with this will open to other pods in cluster and if configured to the world
    * .spec.containers[].ports[].containerPort

### ReplicaSets:
* maintains a replicated number of pods
* References:
    * https://thenewstack.io/kubernetes-deployments-work/

### Deployment:
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

### Services
*  act as the load balancers for Kubernetes traffic, internal and external.
*  uses selectors
*  can create deployment via cli: `kubectl expose deployment web --port=80`
    * also done via .yaml spec file
*  The service will have its own internal IP address (denoted by the name ClusterIP)
*  connections to this IP address on port 80 will be load-balanced across all the pods of this deployment, matching the service’s selector..
*  When we edit the deployment and trigger a rolling update, a new replica set is created. This replica set will create pods, whose labels will include (among others) run=web. As such, these pods will receive connections automatically.
*  This means that during a rollout, the deployment doesn’t reconfigure or inform the load balancer that pods are started and stopped. It happens automatically through the selector of the service associated to the load balancer.
*  `kubectl get endpoints <service name`
     * this is the connection info from service to pods
     * can use it to make sure connectivity
    
* service types
    * clusterIP : exposes the service INSIDE the cluster 
    * NodePort: exposes the service OUTSIDE the cluster on a NODE port
    * LoadBalancer: provision a load balancer EXTERNAL to the app(i.e. an AWS or AZURE load balancer)
    * ExernalName: Maps the service to an external URL, such a database outside of the cluster, or an API

### NetworkPolicies
* Like Security Groups from AWS but for K8's
* By default(with no network policy) all pods are allowed to communicate with any other pod and reach out to any available IP
* As soon as a NetworkPolicy applies to a pod(i.e. valid pod selectors)
    * the default will be to blacklist everything
    * the NetworkPolicy of allowed options will be a whitelist
* Note: numerous k8's cluters don't support NetworkPolicies by default(may need to setup canal or cilium)
* ```
    kubectl get networkpolicies
    kubectl describe networkpolicy my-network-policy
      
      ```
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
### Jobs/CronJobs
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
    
### PersistentVolume
* TBD
    
### PersistentVolumeClaim
* TBD

### Config Map
* k/v store of configuration data that you need to pass into pods/containers
* can pass in two ways
    1. via env section of spec file
    2. via a volume mount
        * if passed in as volume mount, each TOP LEVEL key will be added as a FILE in your mount point
        * so if your mount point is `/etc/config` and you mount a config map there will be one file per TOP LEVEL KEY

### Secrets:
* can create a secret file
* can set it in spec file with apply














