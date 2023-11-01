Thiis project is based on building an architecture that consist of a react app as frontend,Golang 
serving as the backend and a MongoDB server as our database where changes will be reflected on when 
users make actions.

Prerequisite:
1. Knowledge on Kubernetes 
2. Amazon EKS.
3. Cloud9 or IDE such as VsCode.
4. Golang.

1. The first Step is to setup our cluster. This is where our applications will be deployed.
   - Open your AWS console tab and navigate to the EKS dashboard.
   - Select add cluster and create a cluster with any name of your choice.
   - Choose the latest version of kubernetes.
   - You will need to create an IAM role with the "AmazonEKSClusterPolicy" policy attached to it.
   - leave everything else and click on next, select the default VPC or you can choose to create yours
   - select the number of subnets for your cluster.
   - Also choose the default security group for your clusters network interface.
   - You should have a public endpoint access.
   - leave everything else and move on to review and create your cluster.
   - while you're waiting on the cluster to be created,you can set up the EBS addons,This will help
     our cluster to persist data into EBS volumes.
   - under your newly created cluster scroll down add-ons and find the EBS driver add-on , install it.
     
