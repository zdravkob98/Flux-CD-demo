# Flux-CD
Git repository (GitHub) used to run the Flux CD operators: - for a kubernetes cluster I am using Minikube
-----------------------------------------------------------------------------------------------------------

Minikube
=========


Module 1
	1. Cluster up and running
		1.1. Check that it is properly installed
			- minikube version
		1.2. Start the cluster
			- minikube start

	2. Cluster version
		2.1. To check if kubectl is installed you can run the kubectl version command
			-kubectl version

	3. Cluster details
		3.1. Let’s view the cluster details.
			-kubectl cluster-info
		3.2. To view the nodes in the cluster
			-kubectl get nodes


Module 2 - Deploy an app
	1. Kubectl basics
		1.1. Check that kubectl is configured to talk to your cluster
			- kubectl version
		1.2. To view the nodes in the cluster
			-kubectl get nodes

	2. Deploy our app
		2.1. kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1 (For example)
		2.2. To list your deployments
			- kubectl get deployments

	3. View our app
		3.1. The command can create a proxy that will forward communications into the cluster-wide, private network. 
			The proxy enables direct access to the API from these terminals.
			- kubectl proxy
		3.2. You can see all those APIs hosted through the proxy endpoint. For example, we can query the version directly through the API using the curl command.
			- http://localhost:8001/version
	
		- The API server will automatically create an endpoint for each pod, based on the pod name, that is also accessible through the proxy.

		3.3. First we need to get the Pod name
			- kubectl get pod - For example (kubernetes-bootcamp-57978f5f5d-qkjqb)
		3.4. You can access the Pod through the API by running:
			POD_NAME = kubernetes-bootcamp-57978f5f5d-qkjqb
			- curl http://localhost:8001/api/v1/namespaces/default/pods/{POD_NAME}/


Module 3 
	Step 1: Check application configuration
		1.1. Let’s verify that the application we deployed is running. 
			- kubectl get pods
		1.2. Next, to view what containers are inside that Pod and what images are used to build those containers we run the describe pods command.
			-

	Step 2: Show the app in the terminal
		2.1. Run again in second terminal
			-
		2.2. To see the output of our application, run a curl request.
			POD_NAME = kubernetes-bootcamp-57978f5f5d-qkjqb
			- curl http://localhost:8001/api/v1/namespaces/default/pods/{POD_NAME}/proxy/
			The url is the route to the API of the Pod.

	Step 3: View the container logs
		3.1. Anything that the application would normally send to STDOUT becomes logs for the container within the Pod.
			POD_NAME = kubernetes-bootcamp-57978f5f5d-qkjqb
			- kubectl logs {POD_NAME}
			We can retrieve these logs using the kubectl logs command
            
	Step 4: Executing command on the container. We can execute commands directly on the container once the Pod is up and running.
        For this, we use the exec command and use the  name of the Pod as a parameter.
		4.1. Let’s list the environment variables
			- kubectl exec kubernetes-bootcamp-57978f5f5d-qkjqb -- env
		4.2. Start a bash session in the Pod’s container
			- kubectl exec -ti kubernetes-bootcamp-57978f5f5d-qkjqb -- bash
		4.3. We have now an open console on the container where we run our NodeJS application. The source code of the app is in the server.js file
			- cat server.js
		4.4.  Check that the application is up
			- curl localhost:8080
		4.5. To close your container connection type exit
			- exit


Module 4: Using a Service to Expose Your App
	A Kubernetes Service is an abstraction layer which defines a logical set of Pods and enables external traffic exposure, load balancing and service discovery for those Pods.
	
	Step 1: Create a new service
		1.1. Let's verify that the application is still running
			- kubectl get pods
		1.2. List the current Services from our cluster
			- kubectl get services
		1.3. To create a new service and expose it to external traffic we’ll use the expose command with NodePort as parameter
			- kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
		1.4. Now we'll see the kubernetes-bootcamp with IP and Port
			-kubectl get services
		1.5. To find out what port was opened externally (by the NodePort option) we’ll run
			- kubectl describe services/kubernetes-bootcamp
		1.6. We can see the value of theNode port -  31379 (For example)
		1.7. Now we can test that the app is exposed outside of the cluster using curl, the IP of the Node and the externally exposed port
			- curl $(minikube ip):31379

	Step 2: Using labels
		2.1. The Deployment created automatically a label for our Pod. With describe deployment command you can see the name of the label
			- kubectl describe deployment
		2.2. Let’s use this label to query our list of Pods. We’ll use the kubectl get pods command with -l as a parameter, followed by the label values
			- kubectl get pods -l app=kubernetes-bootcamp
		2.3. You can do the same to list the existing services
			- kubectl get services -l app=kubernetes-bootcamp
		2.4. To apply a new label we use the label command followed by the object type, object name and the new label
			- kubectl label pods $POD_NAME version=v1
		2.5. We can check it with the describe pod command
			-kubectl describe pods kubernetes-bootcamp-57978f5f5d-qkjqb
		2.6. And we can query now the list of pods using the new label
			- kubectl get pods -l version=v1

	Step 3: Deleting a service
		3.1. To delete Services you can use the delete service command. Labels can be used also here
			- kubectl delete service -l app=kubernetes-bootcamp
		3.2. Confirm that the service is gone
			- kubectl get services
		3.3. This proves that the app is not reachable anymore from outside of the cluster.
			- kubectl exec -ti kubernetes-bootcamp-57978f5f5d-qkjqb -- curl localhost:8080
			You can confirm that the app is still running with a curl inside the pod:


Module 5: Skaling
	Step 1: Scaling a deployment
		1.1. List your deployments
			- kubectl get deployments
		1.2. To see the ReplicaSet created by the Deployment
			- kubectl get rs
		1.3. Scale the Deployment to 4 replicas
			- kubectl scale deployments/kubernetes-bootcamp --replicas=4
		1.4. List your Deployments once again
			- kubectl get deployments
		1.5. Check if the number of Pods changed
			- kubectl get pods -o wide
		1.6. The change was registered in the Deployment events log. To check that, use the describe command
			- kubectl describe deployments/kubernetes-bootcamp

	Step 2: Load Balancing
		1.1. kubectl describe services/kubernetes-bootcamp - check that the Service is load-balancing the traffic.
			To find out the exposed IP and Port we can use the describe service
		1.2. kubectl get pods -o wide - We hit a different Pod with every request. This demonstrates that the load-balancing is working.

	Step 3: Scale Down
		1.1. To scale down the Service to 2 replicas, run again the scale command
			- kubectl scale deployments/kubernetes-bootcamp --replicas=2
		1.2. List the Deployments to check if the change was applied
			- kubectl get deployments
		1.3. The number of replicas decreased to 2. List the number of Pods
			- kubectl get pods -o wide


Pods - When we create a Deployment on Kubernetes, that Deployment creates Pods with containers inside them.
    A Pod is a Kubernetes abstraction that represents a group of one or more application containers (such as Docker),
    and some shared resources for those containers. Those resources include:
	-Shared storage, as Volumes
	-Networking, as a unique cluster IP address
	-Information about how to run each container, such as the container image version or specific ports to use

Nodes - A Pod always runs on a Node. A Node is a worker machine in Kubernetes and may be either a virtual or a physical machine,
    depending on the cluster. Each Node is managed by the control plane.
	A Node can have multiple pods, and the Kubernetes control plane automatically handles scheduling the pods across the Nodes in the cluster.
	The control plane's automatic scheduling takes into account the available resources on each Node.
	A node is a worker machine in Kubernetes and may be a VM or physical machine, depending on the cluster. Multiple Pods can run on one Node.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

FLUX
=======

1. Install (For windows 10) -> CHOCOLATEY -> choco install flux

2. Export your credentials
	Export your GitHub personal access token and username:
	- export GITHUB_TOKEN=<your-token>
	- export GITHUB_USER=<your-username>

3. Check your Kubernetes cluster
	- flux check --pre

4. Install Flux onto your cluster 
	flux bootstrap github \
  	--owner=$GITHUB_USER \
  	--repository=Flux-CD-demo \
  	--branch=main \
  	--path=./clusters/my-cluster \
  	--personal
	
	The bootstrap command above does following:
		- Creates a git repository Flux-CD (For example) on your GitHub account
		- Adds Flux component manifests to the repository
		- Deploys Flux Components to your Kubernetes Cluster
		- Configures Flux components to track the path /clusters/my-cluster/ in the repository


5. Add podinfo  repository to Flux
	This example uses a public repository github.com/stefanprodan/podinfo, podinfo is a tiny web application made with Go.
	Create a GitRepository manifest pointing to podinfo repository’s master branch:

	flux create source git podinfo \
        --url=https://github.com/stefanprodan/podinfo \
        --branch=master \
        --interval=30s \
        --export > ./clusters/my-cluster/podinfo-source.yaml
	git add -A && git commit -m "Add podinfo GitRepository" && git push

6. Deploy github-battle application
	Configure Flux to build and apply the kustomize directory located in the github-battle repository.
	Use the flux create command to create a Kustomization that applies the github-battle deployment.

	flux create kustomization github-battle \
	  --target-namespace=default \
	  --source=github-battle \
	  --path="./kustomize" \
	  --prune=true \
	  --interval=5m \
	  --export > ./clusters/my-cluster/github-battle-kustomization.yaml

	git add -A && git commit -m "Add github-battle Kustomization"
	git push

7. Watch Flux sync the application
	Use the flux get command to watch the github-battle app

	flux get kustomizations --watch

8. Check github-battle has been deployed on your cluster:

	kubectl -n default get deployments,services

	Changes made to the github-battle Kubernetes manifests in the master branch are reflected in your cluster.

	When a Kubernetes manifest is removed from the github-battle repository, Flux removes it from your cluster.
	When you delete a Kustomization from the fleet-infra repository,
	Flux removes all Kubernetes objects previously applied from that Kustomization.
