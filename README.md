# Origin

This repository is a fork of the IIBv10 and MQv9 docker build repository from https://github.com/DAVEXACOM/IIB-MQ.

# Overview

This repository contains a Dockerfile and some scripts which demonstrate a way in which you might run [IBM Integration Bus](http://www-03.ibm.com/software/products/en/ibm-integration-bus) and IBM MQ in a [Docker](https://www.docker.com/whatisdocker/) container on [IBM Cloud Container Service](https://www.ibm.com/cloud/container-service).

IBM would [welcome feedback](#issues-and-contributions) on what is offered here.

# Before you begin

* [Container registry with namespace configured](https://console.bluemix.net/docs/services/Registry/registry_setup_cli_namespace.html)
* If you do not have a Kubernetes cluster on IBM Cloud Container Service, follow steps 1 and 2 under [Create a Kubernetes cluster](https://console.bluemix.net/docs/tutorials/scalable-webapp-kubernetes.html#deploy-a-scalable-web-application-on-kubernetes) to create a new cluster. You can use an existing cluster if you have one.
* Create a Platform API key by following the tnstructions [here](https://www.ibm.com/blogs/bluemix/2017/10/introducing-ibm-cloud-iam-service-ids-api-keys/).

# To create the IIB-MQ container automatically using IBM Cloud Continuous Delivery, click this button
[![Create toolchain](https://console.bluemix.net/devops/graphics/create_toolchain_button.png)](https://console.bluemix.net/devops/setup/deploy/?repository=https%3A//github.com/andrewyuau/IIB-MQ)

1. Select the region and organization of your Continuous Delivery service if you have one, otherwise it will be created automatically for you.
![toolchain1](images/toolchain1.jpeg)

2. You can leave the settings for Git Repos and Issue Tracking and Eclipse Orion Web IDE as default.
![toolchain2](images/toolchain2.jpeg)

3. For Delivery Pipeline, rename the app name to something easy to remember. Put in the API key created earlier. You should then be able to select your Container registry region and namespace. Select your Cluster region and name, then type in any Cluster namespace, eg. prod. Once finished, click 'Create'.
![toolchain3](images/toolchain3.jpeg)

4. You should see a message saying that our toolchain is ready. Click on the Delivery Pipeline.
![toolchain4](images/toolchain4.jpeg)

5. You should see the Build stage running now. It will take some time to build the Docker image if it is your first time.
![toolchain5](images/toolchain5.jpeg)

6. Lastly, your PROD stage will pass. Click on 'View logs and history' in the PROD section.
![toolchain6](images/toolchain6.jpeg)

7. Click on 'Deploy Helm chart' to view the logs for the deploy job.
![toolchain7](images/toolchain7.jpeg)

8. Scroll to the bottom and click on the links to access your MQ (admin/passw0rd) and IIB consoles. Congratulations!
![toolchain8](images/toolchain8.jpeg)

# To create the IIB-MQ container manually, follow the instructions below

# Configure kubectl and helm

Follow instructions under [Configure kubectl and helm](https://console.bluemix.net/docs/tutorials/scalable-webapp-kubernetes.html#deploy-a-scalable-web-application-on-kubernetes) to configure command line tools [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) and [helm](https://helm.sh/).

# Building the image

1. Clone the app to your local environment from your terminal using the following command

    ```
      git clone https://github.com/andrewyuau/IIB-MQ.git
    ```

2. `cd` into the `IIB-MQ` folder that you cloned

    ```
      cd IIB-MQ
    ```

3. Start the Docker engine on your local computer
   > See the [Docker installation instructions](https://docs.docker.com/engine/installation/) if you don't yet have the Docker engine installed locally or need help in starting it.

4. Log the local Docker client in to IBM Bluemix Container Registry:

   ```
   bx cr login
   ```

   > This will configure your local Docker client with the right credentials to be able to push images to the Bluemix Container Registry.

5. Retrieve the name of the namespace you are going to use to push your Docker images:

   ```
   bx cr namespace-list
   ```

   > If you don't have a namespace, you can create one with `bx cr namespace-add mynamespace` as example.

6. Check that you have installed **Container Registry plugin** and **Container Service plugin** with this command
    ```
    bx plugin list
    ```
    > Output:   
          Listing installed plug-ins...           
          Plugin Name          Version   
          cloud-functions      1.0.7   
          container-registry   0.1.292   
          container-service    0.1.462   
          dev                  1.1.4   
          schematics           1.3.4   
          sdk-gen              0.1.9   
          IBM-Containers       1.0.1028 

7. Build the Docker image of the service

   > In the following steps, make sure to replace `<namespace>` with your namespace name and `ng` with the region of your registry.

   ```
   docker build -t registry.ng.bluemix.net/<namespace>/iib-mq:latest .   
   ```
   This takes a couple of hours, go and have a cup of coffee.

8. Push the image to the registry

   ```
   docker push registry.ng.bluemix.net/<namespace>/iib-mq:latest
   ```



# What the image contains

The built image contains a full installation of [IBM Integration Bus for Developers Edition V10.0](https://ibm.biz/iibdevedn) and [IBM MQ Advanced for Developers V9.0](https://www.ibm.com/au-en/marketplace/secure-messaging).  

# Deploy image using Helm chart

Follow instructions at [Helm chart readme](https://github.com/andrewyuau/IIB-MQ/tree/master/chart/iibmq).

# License

The Dockerfile and associated scripts are licensed under the [Eclipse Public License 1.0](./LICENSE). IBM Integration Bus for Developers is licensed under the IBM International License Agreement for Non-Warranted Programs. This license may be viewed from the image using the `LICENSE=view` environment variable as described above. Note that this license does not permit further distribution.


IBM MQ Advanced for Developers is licensed under the IBM International License Agreement for Non-Warranted Programs. This license may be viewed from the image using the LICENSE=view environment variable as described above or may be found online. Note that this license does not permit further distribution.
