### Imperitive commands
```
kubectl create -f your-object-config.yaml
kubectl delete -f your-object-config.yaml
kubectl replace -f your-object-config.yaml
```

### Declarative commands
```
kubectl diff -f configs/
kubectl apply -f configs/
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

### Gotchas 
* be sure when typing these to use the correct namespace or will get error even if object exists



### Run shell/command in container

* Run a command in a Container
    * `kubectl exec <pod name> -- <bash command here>`
    * `kubectl exec <pod name> -- curl <secure pod cluster ip address>`
    
* get a shell to a container
    * `kubectl exec -it task-pv-pod -- /bin/bash`
    




### Deployments
* `kubectl set image <>` change a deployment
    * -r  or --record flag
    * will store the changes so that can rollback if needed

*  can create deployment via cli: `kubectl expose deployment web --port=80`
    * also done via .yaml spec file

* `kubectl rollout history deployment/rolling-deployment`
* `kubectl rollout history deployment/rolling-deployment --revision=2`
* `kubectl rollout undo deployment/rolling-deployment`
* `kubectl rollout undo deployment/rolling-deployment --to-revision=1`


* impertive namespace creation
    * `kubectl create ns <namespace>`
    
    
* edit running objcts
    * can in place edit running objects
    * `kubectl edit <x> <object name>`
    * `kubectl edit deployment <deployment name>`


* Get the yaml spec from the running in memory k8s cluster
    * `kubectl get pod <pod> -n <namespace> -o yaml --export`
    * only gets the actual yaml defined spec of the pod, ignores the status and other metadata   


* can run images directly from cli
    * ` kubectl run web --image=nginx`

* Run a spec file:
    * `kubectl apply -f <filename>`

