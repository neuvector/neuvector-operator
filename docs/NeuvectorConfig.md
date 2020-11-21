
### Description
NeuVector delivers the only cloud-native Kubernetes security platform with uncompromising end-to-end protection from DevOps vulnerability protection to automated run-time security, and featuring a true Layer 7 container firewall.

The NeuVector Operator runs  in the openshift container platform to deploy and manage the NeuVector Security cluster components. The NeuVector operator contains all necessary information to deploy NeuVector using helm charts. You simply need to install the NeuVector operator from the OpenShift embeded operator hub and create NeuVector instance. You can modify the NeuVector installation configuration by modifying yaml while creating the NeuVector instance such as imagePullSecrets, tag version, etc. Please refer to [NeuVector Helm Chart] (https://github.com/neuvector/neuvector-helm) for the values that can be modifed during installation. To upgrade to a newer version of NeuVector, just reapply the NeuVector instance with desired tag , which in turn pulls the specified NeuVector image tags and upgrades as per upgrade plan configured on the helm chart. 

### Deploying NeuVector Operator
**Complete below steps to create secret for accessing Docker or similar registry and Grant Service Account Access to the Privileged SCC before installation.**

Create the NeuVector namespace

    oc new-project  neuvector
    
Configure OpenShift to pull images from the private NeuVector registry on Docker Hub
    oc create secret docker-registry regsecret -n neuvector --docker-server=https://index.docker.io/v1/ --docker-username=your-name --docker-password=your-pword --docker-email=your-email
    
Where ’your-name’ is your Docker username, ’your-pword’ is your Docker password, ’your-email’ is your Docker email.

Login as system:admin account

     oc login -u system:admin
     
Grant Service Account Access to the Privileged SCC

      oc -n neuvector adm policy add-scc-to-user privileged -z default
      
The following info will be added in the Privileged SCC users:

      - system:serviceaccount:neuvector:default

**Search for and Install the NeuVector Operator from the OpenShift Console -> OperatorHub**

**Subscribe to the NeuVector Operator**
* Choose namespace, channel and upgrade approval strategy
  *  NeuVector Operator available on the chosen namespace
  *  Current latest channel is beta, but may be moved to stable in the future (select stable if available)
  *  Automatic upgrade strategy automatically upgrades to the latest NeuVector Operator (NOT the NeuVector cluster   *   *  components such as Manager, Controller, Enforcer containers)

**Create NeuVector instance from the NeuVector tab within the NeuVector Operator**
* Click NeuVector-Operator from Installed Operators
  *  Create the NeuVector instance

**Customize values such as secret name, runtime engine, etc.**
* ImagePullSecrets should match the one created one in the preparation step.
* The container runtime engine is CRI-O by default
* Update the tag with latest tag such as 3.2.1
* Click Create button to create the NeuVector cluster components and services for Manager, Controller, and Enforcer
* Refer to the [NeuVector Helm Chart] (https://github.com/neuvector/neuvector-helm) other possible options

**Verify NeuVector cluster components installation from Resources**
* Click example-neuvector -> Resources
* Check the status of all pods

**Access the NeuVector Console**
* Click neuvector-service-webui service
* Check the neuvector-service-webui node port address or public IP
* Access NeuVector webui using one of the node IP and above node port address
* e.g. https://10.1.7.171:31172
* Login as username: admin password: admin
* Change the default password

**Add Neuvector license from NeuVector WebUI->setting**

### System Requirements
* Minimum 1GB of memory for controller, manager or all-in-one container; 1GB for enforcer.
* Shared CPU core for standard workloads, dedicated CPU (one or more) for enforcer for higher network throughput in Protect mode, or controller for high volume (10K+) image scanning.
* Registry image scanning is performed by the controller and the image is pulled and expanded in memory. If expanded image sizes larger than 500MB are expected, consider increasing the controller memory to 1.5GB or more to provide capacity and headroom for the controller.
* Recommended browser: Chrome for better performance

### Uninstalling NeuVector Operator

* First uninstall Neuvector cluster compoments by navigating to Neuvector instance
    *  InstalledOperator->Neuvector-Operator->Neuvector Instance
    *  Delete instance
* Delete Neuvector Operator by navigating to Installed Operator
    * uninstall Neuvector Operator

