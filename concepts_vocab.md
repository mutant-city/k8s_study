### Kubernetes meaning
* Kubernetes (“koo-burr-NET-eez”) is the no-doubt-mangled conventional pronunciation of a Greek word, κυβερνήτης, meaning “helmsman” or “pilot.” 
* https://www.geekwire.com/2016/ever-come-kooky-kubernetes-name-heptio/

### namespaces
* without a namespace, 
    * the object will be in the default namespace
    * with a namespace, will not see the object in the basic `kubectl get pods`!!!
    * will have to add the namespace in the `get` call AND ALL calls for that object
        * without the namespace specified k8's will act like the object doesn't exist
* can add a namespace in the `metadata` section in the spec.yaml file
    * see `my-pod-namespace.yaml`
    * .metadata.namespace
* uses: 
    * organizing appplications running the same cluster
    * multiple teams 
Commands:
    create a namespace:
    * kubectl create ns <namespace name>
    specify a namespace in a get:
    * kubectl get pods -n my-ns
 
### State management
* k8s tries to make the status match the deployment spec
* spec 
    * desired state of that object
* status
    * the current state of the object
* can see spec and status via `kubectl describe` commands

### Vocab
* Node: The actual server hardware that a k8's daemon is running on
* Pod: A collection of containers that serve a purpose that run on a Node
* Container: the actual singlar unit of work/container that is running
* ton more objects

### Networking
* Each Pod is assigned a unique IP address for each address family. 
* Every container in a Pod shares the network namespace, including the IP address and network ports. 
* Inside a Pod (and only then), the containers that belong to the Pod can communicate with one another using localhost.

### SecurityContext
* pods run all containers as root by default
* security groups effectively change the user/group of the running the pod 
* can set the pod as a user and group without root access
* interactions between container and node
* defines the containers user/group running on the node
    * if the container doesn't have the permissions then things like volume mounts will mount but the files will deny access
    * i.e. chmod and chown a location on the node for a user that is different than the security context's defined user for the pod
        * the pod won't have access
* Note: Pod status will be `Error` if have issues with permissions of any mounted files

### Resource Requirements
* k8's allows us to specify the resource requirements of a container in the pod spec
* memory/cpu 
* defined in terms of:
    * resource requests 
        * the amount of resources on the node requested/necessary to run a container
        * pod will only run on a node that has enough resources to run it
        * K8s's typically kills container if not enough resources as defined in resources are available 
        * k8s kills a container if the container passes its defined limit, i.e. uses to many resources
    * limits
        * maximum value for the resource usage of a container on a node
        * if goes above limit, container will be killed 
* specified under the `container` root key 
    * .spec.containers[].resources
* notation is here
    * https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#meaning-of-memory
* cpu limits are percentage of a cpu core. 1000 millicpus = 1 cpu core. 
    * so 250m is 1 quarter of a cpu core.
* doing a describe on a container will also show this.
* can see the limits/requests via `kubectl describe`


### Labels
* under the metadata parent node: .metadata.labels
* cli flag = --show-labels
* or there's a labels section in describe
* useable by selectors via other objects or CLI to identify objects

### RestartPolicy
* .spec.restartPolicy
* restartPolicy field with possible values Always, OnFailure, and Never. 
* The default value is Always.
* The restartPolicy applies to all containers in the Pod.

### Annotations
* used to store custom metadata about objects
* not intended to be identifying and not usable by selection
* just attach some kind of custom data 

### Selectors
* provide us a way to select a list of objects based on their lable
* used by objects to grab other objects and apply functionality
* used by cli to obtain objects
* in objects .spec.selector.matchLabels

### Service account:
* service accounts allows the pod to access the kubernetes API
* some pods need acces to k8's cluster
* .spec.serviceAccountName: <account name>
* on CKAD don't need to know how to configure service accounts inside of the k8's cluster

### Monitoring
* One of many kubernetes Monitoring apps
```
git clone https://github.com/linuxacademy/metrics-server
kubectl apply -f ~/metrics-server/deploy/1.8+/
kubectl get --raw /apis/metrics.k8s.io/
```

### Storage
* containers internal storage ephemeral by default
    * when that container is deleted that storage is gone

* Kubernetes is designed to manage stateless containers
    * pods/containers should be easily deletable and replacable
    * since data inside container is gone when container deleted need external storage to container lifecycle

* State persistence: maintaining data outside and potentially beyond the lifecycle of a container
* PersistentVolume and PersistentVolume claims provide easy way to implement/consume storage resources in the context 
    of complex production environment that has numerous storage solutions
    * can abstract out s3, google cloud, EBS, etc. with a PersistenVolume object
    
* a Node represents CPU and Memory resources while a PV represents Storage resources

* PersistentVolume represents a storage resource


* StorageClass: 
    * defines categories of storage
    * ex: fast, slow
    
* accessModes:
    * determine what read/write modes can access the volume
    * how many pods/read, write/etc.
    * ex: ReadWriteOnce
    
        
* storageSystem<-->PersistentVolume<-->PersistentVolumeClaim<-->Pod

* Volume
    * Volumes exists outside the lifetime of the container and aren't ephemeral
    * mounts volumes to specific container
    * setup inside of the pod spec-> .spec.volumes
    * assigned to container/s via -> .spec.containers.volumeMounts[]
    * types of volume:
        * .spec.volumes.emptyDir volume
            * can add an emptyDir to numerous containers who all now share volume
            * in below spec file the two containers share the my-volume storage at /tmp/storage
        * ```
            apiVersion: v1
            kind: Pod
            metadata:
              name: volume-pod
            spec:
              containers:
                  - image: busybox
                    name: busybox
                    command: ["/bin/sh", "-c", "while true; do sleep 3600; done"]
                    volumeMounts:
                    - mountPath: /tmp/storage
                      name: my-volume         
                  - image: busybox
                    name: busybox
                    command: ["/bin/sh", "-c", "while true; do sleep 3600; done"]
                    volumeMounts:
                    - mountPath: /tmp/storage
                      name: my-volume
              volumes:
                - name: my-volume
                  emptyDir: {}
        ```

* Two ways PVs may be provisioned: statically or dynamically
    * static: via statically provisioned PV/PVC
    * dynamic: via StorageClasses
    
* AccessModes
    * ReadWriteOnce -- the volume can be mounted as read-write by a single node
    * ReadOnlyMany -- the volume can be mounted read-only by many nodes
    * ReadWriteMany -- the volume can be mounted as read-write by many nodes


* Steps
    1. create PV `kubectl apply`
        * `kubectl get pv <pv name>`
            * look for status of available
    2. create PVC `kubectl apply`
        * `kubectl get pvc`
        * `kubectl get pv`
            * look for status of bound on both
    3. Add the PVC to the Pod
    
    
* PersistentVolume
    * this sets up the storage ON THE STORAGE SIDE
    * location of the storage on the storage provider or local host
    * types:
        * hostPath PersistentVolume. 
            * Kubernetes supports hostPath for development and testing on a single-node cluster.
            * In a production cluster, you would not use hostPath. 
                Instead a cluster administrator would provision a network resource like a 
                Google Compute Engine persistent disk, an NFS share, or an Amazon Elastic Block Store volume.
    * `kubectl get pv <pv name>`
        * can see if status is Available
    * https://kubernetes.io/docs/concepts/storage/persistent-volumes/
                
* PersistentVolumeClaim
    * PVC is Abstraction between the user(Pod) and the PersistentVolume
    * bind PVC to PV
        * PVC's automatically bind to a PV that has compatible:
            * StorageClassName
            * accessModes
            * enough space to provide the requested amount
        * Can bind to PV before bind to Pod
            * `kubectl get pv`
            * can check if bound to PV
            
    * bind to pod
        * spec.volumes.persistentVolumeClaim
        * instead of volume version: .spec.volumes.emptyDir
        * note: the pod doesn't know about the PV just the PVC

* Retention
    * The reclaim policy for a PersistentVolume tells the cluster what to do with the volume after it has been released of its claim
    * Currently, volumes can either be Retained, Recycled, or Deleted.
    * Reclaim: 
        * When the PersistentVolumeClaim is deleted, the PersistentVolume still exists and the volume is considered "released". 
        * But it is not yet available for another claim because the previous claimant's data remains on the volume. 
        * admin must manually follow steps to reclaim
    * Delete:
        * deletion removes both the PersistentVolume object from Kubernetes, as well as the associated storage asset in the external infrastructure, 
          such as an AWS EBS, GCE PD, Azure Disk, or Cinder volume.
    * Recycle:
        * If supported by the underlying volume plugin, the Recycle reclaim policy performs a basic scrub (rm -rf /thevolume/*) on the volume and makes it available again for a new claim.



* StorageClass
    Volumes that were dynamically provisioned inherit the reclaim policy of their StorageClass, which defaults to Delete