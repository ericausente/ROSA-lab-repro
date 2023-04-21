# ROSA-lab-repro

ROSA stands for Red Hat OpenShift Service on AWS. It is a fully managed service that allows users to deploy and manage OpenShift clusters on Amazon Web Services (AWS) infrastructure. ROSA provides a scalable, secure, and stable platform for running containerized applications.

NGINX Ingress Controller is a software component that manages incoming traffic to a Kubernetes cluster. It acts as a reverse proxy and load balancer, routing traffic to the appropriate Kubernetes services. The NGINX Ingress Controller is widely used in Kubernetes deployments and provides advanced features such as SSL termination, rate limiting, and content-based routing.

F5 Networks and Red Hat have partnered to provide an enterprise-grade solution for managing Kubernetes ingress with the NGINX Ingress Controller. 

Prerequisites: 
1. Before using the service on AWS, one must have registered an account with RedHat. Log in or register to access product trials and purchase subscriptions. Your account, in combination with an active subscription, provides access to technical support knowledge and the ability to manage users, subscriptions, and certifications.
2. An AWS account
3. An NGINX License


Steps: 

1. Prepare your AWS account
2. Enable ROSA Service in AWS account
ROSA needs to be enabled on your AWS account to work properly. Open the AWS Console to enable ROSA.
When you open the RedHat Openshift Service on the AWS console for the first time, users can see the option “Enable Red Hat Openshift” as shown in the below screenshot.
3. Download and install the ROSA and AWS command line tools (CLI) and add it to your PATH.
See: 
https://access.redhat.com/documentation/en-us/red_hat_openshift_service_on_aws/4/html/rosa_cli/rosa-get-started-cli

```
 Download the latest version of the rosa CLI for your operating system from the Downloads page on OpenShift Cluster Manager.

Extract the rosa binary file from the downloaded archive. The following example extracts the binary from a Linux tar archive:

$ tar xvf rosa-linux.tar.gz

Add rosa to your path. In the following example, the /usr/local/bin directory is included in the path of the user:

$ sudo mv rosa /usr/local/bin/rosa

Verify if the rosa CLI tool is installed correctly by querying the rosa version:

$ rosa version

Example output

1.2.15
Your ROSA CLI is up to date.
```

See:
https://aws.amazon.com/cli/
Note:If you have AWS CLI already installed, you can skip downloading

4. Create the service linked role for the Elastic Load Balancer (ELB)
Your AWS account must have a service-linked role set up to allow ROSA to utilize ELB.

To check if the role exists for your account, run this command in your terminal:
```
aws iam get-role --role-name "AWSServiceRoleForElasticLoadBalancing"
```

If the role doesn't exist, create it by running the following command:
```
aws iam create-service-linked-role --aws-service-name "elasticloadbalancing.amazonaws.com"
```

5. Log in to the ROSA CLI with your Red Hat account token and create AWS account roles and policies
To authenticate / re-authenticate: 

Log in to the ROSA CLI with your Red Hat account token and create AWS account roles and policies:

    To authenticate, go to your Red Hat Cluster Console and click on the "Copy login command" button to get the login command for your cluster.
    Run the login command in your terminal to log in to the ROSA CLI.

6. Verify your credentials and quota
```
rosa verify quota
```

Issues: 
No. of VPCs should be available, so if your policy is only up to 10, and you have 10 used VPCs,  so you have to delete a VPC first (10/10 will not allow us to continue) 
No of EIPs available at least 1 available. 


7. Create a ROSA cluster using the rosa create cluster command. You can specify the cluster name, AWS region, and other options. For example:
```
rosa create cluster --cluster-name rosaericausente --sts --mode auto --yes
```

8. Wait for the cluster creation to complete. This can take several minutes to several hours depending on the size of the cluster.

9. Create an admin user using the rosa create admin command. You can specify the user name, password, and other options. For example:

```
% rosa create admin --cluster=rosaericausente
I: Admin account has been added to cluster 'rosaericausente'.
I: Please securely store this generated password. If you lose this password you can delete and recreate the cluster admin user.
I: To login, run the following command:

   oc login https://api.rosaericausente.t6n0.p1.openshiftapps.com:6443 --username cluster-admin --password 6iKIc-*EPaJ-dnyHo-Xkcn*

I: It may take several minutes for this access to become active.

% oc login https://api.rosaericausente.t6n0.p1.openshiftapps.com:6443 --username cluster-admin --password 6iKIc-*EPaJ-dnyHo-Xkcn*
Login successful.

You have access to 100 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default".
```

10. Verify that the pods are running: 
```
e.ausente@C02DR4L1MD6M .kube % oc get pods -A
NAMESPACE                                          NAME                                                                             READY   STATUS                   RESTARTS       AGE
openshift-addon-operator                           addon-operator-catalog-xzzjt                                                     1/1     Running                  0              2d5h
openshift-addon-operator                           addon-operator-manager-56cc97cf96-kcvsk                                          2/2     Running                  0              2d5h
openshift-addon-operator                           addon-operator-webhooks-ff88cf8c6-87dcg                                          1/1     Running                  0              2d5h
openshift-addon-operator                           addon-operator-webhooks-ff88cf8c6-lzzjn                                          1/1     Running                  0              2d5h
openshift-apiserver-operator                       openshift-apiserver-operator-7fcc4f5979-72nfw                                    1/1     Running                  0              2d5h
openshift-apiserver                                apiserver-5c485dc459-dslgb                                                       2/2     Running                  0              2d5h
openshift-apiserver                                apiserver-5c485dc459-m4z2d                                                       2/2     Running                  0              2d5h
openshift-apiserver                                apiserver-5c485dc459-zs7lz                                                       2/2     Running                  0              2d5h
openshift-authentication-operator                  authentication-operator-559895d8c-f8rl8                                          1/1     Running                  0              2d5h
openshift-authentication                           oauth-openshift-784dc959c8-86ccr                                                 1/1     Running                  0              47h
openshift-authentication                           oauth-openshift-784dc959c8-ch4r4                                                 1/1     Running                  0              47h
openshift-authentication                           oauth-openshift-784dc959c8-xtzzk                                                 1/1     Running                  0              47h
openshift-backplane-managed-scripts                osd-delete-backplane-script-resources-28033962-trlpt                             0/1     Completed                0              13h
openshift-backplane-srep                           osd-delete-ownerrefs-serviceaccounts-28034707-9qcc7                              0/1     Completed                0              85m
openshift-backplane-srep                           osd-delete-ownerrefs-serviceaccounts-28034737-fgpmg                              0/1     Completed                0              55m
openshift-backplane-srep                           osd-delete-ownerrefs-serviceaccounts-28034767-hq575                              0/1     Completed                0              25m


```


### If you have various clusters in AWS, you can use the kubectl config use-context command to switch between different contexts/clusters defined in your kubeconfig file.

First, list the available contexts in your kubeconfig file using the command:
```
kubectl config get-contexts
```
This will output a list of contexts in your kubeconfig file.

To switch to a particular context/cluster, use the command:
```
kubectl config use-context <context-name>
```
Replace <context-name> with the name of the context you want to switch to.

For example, to switch to the k8s-cluster-eric context in your kubeconfig file, use the command:
```
kubectl config use-context arn:aws:eks:ap-southeast-1:150380270330:cluster/k8s-cluster-eric
```
After switching to the new context, you can use kubectl commands to interact with the resources in that cluster.



