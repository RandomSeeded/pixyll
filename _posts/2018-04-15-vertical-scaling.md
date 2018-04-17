---
layout:     post
title:      "Pod-based services"
date:       2018-04-15 00:00:00
summary:    Cloning BEAM
categories: Erlang Pods Vertical Scaling
---

# Pod-based scaling: a solution to simultaneous service deploys

One of the most common pain points of a microservice oriented architecture is the difficulty of making changes across multiple services. With traditional horizontal scaling, downstream services must be fully deployed before deploying any upstream changes. As a result, any deployments which require changes to multiple services are inherently unsafe. This post outlines an alternate approach to scaling services which tackles this issue, and talks about some of the difficulties deploying it today.

## Traditional Horizontal Scaling

We'll define an architecture designed for 'traditional horizontal scaling' as one which:

- a deployed service is made up of a pool of instances of that service
- individual services are aware only of the existence of other service pools, not individual services themselves

[Image of service pools, with non-versioned traffic]

The important takeaway of this is that traffic between services are not versioned. 

What happens if we have a new requirement which requires changes to multiple services? Due to the fact that traffic between these services is not versioned, all instances of the downstream services must be updated prior to any instances of the upstream service. Otherwise, we run the potential for the downstream service receiving traffic it can't correctly handle.

[Image of downstream service receiving unexpected traffic]

We must therefore first update all instances of our downstream service. We can do this in a staged deploy...

[Image of downstream service partly through updating]

...but we may not be actually production testing the new code, because the upstream service is still running the old version.

[Image of downstream service with unhit branches]

When we start to update the upstream service, it's possible that even a small number of upstream instances crash **all** of the downstream instances. 

[Image of dead downstream services]

## Pod-based scaling

This is the approach that pod-based scaling attempts to solve. The functional architectural unit to-be-scaled is not an instance of a service, but instead a snapshot of all services at a given point in time. This represents a 'pod'. This does not mean only a single instance of each service within the pod: there should still be service pools within each pod. 

[Image of service pods]

Important characteristics of pods:

- all services (or at minimum, all services which could ever be updated in conjunction) are contained within a pod
- all instances of a service within a given pod are running the same version of the code
- all traffic between services remains within a given pod

With pod-based scaling, staged rollouts of services are now trivial: we make changes to all the services simultaneously, and stage the rollout of pods.

[Image of pod-based scaling rollout]

If we run into any issues with the deploy, these issues are contained to just a single pod, and don't risk catastrophic failure of all instances of a service.

## Approaches to Deploying

### Versioned Traffic

Scaling in this manner requires some thought on AWS, Google Cloud, etc. The most important characteristic we need to preserve is traffic versioning.

For queues such as SQS, this seems easy enough at first glance; we'll version the queues according to a release version representing a 'snapshot' of all services, just like pod-based scaling. However, this is insufficient, because we now have additional issues around messages potentially being unreceived. What happens if we remove all instances of a 'release' of a downstream service prematurely? What happens if we deploy a new release's upstream service before deploying its corresponding downstream service? In either case, we have orphaned messages which go unhandled due to our releases no longer matching our architecture. 

The best we can do here is deploy new instances of all services for all releases, and not expose any of the new services to traffic until all services have been successfully deployed. When removing services, we must first close them off to all external traffic, and then remove them once fully drained.

Things get tougher when using HTTP requests for inter-service communication: we now have to implement versioning both on all of our inter-service requests and additionally build out support in the load balancer. The load balancer is now required to know which services correspond to which release version, and send requests accordingly.

### Containers and Deployments

Our approach should have:

- functional unit of 'pod' containing all services
- all inter-service traffic contained within a single 'pod'
- ability to horizontally scale services within a pod
- ability to horizontally scale entire pods
- efficient utilization of resources

One possible architecture, from broadest to most granular

- A container supervisor, responsible for scaling the entire set of all pods in response to high resource utilization
- A set of containers, where each container represents a pod (or release snapshot)
- Each individual container has within it a set of application supervisors, where each supervisor is responsible for a given service pool
- Each service pool is made up of instances of the service, responsible for business logic (generally receiving messages and acting accordingly)

If there is an increase in total traffic to the system, our container supervisor would be responsible for deploying additional instances of the current pod. If there was a change in relative traffic distribution (such as one service utilizing more resources than another), this would be handled by the application supervisors, which would create additional instances of the higher-utilization instance [reword?]. Staged releases are easy; we can just do a blue-green deploy [link here] of the pods.

**Unfortunately, that architecture just isn't well supported by today's cloud providers.** If we had the outer 'pod' as a docker container, we have no good options for the inner container. We can't run all the services inside the same virtual machine without any additional isolation without running into cross-service contamination of the virtual machine state. What's left, [docker-in-docker](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/)? Horizontal scaling within a pod isn't that well supported either; we need a service-independent program capable of automatically scaling the instances of an application in response to traffic or resource utilization. This approach also struggles if services require extremely different numbers of instances: each pod must have enough instances of an application to ensure redundancy for traffic within that pod, yet can't scale to so many instances that overhead from task switching becomes problematic.

### Erlang

Erlang's ability to solve this problem is remarkable. Each of the problems which stymie a modern cloud implementation is solved. A pod can be represented by a running Erlang cluster or instance, which represents a single release. The issue of nested containers is gone, because we already have isolation of our services from the Erlang VM. Dynamic horizontal scaling of services in response to utilization is also [built-in](http://erlang.org/doc/design_principles/sup_princ.html#id80194), and scaling of the entire system is equally easy. We can scale horizontally with the Erlang release running in a container to be replicated across our resources, or we can scale vertically by spinning up new resources and having them connect to an existing pod (Erlang cluster).

## Where Do We Go From Here?

[General thought for this: there's work to be done. Identify areas that we just don't have great solutions for yet. Identify the blame that cloud providers have here. Talk about best practices if you arent running Erlang]

## About me

[Prep a newsletter]

