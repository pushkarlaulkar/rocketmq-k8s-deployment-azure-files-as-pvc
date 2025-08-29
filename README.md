Steps to deploy **RocketMQ** as Kuberenetes Deployment with **Azure Files** as data persistence.

If there is an Azure Firewall in your environment through which all outbound access should go, then a rule needs to be added for port 445 as shown below

<img width="456" height="577" alt="image" src="https://github.com/user-attachments/assets/58341c28-7cc5-461f-a3bc-68f2b6f13083" />

1. Create an Azure File Share through portal or CLI or Powershell

2. Create PV by applying below command

   ```
   kubectl apply -f pv.yaml
   ```

3. Create PVC

   ```
   kubectl apply -f pvc.yaml
   ```

4. Make sure PVC is bound to the PV. Output should be similar as below

   ```
   NAME                           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE   VOLUMEMODE
   persistentvolume/rocketmq-pv   500Gi      RWX            Retain           Bound    your-namespace/rocketmq-pvc                  <unset>                          34m   Filesystem
  
   NAME                                 STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE   VOLUMEMODE
   persistentvolumeclaim/rocketmq-pvc   Bound    rocketmq-pv   500Gi      RWX                           <unset>                 33m   Filesystem
   ```

5. Apply the deployment

   ```
   kubectl apply -f rocketmq.yaml
   ```
6. If the rmqbroker pod stays in Init condition for a long time, describe the pod using ` kubectl describe pod-name ` and check the error. If below error is shown

   ```
     Warning  FailedMount  8s (x6 over 27s)  kubelet            MountVolume.MountDevice failed for volume "rocketmq-pv" : rpc error: code = InvalidArgument desc = GetAccountInfo(resource-group-name#storage-account-name#share-name) failed with error: POST https://management.azure.com/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.Storage/storageAccounts/storage-account-name/listKeys

   RESPONSE 403: 403 Forbidden
   ERROR CODE: AuthorizationFailed
   
   {
     "error": {
       "code": "AuthorizationFailed",
       "message": "The client 'client-id' with object id 'client-id' does not have authorization to perform action 'Microsoft.Storage/storageAccounts/listKeys/action' over scope '/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.Storage/storageAccounts/storage-account-name' or the scope is invalid. If access was recently granted, please refresh your credentials."
     }
   }
   ```
   Assign below role assignment

   ```
   az role assignment create \
   --assignee client-id \
   --role "Storage Account Contributor" \
   --scope /subscriptions/subscription-id/resourceGroups/resource-group-name
   ```
