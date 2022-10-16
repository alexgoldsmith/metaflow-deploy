V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V=V
Setup instructions for END USERS (e.g. someone running Flows vs the new stack):
-------------------------------------------------------------------------------
There are three steps:
1. Ensuring Azure access
2. Configure Metaflow
3. Run port forwards
4. Install necessary Azure Python SDK libraries


STEP 1: Ensure you have sufficient access to these Azure resources on your local workstation:

- AKS cluster ("aks-greenon-metaflow-default") ("Azure Kubernetes Service Contributor" + "Azure Kubernetes Service Cluster User Role")
- Azure Storage ("metaflow-storage-container" in the storage account "stgreenonmetaflowdefault") ("Storage Blob Data Contributor")

You can use "az login" as a sufficiently capabable account. To see the credentials for the service principal
(created by terraform) that is capable, run this:

$ terraform output -raw SERVICE_PRINCIPAL_CREDENTIALS

Use the credentials with "az login"

$ az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID

Configure your local Kubernetes context to point to the the right Kubernetes cluster:

$ az aks get-credentials --resource-group rg-metaflow-default-westus --name aks-greenon-metaflow-default

STEP 2: Configure Metaflow:

Option 1: Create JSON config directly

Create the file "~/.metaflowconfig/config.json" with this content. If this file already exists, keep a backup of it and move it aside first. 
{
    "METAFLOW_AZURE_STORAGE_BLOB_SERVICE_ENDPOINT": "https://stgreenonmetaflowdefault.blob.core.windows.net/",
    "METAFLOW_DATASTORE_SYSROOT_AZURE": "metaflow-storage-container/tf-full-stack-sysroot",
    "METAFLOW_DEFAULT_DATASTORE": "azure",
    "METAFLOW_DEFAULT_METADATA": "service",
    "METAFLOW_KUBERNETES_NAMESPACE": "default",
    "METAFLOW_KUBERNETES_SECRETS": "metaflow-azure-storage-credentials",
    "METAFLOW_KUBERNETES_SERVICE_ACCOUNT": "default",
    "METAFLOW_SERVICE_INTERNAL_URL": "http://metadata-service.default:8080/",
    "METAFLOW_SERVICE_URL": "http://127.0.0.1:8080/"
}

If deployed with Airflow or Argo then remove the `METAFLOW_KUBERNETES_SERVICE_ACCOUNT` key from the json file. 
If deployed with Airflow set `METAFLOW_KUBERNETES_NAMESPACE` to "airflow". 
If deployed with Argo set `METAFLOW_KUBERNETES_NAMESPACE` to "argo". 

Option 2: Interactive configuration

Run the following, one after another.

$ metaflow configure azure
$ metaflow configure kubernetes

Use these values when prompted:

METAFLOW_DATASTORE_SYSROOT_AZURE=metaflow-storage-container/tf-full-stack-sysroot
METAFLOW_AZURE_STORAGE_BLOB_SERVICE_ENDPOINT=https://stgreenonmetaflowdefault.blob.core.windows.net/
METAFLOW_KUBERNETES_SECRETS=metaflow-azure-storage-credentials
METAFLOW_SERVICE_URL=http://127.0.0.1:8080/
METAFLOW_SERVICE_INTERNAL_URL=http://metadata-service.default:8080/
[For Airflow only] METAFLOW_KUBERNETES_NAMESPACE=airflow
[For Argo only] METAFLOW_KUBERNETES_NAMESPACE=argo

Note: you can skip these:

METAFLOW_SERVICE_AUTH_KEY
METAFLOW_KUBERNETES_SERVICE_ACCOUNT
METAFLOW_KUBERNETES_CONTAINER_REGISTRY
METAFLOW_KUBERNETES_CONTAINER_IMAGE

STEP 3: Setup port-forwards to services running on Kubernetes:

option 1 - run kubectl's manually:
$ kubectl port-forward deployment/metadata-service 8080:8080
$ kubectl port-forward deployment/metaflow-ui-backend-service 8083:8083
$ kubectl port-forward deployment/metadata-ui-static-service 3000:3000
$ kubectl port-forward -n argo deployment/argo-server 2746:2746

option 2 - this script manages the same port-forwards for you (and prevents timeouts)

$ python metaflow-tools/scripts/forward_metaflow_ports.py [--include-argo] [--include-airflow]

STEP 4: Install Azure Python SDK
$ pip install azure-storage-blob azure-identity

#^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^=^
