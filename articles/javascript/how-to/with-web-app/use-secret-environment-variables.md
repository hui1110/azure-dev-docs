---
title: Use Azure Key Vault secrets in Express.js app
description: Store secrets in Azure Key Vault, then pull in those secrets programmatically from Key Vault to the Express.js app. 
ms.topic: how-to
ms.date: 03/30/2021
ms.custom: seo-javascript-september2019, devx-track-js, devx-track-azurecli
#intent: Show a customer how to create a key vault resource, add a key, secret, and certificate, then use those in an Express.js app. 
---

# Use Azure Key Vault secrets in Express.js app

Store secrets in Azure Key Vault, then use those secrets programmatically from Key Vault in your Express.js app. 

* [Sample code](https://github.com/Azure-Samples/js-e2e-express-mongodb/tree/keyvault)

## Prepare your development environment

1. Complete the [Express.js with Cosmos DB tutorial](/azure/app-service/tutorial-nodejs-mongodb-app?tabs=azure-portal%2Cterminal-bash%2Cvscode-deploy%2Cdeploy-instructions-azportal%2Cdeploy-zip-linux-mac%2Cdeploy-instructions--zip-azcli). 

    When you complete the previous tutorial, you should have an Express.js app using a Cosmos DB database deployed to an Azure web app. 

1. Make sure the following are installed on your local developer workstation:

    - An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
    - Azure resource group already created in previous tutorial.
    - Azure resources already created in [previous tutorial](/azure/app-service/tutorial-nodejs-mongodb-app?tabs=azure-portal%2Cterminal-bash%2Cvscode-deploy%2Cdeploy-instructions-azportal%2Cdeploy-zip-linux-mac%2Cdeploy-instructions--zip-azcli) 
        - Azure Cosmos DB resource 
        - Azure App service  
    - [Node.js 10.1+ and npm](https://nodejs.org/en/download) - installed to your local machine.
    - [Visual Studio Code](https://code.visualstudio.com/) - installed to your local machine. 
    - Visual Studio Code extensions:
        - [Azure App Service extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice) for Visual Studio Code.
    - Use [Azure Cloud Shell](/azure/cloud-shell/quickstart) using the bash. If you prefer, [install](/cli/azure/install-azure-cli) the Azure CLI to run CLI reference commands.

## Log in to Azure CLI

In the Visual Studio Code integrated terminal, log in to the Azure CLI. This requires you to authenticate in a browser with your account, which has permission on a valid Azure Subscription.

Use the [az Login](/cli/azure/reference-index#az-login) command to login. 

```azurecli
az login
```



## Create a Key Vault resource with Azure CLI

Use the [az keyvault create](/cli/azure/keyvault#az-keyvault-create) command to create a Key Vault resource in the resource group.

```azurecli
az keyvault create \
    --subscription REPLACE_WITH_YOUR_SUBSCRIPTION_NAME_OR_ID \
    --resource-group REPLACE_WITH_YOUR_RESOURCE_GROUP_NAME \
    --name REPLACE_WITH_YOUR_KEY_VAULT_NAME
```

Your Azure account is the only one authorized to perform any operations on this new vault. Make note of the output values: 
* **Vault Name**: The name you provided to the `--name` parameter above.
* **Vault URI**: The URL format is `https://<YOUR_KEY_VAULT_NAME>.vault.azure.net/`. 

## Create a service principal with Azure CLI

The [service principal](/azure/active-directory/develop/app-objects-and-service-principals) allows you to create and use resources without having to use or expose your personal user account. The service principal is stored as an App Registration in Azure Active Directory. 

This sample uses the [DefaultAzureCredential](/javascript/api/overview/azure/identity-readme#defaultazurecredential), which requires authentication setup. One example of setting up the credential is to create and use a service principal.

1. Use the [az ad sp create-for-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac) command to create a service principal. 

    ```azurecli
    az ad sp create-for-rbac \
    --name REPLACE-WITH-YOUR-NEW-SERVICE-PRINCIPAL-NAME \
    --role Contributor \
    --scopes /subscriptions/REPLACE_WITH_YOUR_SUBSCRIPTION_NAME_OR_ID
    ```

    An example service principal name is `demo-keyvault-service-principal-YOUR-NAME`, where `YOUR-NAME` is postpended to the string. 

    

1. Capture and save the service principal output results of the command to use later.
 
    ```json
    {
        "appId": "YOUR-SERVICE-PRINCIPAL-ID",
        "displayName": "YOUR-SERVICE-PRINCIPAL-NAME",
        "name": "http://YOUR-SERVICE-PRINCIPAL-NAME",
        "password": "!@#$%",
        "tenant": "YOUR-TENANT-ID"
    }
    ```

## Give your service principal access to your key vault

Use the [az keyvault set-policy](/cli/azure/keyvault#az-keyvault-set-policy) command to give your service principal access to your Key Vault with Azure CLI command. The value for `YOUR-SERVICE-PRINCIPAL-ID` is your service principal output's `appId` value. 

```azurecli
az keyvault set-policy \
--subscription REPLACE-WITH-YOUR-SUBSCRIPTION-NAME-OR-ID \
--name "REPLACE-WITH-YOUR-KEY-VAULT-NAME" \
--spn YOUR-SERVICE-PRINCIPAL-ID \
--secret-permissions get list
```

This service principal will only be able to list all secrets or get a specific secret.

## Store your secret environment variable in Key Vault resource

Use the [az keyvault secret set](/cli/azure/keyvault/secret#az-keyvault-secret-set) command to add your MongoDB connection string, created in the [prior tutorial](/azure/app-service/tutorial-nodejs-mongodb-app?tabs=azure-portal%2Cterminal-bash%2Cvscode-deploy%2Cdeploy-instructions-azportal%2Cdeploy-zip-linux-mac%2Cdeploy-instructions--zip-azcli), as a secret named `DATABASEURL` to your key vault.

```azurecli
az keyvault secret set \
--subscription REPLACE-WITH-YOUR-SUBSCRIPTION-NAME-OR-ID \
--vault-name "REPLACE-WITH-YOUR-KEY-VAULT-NAME" \
--name "DATABASEURL" \
--value "YOUR-COSMOS-DB-MONGODB-CONNECTION-STRING"
```

> [!NOTE]
> `DATABASEURL`, as a secret name, is not a keyword. You could choose any name to identify the secret. Just use that name consistently in the remaining instructions. 

## Switch branches in GitHub 

The code to use key vault, instead of an environment variable, is provided in the `keyvault` branch of the sample repository.

1. Using git, stash the changes to your local project, then checkout out the `keyvault` branch. 

    ```bash
    git stash && git checkout keyvault
    ```

1. Install dependencies and open the project in Visual Studio Code. 

    ```bash
    npm install && \
    code .
    ```

    The Azure Key Vault integration requires two additional npm packages, @azure/identity (to use the service principal) and @azure/keyvault-secrets.

## Configure Express.js required environment variables to use Azure Identity

Set these environment variables in the `.env` file of the sample project to create the **REQUIRED context to use DefaultAzureCredential**.

* `AZURE_TENANT_ID`: The `tenant` from the service principal output above.
* `AZURE_CLIENT_ID`: The `appId` from the service principal output above.
* `AZURE_CLIENT_SECRET`: The `password` from the service principal output above.

When you deploy the application to Azure app service, you will also need to add these settings to your web app. 

> [!NOTE]
> These variables names are keywords and must be used as-is, without changes, in order for Azure Identity to work successfully.

## Configure Express.js required environment variables to use Azure Key Vault

Set these environment variables in the `.env` file of the sample project to programmatically determine which Key Vault resource and secret to use.

* `KEY_VAULT_NAME`: Same value as `REPLACE-WITH-YOUR-KEY-VAULT-NAME` used in previous commands.
* `KEY_VAULT_SECRET_NAME_DATABASEURL`: The secret name, `DATABASEURL`.

When you deploy the application to Azure app service, you will also need to add these settings to your web app. 

> [!NOTE]
> These variable names are specific to this sample. You can change them but make sure to change them in the environment file, the source code file, and your deployed web app settings. 

## Run the local program 

1. Run the Express.js app with the following command:

    ```bash
    npm start
    ```

1. Open the Express.js app in the browser: `http://localhost:8080`.
1. You may have names and jobs from the previous tutorial. Interact with the app, adding names and jobs, deleting individual names and jobs, or deleting all names and jobs. 

    :::image type="content" source="../../media/key-vault/use-expressjs-with-key-vault-to-use-cosmos-db-connection.png" alt-text="Run and view Express.js app accessing your Key Vault resource to get the Cosmos DB connection string, then use the connection string to access the MongoDB database.":::

## Deploy the key vault version to Azure app service

Complete this section using VS Code and the App Service extension.

1. In the VS Code activity bar, select the Azure icon.
1. In the side bar, select your web app from your subscription under the **App Service** section.
1. Right-click your app name and select **Deploy to Web app**. 
1. Add the required environment variables from your local app to the Azure app service, by right-clicking on the **Application Settings**. Use the value from the local `.env` file.

    |Setting to add|
    |--|
    |KEY_VAULT_NAME: |
    |KEY_VAULT_SECRET_NAME_DATABASEURL: `DATABASEURL`|
    |AZURE_TENANT_ID=|
    |AZURE_CLIENT_ID=|
    |AZURE_CLIENT_SECRET|

1. Remove the previous tutorial's settings by right-clicking on the setting then selecting **Delete**. 

    |Setting to remove|
    |--|
    |DATABASE_URL: `DATABASEURL`|

1. Right-click your web app name then select **Restart** to have the new app settings take affect.
1. Right-click your web app name then select **Browse Website**. In the subsequent pop-up window, select **Open**.
1. On the sidebar, right-click your web app and select **Start streaming logs** to see the service logs.
1. Interact with the app, adding items and deleting. 

## What Changed in the keyvault branch?

The original tutorial stored the database connection string in the `.env` file locally and in the App Settings in your Azure web app. Anyone who had access to your local workstation or your remote Azure app service would be able to see and use your Cosmos DB connection string. 

This branch of the sample changes from using the environment variables to getting those values from key vault in the [./src/azure/azure-kevault.js](https://github.com/Azure-Samples/js-e2e-express-mongodb/blob/keyvault/src/azure/azure-keyvault.js) file. 

The authentication to connect to Key Vault uses the DefaultAzureCredential. The benefit of this is that the code doesn't need to use or store credentials to your key vault. 

## Understand the sample application Key Vault code

The sample code uses the following Azure SDKs:

* [@azure/identity](https://www.npmjs.com/package/@azure/identity) - uses DefaultAzureCredential and your service principal to access resources on Azure.
* [@azure/keyvault-secrets](https://www.npmjs.com/package/@azure/keyvault-secrets) - used to manage Key Vault secrets.

### Get secret from Key Vault with JavaScript

After you ensure your DefaultAzureCredential is correctly configured with the required environment variables to use Azure Identity, use the DefaultAzureCredential to access your Key Vault secrets with JavaScript. 

1. The following [azure-keyvault.js](https://github.com/Azure-Samples/js-e2e-express-mongodb/blob/keyvault/src/azure/azure-keyvault.js) file gets the secret from your key vault.

    :::code language="javascript" source="~/../js-e2e-express-mongodb-keyvault/src/azure/azure-keyvault.js" range="76-113" :::

1. The following [data.js](https://github.com/Azure-Samples/js-e2e-express-mongodb/blob/keyvault/src/data.js) code pulls in the dependency for the key vault secret function, `getSecret`, and initializes the configuration object.

    :::code language="javascript" source="~/../js-e2e-express-mongodb-keyvault/src/data.js" range="7-8":::

1. The following `data.js` code shows the `getConnection` function to get environment variables and call `getSecret` from `azure-keyvault.js`.

    :::code language="javascript" source="~/../js-e2e-express-mongodb-keyvault/src/data.js" range="15-43" :::

1. The following `data.js` file code calls the `getConnection` function, then returns the function to the Express.js `server.js` file. 
    
    :::code language="javascript" source="~/../js-e2e-express-mongodb-keyvault/src/data.js" range="96-112" :::

## Clean up resources - remove resource group

Once you have completed this tutorial, you need to remove the resource group with the [az group delete](/cli/azure/group#az-group-delete) command.

```azurecli
az group delete \
--name REPLACE_WITH_YOUR_RESOURCE_GROUP_NAME  -y
```

This command may take a few minutes. 

## Clean up resources - remove service principal

Delete your service principal with the 
[az ad sp delete](/cli/azure/ad/sp#az-ad-sp-delete) command. 

```azurecli
az ad sp delete \
--id YOUR-SERVICE-PRINCIPAL-ID
```

## Next steps

* [Configure your Azure Web app](../configure-web-app-settings.md)