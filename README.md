# Origin

This repository is a fork of the IIBv10 and MQv9 docker build repository from https://github.com/DAVEXACOM/IIB-MQ.

# Overview

This repository contains a Dockerfile and some scripts which demonstrate a way in which you might run [IBM Integration Bus](http://www-03.ibm.com/software/products/en/ibm-integration-bus) and IBM MQ in a [Docker](https://www.docker.com/whatisdocker/) container on [IBM Cloud Container Service](https://www.ibm.com/cloud/container-service).

IBM would [welcome feedback](#issues-and-contributions) on what is offered here.

# Before you begin

* [Container registry with namespace configured](https://console.bluemix.net/docs/services/Registry/registry_setup_cli_namespace.html)
* [Basic understanding of Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
* If you do not have a Kubernetes cluster on IBM Cloud Container Service, follow steps 1 and 2 under [Create a Kubernetes cluster](https://console.bluemix.net/docs/tutorials/scalable-webapp-kubernetes.html#deploy-a-scalable-web-application-on-kubernetes) to create a new cluster. You can use an existing cluster if you have one.

# To get started, click this button:
[![Create toolchain](https://console.bluemix.net/devops/graphics/create_toolchain_button.png)](https://console.bluemix.net/devops/setup/deploy/?repository=https%3A//github.com/andrewyuau/IIB-MQ)

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

7. Modify the kubernetes.yml with replacing the **namespace** with your namespace. ![Application Diagram](ReadMeImages/code-snipts.png)  


8. Build the Docker image of the service

   > In the following steps, make sure to replace `<namespace>` with your namespace name and `ng` with the region of your registry.

   ```
   docker build -t registry.ng.bluemix.net/<namespace>/iib-mq:latest .   
   ```
   This takes a couple of hours, go and have a cup of coffee.

9. Push the image to the registry

   ```
   docker push registry.ng.bluemix.net/<namespace>/iib-mq:latest
   ```



# What the image contains

The built image contains a full installation of [IBM Integration Bus for Developers Edition V10.0](https://ibm.biz/iibdevedn) and [IBM MQ Advanced for Developers V9.0](https://www.ibm.com/au-en/marketplace/secure-messaging).  

# Running a container

After building a Docker image from the supplied files, you can [run a container](https://docs.docker.com/userguide/usingdocker/) which will create and start an Integration Node to which you can [deploy](http://www-01.ibm.com/support/knowledgecenter/SSMKHH_10.0.0/com.ibm.etools.mft.doc/af03890_.htm) integration solutions.

In order to run a container from this image, it is necessary to accept the terms of the IBM Integration Bus for Developers license.  This is achieved by specifying the environment variable `LICENSE` equal to `accept` when running the image.  You can also view the license terms by setting this variable to `view`. Failure to set the variable will result in the termination of the container with a usage statement.  You can view the license in a different language by also setting the `LANG` environment variable.

In addition to accepting the license, you can optionally specify an Integration Node name using the `NODENAME` environment variable.

The last important point of configuration when running a container from this image, is port mapping.  The Dockerfile exposes ports `4414` and `7800` for IIB and `1414` and `9883` for MQ by default, for Integration Node administration and Integration Server HTTP traffic respectively.  This means you can run with the `-P` flag to auto map these ports to ports on your host.  Alternatively you can use `-p` to expose and map any ports of your choice.

For example:

~~~
docker run --name myNode -e LICENSE=accept -e NODENAME=MYNODE -P iibv10image -e MQ_QMGR_NAME=MQ1
~~~

This will run a container that creates and starts an Integration Node called `MYNODE` and exposes ports `4414` and `7800` on random ports on the host machine. It also creates a queue manager called `MQ1` and starts this queue manager with some default queues defined.
For more information on configuring MQ please see [README.md](https://github.com/ibm-messaging/mq-docker/blob/master/README.md) for the standalone MQ container.

At this point you can use:
~~~
docker port <container name>
~~~

to see which ports have been mapped then connect to the Node's web user interface as normal (see [verification](# Verifying your container is running correctly) section below).

### Running administration commands

You can run any of the Integration Bus
 commands using one of two methods:

##### Directly in the container

Attach a bash session to your container and execute your commands as you would normally:

~~~
docker exec -it <container name> /bin/bash
~~~

At this point you will be in a shell inside the container and can source `mqsiprofile` and run your commands.

##### Using Docker exec

Use Docker exec to run a non-interactive Bash session that runs any of the Integration Bus commands.  For example:

~~~
docker exec <container name> /bin/bash -c mqsilist
~~~

### Accessing logs

This image also configures syslog, so when you run a container, your node will be outputting messages to /var/log/syslog inside the container.  You can access this by attaching a bash session as described above or by using docker exec.  For example:

~~~
docker exec <container id> tail -f /var/log/syslog
~~~

# Verifying your container is running correctly

Whether you are using the image as provided or if you have customised it, here are a few basic steps that will give you confidence your image has been created properly:

1. Run a container, making sure to expose port 4414, 1414 and 9443 to the host - the container should start without error
2. Run mqsilist to show the status of your node as described above - your node should be listed as running
3. Access syslog as descried above - there should be no errors
4. Connect a browser to your host on the port you exposed in step 1 - the Integration Bus web user interface should be displayed.
5. Connect to a browser and connect via HTTPS to port 9443 to run the MQ administration console.

At this point, your container is running and you can [deploy](http://www-01.ibm.com/support/knowledgecenter/SSMKHH_10.0.0/com.ibm.etools.mft.doc/af03890_.htm) integration solutions to it using any of the supported methods.



# License

The Dockerfile and associated scripts are licensed under the [Eclipse Public License 1.0](./LICENSE). IBM Integration Bus for Developers is licensed under the IBM International License Agreement for Non-Warranted Programs. This license may be viewed from the image using the `LICENSE=view` environment variable as described above. Note that this license does not permit further distribution.


IBM MQ Advanced for Developers is licensed under the IBM International License Agreement for Non-Warranted Programs. This license may be viewed from the image using the LICENSE=view environment variable as described above or may be found online. Note that this license does not permit further distribution.
