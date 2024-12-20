# Module 3 - Connect the cluster to Calico Cloud

> **Note**: In order to complete this module, you will need to a [Calico Cloud account](https://www.calicocloud.io/). If you are participating in a live workshop, you will receive an invite with the information to login into an active Calico Cloud environment and join your AKS cluster in there.  
If you are running this workshop in a self-paced mode, you can create an Calico Cloud environment following the steps [here](submodule-3.1-create-calicloud.md).  

Issues with being unable to navigate menus in the UI are often due to browsers blocking scripts - please ensure that you disabled all blocker scripts.

## Step 1 - Accept the Invitation

1. During the workshop, you will receive an invitation to connect to a Calico Cloud organization, just like in the picture below:

2. Click on the link ACCEPT INVITATION and create an password to access the Calico Cloud.

3. Once you have access to your **Calico Cloud** environment, go to step 2:

## Step 2 - Connecting your cluster to Calico Cloud

1. The welcome screen will allow you to choose among four use cases and will provide a guided tour for each use case. After that you can proceed to connect your first cluster. This option directs you to the **Managed Clusters** section. Click on the "**Connect Cluster**" button to start the process.

   The Connect Cluster window will allow you to choose a name to identify your cluster in Calico Cloud and select which platform you are running the cluster on. The next window presents a link for you to review the cluster requirements for Calico Cloud. A Helm command to run the installation script will be generated, you need to copy and apply this command in your cluster.

   > **Warning**
   > Please choose a unique name with your alias or username for the cluster name as the workshop uses a shared Calico Cloud instance.
   > A good example: "cc-aks-myname-workshop"  
   > Also, check the `Advanced` checkbox to use `Helm` option to generate the command via `Helm` to stay consistent with our installation method!

   ![connect-cluster-helm](https://github.com/user-attachments/assets/e90a18c5-e25d-4bf8-bc61-97ad3a3d396c)

2. Run the installation script in your cluster. Script should look similar to this:

    ```bash
   helm repo add calico-cloud https://installer.calicocloud.io/charts --force-update && helm upgrade --install calico-cloud-crds calico-cloud/calico-cloud-crds --namespace calico-cloud --create-namespace && helm upgrade --install calico-cloud calico-cloud/calico-cloud --namespace calico-cloud --set apiKey=blahblahblah --set installer.clusterName=myname-cc-aks-ebpf --set installer.calicoCloudVersion=v20.3.0
    ```

    > Output should look similar to:

    ```bash
    namespace/calico-cloud created
    customresourcedefinition.apiextensions.k8s.io/installers.operator.calicocloud.io created
    serviceaccount/calico-cloud-controller-manager created
    role.rbac.authorization.k8s.io/calico-cloud-leader-election-role created
    clusterrole.rbac.authorization.k8s.io/calico-cloud-metrics-reader created
    clusterrole.rbac.authorization.k8s.io/calico-cloud-proxy-role created
    rolebinding.rbac.authorization.k8s.io/calico-cloud-leader-election-rolebinding created
    clusterrolebinding.rbac.authorization.k8s.io/calico-cloud-installer-rbac created
    clusterrolebinding.rbac.authorization.k8s.io/calico-cloud-proxy-rolebinding created
    configmap/calico-cloud-manager-config created
    service/calico-cloud-controller-manager-metrics-service created
    deployment.apps/calico-cloud-controller-manager created
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
    100   355  100   355    0     0    541      0 --:--:-- --:--:-- --:--:--   541
    secret/api-key created
    installer.operator.calicocloud.io/aks-cc-repo created
    ```

    Joining the cluster to Calico Cloud can take a few minutes. Meanwhile the Calico resources can be monitored until they are all reporting `Available` as `True`

    ```bash
    kubectl get tigerastatus                                                                                                                    
    ```

    > Output should look similar to:

    ```bash
    NAME                            AVAILABLE   PROGRESSING   DEGRADED   SINCE
   apiserver                       True        False         False      88m
   calico                          True        False         False      62m
   cloud-core                      True        False         False      5h52m
   compliance                      True        False         False      63m
   image-assurance                 True        False         False      3h17m
   intrusion-detection             True        False         False      92m
   ippools                         True        False         False      93m
   log-collector                   True        False         False      62m
   management-cluster-connection   True        False         False      92m
   monitor                         True        False         False      75m
   policy-recommendation           True        False         False      93m
   tiers                           True        False         False      93m
    ```

    You can also monitor your cluster installation on the Calico Cloud UI. Go to the "**Managed Clusters**" section, select your cluster and expand the timestamp dropdown to see the installation logs.
    In a few minutes the status will change from **Installing** to **Done**. Congratulations! You successfully connected your cluster to Calico Cloud.

    ![monitor-install](https://github.com/tigera-solutions/cc-aks-shift-left-workshop/assets/117195889/978cb58e-85b6-43b0-a32e-90850684e78f)

## STEP 3 - Selecting your cluster

Once the installation is completed, you will be able to start interacting with your cluster from the Calico Cloud interface. Calico Cloud provides a single pane of glass for managing multiple clusters. If you followed the previous steps, you would have two clusters connected to Calico Cloud at this point: Your cluster and a pre-configured lab cluster that allows you to explore some of the features in Calico Cloud.

You can switch between clusters by following the steps below:

1. Navigate to the Dashboard section - the first icon under the Calico Cat on the top-left of the UI.

2. Click on the **Cluster** dropdown button on the top-right of the UI.

3. Select your recently added cluster.

   ![select-cluster](https://github.com/tigera-solutions/cc-aks-shift-left-workshop/assets/117195889/4b6b5957-bf7c-4373-aa04-2d38f1eb2601)

The "**Cluster**" dropdown button will be always visible accross the Calico Cloud UI, no matter which section you are viewing. You can change the cluster you want to interact with at any moment.
When you change the cluster, the whole Calico Cloud context will change immediately to reflect the information regarding the current selected cluster.

Next, we can install the demo workloads and observe the traffic flows using Calico Cloud.

---

[:arrow_right: Module 4 - Install Demo Apps](module-4-install-demo-apps.md)  
[:arrow_left: Module 2 - Deploy an AKS cluster](module-2-deploy-aks.md)

[:leftwards_arrow_with_hook: Back to Main](../README.md)  
