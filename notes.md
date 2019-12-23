spec 
	* desired state of that object
status
	* the current state of the object

k8s tries to make the status match the spec



Pods-group of one or more containers with shared storage/network, and a specification for how to run the containers.
Node-actual server the pods run on


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
    * defines privelege and access control settings for pods/containers for a pid
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





















