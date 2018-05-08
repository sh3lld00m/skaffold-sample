# Instructions

If you wish, you can open this tutorial in Cloud Shell:

[![Open in Cloud Shell](http://gstatic.com/cloudssh/images/open-btn.svg)](https://console.cloud.google.com/cloudshell/open?git_repo=https%3A%2F%2Fgithub.com%2Fsh3lld00m%2Fskaffold-sample.git&page=editor&working_dir=skaffold-sample&tutorial=Readme.md)


## Installation and cluster creation
1. Create a Kubernetes Engine cluster:
```bash
gcloud container clusters create skaffold --zone europe-west1-d
```

2. Install Skaffold
```bash
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64
chmod +x skaffold
```
```bash
mkdir ~/bin
export PATH=$PATH:~/bin
sudo mv skaffold ~/bin
```

## Continuous development

We will use a simple Go application that logs a statement every second.

As As a new developer on-boarding you need to start Skaffold in dev mode to
begin iterating on the application and seeing the updates happen in real time. 
The development team working on the application has already setup the Dockerfile,
Kubernetes manifests, and Skaffold manifest necessary to get you started.

1. 	First, change the references in ```skaffold.yaml``` and ```k8s-pod.yaml``` to point to your
	Container Registry:
	```bash
	export PROJECT=$(gcloud info --format='value(config.project)')
	sed -i -e s#k8s-skaffold#${PROJECT}#g skaffold.yaml
	sed -i -e s#k8s-skaffold#${PROJECT}#g k8s-pod.yaml
	```

2. 	Take a look at the different files in the directory.
 	
 	- 	Start with ```skafold.yaml```. You'll notice the different steps (build, deploy)
 		and a specific section for storing profiles. There, there's a profile named ```gcb```
 		that will be using Google Container Builder to build and push your image. The deploy
 		section is configured to use the ```kubectl``` command to apply the Kubernetes
		manifests in our directory (a glob specifying all files starting with ```k8s-```).
		
	-	The ```Dockerfile``` file contains a multistep build of a container, where we're building the Go
  		application and baking it in an Alpine container.
  		
  	- 	File ```k8s-pod.yaml``` is just the pod manifest for the app we just baked. This will be deployed in
  		kubernetes cluster you created previously.
  
3.	Run Skaffold in ```dev``` mode with the ```gcb``` profile enabled.
	This will use Container Builder to build a new image from the local source code,
	push it to your Container Registry and then deploy your application
	to your Kubernetes Engine cluster.

	```bash
	skaffold dev -p gcb
	```

4.	You will see the application's logs printing to the screen.

	```
	Starting deploy...
	Deploying k8s-pod.yaml...
	Deploy complete.
	[getting-started getting-started] Hello world!
	[getting-started getting-started] Hello world!
	[getting-started getting-started] Hello world!
	```

5.	Open the ```main.go``` file with the Cloud Shell Editor:

	```bash
	cloudshell edit main.go
	
	```

6.	The Cloud Shell editor should be opened above your terminal window. Edit the `Hello World`
	message to say something different. Your change will be saved automatically by the editor.
	Once the save is complete Skaffold will detect that a file has been changed and then
	rebuild, repush and redeploy the change. You will see your new log line now streaming back
	from the Kubernetes cluster.