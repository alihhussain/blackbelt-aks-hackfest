# Dockerizing Applications with Azure Container Registry

## Azure Container Registry (ACR)

We will use Azure COntainer Registry to build our containers from Dockerfiles and also host our images to run in AKS

### Create Azure Container Registry instance

1. In the browser, sign in to the Azure portal at https://portal.azure.com. Your Azure login ID will look something like `odl_user_9294@gbbossteamoutlook.onmicrosoft.com`
2. Click **Resource Groups** and select **fabmedical-sol**
3. Select the registry that was created earlier **fabmedicalregistryv1**
4. Use the existing Resource Group **fabmedical-sol**
5. Open the Azure Cloud Shell

    ![Azure Cloud Shell](img/cloudshell.png "Azure Cloud Shell")

6. The first time Cloud Shell is started will require you to create a storage account. In our lab, you must click `Advanced` and enter an account name and share.

7. Once your cloud shell is started, clone the workshop repo into the cloud shell environment
    ```
    git clone https://github.com/Azure/blackbelt-aks-hackfest.git
    ```

8. In the cloud shell, you are automatically logged into your Azure subscription. ```az login``` is not required.
    
9. Verify your subscription is correctly selected as the default
    ```
    az account list
    ```

10. Find your RG name

    ```
    az group list --output table
    ```
    
    ```
    Name                                     Location    Status
    ---------------------------------------  ----------  ---------
    cloud-shell-storage-westus               westus      Succeeded
    DefaultResourceGroup-EUS                 eastus      Succeeded
    fabmedical-sol                           eastus      Succeeded
    jenkins                                  eastus      Succeeded
    MC_fabmedical-sol_fabmedical-k8s_eastus  eastus      Succeeded

    # copy the name from the results above and set to a variable 
    
    RG_NAME=fabmedical-sol

For the first container, we will be creating a Dockerfile from scratch. For the other containers, the Dockerfiles are provided.

### Web Container

1. Create a Dockerfile

    * Access the cloud shell
    * In the `~/blackbelt-aks-hackfest/app/web` directory, add a file called "Dockerfile"
        * you can launch in in-browser code editor in cloud shell by typing `code .` at the bash prompt

    * Add the following lines and save:

        ```
        FROM node:9.4.0-alpine

        ARG VCS_REF
        ARG BUILD_DATE
        ARG IMAGE_TAG_REF

        ENV GIT_SHA $VCS_REF
        ENV IMAGE_BUILD_DATE $BUILD_DATE
        ENV IMAGE_TAG $IMAGE_TAG_REF

        WORKDIR /usr/src/app
        COPY package*.json ./
        RUN npm install

        COPY . .
        RUN apk --no-cache add curl
        EXPOSE 8080

        CMD [ "npm", "run", "container" ]
        ```

2. Create a container image for the node.js Web app

    From the cloudshell session: 

    ```
    #Ignore the warnings
    
    cd ~/blackbelt-aks-hackfest/app/web
    
    # Set environment variable for ACR Name
    ACR_NAME=<registry-name>
    az acr build --registry $ACR_NAME --image azureworkshop/rating-web:v1 .
    
    ```
    1. Return to the Azure Portal in your browser and validate that the image appears in your Container Registry under the "Repositories" area.
    2. Under tags, you will see "v1" listed.

### API Container

In this step, the Dockerfile has been created for you. 

1. Create a container image for the node.js API app

    ```
    #Ignore the warnings

    cd ~/blackbelt-aks-hackfest/app/api

   az acr build --registry $ACR_NAME --image azureworkshop/rating-api:v1 .
    
    ```
    1. Return to the Azure Portal in your browser and validate that the image appears in your Container Registry under the "Repositories" area.
    2. Under tags, you will see "v1" listed.


### MongoDB Container

1. Create a MongoDB image with data files

    ```
    #Ignore the warnings

    cd ~/blackbelt-aks-hackfest/app/db

    az acr build --registry $ACR_NAME --image azureworkshop/rating-db:v1 .
    
    ```
    1. Return to the Azure Portal in your browser and validate that the image appears in your Container Registry under the "Repositories" area.
    2. Under tags, you will see "v1" listed.



