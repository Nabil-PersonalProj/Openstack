# CyberCloud

Project to deploy open stack for on-premise cloud services

## Installation
Openstack have many different ways of deployment. The deployment method that we have choosen is using kolla-ansible. Refer to [here](./CyberCloud%20Manual.pdf) for our installation method. <br />
https://docs.openstack.org/kolla-ansible/2023.1/user/quickstart.html (official openstack documentation)

## Current Set-up

### Hardware:
| Machine | Specs |
|---|---|
| PER740 (Controller) | CPU: Intel(R) Xeon(R) Bronze 3106 CPU @ 1.70GHz<br /> RAM: 65 GB<br /> Storage: |
| PER730 (Compute/Storage) | CPU: Intel(R) Xeon(R) Bronze 3106 CPU @ 1.70GHz<br /> RAM: 65 GB<br /> Storage: |
| PER520 (Storage/Compute) | CPU: Intel(R) Xeon(R) CPU E5-2430 0 @ 2.20GHz<br /> RAM: 16GB<br /> Storage: |

### Current Services
| Service | Description |
|---|---|
| Horizon  |   Horizon provides a dashboard for the server for ease of management and visualisation |
| Keystone (Core-service)  |  Keystone is the identity service for authentication |
| Glance (Core-service)  | Glance is an image service for registering new images or retrieving vm images |
| Nova (Core-service)  | Nova is a service that provides a way to provision isntances |
| Placement (Core-service) | Placement is a scheduling service |
| Neutron (Core-service) | Neutron is the networking service for openstack |
| Cinder | Cinder is a block storage service |
| Heat | Heat service is a orchestration service |
| Kuryr | Kuryr service is a docker network plugin to connect docker and openstack together |
| Zun | Zun is a service to create and manage components |
| Central_Logging (Opensearch) | Centralise logging for all services |

### Network
Current Subnet: 10.221.36.0/24 <br />
![alt text](Net-diagram.drawio.png)

10.221.36.0 - 10.221.36.26 : reserved ips<br />
10.221.36.27 - 10.221.36.126: Available to use for all instances<br />
10.221.36.127 - 10.221.36.191: container ip address (future work)
remaining: should be available for other uses

## Future Works

### Services
| Service | Description | Advantages
|---|---|---|
| Magnum | Service for deploying and managing cluster containers | 1. Container Orchestration:<br /> Magnum simplifies the deployment and management of container orchestration engines, making it easier to run and manage containerized applications in a production environment. It allows you to leverage popular orchestrators like Kubernetes to manage containers efficiently. <br /><br /> 2. Resource Efficiency: <br /> By integrating container orchestration with OpenStack, you can efficiently utilize your cloud resources, optimizing the use of VMs and ensuring that containers are scheduled and scaled appropriately.
| Murano | Murano is an application catalog for application that can work with openstack cloud services | 1. Application Abstraction:<br /> Murano abstracts complex application deployment processes, allowing users to focus on the high-level application components and configurations, rather than the underlying infrastructure.  This simplifies the deployment of applications, even for users who may not be experts in OpenStack or infrastructure management.
| Swift | Service for object storage | 1. Scalability, redundancy & permanence: <br /> Swift uses a distributed architecture with no central point of control, providing greater scalability, redundancy, and permanence, and it ensures data replication and integrity across the cluster.
| Octavia | Offers loadbalancing service. While neutron have its own capabilities octavia solely provides loadbalancing on its own |1. Advanced Load Balancing:<br /> Octavia provides advanced load balancing features, such as SSL/TLS termination, content switching, and session persistence, which are essential for managing modern web applications and microservices. <br /><br />2. Centralized Management:<br /> With Octavia, you can manage load balancers and their configurations centrally, which is especially beneficial in multi-tenant or large-scale cloud environments.
| Aodh | Alarm service to trigger should any actions made go against any rules defined by admin | 1. Alerting and Alarm Management:<br /> Aodh provides a framework for defining and managing alarms based on predefined thresholds and criteria. When certain conditions are met, Aodh can trigger alarms, which can be used to notify administrators or automatically initiate actions. <br /><br />2. Event and Log Handling:<br /> Aodh can consume events and logs from different OpenStack services and resources, allowing you to set alarms based on specific events and patterns. This is crucial for identifying and responding to anomalies and issues in your cloud environment.

### Server Expansion
Controller Nodes should minimally have atleast 3 nodes. Number of controller node should follow the quorum rule. Compute Nodes and storage nodes does not have a limit nor any rule to follow.

| Type of Expansion | Advantages | Disadvantages 
|---|---|---|
| Horizontal Scaling | 1. Distributing data and operations among nodes increases fault tolerance. If one of the servers goes down, the rest of the system keeps services online and maintains availability. <br /><br /> 2. The system does not go offline during the scaling process <br /><br /> 3. Sysadmins get to distribute data backups across several nodes.| 1. Day-to-day server management gets more complex the more nodes you add to a system. <br /><br /> 2. You must add software for load balancing and ensure nodes synchronize effectively. This setup is challenging for less experienced teams.|
|Vertical Scaling | 1. Fast response times as there are no synchronized nodes that communicate with each other. <br /><br /> 2. Everything runs on a single node, which leads to better data consistency and integrity. | 1. Higher possibility for service downtime as you rely on one machine that acts as a single point of failure. <br /><br /> 2. Increased risk of permanent data loss due to hosting everything on a single server.|

Steps to expand:
Refer to [here](https://docs.openstack.org/kolla-ansible/2023.1/user/adding-and-removing-hosts.html) to know more about the horizonal scaling expansion of nodes.


### Network
As of right now, docker network, kuryr is set be a private subnet. Kuryr can also be put on the provider network.

## Useful Links 
https://wiki.infn.it/progetti/cloud-areapd/operations/production_cloud/remove_a_compute_node_howto
adding a compute or control host
https://docs.openstack.org/kolla-ansible/train/user/adding-and-removing-hosts.html