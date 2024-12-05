# Module 6: Switching to eBPF Dataplane

## Configure Calico Cloud to talk directly to the API server

In eBPF mode, Calico Cloud implements Kubernetes service networking directly (rather than relying on kube-proxy). Of course, this makes it highly desirable to disable kube-proxy when running in eBPF mode to save resources and avoid confusion over which component is handling services.

To be able to disable kube-proxy, Calico Cloud needs to communicate to the API server directly rather than going through kube-proxy. To make that possible, we need to find a persistent, static way to reach the API server. The best way to do that varies by Kubernetes distribution, but for AKS we should use **the FQDN of the API server's load balancer**

1. Get the values from AKS cluster settings:

   ```bash
   KUBERNETES_SERVICE_HOST=$(az aks show -g $RESOURCE_GROUP -n $CLUSTERNAME --query 'fqdn' --output tsv)
   KUBERNETES_SERVICE_PORT=$(kubectl get endpoints kubernetes -ojsonpath='{.subsets[0].ports[0].port}')
   ```

2. Configure the service endpoint `ConfigMap`:

   ```bash
   kubectl apply -f - << EOF
   kind: ConfigMap
   apiVersion: v1
   metadata:
     name: kubernetes-services-endpoint
     namespace: tigera-operator
   data:
     KUBERNETES_SERVICE_HOST: "${KUBERNETES_SERVICE_HOST}"
     KUBERNETES_SERVICE_PORT: "${KUBERNETES_SERVICE_PORT}"
   EOF
   ```

   The operator will pick up the change to the config map automatically and do a rolling update of Calico Cloud to pass on the change. Confirm that pods restart and then reach the Running state with the following command:

   ```bash
   watch kubectl get pods -ncalico-system
   ```

3. Disable `kube-proxy`:

   ```bash
   kubectl patch ds -n kube-system kube-proxy -p '{"spec":{"template":{"spec":{"nodeSelector":{"non-calico": "true"}}}}}'
   ```

4. Finally, enable `ebpf` by patching the `Installation` CR:

   ```bash
   kubectl patch installation.operator.tigera.io default --type merge -p '{"spec":{"calicoNetwork":{"linuxDataplane":"BPF"}}}'
   ```

   When enabling eBPF mode, preexisting connections continue to use the non-BPF datapath; such connections should not be disrupted, but they do not benefit from eBPF mode’s advantages.

5. For the purpose of this workshop, we will cycle all the existing cluster pods so that they all get onto the ebpf dataplane and stop using nftables for the existing flows(of course, this is not a good idea in prod or any real-world scenario unless planned)

   ```bash
   kubectl get pods --all-namespaces -o custom-columns=":metadata.name,:metadata.namespace" | tail -n +2 | while read pod namespace; do kubectl delete pod "$pod" -n "$namespace"; done
   ```

Once it has been verified that ```calico-system``` is stable, and ```tigerastatus``` shows us a clean result, we are ready to test workloads once more to look at how the `ebpf` dataplane handles policies.

[:arrow_right: Module 7 - Policy Debugging with eBPF](module-7-policy-debugging-ebpf.md)  

[:arrow_left: Module 5 - Policy Debugging with nftables](module-5-policy-debugging-nftables.md)

[:leftwards_arrow_with_hook: Back to Main](../README.md)
