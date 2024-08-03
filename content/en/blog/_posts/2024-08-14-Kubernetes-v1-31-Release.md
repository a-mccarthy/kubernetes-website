---
layout: blog
title: 'Kubernetes v1.31: TITLE'
date: 2024-08-14
slug: kubernetes-v1-31-release
author: >
  [Kubernetes Release Team](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.31/release-team.md)
---
**Authors:** [Kubernetes 1.31 Release Team](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.31/release_team.md)



## Release theme and logo
<Logo image size is recommended to be no more than 2160px/>

## Highlights of features graduating to Stable

_This is a selection of some of the improvements that are now stable following the v1.31 release._

### Improved ingress connectivity reliability for Kube-proxy

Kube-proxy improved ingress connectivity reliability is GA in v1.31. One of the common problems with Load Balancers in Kubernetes is the synchronization between the different components involved to avoid traffic drop, this feature implements a mechanism in kube-proxy for load balancers to do connection draining for terminating Nodes and establish some best practices for cloud providers and Kubernetes load balancers implementations.


exerpt from SIG:
This feature enables connection draining for workloads running on terminating/deleting Kubernetes nodes which are exposed by services of type: LoadBalancer and externalTrafficPolicy: Cluster . Note: the predicates for using this feature are: a) kube-proxy needs to run as default service proxy on the cluster and b) the load balancer needs to support connection draining. There are no specific changes required for using this feature, it has been enabled by default in Kube-proxy since 1.30 and been promoted to stable in 1.31.

This work was done as part of [KEP #3836](https://github.com/kubernetes/enhancements/issues/3836) by [SIG Network](https://github.com/kubernetes/community/tree/master/sig-network). /docs/reference/networking/virtual-ips/#external-traffic-policy

### AppAprmor support is now stable - Abbie

Kubernetes support for AppArmor is now GA. Protect your Containers using AppArmor by setting the `appArmorProfile.type` field in the Container's `securityContext`. Note that before Kubernetes v1.30, AppArmor was controlled via annotations; starting in v1.30 it is contorlled using fields. Its recommended that you should migrate away from using annotations and start using the `appArmorProfile.type` field.

To learn more read the [AppArmor tutorial](/docs/tutorials/security/apparmor/). This work was done as a part of [KEP #24](https://github.com/kubernetes/enhancements/issues/24) lead by  [SIG Node](https://github.com/kubernetes/community/tree/master/sig-node).
    

### Persistent Volume last phase transition time

Persistent Volume last phase transition time feature moved to GA in v1.31. This feature adds a `PersistentVolumeStatus` field which holds a timestamp of when a PersistentVolume last transitioned to a different phase.
With this feature enabled, every PersistentVolume object will have a new field `.status.lastTransitionTime`, that holds a timestamp of
when the volume last transitioned its phase. This change is not immediate; the new field will be populated whenever a PersistentVolume is updated and first transitions between phases (`Pending`, `Bound`, or `Released`) after upgrading to Kubernetes v1.31.
This allows you to measure time between when a PersistentVolume moves from `Pending` to `Bound`. This can be also useful for providing metrics and SLOs.

For more details about this feature please visit the [PersistentVolume documentation page](https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-v1/#PersistentVolumeStatus).

This work was done as a part of [KEP #3762](https://github.com/kubernetes/enhancements/issues/3762) lead by  [SIG Storage](https://github.com/kubernetes/community/tree/master/sig-storage).

## Highlights of features graduating to Beta

_This is a selection of some of the improvements that are now beta following the v1.31 release._
    
### Kube-proxy nftables backend

Kube-proxy nftables backend moves to beta in v1.31 and introduces a new feature gate, `NFTablesProxyMode`, disabled by default.
In this mode, kube-proxy configures packet forwarding rules using the nftables API of the kernel netfilter subsystem. 
For each endpoint, it installs nftables rules which, by default, select a backend Pod at random.

The nftables API is the successor to the iptables API and is designed to provide better performance and scalability than iptables. 
The `nftables` proxy mode is able to process changes to service endpoints faster and more efficiently than the `iptables` mode, and is also able to more efficiently process packets in the kernel (though this only
becomes noticeable in clusters with tens of thousands of services).

As of Kubernetes v1.31, the `nftables` mode is still relatively new, and may not be compatible with all network plugins; consult the documentation for your network plugin. 
This proxy mode is only available on Linux nodes, and requires kernel 5.13 or later.
See the full [migration guide](/docs/reference/networking/virtual-ips/#proxy-mode-nftables) for more details. 

This work was done as part of [KEP #3866](https://github.com/kubernetes/enhancements/issues/3866) by [SIG Network](https://github.com/kubernetes/community/tree/master/sig-network).

### Always Honor PersistentVolume Reclaim Policy

The Always Honor PersistentVolume Reclaim Policy feature has advanced to beta in Kubernetes v1.31. This enhancement ensures that the PersistentVolume (PV) reclaim policy is respected even after the associated PersistentVolumeClaim (PVC) is deleted, thereby preventing the leakage of volumes.

Prior to this feature, the reclaim policy linked to a PV could be disregarded under specific conditions, depending on whether the PV or PVC was deleted first. Consequently, the corresponding storage resource in the external infrastructure might not be removed, even if the reclaim policy was set to "Delete". This led to potential inconsistencies and resource leaks.

With the introduction of this feature, Kubernetes now guarantees that the "Delete" reclaim policy will be enforced, ensuring the deletion of the underlying storage object from the backend infrastructure, regardless of the deletion sequence of the PV and PVC.

This work was done as a part of [KEP #2644](https://github.com/kubernetes/enhancements/issues/2644) and lead by [SIG Storage](https://github.com/kubernetes/community/tree/master/sig-storage).
    
### Bound service account token improvements

This work was done as part of [KEP #4193](https://github.com/kubernetes/enhancements/issues/4193) by [SIG Auth](https://github.com/kubernetes/community/tree/master/sig-auth).

reached out: https://kubernetes.slack.com/archives/C0EN96KUY/p1722015619684019

/docs/reference/access-authn-authz/service-accounts-admin/#bound-service-account-tokens


### Multiple Service CIDRS

Multiple Service CIDRs moves to beta in v1.31 (disabled by default).

There are multiple components in a Kubernetes cluster that consume IP addresses: Nodes, Pods and Services. Nodes and Pods IP ranges can be dynamically changed because depend on the infrastructure or the network plugin respectively. However, Services are defined during the cluster creation as a hardcoded flag in the kube-apiserver. 
IP exhaustion has been a problem for long lived or large clusters, as admins needed to expand, shrink or even replace entirely the assigned Service CIDR range. 
These operations were never supported natively and were performed via complex and delicate maintenance operations, often causing downtime on their clusters. This KEP allows users and cluster admins to dynamically modify Service CIDR ranges with zero downtime.

For more details about this feature please visit the [Virtual IPs and Service Proxies documentation page](https://kubernetes.io/docs/reference/networking/virtual-ips/#ip-address-objects)

This work was done as part of [KEP #1880](https://github.com/kubernetes/enhancements/issues/1880) by [SIG Network](https://github.com/kubernetes/community/tree/master/sig-network).


### Traffic Distribution for Services

Traffic Distribution for Services moves to beta in v1.31, enabled by default. 

After several iterations on finding the best user experience and traffic engineering capabilities for Services networking, the SIG Networking implemented a `trafficDistribution` field in the Service specification, that serves as a guideline for the underlying implementation to consider while making routing decisions.
It supersedes the functionality formerly provided by the service.kubernetes.io/topology-mode annotation and it’s precursor `topologyKeys` field (which has been deprecated since Kubernetes 1.21)

For more details about this feature please read the [1.30 Release Blog](https://kubernetes.io/blog/2024/04/17/kubernetes-v1-30-release/#traffic-distribution-for-services-sig-network-https-github-com-kubernetes-community-tree-master-sig-network) or visit the [Service documentation page](https://kubernetes.io/docs/concepts/services-networking/service/#traffic-distribution).

This work was done as part of [KEP #4444](https://github.com/kubernetes/enhancements/issues/4444) by [SIG Network](https://github.com/kubernetes/community/tree/master/sig-network).


### Kubernetes VolumeAttributesClass ModifyVolume

[VolumeAttributesClass](/docs/concepts/storage/volume-attributes-classes/) API is moving to beta in v1.31.
The VolumeAttributesClass provides a generic,
Kubernetes-native API for modifying dynamically volume parameters like provisioned IO. This allows workloads to vertically scale their volumes on-line to balance cost and performance, if supported by their provider. This feature has been alpha since 1.29 and will be beta in 1.31. 

This work was done as a part of [KEP #3751](https://github.com/kubernetes/enhancements/issues/3751) and lead by [SIG Storage](https://github.com/kubernetes/community/tree/master/sig-storage).

## New features in Alpha

_This is a selection of some of the improvements that are now alpha following the v1.31 release._
    
### New DRA APIs for better accelerators and other hardware management: DRA: control plane controller ("classic DRA")

Kubernetes v1.31 brings an updated dynamic resource allocation (DRA) API and design. The main focus in the update is on structured parameters because they make resource information and requests transparent to Kubernetes and clients and enable implementing features like cluster autoscaling. DRA support in the kubelet was updated such that version skew between kubelet and the control plane is possible. With structured parameters, the scheduler allocates `ResourceClaims` while scheduling a pod. Allocation by a DRA driver controller is still supported through what is now called "classic DRA". 

With Kubernetes v1.31, classic DRA has a separate feature gate and needs to be enabled with `DRAControlPlaneController`. With such a control plane controller, a DRA driver can implement allocation policies that are not supported yet through structured parameters.

This work was done as part of [KEP #3063](https://github.com/kubernetes/enhancements/issues/3063) by [SIG Node](https://github.com/kubernetes/community/tree/master/sig-node).
    
### OCI artifacts as VolumeSource to deliver model weights more efficiently and user friendly: VolumeSource: OCI Artifact and/or Image

The Kubernetes community is moving towards fulfilling more Artificial Intelligence (AI) and Machine Learning (ML) use cases in the future. 

One of the requirements to support these use cases is to support Open Container Initiative (OCI) compatible images and artifacts (referred as OCI objects) directly as a native volume source. This allows users to focus on OCI standards as well as enables them to store and distribute any content using OCI registries.

Given that, the Image Volume Source feature is introduced as a new alpha feature in v1.31. This feature allows users to specify an image reference as volume in a pod while reusing it as volume mount within containers.

This work was done as part of [KEP #4639](https://github.com/kubernetes/enhancements/issues/4639) by [SIG Node](https://github.com/kubernetes/community/tree/master/sig-node) and [SIG Storage](https://github.com/kubernetes/community/tree/master/sig-storage).

### Better devices errors visibility - the device health is now in pod status: Add Resource Health Status to the Pod Status for Device Plugin and DRA

Expose device health information through the Pod Status is added as a new alpha feature in v1.31, disabled by default.

Before Kubernetes v1.31, the way to know whether or not a Pod is associated with the failed device is to use the [PodResources API](#monitoring-device-plugin-resources). 

By enabling this feature, the field `allocatedResourcesStatus` will be added to each container status, within the `.status` for each Pod. The `allocatedResourcesStatus` field reports health information for each device assigned to the container.
 

This work was done as part of [KEP #4680](https://github.com/kubernetes/enhancements/issues/4680) by [SIG Node](https://github.com/kubernetes/community/tree/master/sig-node).


### Authorize with field and label selectors going Alpha in 1.31

This feature allows webhook authorizers and future (but not currently designed) in-tree authorizers to allow list and watch requests based on field and label selectors.  It is now possible for an authorizer to express: this user cannot list all pods, but can list all pods where `.spec.nodeName=foo`.  Combined with CRD field selectors (moving to beta in 1.31), it is possible to write more secure per-node extensions.
    
This work was done as part of [KEP #4601](https://github.com/kubernetes/enhancements/issues/4601) by [SIG Auth](https://github.com/kubernetes/community/tree/master/sig-auth).
    

### Only allow anonymous auth for configured endpoints going Alpha in 1.31

By enabling the feature gate `AnonymousAuthConfigurableEndpoints` users can now use the authentication configuration file to configure the endpoints that can be accessed by anonymous requests. This allows users to protect themselves against RBAC misconfigurations that can give anonymous users broad access to the cluster.
    

This work was done as a part of [KEP #4633](https://github.com/kubernetes/enhancements/issues/4633) and lead by [SIG Auth](https://github.com/kubernetes/community/tree/master/sig-auth).
    
## Graduations, deprecations, and removals in 1.31
    

### Graduations to Stable

This lists all the features that graduated to stable (also known as _general availability_). For a full list of updates including new features and graduations from alpha to beta, see the release notes.

This release includes a total of 11 enhancements promoted to Stable:

* [PersistentVolume last phase transition time](	https://github.com/kubernetes/enhancements/issues/3762)
* [Metric cardinality enforcement](	https://github.com/kubernetes/enhancements/issues/2305)
* [Kube-proxy improved ingress connectivity reliability](	https://github.com/kubernetes/enhancements/issues/3836)
* [Add CDI devices to device plugin API](	https://github.com/kubernetes/enhancements/issues/4009)
* [Move cgroup v1 support into maintenance mode](	https://github.com/kubernetes/enhancements/issues/4569)
* [AppArmor support](	https://github.com/kubernetes/enhancements/issues/24)
* [PodHealthyPolicy for PodDisruptionBudget](	https://github.com/kubernetes/enhancements/issues/3017)
* [Retriable and non-retriable Pod failures for Jobs](	https://github.com/kubernetes/enhancements/issues/3329)
* [Elastic Indexed Jobs](	https://github.com/kubernetes/enhancements/issues/3715)
* [Allow StatefulSet to control start replica ordinal numbering](	https://github.com/kubernetes/enhancements/issues/3335)
* [Random Pod selection on ReplicaSet downscaling](	https://github.com/kubernetes/enhancements/issues/2185)
    
    
### Deprecations and Removals 

Also see our [Kubernetes Removals and Major Changes In v1.31](https://kubernetes.io/blog/2024/07/19/kubernetes-1-31-upcoming-changes/) blog.
    
#### Cgroup v1 enters the maintenance mode

As Kubernetes continues to evolve and adapt to the changing landscape of container orchestration, the community has decided to move cgroup v1 support into maintenance mode in v1.31.
This shift aligns with the broader industry's move towards [cgroup v2](https://kubernetes.io/docs/concepts/architecture/cgroups/), offering improved functionality, scalability, and a more consistent interface. 
Maintance mode means that no new features will be added to cgroup v1 support. 
Critical security fixes will still be provided, however, bug-fixing is now best-effort, meaning major bugs may be fixed if feasible, but some issues might remain unresolved.

Its recommended that you start switching to use cgroup v2 as soon as possible. For more details on the reasoning behind placing cgroup v1 in maintaince mode and starting the migration process see the [Moving cgroup v1 Support into Maintenance Mode in Kubernetes 1.31](/blog/kubernetes-1-31-moving-cgroup-v1-support-maintenance-mode) article. 

Please report any problems encounter by filing an [issue](https://github.com/kubernetes/kubernetes).

This work was done as part of [KEP #4569](https://github.com/kubernetes/enhancements/issues/4569) by [SIG Node](https://github.com/kubernetes/community/tree/master/sig-node).

## Release Notes and Upgrade Actions Required

Check out the full details of the Kubernetes 1.31 release in our [release notes](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.31.md).

### KubeSchedule now uses QueueingHint when `SchedulerQueueingHints` is enabled
Added support to the scheduler to start using QueueingHint registered for Pod/Updated event to determine whether unschedulable Pods update make them schedulable, when the feature gate `SchedulerQueueingHints` is enabled. Previously, when unschedulable Pods are updated, the scheduler always put Pods back to activeQ/backoffQ. But, actually not all updates to Pods make Pods schedulable, especially considering many scheduling constraints nowadays are immutable. Now, when unschedulable Pods are updated, the scheduling queue checks with QueueingHint(s) whether the update may make the pods schedulable, and requeues them to activeQ/backoffQ only when at least one QueueingHint(s) return Queue.

**Action required for custom scheduler plugin developers**: 
Plugins have to implement a QueueingHint for Pod/Update event if the rejection from them could be resolved by updating unscheduled Pods themselves. Example: suppose you develop a custom plugin that denies Pods that have a schedulable=false label. Given Pods with a schedulable=false label will be schedulable if the schedulable=false label is removed, this plugin would implement QueueingHint for Pod/Update event that returns Queue when such label changes are made in unscheduled Pods. [#122234](https://github.com/kubernetes/kubernetes/pull/122234)

### Removal of kubelet --keep-terminated-pod-volumes command line flag
The kubelet flag --keep-terminated-pod-volumes, which was deprecated in 2017, will be removed as part of the v1.31 release.

You can find more details in the pull request [#122082](https://github.com/kubernetes/kubernetes/pull/122082).


## Availability

Kubernetes 1.31 is available for download on [GitHub](https://github.com/kubernetes/kubernetes/releases/tag/v1.31.0). To get started with Kubernetes, check out these [interactive tutorials](https://kubernetes.io/docs/tutorials/) or run local Kubernetes clusters using [minikube](https://minikube.sigs.k8s.io/). You can also easily install 1.31 using [kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/). 

## Release Team

Kubernetes is only possible with the support, commitment, and hard work of its community. Each release team is made up of dedicated community volunteers who work together to build the many pieces that make up the Kubernetes releases you rely on. This requires the specialized skills of people from all corners of our community, from the code itself to its documentation and project management.

We would like to thank the entire [release team](https://github.com/kubernetes/sig-release/blob/master/releases/release-1.31/release-team.md) for the hours spent hard at work to deliver the Kubernetes v1.31 release to our community. The Release Team's membership ranges from first-time shadows to returning team leads with experience forged over several release cycles. A very special thanks goes out our release lead, Angelos Kolaitis, for supporting us through a successful release cycle, advocating for us, making sure that we could all contribute in the best way possible, and challenging us to improve the release process.



## Project Velocity

The CNCF K8s DevStats project aggregates a number of interesting data points related to the velocity of Kubernetes and various sub-projects. This includes everything from individual contributions to the number of companies that are contributing and is an illustration of the depth and breadth of effort that goes into evolving this ecosystem.

In the v1.31 release cycle, which ran for 14 weeks (May 7th to August 13th), we saw contributions to Kubernetes from 113 different companies and 528 individuals.

In the whole Cloud Native ecosystem we have 379 companies counting 2268 total contributors - which means that respect to the previous release cycle we experienced an astounding 63% increase on individuals contributing! 

Source for this data: 
- [Companies contributing to Kubernetes](https://k8s.devstats.cncf.io/d/11/companies-contributing-in-repository-groups?orgId=1&from=1715032800000&to=1723586399000&var-period=d28&var-repogroup_name=Kubernetes&var-repo_name=kubernetes%2Fkubernetes)
- [Overall ecosystem contributions](https://k8s.devstats.cncf.io/d/11/companies-contributing-in-repository-groups?orgId=1&from=1715032800000&to=1723586399000&var-period=d28&var-repogroup_name=All&var-repo_name=kubernetes%2Fkubernetes)

By contribution we mean when someone makes a commit, code review, comment, creates an issue or PR, reviews a PR (including blogs and documentation) or comments on issues and PRs.

If you are interested in contributing visit [this page](https://www.kubernetes.dev/docs/guide/#getting-started) to get started. 

[Check out DevStats](https://k8s.devstats.cncf.io/d/11/companies-contributing-in-repository-groups?orgId=1&var-period=m&var-repogroup_name=All) to learn more about the overall velocity of the Kubernetes project and community.

## Event Update

Explore the upcoming Kubernetes and cloud-native events from August to November 2024, featuring KubeCon, KCD, and other notable conferences worldwide. Stay informed and engage with the Kubernetes community.

**August 2024**
- [**KCD Taipei 2024**](https://community.cncf.io/events/details/cncf-kcd-taiwan-presents-kcd-taipei-2024/): August 3-4, 2024 | Taipei City, Taiwan
- [**KubeCon + CloudNativeCon + Open Source Summit China 2024**](https://events.linuxfoundation.org/kubecon-cloudnativecon-open-source-summit-ai-dev-china/): August 21-23, 2024 | Hong Kong
- [**KubeDay Japan**](https://events.linuxfoundation.org/kubeday-japan/): August 27, 2024 | Tokyo, Japan


**September 2024**
- [**KCD Lahore - Pakistan 2024**](https://community.cncf.io/events/details/cncf-kcd-lahore-presents-kcd-lahore-pakistan-2024/): September 1, 2024 | Lahore, Pakistan
- [**KuberTENes Birthday Bash Stockholm**](https://community.cncf.io/events/details/cncf-stockholm-presents-kubertenes-birthday-bash-stockholm-a-couple-of-months-late/): September 5, 2024 | Stockholm, Sweden
- [**KCD Sydney ’24**](https://community.cncf.io/events/details/cncf-kcd-australia-presents-kcd-sydney-24/): September 5-6, 2024 | Sydney, Australia
- [**KCD Washington DC 2024**](https://community.cncf.io/events/details/cncf-kcd-washington-dc-presents-kcd-washington-dc-2024/): September 24, 2024 | Washington, DC, United States
- [**KCD Porto 2024**](https://community.cncf.io/events/details/cncf-kcd-porto-presents-kcd-porto-2024/): September 27-28, 2024 | Porto, Portugal

**October 2024**
- [**KubeDay Australia**](https://events.linuxfoundation.org/kubeday-australia/): October 1, 2024 | Melbourne, Australia
- [**KCD Austria 2024: October 8-10, 2024 | Wien, Austria**](https://community.cncf.io/events/details/cncf-kcd-austria-presents-kcd-austria-2024/) 
- [**KCD UK - London 2024**](https://community.cncf.io/events/details/cncf-kcd-uk-presents-kubernetes-community-days-uk-london-2024/): October 22-23, 2024 | Greater London, United Kingdom

**November 2024**
- [**KubeCon + CloudNativeCon North America 2024**](https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/): November 12-15, 2024 | Salt Lake City, United States
- [**Kubernetes on EDGE Day North America**](https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/co-located-events/kubernetes-on-edge-day/): November 12, 2024 | Salt Lake City, United States

## Upcoming Release Webinar - Abbie

<RELEASE WEBINARE WILL TAKE PLACE NORMALLY 30 DAYS AFTER RELEASE, ALIGN WITH CNCF TO HIGHLIGHT THE WEBINAR>

## Get Involved

The simplest way to get involved with Kubernetes is by joining one of the many [Special Interest Groups](https://github.com/kubernetes/community/blob/master/sig-list.md) (SIGs) that align with your interests. Have something you’d like to broadcast to the Kubernetes community? Share your voice at our weekly [community meeting](https://github.com/kubernetes/community/tree/master/communication), and through the channels below. Thank you for your continued feedback and support.

- Follow us on X [@Kubernetesio](https://x.com/kubernetesio) for latest updates
- Join the community discussion on [Discuss](https://discuss.kubernetes.io/)
- Join the community on [Slack](http://slack.k8s.io/)
- Post questions (or answer questions) on [Stack Overflow](http://stackoverflow.com/questions/tagged/kubernetes)
- Share your Kubernetes [story](https://docs.google.com/a/linuxfoundation.org/forms/d/e/1FAIpQLScuI7Ye3VQHQTwBASrgkjQDSS5TP0g3AXfFhwSM9YpHgxRKFA/viewform)
- Read more about what’s happening with Kubernetes on the [blog](https://kubernetes.io/blog/)
- Learn more about the [Kubernetes Release Team](https://github.com/kubernetes/sig-release/tree/master/release-team)
