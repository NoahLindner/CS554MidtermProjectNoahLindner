Base Project:


Part 1:
Just make the files and stuff


Part 2:

to run compose, you must first download a linux distribution and virtual box.
Before launching the VM, you must go into virtualbox settings, network, advanced, port forwarding
	Then add a new TCP rule host - guest on port 8080 (I also added host IP 127.0.0.1, but that might not be required)
run the VM with the linux ISO
Make sure that the docker is in groups and that you have successfully installed ubuntu (not in vbox settings, but the actual VM)
open a CMD Terminal and run the following:
	sudo apt update
	sudo apt install docker.io
	sudo systemctl start docker
Then you can check to see if it is running with:
	sudo systemctl status docker
Then you need to install the project files from github
extract and make a new folder in the directory called scr
Place server.js into the src folder
You might need to use (depending on your linux distribution):
	sudo apt install docker-compose
	sudo apt install docker-buildx
Build the image:
	sudo docker compose build
Or with:
	sudo docker build -t cs554-midterm .
And to initiate, just use:
	sudo docker compose up
To check whether or not the app is runing on the vm, you can use:
	wget -qO- http://localhost/8080/
And finally to access the app on the host, open a cmd on the host computer and us the command:
	curl http://localhost:8080/
To check the service status, go into the vm and use the terminal command:
	sudo docker compose ps
similarly to check the logs, just go into the vm and use the terminal command:
	sudo docker compose logs
To restart the app and check the request count, go into the vm and use the terminal commands:
	sudo docker compose restart
	wget -qO- http://localhost/8080/

The diagram should be a png file in the github (MAKE THIS LATER)


Part 3:

Instead of running docker kubernetes on the VM we will use the windows host. Download docker desktop and open it. go to the kubernetes tab and create a cluster.
cd to the folder where the deployment.yaml is in the project folder
Use the following commands in your terminal:
	kubectl create deployment cs454-midterm --image=cs454-midterm:latest
	kubectl run cs454-midterm --image cs454-midterm:latest
	kubectl run cs454-midterm2 --image cs454-midterm:latest
	kubectl run cs454-midterm3 --image cs454-midterm:latest
	kubectl expose deployment cs454-midterm --type=ClusterIP --port=80 --target-port=8080 --name=cs454-midterm-service
	kubectl port-forward svc/cs454-midterm-service 8080:80
If you want to connect to the VM, you need to use the following command and make sure that your firewall allows the connection
	kubectl port-forward svc/cs454-midterm-service 8080:80 --address 0.0.0.0
I didn't want to add a new rule to my firewall and have to delete it afterward, so I got proof of the VM being able to access the address and showed proof of the kubernetes on
my host computer.
You must check the connection though this command to prove that you can coomunicate to the host:
	nc -zv 192.168.56.1 8080
You can delete one of the pods with the command in a new cmd:
	kubectl delete pod cs454-midterm
note that a pod name can be found from the docker desktop for whatever name you used.

Kubernetes can have multiple replacement pods to use in place of the current pod in the case that the current pod is deleted or dies. The application is able to continue to exist despite
the possibility of error. This is able to preserve the information of the application, unlike Part 2 where the volitile data is lost when the app is forced to restart.
The service is able to reroute to a new port if it returns "healthy", allowing the app to self correct some problems it may encounter.


Part 4:

Use this command to edit the service selector
	kubectl edit svc cs454-midterm-service
look for: 
	spec:
		selector:
			app: cs454-midterm
and change the app
Then go back to the original terminal window that is still running the "kubectl port-forward svc/cs454-midterm-service 8080:80" command
Back out of it using ctrl+c and run the code again:
	kubectl port-forward svc/cs454-midterm-service 8080:80
It should no longer work at that point. Check the web page, use kubectl get pods, use kubectl describe svc cs454-midterm-service, 
kubectl get endpointslices -l kubernetes.io/service-name=cs454-midterm-service, use kubectl port-forward svc/cs454-midterm-service 8080:80 --v=4,
and use kubectl logs cs454-midterm to record the required info for part 4. (this should be in the video)
Change it back using the same method.

The pods and deployment should still be perfectly healthy and fine, even when the service selector is incorrect, but you still won't have access to the app.
Therefore, the pod indicators are not always going to be a valid way of determening when something is wrong with the system.


writeup:
	Starting off with the resouce choices, I used the most recent ubuntu iso distribution with 4 Gigabytes of RAM. 
My host computer is very slow, as it only has 8 Gigabytes of RAM, So I had to make sure that I didn't use up 
too much of it. In order to connect, one would need to use an SSH key pair with the public key working with the VM
and the private key allowing me to modify work with Docker. I would prefer to use the local network associated 
with my host system, though an external one may also work better. I'd use ports 22 and 8080 (mainly because that's
default for Docker Compose). I looked up Openstack floating IPs and it turns out that it should assign me one on its
own (starting with 172 of 128 using port 8080). I'd be responsible for making the VM, IP, and files, while everything 
else would preferable be provided (If not I'd figure it out). Persistant storage is unecessary considering that there isn't
anything important that the "workload" does other than the single message, status, and extra CS554 requirments. 
	Keystone authenticates the OpenStack environments, Glance is responsible for getting the linux ISO image, Nova handles
the virtual machine used with the ISO, and Neutron connects the networking for the host and VM(s). Docker can be run 
Through the VM with "docker compose up", where Docker Compose runs the application, while OpenStack provides the 
hosting requirements. Cinder could be used to store datain the case that you'd have a more important application service
or just wanted to have the number of requests for some reason. 
	Starting off, a user would connect and make a request via the IP address and port, maybe with their browser 
or a cmd command. the request travels to the virtual machine and Neutron brings it into the application's network 
Security decides what is then done with it (passes to port 8080 as port 22 should be used for SSH) It arrives at Docker and then 
is processed by the server.js file you provided. Then a response is sent back to the user. ICMP doesn't apply here, as it
mainly exists for connectivity issue bug fixes. 
	First, double check to make sure that the container is running (and isn't frozen or crashing). Then, send a test 
request meant just to get a response (like if the program was a calculator, request 1+1). Next, check the port Docker uses
to externalize the app (8080). Just test the port itself to make sure it is taking in information. The IPs are next, Openstack would
have to be the main culprit that this point (check my security rules here too). If everything is fine then it might be that 
everything works locally, but not externally. Last would be to check from another network to see if it was just that instance
or it truely cannot take requests from outside the network.
	OpenStack handles much more that the connectivity of an application. it has to handle Keystone, Glance, Nova, and Neutron, 
data storage and transferance between VMs. It's just much more interconnected, to function more directly for an app. 
An infrastructure problem relates to the connection of the VMs to the system, while an application problem is more confined to 
The server.js app itself. So an incorrect output of the app would be an application problem, as the output still reaches the 
user's VM.
	Compose is good for smaller, less important applications. Kubernetes on the otherhand can handle many difference instances 
Without running into catastrophic errors. By removing any misfunctional instances and stopping them before launch, it is a 
much safer and more cost efficient method.


Additional Requirements:

to fulfill the first part, use this command to directly edit the deployment file
	kubectl edit deployment cs454-midterm
change resources {} into the following (make sure there aren't any tab indentions, only single spaces allowed):

resources:
  requests:
    memory: "256Mi"   # 256 Megabytes
    cpu: "250m"       # 0.25 of a CPU Core
  limits:
    memory: "512Mi"   # 512 Megabytes
    cpu: "500m"       # 0.50 of a CPU Core
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10


use this to check if the file was safely edited:
	kubectl describe deployment cs454-midterm
the service should have been shut down, so go to the first terminal tab and reinstate the following command:
	kubectl port-forward svc/cs454-midterm-service 8080:80

The resource request guarantees that there is a minimum amount of space to run the app, while the limit represents the maximum amount of memory to use.
I believe that the stats I used are standard for simple stuff in Node.js and Python (I just looked it up), so I just used those numbers.
A readiness probe determines if a pod is ready to connect to its network. it pauses and isolates a faulty pod and waits until it recovers before reinstating it.
They are very helpful in the case of temporary problems like traffic bottlenecks and whatnot.

Additional Requirements Experiment:

Use this command to utilize two pods at once:
	kubectl scale deployment cs454-midterm --replicas=2
Using this command, you can attempt to edit a pod:
	kubectl edit pod cs454-midterm
Basically no edits are allowed to running pods so I have no idea why you would suggest the readiness state
create a new unhealthy pod by changing the image (one of the only variable that can be altered in a pod with today's updated docker)
	kubectl run cs454-midterm-experiment --image=cs454-midterm:broken --restart=Never --overrides="{\"spec\":{\"containers\":[{\"name\":\"cs454-midterm-experiment\",\"image\":\"cs454-midterm:broken\",\"ports\":[{\"containerPort\":8080}],\"readinessProbe\":{\"httpGet\":{\"path\":\"/health\",\"port\":8080},\"initialDelaySeconds\":3,\"periodSeconds\":5}}]}}"
Delete all pods except for one safe one and one broken one
	kubectl delete pod cs454-midterm...
check pods and endpoints:
	kubectl get pods
	kubectl get endpointslices -l kubernetes.io/service-name=cs454-midterm-service
Fix the experimental pod by changing the image 
	kubectl set image pod cs454-midterm-experiment cs454-midterm-experiment=cs454-midterm:latest
check pods and endpoints:
	kubectl get pods
	kubectl get endpointslices -l kubernetes.io/service-name=cs454-midterm-service

Additional Requirements 200-300 words:

So, to summarize the additional requirements setup, I added in some resource limits on the top and bottom ends to make sure the app had enough memory and CPU usage 
to perform simple tasks, but not too much to prevent a significant amount of resource abuse / usage. Not only that, but the additional readiness probe added a bit
more safety when starting up new pods. Specifically performing delayed checks on whether or not the pod is safe and "ready" to launch. Afterward I performed the experiment
by creating a simultaneously running and not ready pod. Everywhere I've looked, I could never find any method to edit a currently running pod's readiness values. Thus, 
I have come up with this still somewhat working method. Also it is necessary to mention that I upped the amount pod pods used by the deployment by one as per the 
requirments direct request. I created the new pod with it's image being incorrect, as that is one of the few variables (`spec.containers[*].image`,
`spec.initContainers[*].image`,`spec.activeDeadlineSeconds`,`spec.tolerations`, and `spec.terminationGracePeriodSeconds`) that is able to be altered during runtime.
This was the only method I could think of to fix the pod after it had been created, instead of deteling it and creating a new healthy pod.

The video for the proof is in the github repository!!!!!!!