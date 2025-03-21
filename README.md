# Deploying Simple Web APP on DOKS
Simple K8s application setup in DigitalOcean Cluster

## Pre-requisites
- DigitalOcean Cluster 2 nodes
- DigitalOcean Registry
- Doctl installation [here](https://docs.digitalocean.com/reference/doctl/how-to/install/)
- Docker app for building and pushing the image

## Steps to Build the code and push to DigitalOcean Registry

**Step 1:** Clone the repository and CD into the webapp folder by using the following commands
```
git clone https://github.com/AshokMookkaiah/TAMAssignDO.git
cd TAMAssignDO/webapp
```
**Step 2:** Authenticate with doctl
Ensure you have doctl (DigitalOcean CLI) installed and authenticated.

```
doctl auth init
```
**Step 3:** Log in to DigitalOcean Container Registry
Run the following command to authenticate Docker with DigitalOcean:
```
doctl registry login
```
**Step 4:** Build the Docker image 
```
docker build -t myapp .
```
**Step 5:** Tag the Image for DigitalOcean Registry
```
docker tag my-web-app registry.digitalocean.com/<your-registry-name>/myapp:1.0 
```
**Step 6:** Push the image to Digital Ocean Registry
```
docker push registry.digitalocean.com/<your-registry>/my-web-app:1.0 
```
Now the image is successfully built and pushed to the Digital Ocean Registry

## Connecting the DigitalOcean Private Registry to the DOKS Cluster
Visit the [registry page](https://cloud.digitalocean.com/registry) and click the Settings tab. In the DigitalOcean Kubernetes integration section, click Edit to display the available Kubernetes clusters. Select the clusters you wish to add and click Save.
In the control panel, you can select the Kubernetes clusters to use with your registry. This generates a secret, adds it to all the namespaces in the cluster and updates the default service account to include the secret, allowing you to pull images from the registry.

> [!Note]
> You can only integrate the latest Kubernetes patch versions (1.19+) with the registry. For more information on upgrading your Kubernetes clusters, see How to Upgrade DOKS Clusters to Newer Versions.

## Deploying the application in DOKS

**Step 1:** Connect to the cluster by using doctl method or by downloading the kubeconfig from the Digitalocean console.

**Step 2:** Now cd into the k8s directory in the cloned directory
```
cd TAMAssignDO/k8s
```
**Step 3:** Create the Web-app namespace in the DOKS Cluster using the following command
```
kubectl create ns web-app
```
**Step 4:** After creating the namespace we wil deploy the application using the manifest file present in the directory
```
kubectl apply -f app-deploy.yaml
```
**Step 5:** After deploying the application wait for all the pods to comeup and see the status of the pod using the following command
```
kubectl get po -n web-app
NAME                      READY   STATUS    RESTARTS   AGE
web-app-8c8ccc5d6-rmx7n   1/1     Running   0          53m
web-app-8c8ccc5d6-xn97f   1/1     Running   0          53m
web-app-8c8ccc5d6-zjgnj   1/1     Running   0          53m
```
**Step 6:** Now apply the service configuration file to create a loadbalancer for the application
```
kubectl apply -f app-svc.yaml
```
**Step7:** Wait for few minutes till the DigitalOcean Load Balancer gets created in the service we can check if the loadbalancer is assigned by using the following command
```
kubectl get svc -n web-app
NAME              TYPE           CLUSTER-IP     EXTERNAL-IP          PORT(S)        AGE
web-app-service   LoadBalancer   10.108.90.89   <Load-Balancer-ip>   80:30582/TCP   56m
```
**Step 8:** Before enabling the Horizontal Pod Auto scalar we must enable the metrics server in the DOKS cluster using the following command
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
**Step 9:** Now create the HPA in the web-app namespace using the following manifest file
```
kubectl apply -f app-hpa.yaml
```
Sample Web App with HPA and DigitalOcean registry is deployed in the DOKS cluster.
