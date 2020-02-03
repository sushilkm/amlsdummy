# Simple Model
<sup> Daniel Grecoe - A Microsoft Employee</sup>

This code creates a simple Azure Machine Learning web service. 

The code can be run on Windows or Linux, but if run on Windows you cannot test the docker container image locally. That is the only restriction.

The project creates a no-op ML service that simply returns a string message to the user. That is, the ML model backing it is a NO-OP. It was created simply to go through all of the motions of creating a model, registering it, creating a container, creating an AKS compute cluster, and finally, deploying it as a REST endpoing. 

You will need
- Azure Subscription 
- Azure Machine Learning compute cores
- Windows or Linux box to run the project
    - If running on Windows you will not be able to test the docker container locally as the image is Linux based. However, you'll still be able to create and deploy a service.

#### Linux only
To test the container image you have to perform the following commands from the bash shell.

```
sudo usermod -aG docker $USER
newgrp docker
```

<b>NOTE</b>: Re-running the code over and over will not produce anything outside of the original scope. At each step before the model, container image, aks service, or REST endpoint is created, the Azure Machine Learning workspace is scanned for the item. If it exists, no new service is created. 

### API Input
This example produces a REST API with the expected input:

|||
|----|----|
|URL| URL of the web service|
|API KEY| As bearer token in request headers|
|BODY|Simple JSON body:
```
{
    "name" : "Any name or string"
}
```

The API returns a simple JSON structure in the form:
```
{
    "GoAway" : "[name]'s not here."
}
```
Where [name] is the input value. 

### Execution Steps
1. Clone this repo to your machine
2. Create the conda environment
    - conda env create -f environment.yml
    - conda activate SimpleModel
3. Modify whatever settings you need in
    - scripts\general_utils.py :: loadArguments()
    - Static class definitions in run.py :: class Context
4. Run run.py to create the solution
5. When done, delete the resource group identified in the program arguments
    - scripts\general_utils.py :: loadArguments()

## Run.py Steps
1. Create the context object
2. Generate an Azure Machine Learning Service Workspace
    - If workspace already exists by name in the sub with the provided resource group, that workspace it will be added to the Context object. 
    - If it doesn't exist it is created and added to the Context object.
3. Generate the AMLS Experiment
    - If the experiment exists in the workspace by name, it will be added to the Context object.
    - If the experiment doesn't exist it is created and added to the Context object..
4. Create a model
    -This is really a no-op model. A pkl file is generated from nothing useful, but the model file is required to create a container.
    - If the model already exists by name it will be added to the Context object
    - If it doesn't exist, a simple pkl file is generated and registered as a new model. The result is added to the Context object.
5. Create a conatiner image. 
    - Attempts to load a container image associated with the current workspace. 
        - If found, it is added to the Context object
        - If not found, creates a new container image and adds it to the Context object.
            - It is ok to just create new images, it just updates the version in the ACR
            - Once the image is created, attempts to test the container image locally. However this is a Linux based image so if the current system is not Linux test will not run. 
6. Creates or attaches a compute target (in this case AKS)
    - If you have a cluster alreayd, use the function Context.generateComputeTarget by supplying the cluster name and resource group. The existing cluster is then added to the Context object.
    - If you need a new cluster, leave those fields blank and:
        - If a compute with the same name is already associated with the workspace, that compute is added to the Context object.
        - If a compute doesn't exist, it is created and added to the Context object.
7. Create the web service to serve up the REST api endpoint.
    - If a service already exists in the workspace for the expected container image name, that service is added to the Context object.
    - If the service doessn't exist, a new one is created and added to the Context object.
8. The web service is tested and the result is printed to the console along with the connection info for the service, i.e. URI and KEY. When this succeeds, you can take that connection info and use it elsewhere. 
    - If you don't record the API information, you can simply re-run this script and it will be collected for you without creating any new objects/resources (assuming you have not changed the configuration)