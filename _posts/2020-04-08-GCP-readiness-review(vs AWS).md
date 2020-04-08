## Scope
- This readiness review is scoped at the moving our companies infra from AWS to GCP.
- Since we only use EC2, ASG, LB, and ECR, We are not tied to AWS. So explore ways to improve the performance and reliability while keeping cost in check. 

## Why GCP?
### Reliability and performance
- GCP offers a low-latency 10Gbps interconnect across the board.
- GCP offers a global Anycast network as part of its load balancing service.
- GCP also has a track record of exceeding its uptime SLAs for compute VMs.
### Google Kubernetes Engine
- We hate having something in prod which we don’t have complete expertise on. Even though KOPS is stable, it’s still a dark horse.
- GKE spin uptime is very less (<5 min)
- Easy to manage RBAC in GKE.
- One zonal k8 cluster is a free per user account 
### Pricing
- Google offers sustained use discounts and per second billing
     What we mean is that AWS bills its services in a prepaid format, which means that you have to pay for a specific portion of resources, irrespective of how much you use. if you are running their system for 1 hour and 5 minutes, it would still cost you the price of 2 full hours to get it done. It is the same for other resources as well. For example, if the task at hand requires only 1.2GB of RAM, you would still have to go for 2GB.
     GCP, on the other hand, offers you the option of per-second billing. This means that the moment you stop using the service, they stop charging you. It is the same with resources as well, as you can configurable the system for any specific figure of RAM, whether it be 1.5 GB, 3.25GB RAM or any other arbitrary figure.
### Supporting features while designing DR for our infra
- VPC and EIP are not region-specific since GCP offers _Anycast network_
- Anthos: Services like Anthos can be leveraged to build a multi-cloud strategy for our product.
- NodeGrouping feature while doing major upgrades/region migration
### Observability
- StackDriver is one of the best, Plugging StackDriver to GKE is pretty straight forward. And it can monitor and alert for node, service, namespace related checks
- We can push logs to stackdriver itself.
- Integrating Prometheus with StackDriver is also supported.
### Having multiple environments for devs to test their feature on.
- GKE provides abstraction, Ops Engineer would be handling namespace rather than dealing with nodes. So If devs want to test their feature on staging then it’s easy to spin up new namespace for their feature branch
### Security and Maintenance effort:
- Since GKE is managed service, no need to worry about upgrading the Kubernetes control plane.
- Google patches security vulnerabilities of the K8 control plane as well as node very often.

## Why not GCP?
- GCP has a very limited number of services but offerings are relatively tamer, focused more on IaaS & PaaS services.
- We don’t have full control over GKE, If we want to integrate some security tool then it will be difficult/impossible
- GCP customer technical support is not as good as AWS. 