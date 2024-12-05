# Module 2 - Deploy an Azure AKS Cluster

## Initial Setup

1. Define the environment variables to be used by the resources definition.

   > **NOTE**: In the following sections we'll be generating and setting some environment variables. If you're terminal session restarts you may need to reset these variables. You can use that via the following command:
   >
   > source envLabVars.env

   Start in the root of the repository folder

   ```bash
   cd cc-aks-ebpf
   ```

   ```bash
   export RESOURCE_GROUP=rg-cc-aks-ebpf
   export CLUSTERNAME=myname-cc-aks-ebpf
   export LOCATION=canadacentral
   export K8S_VERSION=1.30.6
   export POD_CIDR='10.244.0.0/16'
   # Persist for later sessions in case of disconnection.
   echo export RESOURCE_GROUP=$RESOURCE_GROUP >> envLabVars.env
   echo export CLUSTERNAME=$CLUSTERNAME >> envLabVars.env
   echo export LOCATION=$LOCATION >> envLabVars.env
   echo export K8S_VERSION=$K8S_VERSION >> envLabVars.env
   echo export POD_CIDR=$POD_CIDR >> envLabVars.env
   ```

2. If not created, create the Resource Group in the desired Region.

   ```bash
   az group create \
     --name $RESOURCE_GROUP \
     --location $LOCATION
   ```

3. Create the AKS cluster without a network plugin. We will install Calico OSS CNI afterwards.

   ```bash
   az aks create \
     --resource-group $RESOURCE_GROUP \
     --name $CLUSTERNAME \
     --kubernetes-version $K8S_VERSION \
     --nodepool-name 'aks-ebpf-ng' \
     --location $LOCATION \
     --node-count 3 \
     --network-plugin none \
     --pod-cidr $POD_CIDR \
     --node-osdisk-size 50 \
     --node-vm-size Standard_B2ms \
     --max-pods 70 \
     --generate-ssh-keys \
     --enable-managed-identity \
     --output table
   ```

4. Verify your cluster status. The `ProvisioningState` should be `Succeeded`

   ```bash
   az aks list -o table | grep $CLUSTERNAME
   ```

   Or

   ```bash
   watch az aks list -o table 
   ```

   You may get an output like the following

   ```bash
   Name                     Location       ResourceGroup              KubernetesVersion    CurrentKubernetesVersion    ProvisioningState    Fqdn
   -----------------------  -------------  -------------------------  -------------------  --------------------------  -------------------  -----------------------------------------------------------------------
   aks-cc-aks-ebpf                canadacentral  rg-cc-aks-ebpf                1.30                 1.30.6                      Succeeded          aks-rg-cc-aks-03cfb8-ub5gqil0.hcp.canadacentral.azmk8s.io
   ```

5. Get the credentials to connect to the cluster.

   ```bash
   az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTERNAME
   ```

6. Verify you have API access to your new AKS cluster

   ```bash
   kubectl get nodes
   ```

   The output will be something similar to the this:

   ```bash
    NAME                                STATUS   ROLES   AGE    VERSION
    aks-nodepool1-24664026-vmss000000   NotReady    agent   7m1s   v1.30.6
    aks-nodepool1-24664026-vmss000001   NotReady    agent   7m4s   v1.30.6
   ```

   The nodes are showing `NotReady` because we haven't actually installed Calico as a CNI yet, which is the next step.

---

## Install Calico CNI in iptables/nftables mode

By default, when installing Calico as a CNI on an AKS cluster as a BYOCNI option, the default dataplane will be iptables/nftables (depending on the OS) unless the user configures it as eBPF. For the sake of this workshop, we want to compare the two dataplanes so we start with iptables and then convert the cluster to eBPF later.

1. Configure Calico Helm repo

   ```bash
   helm repo add projectcalico https://docs.tigera.io/calico/charts
   ```

2. Configure Helm values

   ```bash
   cat > values.yaml <<EOF
   installation:
     kubernetesProvider: AKS
     cni:
      type: Calico
      ipam:
        type: Calico
     calicoNetwork:
       hostPorts: Disabled
       bgp: Disabled
       ipPools:
       - cidr: $POD_CIDR
         encapsulation: VXLAN
   EOF
   ```

3. Install Calico OSS using Helm values file

   ```bash
   kubectl create namespace tigera-operator
   ```

   ```bash
   helm install calico projectcalico/tigera-operator --version v3.29.1 -f values.yaml --namespace tigera-operator
   ```

4. The nodes should now be networked properly with Calico as CNI and should show status as `Ready`

   ```bash
    NAME                                STATUS   ROLES   AGE    VERSION
    aks-nodepool1-24664026-vmss000000   Ready    agent   9m3s   v1.30.6
    aks-nodepool1-24664026-vmss000001   Ready    agent   9m3s   v1.30.6
   ```

5. At this point, the Calico components should also be functional and show as `Available`

   ```bash
   kubectl get tigerastatus 
   ```

   The output should look something like this:

   ```bash
   NAME                            AVAILABLE   PROGRESSING   DEGRADED   SINCE
   apiserver                       True        False         False      11m
   calico                          True        False         False      11m
   ```

Now we are ready to connect the cluster to Calico Cloud.

---

[:arrow_right: Module 3 - Connect the cluster to Calico Cloud](module-3-connect-calicocloud.md)  

[:arrow_left: Module 1 - Getting Started](module-1-getting-started.md)

[:leftwards_arrow_with_hook: Back to Main](../README.md)  
