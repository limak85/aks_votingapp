# Simple voting app hosted on Azure Kubernetes Service (AKS)

Aim of this project is to create simple AKS cluster and deploy an application to the cluster using ansible.
There are many aspects of the AKS configuration and application deployment which I'm not going to address:
 - ingress controller
 - persistent storage
 - network policies
 - proper access control to Azure and AKS
to name just a few.

## Requirements
* [Azure account](https://azure.microsoft.com/en-us/free/)
* [Ansible](https://docs.ansible.com/ansible/latest/scenario_guides/guide_azure.html)
* [kubectl](https://kubernetes.io/docs/reference/kubectl/kubectl/)

## Deployment instruction
1. Install ansible and required components
    ```
    pip3 install ansible
    ansible-galaxy collection install azure.azcollection
    ansible-galaxy collection install community.kubernetes
    wget https://raw.githubusercontent.com/ansible-collections/azure/dev/requirements-azure.txt
    pip3 install -r requirements-azure.txt
    pip3 install openshift
    ```
2. Install [kubectl](https://kubernetes.io/docs/tasks/tools/)
3. Generate ssh keys. Unless ~/.ssh/id_rsa.pub is already present.
    ```
    ssh-keygen
    ```
4. [Install Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
5. [Sign in to Azure](https://docs.microsoft.com/en-us/cli/azure/authenticate-azure-cli?view=azure-cli-latest)
6. Download voting app file
    ```
    git clone https://github.com/limak85/aks_votingapp.git
    cd aks_votingapp
    ```
7. Create Azure [Service Principal Name](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli)
    ```
    az ad sp create-for-rbac --name AKSServicePrincipalName
    ```
8. Copy output from above command to infrastructure/az_sp.json file. File content should look like this:
    ```
    {
        "appId": "1692c558-9887-401f-9ae1-f3255ce78f89",
        "displayName": "AKSServicePrincipalName",
        "name": "http://AKSServicePrincipalName",
        "password": "YYYYY.00000_tBS4171.ZZZZZZ",
        "tenant": "abcde-1111-2222-3333-abcde12345"
    }    
    ```
9. Create Azure Kubernetes Service (AKS)
    ```
    ansible-playbook infrastructure/az_create_aks.yml
    ```
10. Configure kubectl
    ```
    az aks get-credentials --resource-group rg_aksiac --name aksiac
    ```
11. Deploy Voting app
    ```
    ansible-playbook voting-app/k8s_create_app.yml
    ```
## Verification
To verify app deployment run:
```
kubectl get deployment --namespace votingapp
```
should output:
```
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
redisdb     1/1     1            1           2m33s
votingapp   1/1     1            1           2m32s
```
and
```
kubectl get service --namespace votingapp
```
should give similar output to:
```
NAME        TYPE           CLUSTER-IP   EXTERNAL-IP    PORT(S)        AGE
redisdb     ClusterIP      10.2.0.205   <none>         6379/TCP       2m13s
votingapp   LoadBalancer   10.2.0.215   20.90.99.127   80:32483/TCP   2m13s
```

External IP can be used to access the app http://20.90.99.127

## Cleaning up
### To remove voting app run:
```
ansible-playbook voting-app/k8s_delete_app.yml
```
### To remove AKS cluster run:
```
ansible-playbook infrastructure/az_delete_aks.yml
```
## Other notes
Voting app comes from [https://github.com/Azure-Samples/azure-voting-app-redis](https://github.com/Azure-Samples/azure-voting-app-redis)

az_sp.json may contain sensitive information and should be secured with [ansible-vault](https://docs.ansible.com/ansible/latest/cli/ansible-vault.html)