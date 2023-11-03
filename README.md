Thiis project is based on building an architecture that consist of a react app as frontend,Golang 
serving as the backend and a MongoDB server as our database where changes will be reflected on when 
users make actions. All applications are containerized because this is a microservice architecture, having
such an architecture will help the software development team in managing and maintaining the application easily.

We will be creating a mongodb as a statefulset. Well statefulset is a higher-level resource used to manage stateful
applications, such as databases or distributed systems, where each pod has a unique identity and stable network
identity. StatefulSets are designed to maintain the ordering and uniqueness of pods and 
provide stable network hostnames for these pods. We will also be creating secrets. Secrets in kubernetes are mainly
used to store secret credentials or entities such as access id,passwoards etc. We passed in the credentials for the 
database into secret to help authenticate access into our db.

We created services for accessing our application pods. A cluster type service and two loadbalancers services for 
our frontend and backend. We used a cluster type service for our db because we only want our backend to make calls 
to our db and nothing else,so its only accessible to resources within our cluster while our loadbalancers type will 
be accessibleto the outside world. Creating a loadBalancer type will automatically create elastic loadbalancers in our
AWS EC2 console to help direct and control traffics between the pods in our deployment.

Prerequisite:
1. Knowledge on Kubernetes 
2. Amazon EKS.
3. Cloud9 or IDE such as VsCode.
 
The first Step is to setup our cluster. This is where our applications will be deployed.
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
   - setup your cloud9 workspace, the cloud9 workspace is an IDE that can be attached to an amazon
     instance, we will install the kubectl on the workspace which is a command utility tool for kubernetes. 
     It helps us pass commands like "kubectl get pods" etc.
   - We need to attach certain polcies to the role of our ec2 instance of our IDE. The policy file can be 
 ]   found in this directory called eks-policy.JSON. This policy gives the ec2 instance attached to our 
     cloud9 IDE permissions to be able to carry out certain commands on cluster.
   - install kubectl:
   -    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.11/2023-03-17/bin/linux/amd64/kubectl
        chmod +x ./kubectl
        sudo cp ./kubectl /usr/local/bin
        export PATH=/usr/local/bin:$PATH

   - Since we will be using cloud9 we won't need to install AWS_CLI because it's already preconfigured 
     when we initialised the cloud9 IDE.
   - next we create our node group with any name of your choice and attach an EKS node role to the node
     group which has the following polices:
        AmazonEC2ContainerRegistryReadOnly
        AmazonEKS_CNI_Policy
        AmazonEKSWorkerNodePolicy
        AmazonEBSCSIDriverPolicy.
   - After creating a node group we can then use our instance which we have installed kubectl and aws-CLI
     to access our cluster. Doing so we will have to add EKS context to our machine using this command:
          aws eks update-kubeconfig --name cluster-1 --region us-east-1.
  
   - create an access key with the IAM user you are using then run "aws configure" on your machine for accessing
     your cluster. After this test your kubectl with:

               kubectl get pods -A


   you should be able to see custom resources already created.
   
   - Now that we have our EKS all setup, we can now create our resources using manifests.
   - create a ns called voting-app using:
     
                         kubectl create ns cloudchamp
          kubectl config set-context --current --namespace cloudchamp
   
   After creating ns,create a directory called manifests where all your files will be 
   stored. all manifests are in this repo in the manifests directory.
   
Resources that will be created:
1. a frontend app in a deployment file with two pods.
2. a database app in a statefulset with three replicas.
3. a secret for accessing our mongodb service.
4. A backend golang API app in a deployment with two pods.
5. A cluster svc for our mongodb Database,as we only want our api to make calls within our cluster.

cd into the manifests directory and run:

          kubectl create -f .
          
After creating it, we will have to get into any of our Mongodb pods to create a database and 
add values to the database. run:

          kubectl exec -it mongo-0 -n cloudchamp -- mongo 
          
When you're in the mongodb terminal run:

               rs.initiate();
               sleep(2000);
               rs.add("mongo-1.mongo:27017");
               sleep(2000);
               rs.add("mongo-2.mongo:27017");
               sleep(2000);
               cfg = rs.conf();
               cfg.members[0].host = "mongo-0.mongo:27017";
               rs.reconfig(cfg, {force: true});
               sleep(5000);
Next run:

     use langdb;
     db.languages.insert({"name" : "csharp", "codedetail" : { "usecase" : "system, web, server-side", "rank" : 5, "compiled" : false, "homepage" : "https://dotnet.microsoft.com/learn/csharp", "download" : "https://dotnet.microsoft.com/download/", "votes" : 0}});
     db.languages.insert({"name" : "python", "codedetail" : { "usecase" : "system, web, server-side", "rank" : 3, "script" : false, "homepage" : "https://www.python.org/", "download" : "https://www.python.org/downloads/", "votes" : 0}});
     db.languages.insert({"name" : "javascript", "codedetail" : { "usecase" : "web, client-side", "rank" : 7, "script" : false, "homepage" : "https://en.wikipedia.org/wiki/JavaScript", "download" : "n/a", "votes" : 0}});
     db.languages.insert({"name" : "go", "codedetail" : { "usecase" : "system, web, server-side", "rank" : 12, "compiled" : true, "homepage" : "https://golang.org", "download" : "https://golang.org/dl/", "votes" : 0}});
     db.languages.insert({"name" : "java", "codedetail" : { "usecase" : "system, web, server-side", "rank" : 1, "compiled" : true, "homepage" : "https://www.java.com/en/", "download" : "https://www.java.com/en/download/", "votes" : 0}});
     db.languages.insert({"name" : "nodejs", "codedetail" : { "usecase" : "system, web, server-side", "rank" : 20, "script" : false, "homepage" : "https://nodejs.org/en/", "download" : "https://nodejs.org/en/download/", "votes" : 0}});

     db.languages.find().pretty();
     
Since we have all our pods running next we expose our frontend and backend pods by creating services for them using imperative commands.

For frontend:

     kubectl expose deploy frontend \
     --name=frontend \
     --type=LoadBalancer \
     --port=80 \
     --target-port=8080
     
For API:

     kubectl expose deploy api \
     --name=api \
     --type=LoadBalancer \
     --port=80 \
     --target-port=8080
     
Test the connections: it should return an OK status:

     {
          API_ELB_PUBLIC_FQDN=$(kubectl get svc api -ojsonpath="{.status.loadBalancer.ingress[0].hostname}")
          until nslookup $API_ELB_PUBLIC_FQDN >/dev/null 2>&1; do sleep 2 && echo waiting for DNS to propagate...; done
          curl $API_ELB_PUBLIC_FQDN/ok
          echo
     }


     {
          API_ELB_PUBLIC_FQDN=$(kubectl get svc api -ojsonpath="{.status.loadBalancer.ingress[0].hostname}")
          echo API_ELB_PUBLIC_FQDN=$API_ELB_PUBLIC_FQDN
     }

Get the DNS for your frontend Application:

     echo http://$FRONTEND_ELB_PUBLIC_FQDN
That's it. You can now reach the applications frontend by copying the link after you run the code above.