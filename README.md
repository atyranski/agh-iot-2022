# Microservices/Edge Computing – K3s

### Authors
* Mateusz Łopaciński
* Piotr Socała
* Marcin Szwed
* Arkadiusz Tyrański

### Project description
The aim of the project is to develop an example application built in architecture
microservice and its launch using the K3s platform. The project should include
analysis of application management capabilities as well as comparison to similar solutions
used in computing clouds, e.g. Kubernetes.

### Agenda
1. K3s (Lightweight Kubernetes) vs K8s (stock Kubernetes)
2. Drivers, requirements, remote access and tools
3. Demo application

## 1. K3s (Lightweight Kubernetes) vs K8s (stock Kubernetes)

```
Easy to install, half the memory, all in a binary of less than 100 MB 
                                                               ~ K3s docs"
```

### What is Kubernetes?


#### Historical background
Businesses around the world are driven by the demand for scalable and reliable services. Kubernetes evolved from a system called Borg that Google used internally for years before they shared it with the public. 

Kubernetes was designed to accommodate large configurations and make them scalable and resilient. But stock Kubernetes is heavy, complex, difficult to manage, and comes with a steep learning curve.
Almost immediately after its release, people began changing their applications to fit the constraints of Kubernetes instead of using Kubernetes as a tool with a specific purpose.

Unused resources result in longer installation times and additional complexity. Best practices for building containers state that ***the container should only contain the resources that it needs to perform its function***, so why should the container orchestrator be any different?

By getting rid of unnecessary components, K3s manages to solve many of the issues developers face in stock Kubernetes.


#### Specification
K3s is a lightweight Kubernetes distribution created by Rancher Labs. K3s is highly available and production-ready. 

```
It is important to note that K3s is not a fork, as it doesn’t change any of the core Kubernetes functionalities 
                                  and remains close to stock Kubernetes.
```

K3s is a single binary solution that unpacked to include `containerd`, the networking stack, the control plane (server) and worker (agent) components, `SQLite` instead of etcd, a service load balancer, a storage class for local storage, and manifests that pulled down and installed other core components like `CoreDNS` and `Traefik Proxy` as the ingress controller.

There are two major ways that K3s is lighter weight than stock Kubernetes:

1. The memory footprint to run it is smaller
    - The memory footprint is reduced primarily by running many components inside of a single process. This eliminates significant overhead that would otherwise be duplicated for each component. 
    - Unlike traditional Kubernetes, which runs its components in different processes, K3s runs the control plane, kubelet, and kube-proxy in a single Server or Agent process, with containerd handling the container lifecycle functions.
    - The tiny footprint of K3s makes it possible to orchestrate and run container workloads in places that Kubernetes could never reach before. Many of these locations use single-node clusters on hardened equipment designed to operate in harsh environments with limited or intermittent connectivity.
    - K3s has been used successfully in satellites, airplanes, submarines, vehicles, wind farms, retail locations, smart cities, and other places where running Kubernetes wouldn’t normally be possible.

2. The binary, which contains all the non-containerized components needed to run a cluster, is smaller
    - The binary is smaller by removing third-party storage drivers and cloud providers. K3s removes several optional volume plugins and all built-in (sometimes referred to as "in-tree") cloud providers.
    - We do this in order to achieve a smaller binary size and to avoid dependence on third-party cloud or data center technologies and services, which may not be available in many K3s use cases. We are able to do this because their removal affects neither core Kubernetes functionality nor conformance.
    - If you want to run Kubernetes on Arm hardware like a Raspberry Pi, then K3s will give you the complete functionality of Kubernetes while leaving more of the CPU and RAM available for your workloads.
    
![image](https://user-images.githubusercontent.com/72259657/210011245-480cdd55-0056-421f-bddf-85fa3ea06231.png)

The following volume plugins have been removed from K3s:

* cephfs - The Ceph File System (CephFS) is a file system compatible with POSIX standards that uses a Ceph Storage Cluster to store its data
* fc
* flocker - allows you to launch and move stateful containers by integrating with a Cluster Manager of your choice
* git_repo
* glusterfs
* portworx
* quobyte
* rbd
* storageos - Persistent storage for containers
 
 Advantages of K3s:
* Small in size — K3s is less than 100MB and this is, possibly, its biggest advantage.
* Lightweight — The binary containing the non-containerized components is smaller than K8s.
* Fast deployment — You can use a single command to install and deploy K3s, and it will take you less than 30 seconds to do so.
* Batteries included — CRI, CNI, service load balancer, and ingress controller are included.
* Easy to update — Thanks to its reduced dependencies.
* Flexibility - The same binary can become a control server node or join an existing cluster as a worker.

```
"It’s neither less than, nor a fork of Kubernetes. It’s 100% upstream Kubernetes, optimized for a specific 
                                use case. (certified by the CNCF)"
```

## Drivers, requirements, remote access and tools

### K3s primary resource utilization drivers

K3s server utilization figures are primarily driven by support of the Kubernetes datastore (kine or etcd), API Server, Controller-Manager, and Scheduler control loops, as well as any management tasks necessary to effect changes to the state of the system. Operations that place additional load on the Kubernetes control plane, such as creating/modifying/deleting resources, will cause temporary spikes in utilization. Using operators or apps that make extensive use of the Kubernetes datastore (such as Rancher or other Operator-type applications) will increase the server's resource requirements. Scaling up the cluster by adding additional nodes or creating many cluster resources will increase the server's resource requirements.

K3s agent utilization figures are primarily driven by support of container lifecycle management control loops. Operations that involve managing images, provisioning storage, or creating/destroying containers will cause temporary spikes in utilization. Image pulls in particular are typically highly CPU and IO bound, as they involve decompressing image content to disk. If possible, workload storage (pod ephemeral storage and volumes) should be isolated from the agent components (/var/lib/rancher/k3s/agent) to ensure that there are no resource conflicts.

### K3s requirements
K3s is very lightweight, but has some minimum requirements as outlined below.

Whether someone's configuring K3s to run in a container or as a native Linux service, each node running K3s should meet the following minimum requirements. These requirements are baseline for K3s and its packaged components, and do not include resources consumed by the workload itself.

#### Prerequisites
Two nodes cannot have the same hostname.

If multiple nodes will have the same hostname, or if hostnames may be reused by an automated provisioning system, use the `--with-node-id` option to append a random suffix for each node, or devise a unique name to pass with `--node-name` or `$K3S_NODE_NAME` for each node you add to the cluster.

#### Operating Systems
K3s is expected to work on most modern Linux systems.

Some OSs have specific requirements and need additional setup:
* (Red Hat/CentOS) Enterprise Linux
* Raspberry Pi OS

In order to get it working on above OSs, one should follow detailed configuration in official website: https://docs.k3s.io/advanced

#### Hardware
Hardware requirements scale based on the size of one's deployments. Minimum recommendations are outlined here.

* RAM: 512MB Minimum (we recommend at least 1GB)
* CPU: 1 Minimum

K3s performance depends on the performance of the database. To ensure optimal speed, using an SSD when possible is reccommended. Disk performance will vary on ARM devices utilizing an SD card or eMMC.

#### Networking

The K3s server needs port 6443 to be accessible by all nodes.

The nodes need to be able to reach other nodes over UDP port 8472 when Flannel VXLAN is used or over UDP ports 51820 and 51821 (when using IPv6) when Flannel Wireguard backend is used. The node should not listen on any other port. K3s uses reverse tunneling such that the nodes make outbound connections to the server and all kubelet traffic runs through that tunnel. However, if one does not use Flannel and provides it's own custom CNI, then the ports needed by Flannel are not needed by K3s.

#### Large clusters

Hardware requirements are based on the size of your K3s cluster. For production and large clusters, using a high-availability setup with an external database is reccommended. The following options are recommended for the external database in production:

*MySQL
*PostgreSQL
*etcd

### K3s Binary Tools

The K3s binary contains a number of additional tools the help you manage your cluster:

| Command  | Description |
| ------------- | ------------- |
| `k3s server` |	Run the K3s management server, which will also launch Kubernetes control plane components such as the API server, controller-manager, and scheduler.  |
| `k3s agent` |	Run the K3s node agent. This will cause K3s to run as a worker node, launching the Kubernetes node services `kubelet` and `kube-proxy`.  |
| `k3s kubectl` |	Run an embedded kubectl CLI. If the `KUBECONFIG` environment variable is not set, this will automatically attempt to use the config file that is created at `/etc/rancher/k3s/k3s.yaml` when launching a K3s server node.  |
| `k3s crictl` |	Run an embedded crictl. This is a CLI for interacting with Kubernetes's container runtime interface (CRI). Useful for debugging. |
| `k3s ctr` |	Run an embedded ctr. This is a CLI for containerd, the container daemon used by K3s. Useful for debugging. |
| `k3s etcd-snapshot` |	Perform on demand backups of the K3s cluster data and upload to S3. |
| `k3s secrets-encrypt` |	Configure K3s to encrypt secrets when storing them in the cluster. |
| `k3s certificate` |	Certificates management. |
| `k3s completion` |	Generate shell completion scripts for k3s. |
| `k3s help` |	Shows a list of commands or help for one command. |

Apart from tools above, in K3s there are also Kubernetes tools available, like Dashboard or Helm

### Remote access

The kubeconfig file stored at `/etc/rancher/k3s/k3s.yaml` is used to configure access to the Kubernetes cluster. If there are installed upstream Kubernetes command line tools such as kubectl or helm you configuring them with the correct kubeconfig path will be needed. This can be done by either exporting the `KUBECONFIG` environment variable or by invoking the `--kubeconfig` command line flag. Refer to the examples below for details.

Leverage the KUBECONFIG environment variable:
```
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
kubectl get pods --all-namespaces
helm ls --all-namespaces
```

Or specify the location of the kubeconfig file in the command:
```
kubectl --kubeconfig /etc/rancher/k3s/k3s.yaml get pods --all-namespaces
helm --kubeconfig /etc/rancher/k3s/k3s.yaml ls --all-namespaces
```

#### Accessing the Cluster from Outside with kubectl

Copy `/etc/rancher/k3s/k3s.yaml` on machine located outside the cluster as `~/.kube/config`. Then replace the value of the `server` field with the IP or name of your K3s server. `kubectl` can now manage your K3s cluster.

### Sources
* [K3s Github](https://github.com/k3s-io/k3s)
* [K3s Docs](https://docs.k3s.io)
* [k3s i Raspberry Pi 4, czyli taki mały własny Kubernetes - Łukasz Kałużny](https://kaluzny.io/k3s-i-raspberry-pi-4-czyli-taki-maly-wlasny-kubernetes/)
* [K3s Explained:
What is it and How Is It Different From Stock Kubernetes (K8s)?](https://traefik.io/glossary/k3s-explained/)
