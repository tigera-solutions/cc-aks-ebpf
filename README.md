# Azure: Kubernetes Networking with Calico eBPF for AKS Clusters

## Welcome

In this AKS-focused workshop, you will work with Azure and Calico Cloud to learn about Calico's ebpf dataplane and how to examine and debug Calico Network Policies to secure workloads in Calico Cloud.

AKS provides a [Bring your own CNI plugin option](https://learn.microsoft.com/en-us/azure/aks/use-byo-cni?tabs=azure-cli) where each pod gets networked by a third-party CNI getting a private IP from configured POD CIDR. This mode can be leveraged to take full advantage of Calico's eBPF dataplane for networking.

## What is eBPF?

eBPF is a feature available in Linux kernels that allows you to run a virtual machine inside the kernel. This virtual machine allows you to safely load programs into the kernel, to customize its operation. Why is this important?

In the past, making changes to the kernel was difficult: there were APIs you could call to get data, but you couldn’t influence what was inside the kernel or execute code. Instead, you had to submit a patch to the Linux community and wait for it to be approved. With eBPF, you can load a program into the kernel and instruct the kernel to execute your program if, for example, a certain packet is seen or another event occurs.

With eBPF, the kernel and its behavior become highly customizable, instead of being fixed. This can be extremely beneficial, when used under the right circumstances.

## Calico Cloud + eBPF

Calico Cloud offers an eBPF data plane as an alternative to our standard Linux dataplane (which is iptables/nftables based). While the standard data plane focuses on compatibility by working together with kube-proxy and your own iptables rules, the eBPF data plane focuses on performance, latency, and improving user experience with features that aren’t possible with the standard data plane.

If you enable eBPF within Calico Cloud but have existing iptables flows, we won’t touch them. Because maybe you want to use connect-time load balancing, but leave iptables as is. With Calico Cloud, it’s not an all-or-nothing deal—we allow you to easily load and unload our eBPF data plane to suit your needs, which means you can quickly try it out before making a decision.

## Use-Cases for eBPF

- XDP
- Connect-time Load Balancing
- Better routing to proxies like Envoy

## Value

The eBPF dataplane mode has several advantages over standard Linux networking pipeline mode:

- It scales to higher throughput.
- It uses less CPU per GBit.
- It has native support for Kubernetes services (without needing kube-proxy) that:
  - Reduces first packet latency for packets to services.
  - Preserves external client source IP addresses all the way to the pod.
  - Supports DSR (Direct Server Return) for more efficient service routing.
  - Uses less CPU than kube-proxy to keep the dataplane in sync.

### Time Requirements

The estimated time to complete this workshop is 60-90 minutes.

### Target Audience

- Cloud Professionals
- DevSecOps Professional
- Site Reliability Engineers (SRE)
- Solutions Architects
- Anyone interested in Calico Cloud :)

## Modules

Module 1 - [Getting Started](modules/module-1-getting-started.md)  
Module 2 - [Deploy an Azure AKS Cluster](modules/module-2-deploy-aks.md)  
Module 3 - [Connect the cluster to Calico Cloud](modules/module-3-connect-calicocloud.md)  
Module 4 - [Install Demo Apps](modules/module-4-install-demo-apps.md)  
Module 5 - [Policy Debugging with nftables](modules/module-5-policy-debugging-nftables.md)  
Module 6 - [Switching to ebpf Dataplane](modules/module-6-switch-ebpf-dataplane.md)  
Module 7 - [Policy Debugging with ebpf](modules/module-7-policy-debugging-ebpf.md)  
Module 8 - [Cleanup](modules/module-8-cleanup.md)  

**Follow us on social media:**

- [LinkedIn](https://www.linkedin.com/company/tigera/)
- [Twitter/X](https://twitter.com/tigeraio)
- [YouTube](https://www.youtube.com/channel/UC8uN3yhpeBeerGNwDiQbcgw/)
- [Slack](https://calicousers.slack.com/)
- [Github](https://github.com/tigera-solutions/)
- [Discuss](https://discuss.projectcalico.tigera.io/)

> [!NOTE]
> The examples and sample code provided in this workshop are intended to be consumed as instructional content. These will help you understand how Calico Cloud can be configured to build a functional solution. These examples are not intended for use in production environments.
