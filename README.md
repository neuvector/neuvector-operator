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


### Troubleshooting

#### Troubleshooting NeuVector Deployments
The NeuVector containers are deployed, managed, and updated using the same orchestration tool used for application workloads. Please be sure to review the online documentation for each step necessary during deployment. Often deployments are attempted by just copying the sample yaml files and deploying them without reviewing the steps prior, such as properly configuring registries, secrets, or RBACs/rolebindings.

##### Initial Deployment
* Check that the NeuVector containers can be pulled with correct authentication. Check the secret used and make sure the cluster is able to access the appropriate registry.
* Make sure the changes to the yaml required (e.g. NodePort or LoadBalancer) or Helm values settings are set appropriately.
* Check the platform and container run-time and make changes as needed (e.g. PKS, containerd, CRI-O).

##### Login and Initial Configuration
* Check to make sure appropriate access to the manager (IP address, port, route) is allowed through firewalls.
* Apply the license file in order to see all containers, nodes, and Enforcers

##### Ongoing Operation
* Directory integration. NeuVector supports specific configurations for LDAP/AD and other integrations for groups and roles. Contact NeuVector for additional troubleshooting steps and a tool for AD troubleshooting.
* Registry scanning. Most issues are related to registry authentication errors or inability for the controller to access the registry from the cluster.
* For performance issues, make sure the controller is allocated enough memory for scanning large images. Also, CPU and memory minimums can be specified in the pod policy to ensure adequate performance at scale.
* Admission Control. See the Troubleshooting section in the section Security Risks... -> Admission Controls.

##### Updating
* Use rolling updates for the controller. If you are rebooting hosts, make sure to monitor the controllers as they move to other hosts, or redeploy on the rebooted hosts, to make sure they are able to start, join the controller cluster, and stabilize/sync. Rebooting all hosts at once or too quickly can result in unknown states for the controllers.
* Use a persistent volume claim to store the NeuVector configuration for the case that all controllers/nodes go down in the cluster.
* When updating to a new version, review the online documentation to identify changes/additions to the yaml required, as well as other changes such as rolebindings or new services (e.g. admission control webhook, persistent volume claim etc).

##### Debug Logs
To view the logs of a NeuVector container, for example a controller pod

```kubectl logs neuvector-controller-pod-777fdc5668-4jkjn -n neuvector```

These logs may show cluster connectivity issues, admin actions, scanning activity and other useful entries. If there are multiple controllers running it may be necessary to inspect each one. These logs can be piped to a file to send to NeuVector support.

#### Turning on Debug mode for NeuVector Controllers
For issues that require in-depth investigation, debug mode can be enabled for the controllers/allinones, which will log detailed information. This can increase the log file size by a large amount, so it is recommended to turn it off after collecting them.

To turn on Debug mode using the REST API, please see the section below. After collecting the log files, please email to support@neuvector.com.

#### Kubernetes, OpenShift and Other Orchestration Logs
It can be helpful to inspect the logs from orchestration tools to see all deployment activity including pod creation timestamps and status, deployments, daemonsets and other management actions of the NeuVector containers performed by the orchestration tool. These can also be collected and sent to support@neuvector.com

```kubectl get events -n neuvector```
    
##### Support Log
The support log contains additional information which is useful for NeuVector Support, including system configuration, containers, policies, notifications, and NeuVector container details.

To download the support log, go to Settings -> Configuration and select Collect Log. Please email the log file to support@neuvector.com    