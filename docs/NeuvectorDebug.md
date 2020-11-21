
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
