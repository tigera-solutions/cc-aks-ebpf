# Module 8 - Clean up

1. Delete the example application stacks and namespaces.

   ```bash
   kubectl delete -f pre/3-stars.yaml
   ```

2. Delete the AKS cluster.

   ```bash
   az aks delete \
     --resource-group $RESOURCE_GROUP \
     --name $CLUSTERNAME
   ```

3. Delete the resource group.

   ```bash
   az group delete \
     --name $RESOURCE_GROUP
   ```

4. Delete environment variables backup file.

   ```bash
   rm envLabVars.env
   ```

---

[:arrow_left: Module 7 - Policy Debugging with eBPF](module-7-policy-debugging-ebpf.md)  

[:leftwards_arrow_with_hook: Back to Main](../README.md)
