---
layout:     post
title:      Cilium Identity Handling
date:       2021-08-04
summary:    Cilium is an Identity-aware network policy CNI. Cilium adopts a very different approach at deriving the identities and validating those identities in the data path using eBPF.
categories: cilium, cni, k8s
---

## Key Goals for Cilium's Identity solution
Cilium uses set of labels to do policy enforcement. One would argue that most
other CNIs also use a similar set of labels for policy enforcement. However the
devil is in the details. While most other CNIs eventually map the set of labels
to the set of pods and then use the corresponding IP addresses of the pods for
policy enforcement (for e.g for building iptables based rules), Cilium differs
in its approach. Cilium derives an identity value (an integer value) from the
set of labels and then uses this identity value to subsequently enforce the
policies. This way Cilium decouples its policy enforcement logic from an IP
addressing scheme.

This has been explained with an [example in Cilium's
documentation](https://docs.cilium.io/en/v1.9/concepts/security/identity/).

This article tries to uncover the internals of this identity value handling.

## Environment
Cilium: v1.9

k8s: 1.19

Cluster: Two nodes, using Cilium as CNI in systemd mode. [Deathstar sample
application](https://docs.cilium.io/en/v1.9/gettingstarted/http/) is used.

Using [Cilium's vagrant based development
setup](https://docs.cilium.io/en/v1.9/contributing/development/dev_setup/) for
experimentation.

## Identity Generation
An identity is associated with every pod. If there are multiple instances of
the same pod, then the identity value across all those pods remain same.

Note that there is also a notion of ENDPOINT which is a node-local value for a
given pod.

### How is identity generated?
Identity is assigned on the endpoint startup. Cilium [selects the set](https://github.com/cilium/cilium/blob/b7228c8b9b5897300ba5db754fe85b57bee61576/pkg/labels/oplabels.go#L83) of labels of the endpoints and then [randomly allocates](https://github.com/cilium/cilium/blob/c0b68419e6d82db63292f77aa120de22eeaa729f/pkg/identity/cache/allocator.go#L302) an identity from an identity pool. The [selection filter](https://github.com/cilium/cilium/blob/2d34336206d51e1f6b782bf63f16e9d4368aec80/pkg/labelsfilter/filter.go#L163) is applied for the set of labels for generating a security identity.
```go
	expressions := []string{
		k8sConst.PodNamespaceLabel,      // include io.kubernetes.pod.namespace
		k8sConst.PodNamespaceMetaLabels, // include all namespace labels
		k8sConst.AppKubernetes,          // include app.kubernetes.io
		"!io.kubernetes",                // ignore all other io.kubernetes labels
		"!kubernetes.io",                // ignore all other kubernetes.io labels
		"!.*beta.kubernetes.io",         // ignore all beta.kubernetes.io labels
		"!k8s.io",                       // ignore all k8s.io labels
		"!pod-template-generation",      // ignore pod-template-generation
		"!pod-template-hash",            // ignore pod-template-hash
		"!controller-revision-hash",     // ignore controller-revision-hash
		"!annotation.*",                 // ignore all annotation labels
		"!etcd_node",                    // ignore etcd_node label
```

In some cases, the endpoint's labels are not be available on endpoint startup,
in which case `reserved:init` label is attached to the endpoint and the
security identity is created accordingly. The security identity is updated as
and how the endpoint labels are fetched. Cilium's documentation in the [context
to endpoint
lifecycle](https://docs.cilium.io/en/v1.9/policy/lifecycle/#init-identity)
clarifies the scenarios and other aspect on how to use `reserved:init` label.

## Identity Transmission

### Per-packet Identity Transmission in VXLAN tunneled mode
This is the default mode for Cilium to operate. In this case, Cilium
establishes a vxlan tunnel between the nodes on UDP port 8472. The inter-host
traffic is tunneled in the UDP packets using vxlan encapusation and the `VXLAN
Network Identifier (VNI)` contains the source identity of the pod originating
the traffic.

In my case, `cilium identity list` shows up:

```
. . .
12643   k8s:class=deathstar
        k8s:io.cilium.k8s.policy.cluster=default
        k8s:io.cilium.k8s.policy.serviceaccount=default
        k8s:io.kubernetes.pod.namespace=default
        k8s:org=empire
        : : :
36927   k8s:class=xwing
        k8s:io.cilium.k8s.policy.cluster=default
        k8s:io.cilium.k8s.policy.serviceaccount=default
        k8s:io.kubernetes.pod.namespace=default
        k8s:org=alliance
. . .
```

where 36927 is the identity of the xwing pod. An execution of a command
`kubectl exec xwing -- curl -s -XPOST
deathstar.default.svc.cluster.local/v1/request-landing` results in the VXLAN
UDP packets getting generated and if you capture the packets on the host, you
should be able to see the VNI field with the identity value of the source pod.

![Identity used for pod xwing](../res/imgs/vxlan-network-identifier-with-identity.png)

In this you can see that the tunneled traffic originating from xwing has a VNI
value of 36927 indicating the source identity of the pod xwing.

![Identity used for pod deathstar](../res/imgs/vxlan-network-identifier-with-identity-rsp.png)

In this you can see that the tunneled traffic originating from xwing has a VNI
value of 12643 indicating the source identity of the pod deathstar.

### Use of ipcache for resolving identity in case of direct-routing mode
In case where `tunnel: disabled` mode is used, Cilium assumes direct-routing
mode i.e, it uses kernel routing functionality to be used for packet routing.
Since the vxlan headers in this case aren't available the cilium resolves the
security identity by resolving the IP address of the pod.

```
vagrant@k8s1:~/go/src/github.com/cilium/cilium$ cilium ip list
IP                              IDENTITY                                          SOURCE
0.0.0.0/0                       world                                             
::/0                            world                                             
10.0.2.15/32                    host                                              
10.11.0.6/32                    k8s:io.kubernetes.pod.namespace=default           k8s
                                k8s:org=empire                                    
                                k8s:class=deathstar                               
                                k8s:io.cilium.k8s.policy.cluster=default          
                                k8s:io.cilium.k8s.policy.serviceaccount=default   
10.11.0.49/32                   host                                              
10.11.0.78/32                   k8s:io.cilium.k8s.policy.serviceaccount=coredns   k8s
                                k8s:io.cilium.k8s.policy.cluster=default          
                                k8s:k8s-app=kube-dns                              
                                k8s:io.kubernetes.pod.namespace=kube-system       
10.11.0.210/32                  health                                            
10.11.1.116/32                  k8s:io.cilium.k8s.policy.cluster=default          k8s
                                k8s:io.cilium.k8s.policy.serviceaccount=default   
                                k8s:io.kubernetes.pod.namespace=default           
                                k8s:org=alliance                                  
                                k8s:class=xwing                                   
            ...
```

As shown above, Cilium maintains an ipcache which resolves a pod IP address to
the corresponding set of pod labels. Note that Cilium keeps this information as
part of etcd. In case of a multi-cluster deployment, a managed etcd is used
which synchronizes the multi-cluster wide state.

```
vagrant@k8s1:~/go/src/github.com/cilium/cilium$ etcdctl get --prefix "cilium/state/ip/v1/default/10.11.0.6" | tail -1 | jq 
```
```json
{
    "IP": "10.11.0.6", <-------------\
    "Mask": null,                    |
    "HostIP": "192.168.33.11",       |
    "ID": 12643,   <-----------------/----- NOTE THIS
    "Key": 0,
    "Metadata": "cilium-global:default:k8s1:3280",
    "K8sNamespace": "default",
    "K8sPodName": "deathstar-c74d84667-pb54w"
}
```

In the above case, pod IP address 10.11.0.6 is resolved to `ID: 12643`.

Note that, unlike other CNIs, Cilium does not use the pod IP address to create
an iptables/netfilter rule. An addition of new pod will result in addition of a
new entry in the kvstore and that's about it. CNIs dependent on iptables will
have to add a new iptables rule on all the corresponding hosts because of new
pod addition.

The verification of source identity is done in the ebpf code in the ingress
node.

