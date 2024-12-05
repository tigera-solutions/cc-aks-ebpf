# Module 4 - Install Demo Apps

During this workshop, we will use the `Stars` application as an example to work on. In this module, we will set up some quick network policies to control access to and from the namespaces the app is deployed across.  Once the application is deployed and policies are applied, we will see how they are enforced on the node on the dataplane in `nftables` first to understand policy enforcement before we switch to `ebpf` and compare there.

1. Clone this repository in your Azure Cloud Shell/terminal.

   ```bash
   git clone https://github.com/tigera-solutions/cc-aks-ebpf.git && \
   cd cc-aks-ebpf
   ```

2. Install the example application stack:

   From the cloned directory, execute:

   ```bash
   kubectl create -f pre
   ```

   Included in the `pre` folder, there is the application that will be used in the exercises during the workshop. The diagram below shows how the applications' microservices communicate between themselves.

   ![stars](https://github.com/user-attachments/assets/5449ad0a-b7d3-45ac-bf86-fc629e3ff174)

   The necessary policies and tiers will also be created.

   > [!IMPORTANT]
   > Wait until all the pods are up and running to move to the next step.

3. Apply `Felix` optimizations:

   ```bash
   kubectl patch felixconfiguration default -p '{"spec":{"flowLogsFlushInterval":"15s"}}'
   kubectl patch felixconfiguration default -p '{"spec":{"dnsLogsFlushInterval":"15s"}}'
   kubectl patch felixconfiguration default -p '{"spec":{"flowLogsFileAggregationKindForAllowed":1}}'
   kubectl patch felixconfiguration default -p '{"spec":{"flowLogsFileAggregationKindForDenied":0}}'
   kubectl patch felixconfiguration default -p '{"spec":{"dnsLogsFileAggregationKind":0}}'
   ```

   Configure Felix to collect TCP stats - this uses eBPF TC program and requires miniumum Kernel version of v5.3.0/v4.18.0-193.

   ```bash
   kubectl patch felixconfiguration default -p '{"spec":{"flowLogsCollectTcpStats":true}}'
   ```

## Service Graph

Connect to Calico Cloud GUI. From the menu, select `Service Graph > Default`. Explore the options.

![service_graph](https://user-images.githubusercontent.com/104035488/192303379-efb43faa-1e71-41f2-9c54-c9b7f0538b34.gif)

Next, we will send some traffic to test the network policies and see how the dataplane enforces it.

[:arrow_right: Module 5 - Policy Debugging with nftables](module-5-policy-debugging-nftables.md)  

[:arrow_left: Module 3 - Connect the cluster to Calico Cloud](module-3-connect-calicocloud.md)

[:leftwards_arrow_with_hook: Back to Main](../README.md)
