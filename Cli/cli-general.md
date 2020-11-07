## CLI
* k8's has an elegant and predictable cli

* Two ways to add objects to cluster
    1. imperative
        * directly via cli
    2. declarative
        * use yaml spec files
    * https://www.digitalocean.com/community/tutorials/imperative-vs-declarative-kubernetes-management-a-digitalocean-comic

### Imperative 
* Advantages:
 * Simple, easy to learn and easy to remember.
 * Require only a single step to make changes to the cluster.

* Disadvantages:
 * Do not integrate with change review processes.
 * Do not provide an audit trail associated with changes.
 * Do not provide a source of records except for what is live.
 * Do not provide a template for creating new objects.

### Declarative

* Advantages compared to imperative object config:
    * Changes made directly to live objects are retained, even if they are not merged back into the configuration files.
    * Better support for operating on directories and automatically detecting operation types (create, patch, delete) per-object.

* Disadvantages compared to imperative object configuration:
    * Harder to debug and understand results when they are unexpected.
    * Partial updates using diffs create complex merge and patch operations.



        


### Cli help
* `kubectl explain <resource>`
    * `kubectl explain pod.spec.containers.env`
    

### Increase efficiency
* `alias k=kubectl`


