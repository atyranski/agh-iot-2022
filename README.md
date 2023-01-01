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
2. Drivers, modules, requirements, remote access and tools
3. Demo application

## 1. K3s (Lightweight Kubernetes) vs K8s (stock Kubernetes)

```
Easy to install, half the memory, all in a binary of less than 100 MB 
                                                               ~ K3s docs"
```

### What is Kubernetes?
Businesses around the world are driven by the demand for scalable and reliable services. Kubernetes evolved from a system called Borg that Google used internally for years before they shared it with the public. 

Kubernetes was designed to accommodate large configurations and make them scalable and resilient. But stock Kubernetes is heavy, complex, difficult to manage, and comes with a steep learning curve.
Almost immediately after its release, people began changing their applications to fit the constraints of Kubernetes instead of using Kubernetes as a tool with a specific purpose.

Unused resources result in longer installation times and additional complexity. Best practices for building containers state that ***the container should only contain the resources that it needs to perform its function***, so why should the container orchestrator be any different?

By getting rid of unnecessary components, K3s manages to solve many of the issues developers face in stock Kubernetes.

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

### Sources
* [K3s Github](https://github.com/k3s-io/k3s)
* [K3s Docs](https://docs.k3s.io)
* [k3s i Raspberry Pi 4, czyli taki mały własny Kubernetes - Łukasz Kałużny](https://kaluzny.io/k3s-i-raspberry-pi-4-czyli-taki-maly-wlasny-kubernetes/)
* [K3s Explained:
What is it and How Is It Different From Stock Kubernetes (K8s)?](https://traefik.io/glossary/k3s-explained/)
