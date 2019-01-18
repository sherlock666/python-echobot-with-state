# EchoBot with State

## To try this sample
- Clone the repository
```bash
git clone https://github.com/Microsoft/botbuilder-python.git
```


### Visual studio code
- Activate your desired virtual environment
- Open `botbuilder-python\samples\EchoBot-with-State` folder
- Bring up a terminal, navigate to `botbuilder-python\samples\EchoBot-with-State` folder
- In the terminal, type `pip install -r requirements.txt`
- In the terminal, type `python main.py`

## Testing the bot using Bot Framework Emulator
[Microsoft Bot Framework Emulator](https://github.com/microsoft/botframework-emulator) is a desktop application that allows bot developers to test and debug their bots on localhost or running remotely through a tunnel.

- Install the Bot Framework emulator from [here](https://github.com/Microsoft/BotFramework-Emulator/releases)

### Connect to bot using Bot Framework Emulator **V4**
- Launch Bot Framework Emulator
- File -> Open bot and navigate to samples\EchoBot-with-State folder
- Select EchoBot-with-State.bot file

## Deploy to Azure

### Create a Resource Group
A resource group is a logical container into which Azure resources like web apps, databases, and storage accounts are deployed and managed. 

In the Cloud Shell, create a resource group with the az group create command. The following example creates a resource group named myResourceGroup in the West US location.
```bash
az group create --name myResourceGroup --location "West US"
```
### Create an Azure App Service Plan 
Create an App Service plan in the resource group with the az appservice plan create command.
```bash
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku B1 --is-linux
```
### Create a Web App
Create a web app in the `myAppServicePlan` App Service plan. We are setting the runtime to Python 3.7 and setting the deployment to local git.
```bash
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app_name> --runtime "PYTHON|3.7" --deployment-local-git
```
Running this command should output the URL of the Git remote where we will push our bot. Save this URL as we will need it later.

### Register Your Bot
Next we need to create a bot registration so we can deploy our bot to multiple channels like Microsoft Teams, Skype, and Facebook Messenger. Make sure to add your `<app_name>` to the enpoint url
```bash
az bot create --resource-group myResourceGroup --name myPyhtonBot --kind registration --endpoint https://<app_name>.azurewebsites.net/api/messages
```

### Create Startup File
Create a `startup.txt` file to your project and add the following line: 
```
gunicorn --bind 0.0.0.0 --worker-class aiohttp.worker.GunicornWebWorker --timeout 600 main:app
```
This configures the Gunicorn WSGI HTTP Server to use `aiohttp` workers and starts the app.

### Add Enviorment Varaibles
We need to add our `APP_ID` and `APP_SECRET` to our app as environment variables. To get our the `APP_ID` and `APP_SECRET`, open the [Azure Portal](https://portal.azure.com) and naviage to the resource group we created earlier. Click on the Deployments blade on the left side, and then click on the Bot you ceated under Deployment Name. Then click on the Inputs blade and copy the `APPID` and `APPSECRET`.

Now go back to the resource group and navigate to the Web App Service you created. Open the Application settings blade. In the Applications settings section, add the `APP_ID` and `APP_SECRET` as key value pairs.

While we're in this window, set the Start File field to `startup.txt`. This will run the startup command we created earlier. 

### Create Local Git Reposity and Push to Web App Remote
Create a local git repository and add your project.

```bash
git init
git add .
git commit -m "init"
```

In the Web App Service on Azure, click on the Deployment Center Blade. It should be right above the Application settings blade from the previous step. Copy the Git Clone Uri and add it as a remote to your local git repository. Then click on the Deployment Credntials button to get the Username and Password for the Kudo Repository. When you push your project to Azure, you will be prompted for the username and password.

```bash
git remote add azure https://<app_name>.scm.azurewebsites.net:443/<app_name>.git
git push azure master
```

### Test in WebChat on Azure
Navigate to your Bot - the simpilest way to find it is in your Resource Group. Click on the Test in WebChat blade and message your bot. 

## Bot State

A key to good bot design is to track the context of a conversation, so that your bot remembers things like the answers to previous questions. Depending on what your bot is used for, you may even need to keep track of state or store information for longer than the lifetime of the conversation. A bot's state is information it remembers in order to respond appropriately to incoming messages. The Bot Builder SDK provides classes for storing and retrieving state data as an object associated with a user or a conversation.

# Further reading

- [Azure Bot Service Introduction](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0)
- [Bot State](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-storage-concept?view=azure-bot-service-4.0)
- [Write directly to storage](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-v4-storage?view=azure-bot-service-4.0&tabs=csharpechorproperty%2Ccsetagoverwrite%2Ccsetag)
- [Managing conversation and user state](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-v4-state?view=azure-bot-service-4.0)