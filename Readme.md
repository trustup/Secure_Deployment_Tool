# Secure Deplyment Tool - Overview

The purpose of the Secure Deployment tool is to offer a secure solution for containerized applications by using Hardware-Assisted Security to protect them at runtime. This protection is achieved by executing computations in a Trusted Execution Environment (TEE) based on hardware, thus preventing unauthorized access to data rather than transforming or dispersing it.

![](/image/Architettura_SD.png)

Exactly for the tool we expolit the TEE technology of Intel SGX. 

## **SGX Runtime**
To secure applications with the SGX TEE we use LibOS-based Runtimes. 

There are some different LibOS, in this proof of concept we’re going to use Gramine; especially, using Gramine Shielded Containers (GSC) we provide the infrastructure to deploy Docker containers protected by Intel SGX enclaves. This technology transforms a Docker image into a new image which includes the Gramine LibOS, manifest files, Intel SGX related information, and executes the application inside an Intel SGX enclave using Gramine. 


### **GSC repository e documentation**
The official GSC git repository can be found at: https://github.com/gramineproject/gsc

The official GSC documentation can be found at: https://gramine-gsc.readthedocs.io

# Tool's operability
## Prerequisites

For the tool's operability we need: 
1) Docker image repository - Docker Hub
2) Jenkins server
3) SGX-enabled VM 
4) GSC on the VM

![](/image/Architettura.png)


## Jenkins UI
We use Jenkins as a UI to make the tool more user-friendly, in fact, thanks to Jenkins, the user can simply choose from all the available images on the Docker Hub repository which image to deploy, also selecting the version; moreover, and most importantly, he can decide whether to deploy a secured container or not.

![](/image/jenkins_job.png)

### SGX-secured_container
For the deployment of the SGX-secured container the user needs to provide Jenkins with a configuration file that allows to take advantage of this technology, the Manifest file. In this repository there is a simply example of [Manifest file](/Manifestfile.manifest)

The official manifest syntax documentation can be found at: https://gramine.readthedocs.io/en/latest/manifest-syntax.html


