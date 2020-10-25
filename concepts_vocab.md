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

### Vocab
* Node: The actual server hardware that a k8's daemon is running on
* Pod: A collection of containers that serve a purpose that run on a Node
* Container: the actual singlar unit of work/container that is running

### Networking
* Each Pod is assigned a unique IP address for each address family. 
* Every container in a Pod shares the network namespace, including the IP address and network ports. 
* Inside a Pod (and only then), the containers that belong to the Pod can communicate with one another using localhost.

### SecurityContext
* defines privilege and access control settings for pods/containers for a pid
* can give a container special operating system privileges
* `securityContext` defined as an element in the `spec` root element 

### Resource Requirements
* k8's allows us to specify the resource requirements of a container in the pod spec
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


### Labels
* under the metadata parent node: .metadata.labels
* cli flag = --show-labels
* or there's a labels section in describe
* useable by selectors via other objects or CLI to identify objects

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
* can allow certain service accounts to use a certain pod
* service accounts allows the pod to access the kubernetes API

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
    * 
* Volume
    * Volumes exists outside the lifetime of the container and aren't ephemeral
    * mounts volumes to specific container
    * setup inside of the pod spec-> .spec.volumes
    * assigned to container/s via -> .spec.containers.volumeMounts[]
    * types of volume:
        * emptyDir volume
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
          
       
* PersistentVolume
