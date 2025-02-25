# Save user and conversation data

This bot has been created using [Bot Framework](https://dev.botframework.com).

This sample demonstrates how to save user and conversation data in a bot. The bot maintains conversation state to track and direct the conversation and ask the user questions. The bot maintains user state to track the user's answers.

By default this bot uses MemoryStorage for Conversation and User state. Memory Storage is great for testing purposes, but in a production scenario you will need to use persistent storage.

Application.java contains commented examples of using Azure Blob Storage or Cosmos DB instead.

This sample is a Spring Boot app and uses the Azure CLI and azure-webapp Maven plugin to deploy to Azure.

# Azure Blob Storage

To use Azure Blob Storage, create a Blob Storage account in your Azure subscription. You can then use the following code to
create your storage layer, passing in your Blob Storage connection string and a blob container name.

***Note: You do not need to create the container manually, the bot will create the container for you if it does not exist.***

```java
return new BlobsStorage("<blob-storage-connection-string>", "bot-state");
```

# CosmosDB Storage

To use CosmosDB Storage, you need to create a CosmosDB instance in your Azure subscription. You can then use the following code to
create your storage layer.

***Note: It is your responsibility to create an appropriate database within your CosmosDB instance. However, you should **not**
create the container within the database, as the bot will do this for you and ensure the container is configured correctly.***

```java
CosmosDbPartitionedStorageOptions options = new CosmosDbPartitionedStorageOptions();
options.setCosmosDbEndpoint("<endpoint-for-your-cosmosdb-instance>");
options.setAuthKey("<your-cosmosdb-auth-key>");
options.setDatabaseId("<your-database-id>");
options.setContainerId("<cosmosdb-container-id>");
return new CosmosDbPartitionedStorage(options);
```

## Prerequisites

- Java 1.8+
- Install [Maven](https://maven.apache.org/)
- An account on [Azure](https://azure.microsoft.com) if you want to deploy to Azure.

## To try this sample
- From the root of this project folder:
  - Build the sample using `mvn package`
  - Run it by using `java -jar .\target\bot-statemanagement-sample.jar`

## Testing the bot using Bot Framework Emulator

[Bot Framework Emulator](https://github.com/microsoft/botframework-emulator) is a desktop application that allows bot developers to test and debug their bots on localhost or running remotely through a tunnel.

- Install the latest Bot Framework Emulator from [here](https://github.com/Microsoft/BotFramework-Emulator/releases)

### Connect to the bot using Bot Framework Emulator

- Launch Bot Framework Emulator
- File -> Open Bot
- Enter a Bot URL of `http://localhost:3978/api/messages`

## Deploy the bot to Azure

As described on [Deploy your bot](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-deploy-az-cli), you will perform the first 4 steps to setup the Azure app, then deploy the code using the azure-webapp Maven plugin.

### 1. Login to Azure

From a command (or PowerShell) prompt in the root of the bot folder, execute:
`az login`

### 2. Set the subscription

```
az account set --subscription "<azure-subscription>"
```

If you aren't sure which subscription to use for deploying the bot,  you can view the list of subscriptions for your account by using `az account list` command.

### 3. Create an App registration

```
az ad app create --display-name "<botname>" --password "<appsecret>" --available-to-other-tenants
```

Replace `<botname>` and `<appsecret>` with your own values.

`<botname>` is the unique name of your bot.
`<appsecret>` is a minimum 16 character password for your bot.

Record the `appid` from the returned JSON

### 4. Create the Azure resources

Replace the values for `<appid>`, `<appsecret>`, `<botname>`, and `<groupname>` in the following commands:

#### To a new Resource Group

```
az deployment sub create --name "stateManagementDeploy" --location "westus" --template-file ".\deploymentTemplates\template-with-new-rg.json" --parameters appId="<appid>" appSecret="<appsecret>" botId="<botname>" botSku=S1 newAppServicePlanName="stateManagementPlan" newWebAppName="stateManagement" groupLocation="westus" newAppServicePlanLocation="westus"
```

#### To an existing Resource Group

```
az deployment group create --resource-group "<groupname>" --template-file ".\deploymentTemplates\template-with-preexisting-rg.json" --parameters appId="<appid>" appSecret="<appsecret>" botId="<botname>" newWebAppName="stateManagement" newAppServicePlanName="stateManagementPlan" appServicePlanLocation="westus" --name "stateManagement"
```

### 5. Update app id and password

In src/main/resources/application.properties update

- `MicrosoftAppPassword` with the botsecret value
- `MicrosoftAppId` with the appid from the first step

### 6. Deploy the code

- Execute `mvn clean package`
- Execute `mvn azure-webapp:deploy -Dgroupname="<groupname>" -Dbotname="<bot-app-service-name>"`

If the deployment is successful, you will be able to test it via  "Test in Web Chat" from the Azure Portal using the "Bot Channel  Registration" for the bot.

After the bot is deployed, you only need to execute #6 if you make changes to the bot.

## Bot State

A key to good bot design is to track the context of a conversation, so that your bot remembers things like the answers to previous questions. Depending on what your bot is used for, you may even need to keep track of state or store information for longer than the lifetime of the conversation. A bots state is information it remembers in order to respond appropriately to incoming messages. The Bot Builder SDK provides classes for storing and retrieving state data as an object associated with a user or a conversation.

## Further reading
- [Maven Plugin for Azure App Service](https://github.com/microsoft/azure-maven-plugins/tree/develop/azure-webapp-maven-plugin)
- [Spring Boot](https://spring.io/projects/spring-boot)
- [Azure Bot Service](https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0)
- [Bot Storage](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0)
- [Azure for Java cloud developers](https://docs.microsoft.com/en-us/azure/java/?view=azure-java-stable)