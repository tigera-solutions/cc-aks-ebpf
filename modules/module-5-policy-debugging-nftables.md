# Module 5 - Policy Debugging with nftables

## Test and Visualize Allowed & Denied Flows

1. First review the policy for the `client` namespace (and associated pod) in the `Policy Board` on Calico Cloud:

   ![client-policy](https://github.com/user-attachments/assets/8c2c02da-3c27-493d-b27f-27892b1bc8a8)

   We see that `TCP` egress traffic from the `client` namespace is allowed to the pods in the `stars` namespace on destination ports `6379` and `80`

2. Test traffic flow from the `client` pod to the `frontend` pod in the `stars` namespace:

   ```bash
   kubectl -n client exec -it $(kubectl get po -n client -l role=client -ojsonpath='{.items[0].metadata.name}')  -- /bin/bash -c 'curl -m3 -I http://frontend.stars'
   ```

   The expected result is to get a `HTTP/1.1 404 NotFound` response in the header like so:

   ```bash
   HTTP/1.1 404 Not Found
   Date: Thu, 05 Dec 2024 08:24:21 GMT
   Transfer-Encoding: chunked
   ```

   Also observe the result of the flow in service graph and the policy that allowed it.

3. Next test the traffic flow from the `client` pod to a pod that isn't allowed by our policy:

   ```bash
   kubectl -n client exec -it $(kubectl get po -n client -l role=client -ojsonpath='{.items[0].metadata.name}')  -- /bin/bash -c 'curl -m3 -I http://management-ui.management-ui'
   ```

   We see that the `curl` request timed out after the set timeout of `3` seconds:

   ```bash
   curl: (28) Connection timed out after 3000 milliseconds
   command terminated with exit code 28
   ```

   This traffic is denied by the policy through the implicit deny due to not having a rule present to allow this traffic explicitly. Once again, this can be visualized on the service graph as a denied flow.

## Debug Policy Enforcement by `nftables` on the node

1. First, find out the node that the `client` pod is running on:

   ```bash
   NODE_NAME=$(kubectl get pod -n client -o=jsonpath='{.items[0].spec.nodeName}')
   CLIENT_POD_IP=$(kubectl get pod -n client -o=jsonpath='{.items[0].status.podIP}')
   echo $CLIENT_POD_IP
   ```

2. Then utilize the `privileged` pod that was deployed to get a shell into the node:

   ```bash
   PRIV_POD=$(kubectl get pods -o=jsonpath='{.items[?(@.spec.nodeName=="'"$NODE_NAME"'")].metadata.name}')
   kubectl exec -it "$PRIV_POD" -- /bin/sh
   ```

   Once in, get into the host namespace:

   ```bash
   chroot /host
   ```

3. Find the `cali*` interface associated with the `client` pod IP using `ip route`

   Assuming as an example, if `CLIENT_POD_IP` is `10.244.24.179` then do:

   ```bash
   ip route | grep 10.244.24.179
   ```

   This might give an output like:

   ```bash
   10.244.24.179 dev cali0cdd47bbdcb scope link
   ```

4. Find the filter table and associated `nftables` chain with that `cali0cdd47bbdcb` interface:

   ```bash
   nft list ruleset | grep cali0cdd47bbdcb
   ```

   This should give a result like:

   ```bash
   oifname "cali0cdd47bbdcb"  counter packets 4949 bytes 296940 goto cali-tw-cali0cdd47bbdcb
   iifname "cali0cdd47bbdcb"  counter packets 53584 bytes 4836559 goto cali-fw-cali0cdd47bbdcb
   chain cali-tw-cali0cdd47bbdcb {
   chain cali-fw-cali0cdd47bbdcb {
   ```

   These are `nftables` chains that the policy rules we defined ultimately link to

   Here `cali-tw` means `to-workload`  AKA corresponding to the `Ingress` to the `client` pod/workload
   and  `cali-fw` means `from-workload` AKA corresponding to the `Egress` from the `client` pod/workload

5. Focus into the `cali-fw` chain to look at our egress rule that was defined from the `client` pod:

   ```bash
   nft list chain ip filter cali-fw-cali0cdd47bbdcb
   ```

   This dumps the output of that chain that marks the packets with the final chain that matches the rule.
   What we are looking for is the `drop` action that drops packets if the rule isn't matched for the `allow` rule:

   ```bash
   meta mark & 0x00020000 == 0x00000000 counter packets 1070 bytes 77166 log prefix "DPE|platform" group 2 snaplen 80
   meta mark & 0x00020000 == 0x00000000 counter packets 1070 bytes 77166 drop
   ```

6. We can run the command again to try and send traffic to `management-ui` service which should fail and you will see the counter increment here for the dropped packets.

   ```bash
   meta mark & 0x00020000 == 0x00000000 counter packets 1072 bytes 77286 log prefix "DPE|platform" group 2 snaplen 80
   meta mark & 0x00020000 == 0x00000000 counter packets 1072 bytes 77286 drop
   ```

We have successfully traced the network policy we applied to the `nftables` rule corresponding to it and seen that when we tested a dropped packet it indeed did so on the host.

Next, we will change the dataplane of our cluster from `nftables` to `ebpf` and see how we can debug the policy in the new dataplane

[:arrow_right: Module 6 - Switching to eBPF Dataplane](module-6-switch-ebpf-dataplane.md)  

[:arrow_left: Module 4 - Install Demo Apps](module-4-install-demo-apps.md)

[:leftwards_arrow_with_hook: Back to Main](../README.md)
