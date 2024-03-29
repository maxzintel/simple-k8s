# Deploying (locally) with MiniKube
### Initially built with pods for dev, swapped to deployments for dev/prod.

### Steps:
1. Start minikube (network = public, not internal)
  * `minikube start`
    * `minikube status` to confirm everything is running correctly.
    * `minikube ssh` to get into the cluster.
    * `minikube ip` for ip of the cluster.
2. Apply configuration files to create/provision k8s objects.
  * `kubectl apply -f $file-name$.yml`
    * Here, use `client-deployment.yml` and `client-node-port.yml`
  * How does our deployment manage/monitor the pods it creates after creation?
    * The `spec:selector:matchLabels:component:web` matches the labels in the pod template.
3. Monitor new objects.
  * `kubectl get pods` Show pods in cluster and various attributes of them.
  * `kubectl get deployments` Same as above, for Deployment objects.
  * `kubectl get services` Same as above, for Service objects.
  * If pod is unhealthy, crashlooping, etc...
    * `kubectl run -it $arbitrary-name$ --image $docker-hub-id$/$image-name$`
    * Above command will pull down a new copy of your image, and exec inside it.
    * Idea here is to run commands from the docker-compose.yml
      * Allows for troubleshooting (via output logs) of pods that may otherwise be crashlooping.
4. Access running containers in your browser:
  * Navigate to `$minikube-ip$:$nodePort$`
  * For me, this is `192.168.99.103:31515`
  * If using Docker-For-Desktop rather than minikube, swap out the minikube ip with localhost.

### Updating pods via Declarative Deployments:
#### Updating pods in place.
1. Have a running local cluster (from above).
2. Update config files (aka the desired state of the object)
  * Ex: Change the desired image to be used in the client-pod from `multi-client` to `multi-worker`
3. Update the pods in place via k8s cli
  * `kubectl apply -f client-pod.yml` should output `pod/client-pod configured`
4. Inspect/Verify Updates
  * `kubectl describe $object-type$ $object-name$`
    * `$object-name$` is optional.
    * Ex: `kubectl describe pods client-pod`
  * Output of above will display the container/image we are now running (`multi-worker`).

### Updating images & enabling our Deployments to pull the new image version:
1. Make a change to some existing Docker Hub image.
  * Ex: Change the app title in App.js in the multi-client container to "... v2"
    * Can be found in the s1dequest/multi-docker GitHub project.
2. Rebuild the image with the new changes.
  * `docker build -t $docker-id$/$image-name$:$version-number$`
    * Example here uses `s1dequest/multi-client:v2`.
  * The version tag after the image name allows us to differentiate between versions in our Deployment object.
3. Push changes to Docker Hub
  * `docker push $docker-id$/$image-name$:$version-number$`
4. Use an imperative command to update the image version the deployment should use.
  * AKA: Run a `kubectl` command forcing the deployment ot use the new image version.
  * `kubectl set image $object-type$/$object-name$ $container-name$=$new-image-to-use$`
  * `kubectl set image Deployment/client-deployment client=s1dequest/multi-client:v2`
