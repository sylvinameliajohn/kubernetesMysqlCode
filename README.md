# Deploy application using NodeJS and MySQL on OKE

## Introduction 
This guide shows detailed steps of deploying MySQL on Kubernetes cluster on Oracle managed OKE cluster. 

* A Kubernetes cluster is a group of nodes. The nodes are the machines running applications. Each node can be a physical machine or a virtual machine. The node's capacity (its number of CPUs and amount of memory) is defined when the node is created. A cluster comprises:
  - one or more master nodes (for high availability, typically there will be a number of master nodes)
  - one or more worker nodes (sometimes known as minions)
* A Kubernetes cluster can be organized into namespaces to divide the cluster's resources between multiple users. Initially, a cluster has the following namespaces:
  - default, for resources with no other namespace
  - kube-system, for resources created by the Kubernetes system
  - kube-node-lease, for one lease object per node to help determine node availability
  - kube-public, usually used for resources that have to be accessible across the cluster
## Step 1 - Sign in to OCI Console
1. Go to [cloud.oracle.com](https://cloud.oracle.com/). Enter your Cloud Account Name and click Next. This is the name you chose while creating your account in the previous section. It's NOT your email address. If you've forgotten the name, see the confirmation email.
![image](https://user-images.githubusercontent.com/77334410/135283727-20e2f7eb-e5e2-46f1-ac40-6d6d9ec1dc11.png)
2. Click Continue to sign in using the "oraclecloudidentityservice".
![image](https://user-images.githubusercontent.com/77334410/135283805-ebbff5ed-73c1-4513-939d-cf528248f1b2.png)
3. When you sign up for an Oracle Cloud account, a user is created for you in Oracle Identity Cloud Service with the username and password you selected at sign up. You can use this single sign-on option to sign in to Oracle Cloud Infrastructure and then navigate to other Oracle Cloud services without reauthenticating. This user has administrator privileges for all the Oracle Cloud services included with your account.
4. Enter your Cloud Account credentials and click Sign In. Your username is your email address. <br/>
   The password is what you chose when you signed up for an account.<br/>
  ![image](https://user-images.githubusercontent.com/77334410/135283955-f443dff0-3d0d-4367-b6cb-452beb9577df.png)
5. You are now signed in to Oracle Cloud!

## Step 2 - Create Kubernetes Cluster in OKE
1. From OCI Services menu, Click **Container Clusters (OKE)** under **Developer Services**.
![image](https://user-images.githubusercontent.com/77334410/135286277-4be4ab09-743a-442e-8b61-cc0cc2fa9b74.png)
2. No need to create any policies for OKE, all the policies are pre-configured 
3. Under **List Scope**, select the compartment in which you would like to create a cluster. 
4. Click **Create Cluster**. Choose Quick Create and click Launch Workflow.
5. Fill out the dialog box:
  - NAME: Provide a name (oke-cluster in this example)
  - COMPARTMENT: Choose your compartment
  - CHOOSE VISIBILITY TYPE: Public
  - SHAPE: Choose a VM shape
  - NUMBER OF NODES: 3
  - KUBERNETES DASHBOARD ENABLED: Make sure flag is checked
  - Click Next and Click "Create Cluster".<br/>
  ![image](https://user-images.githubusercontent.com/77334410/135286814-59e3d5c2-2c0a-4258-a168-095d49d14a65.png)
6. We now have a OKE cluster with 3 nodes and Virtual Cloud Network with all the necessary resources and configuration needed

## Step 3 - Access the Kubernetes cluster in Cloud Shell
1. Once the cluster is provisioned(Status changes to Active), click on the clustername to view details about the cluster.
![image](https://user-images.githubusercontent.com/77334410/135289276-6d80d414-b831-487e-ae50-e4883f6b8563.png)
2. Click on **Access Cluster** to get the commands required for connecting to the Kubernetes cluster via Cloud shell
![image](https://user-images.githubusercontent.com/77334410/135289539-f5ba4ba7-354e-47e5-b263-66ed5d944d57.png)
3. Follow the steps mentioned in the popup screen. <br/>
![image](https://user-images.githubusercontent.com/77334410/135289928-d4a974d5-9aa3-4daa-9a95-a2b8db6b5406.png)
4. Validate the access to the cluster by typing the following command.
```` bash
kubectl get nodes
````
![image](https://user-images.githubusercontent.com/77334410/135291376-4095f9dd-8a42-479e-beeb-1cc735f79e2b.png)

## Step 4 - Deploy the NodeJS-MySQL application in the cluster
1. Clone the application code from Github by typing the following command in cloudshell
```` bash
git clone https://github.com/sylvinameliajohn/kubernetesMysqlCode.git
````
![image](https://user-images.githubusercontent.com/77334410/135292414-cf3e0a27-4a4d-4796-ab24-9b0cccf2136d.png)<br/>
The code is downloaded in a folder called **kubernetesMysqlCode**

2. Now run the following commands to create the MySQL containers
```` bash
kubectl create -f mysql-secret.yaml
kubectl get secrets
kubectl create -f mysql-pod.yaml
kubectl get pod
````
Wait till the status of the above command shows **Running**<br/>
![image](https://user-images.githubusercontent.com/77334410/135293630-a1c93ccd-bce1-47d9-99ab-94d8d184dd8f.png)

3. Create a MySQL database inside the newly created MySQL pod by following the commands below
```` bash
kubectl exec k8s-mysql -it -- bash
echo $MYSQL_ROOT_PASSWORD
mysql --user=root --password=$MYSQL_ROOT_PASSWORD
show databases;
create database k8smysqldb;
````
![image](https://user-images.githubusercontent.com/77334410/135294273-c6948c21-ef8b-4aaa-a8e3-c7ef0592ceb1.png)
4. Enter CTRL+Z followed by CTRL+D(twice) to exit the container
5. Create a kubernetes service by typing the command below
```` bash
kubectl create -f mysql-service.yaml
````
6. Get the MySQL pod IP using the command below and make a note of the IP returned
```` bash
kubectl get pod k8s-mysql -o template --template={{.status.podIP}}
````
7. Use the above IP to modify the application code by typing the following commands
```` bash
cd nodejs-app
vim api.js
````
8. Change the value of **host** in the opened file and replace it with the IP from previous step and save the file
![image](https://user-images.githubusercontent.com/77334410/135295930-76964678-ecf2-45a2-86cd-5315810e9263.png)

## Step 5 - Push the docker container images to OCIR 
1. Login to docker using the following command after replacing the identifiers
```` bash
docker login <region_code>.ocir.io --username=<objectstorage_namespace>/oracleidentitycloudservice/<user_name> --password='<auth_token>'
````
  - Replace **<region_code>** with value of home region
  - Replace **<objectstorage_namespace>** with value returned from **Tenancy Details**<br/>
  ![image](https://user-images.githubusercontent.com/77334410/135297544-0293cb53-aca9-4719-9afa-0a432641748a.png)
  - Replace **<user_name>** with user details used to login
  - Replace **<auth_token>** with the auth token generated under the logged in User details page(by clicking on generate Token)
 ![image](https://user-images.githubusercontent.com/77334410/135300213-53216a53-eeb6-486c-a913-d83998c5e11d.png)
  Example of the replaced command will look like below -
  ```` bash
docker login fra.ocir.io --username=sehubemeaprod/oracleidentitycloudservice/sylvin.amelia@oracle.com --password='WmR2[pn6t8}gz#Zn02#9'![image](https://user-images.githubusercontent.com/77334410/135298489-5196bc6a-8ce3-465e-b803-c51f52cf15b1.png)

````
2. Create a Kubernetes secret by replacing the placeholders in the command below
```` bash
kubectl create secret docker-registry ocirsecret --docker-server=<region_code>.ocir.io  --docker-username='<objectstorage_namespace>/oracleidentitycloudservice/<user_name>' --docker-password='<auth_token>'  --docker-email='<user_name>'
kubectl get secret ocirsecret --output=yaml
````
3. Run docker build and docker push by replacing the placeholders
```` bash
docker build -t <region_code>.ocir.io/<objectstorage_namespace>/mysql_repo/mysql-sample-app:latest . --no-cache=true
docker push <region_code>.ocir.io/<objectstorage_namespace>/mysql_repo/mysql-sample-app:latest

````
4. Create the application pod as follows. Ensure that the image name is correct in msg-api-pod.yaml and deployment.yaml by modifying the existing values
```` bash
kubectl create -f msg-api-pod.yaml
kubectl get pod
````
Wait for the state to change to **Running**<br/>
![image](https://user-images.githubusercontent.com/77334410/135303092-c9207495-0258-41f6-871f-24dabfcaff11.png)

5. Create the kubernetes deployment as follows
```` bash
kubectl apply -f deployment.yaml
````

6. Verify the Load balancer IP by typing the following command
```` bash
kubectl get all
````
Note the external IP of the Load balancer and the port<br/>
![image](https://user-images.githubusercontent.com/77334410/135304471-5f68e220-8099-4b06-b18a-6be28d833973.png)

7. Verify the application by opening the URL in browser http://<LB external IP>:<LB port>/ping <br/>
  Example is http://152.70.181.51:8080/ping <br/>
  ![image](https://user-images.githubusercontent.com/77334410/135304904-4b21785a-9f24-45cd-ad17-a752221f7aad.png)

## Step 6 - Verify the values returned from MySQL
1. Now let us test the application by adding some values in the MySQL database and verify if it is returned by the application
```` bash  
kubectl exec k8s-mysql -it -- bash
mysql --user=root --password=$MYSQL_ROOT_PASSWORD
use k8smysqldb;
show tables;

insert into messages(text) values ('this is the first msg!');
insert into messages(text) values ('this is the second msg!');
````
2. Now verify the application URL opening the URL in browser http://<LB external IP>:<LB port>/msg-api/all
  Example is http://152.70.181.51:8080/msg-api/all <br/>
  
3. The newly added values should be reflected as follows
  ![image](https://user-images.githubusercontent.com/77334410/135305771-415e6e9a-cb5e-48ce-ae68-f7bc1fa3cf84.png)



  







