# Kubernetes research

## Comparison with AtYourService:
See https://platform9.com/blog/kubernetes-docker-swarm-compared/

## Comparison with AtYourService and 0-orchestrator:
AtYourService integration with orchestrator and k8s(Kubernetes) both serve as a virtualization orchestrator,
and so have a lot of similarities when it comes to orchestration concepts. Although they might implement each concept
differently it is important to keep in mind what they do for a comprehensive Comparison.
similarities include but are not limited to :


| similarity    | AYS           | K8s  |
| ------------- |:-------------:| -----:|
| service abstraction      |**AYS Services, Actions and framework** |**K8s Pods, Deployment, Nodes, Services**|
| monitoring      |**AYS long running jobs and recurring jobs** | **K8s Heapster**, **K8s Replication controllers**|
| logging | **AYS logging**     |  **K8s logs** |
| self-healing  |  **Orchestrator watchdog**  |   **K8s Liveness-probe** |
| service grouping or labeling  | **AYS Services and Orchestrator** | **K8s Labels**, **K8s Selectors**, **K8s Namepsaces** |
| rest api for querying running and controlling services  |   **AYS API & orchestrator API**   | **K8s API and clients** |


### AYS

**AYS Services** : ays services allow grouping and creating relations between actions and running process as well as other services.

**AYS Actions and framework** : implemented through Capnp and yaml files with child parent relations for stateful
abstraction of running services,the state transition is handled through easy to use python framework.

**Orchestrator** : orchestrator is another abstraction layer over services to define a cpu and storage cloud enviroment.

**AYS long running jobs and recurring jobs** : AYS framework allow for custom jobs to be run on a specific service/s multiple
times through a set period or indefinitely every set period.

**Orchestrator watchdog** : listens to a que from the nodes that outputs logs from specific jobs and checks on certain conditions that are defined in the actions.py of the node and the responding action to be taken is defined in the respective service's watchdog_handler action.

**AYS API & orchestrator API** : both have  a restful api and python clients.Ays has a command line interface to control as well.

**AYS logging** : logs are labeled and can be filtered though job action and service


### Kubernetes

**K8s Pods**: This is the first layer of abstraction above the basic building blocks of kubernetes which are the containers (which can either be docker, RKT, or any container runtime that implements the [OpenContainerIinitiative](https://www.opencontainers.org/)specs) this layer can container one or more containers and can share storage , load balancers and other resources.Also allows some policies and naming conventions to make it a stateful object.

**K8s Replica-sets**: The second layer of abstraction , a replica set has one or more Pods and is usually used for replication and to maintain a certain number of replica pods , this layer does not handle networking and other issues in replication and so is not commonly used, while a deployment is.

**K8s Deployments**: The layer above replica-sets a deployment while handle the replication adjusting the network traffic in case a pod is down and was restarted and in some cases can handle connecting resources such as storage.

**K8s Services**: The service here defines a network routing application that handles networking issues such as load balancing dns and port forwarding.

**K8s Nodes**: The actual machines that hosts the container runtime , it is what pods deploy on , also refrenced as kubelet

**K8s Replication controllers** :A ReplicationController ensures that a specified number of pod replicas are running at any
one time implemented through a controllers framework implemented in go, watchers can be implemented to react to kubernetes level events such deployment start stop delete cpu consumption and container failure , but cannot handle application failure

**K8s Liveness-probe** : A liveness probe is an object that has a command specified that will run on the container(pods)(deployment) or through another supported interface such as an http request every set time if the command returns a success code the kubelet considers the pod alive and healthy if it returns a failure , the kubelet kills the Container and restarts it.Another type of liveness probe uses a TCP Socket. With this configuration, the kubelet will attempt to open a socket to your container on the specified port. If it can establish a connection, the container is considered healthy, if it can’t it is considered a failure.

**K8s Readiness-probe** : A Readiness probe is the same as a liveness probe except on succcess and failure it marks the pod as ready for success and on failure the endpoints controller removes the Pod’s IP address from the endpoints of all Services that match the Pod.


**K8s Heapster** : Heapster enables Container Cluster Monitoring and Performance Analysis for K8s, this will allow container
level monitoring and above so all higher K8s abstraction concepts can be monitored almost out of the box.
However, this does not support custom monitoring or application monitoring. To allow custom monitoring another solution,
Cadvisor must be used.

**K8s Labels** : as defined in https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/ : "Labels are key/value pairs that are attached to objects, such as pods. Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system. " basically they are tags that can be referenced later on when trying to apply a change or action on a specific object.

**K8s Selectors** : This is a way to filter on objects that have the same label ,the label selector is the core grouping primitive in Kubernetes.

**K8s Namespaces** : Kubernetes namespaces help different projects, teams, or customers to share a Kubernetes cluster. This is  way to subdivide a cluster with different permissions and policies and available objects

**K8s API and clients** : k8s has a restful api which can be accessed either through the command line using a tool called Kubectl or through multiple implementations of wrapper clients in go, python , java ...

**K8s logs** : output from the command on the container are logged and can be accessed using kubectl logs or through the api endpoint it can also filter on specific deployment, pod or  container outputs.