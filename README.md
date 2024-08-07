# Developing Solutions for Microsoft Azure

1. [Azure App Service](#1)

- [Explore Azure App Service](#101)
  - [Service plans](#1011)
  - [Deploy to App Service](#1012)
  - [Authentication and authorization](#1013)
  - [Networking features](#1014)
    - [Examples](#1015)
- [Configure web app settings](#102)
  - [Configure general settings](#1021)
  - [Configure path mappings](#1022)
  - [Enable diagnostic logging](#1023)
  - [Configure security certificates](#1024)
- [Scale apps in Azure App Service](#103)
  - [Identify autoscale factors](#1031)
  - [Enable autoscale](#1032)
  - [Autoscale best practices](#1033)
- [Azure App Service deployment slots](#104)

2. [Functions](#2)

- [Explore Azure Function](#201)
- [Develop Azure Function](#202)
  - [Example](#2021)

3. [Blob storage](#3)

- [Explore Azure Blob storage](#301)
  - [ Azure Storage security features](#3011)
- [Manage the Azure Blob storage lifecycle](#302)
- [Azure Blob storage client library for .NET](#303)
- [Set and retrieve properties and metadata for blob resources by using REST](#304)
  - [Example](#3041)

4. [Cosmos DB](#4)

- [Explore Azure Cosmos DB](#401)
  - [Consistency levels](#4011)
  - [Supported APIs](#4012)
  - [Request units](#4013)
    - [Example](#4014)
- [Develop solutions that use Cosmos DB](#402)
  - [Explore Microsoft .NET SDK v3](#4021)
    - [Example](#4022)
  - [Create stored procedures](#4023)
  - [Create triggers and user-defined functions](#4024)
  - [Change feed in Azure Cosmos DB](#4043)

5. [Implement containerized solutions](#5)

- [Discover the Azure Container Registry](#501)
- [Build and manage containers with tasks](#502)
- [Elements of a Dockerfile](#503)
  - [Example](#5031)
- [Explore Azure Container Instances](#504)
  - [Example](#5041)
- [Implement Azure Container Apps](#505)
  - [Example](#5051)
- [Explore containers in Azure Container Apps](#506)
- [Implement authentication and authorization in Azure Container Apps](#507)
- [Manage revisions and secrets in Azure Container Apps](#508)
- [Explore Dapr integration with Azure Container Apps](#509)

6. [Implement user authentication and authorization](#6)

- [Implement authentication by using the Microsoft Authentication Library](#601)
  - [Example](#6011)
- [Implement shared access signatures](#602)
- [Microsoft Graph](#603)

7. [Implement secure Azure solutions](#7)

- [Azure Key Vault](#701)
  - [Example](#7011)
- [Implement managed identities](#702)
  - [Example](#7021)
- [Implement Azure App Configuration](#703)

8. [Implement API Management](#8)

- [Example](#801)

## 1. Azure App Service <a name="1"></a>

### 📒 Explore Azure App Service <a name="101"></a>

`Azure App Service` is an HTTP-based service for hosting web applications, REST APIs, and mobile back ends. You can develop in your favorite programming language or framework. Applications run and scale with ease on both Windows and Linux-based environments.

<b>Built-in auto scale support</b>

The ability to scale up/down or scale out/in is baked into the Azure App Service. Depending on the usage of the web app, you can scale the resources of the underlying machine that is hosting your web app up/down. Resources include the number of cores or the amount of RAM available. `Scaling out/in` is the ability to increase, or decrease, the number of machine instances that are running your web app.

<b>Container support</b>

With Azure App Service, you can deploy and run containerized web apps on Windows and Linux. You can pull container images from a private Azure Container Registry or Docker Hub. Azure App Service also supports multi-container apps, Windows containers, and Docker Compose for orchestrating container instances.

<b>Continuous integration/deployment support</b>

The Azure portal provides out-of-the-box continuous integration and deployment with Azure DevOps Services, GitHub, Bitbucket, FTP, or a local Git repository on your development machine. Connect your web app with any of the above sources and App Service will do the rest for you by auto-syncing code and any future changes on the code into the web app. Continuous integration and deployment for containerized web apps is also supported using either Azure Container Registry or Docker Hub.

<b>Deployment slots</b>
When you deploy your web app you can use a separate deployment slot instead of the default production slot when you're running in the Standard App Service Plan tier or better. Deployment slots are live apps with their own host names. App content and configurations elements can be swapped between two deployment slots, including the production slot.

<b>App Service on Linux</b>

App Service can also host web apps natively on Linux for supported application stacks. It can also run custom Linux containers (also known as Web App for Containers). App Service on Linux supports many language specific built-in images. Just deploy your code. Supported languages and frameworks include: Node.js, Java (JRE 8 & JRE 11), PHP, Python, .NET, and Ruby. If the runtime your application requires isn't supported in the built-in images, you can deploy it with a custom container.

The languages, and their supported versions, are updated regularly. You can retrieve the current list by using the following command in the Cloud Shell.

```
az webapp list-runtimes --os-type linux
```

App Service on Linux does have some limitations:

- App Service on Linux isn't supported on Shared pricing tier.
- The Azure portal shows only features that currently work for Linux apps. As features are enabled, they're activated on the portal.
- When deployed to built-in images, your code and content are allocated a storage volume for web content, backed by Azure Storage. The disk latency of this volume is higher and more variable than the latency of the container filesystem. Apps that require heavy read-only access to content files may benefit from the custom container option, which places files in the container filesystem instead of on the content volume.

### 📒 Service plans <a name="1011"></a>

In App Service, an app always runs in an App Service plan. An `App Service plan` defines a set of compute resources for a web app to run. One or more apps can be configured to run on the same computing resources (or in the same App Service plan).

When you create an App Service plan in a certain region (for example, West Europe), a set of compute resources is created for that plan in that region. Whatever apps you put into this App Service plan run on these compute resources as defined by your App Service plan.

Each App Service plan defines:

- Operating System (Windows, Linux)
- Region (West US, East US, etc.)
- Number of VM instances
- Size of VM instances (Small, Medium, Large)
- Pricing tier (Free, Shared, Basic, Standard, Premium, PremiumV2, PremiumV3, Isolated, IsolatedV2)

The `pricing tier` of an App Service plan determines what App Service features you get and how much you pay for the plan. There are a few categories of pricing tiers:

- `Shared compute: Free and Shared`, the two base tiers, runs an app on the same Azure VM as other App Service apps, including apps of other customers. Some apps might belong to other customers. These tiers are intended to be used only for development and testing purposes. These tiers allocate CPU quotas to each app that runs on the shared resources, and the resources can't scale out.
- `Dedicated compute: The Basic, Standard, Premium, PremiumV2, and PremiumV3` tiers run apps on dedicated Azure VMs. Only apps in the same App Service plan share the same compute resources. The higher the tier, the more VM instances are available to you for scale-out.
- `Isolated: The Isolated and IsolatedV2` tiers run dedicated Azure VMs on dedicated Azure Virtual Networks. It provides network isolation on top of compute isolation to your apps. It provides the maximum scale-out capabilities.

<b> How does my app run and scale? </b>

In the Free and Shared tiers, an app receives CPU minutes on a shared VM instance and can't scale out.

In other tiers, an app runs and scales as follows:

- An app runs on all the VM instances configured in the App Service plan.
- If multiple apps are in the same App Service plan, they all share the same VM instances.
- If you have multiple deployment slots for an app, all deployment slots also run on the same VM instances.
- If you enable diagnostic logs, perform backups, or run WebJobs, they also use CPU cycles and memory on these VM instances.

In this way, the App Service plan is the `scale unit` of the App Service apps. If the plan is configured to run five VM instances, then all apps in the plan run on all five instances. If the plan is configured for autoscaling, then all apps in the plan are scaled out together based on the autoscale settings.

<b>What if my app needs more capabilities or features?</b>
Your App Service plan can be scaled up and down at any time. It's as simple as changing the pricing tier of the plan. If your app is in the same App Service plan with other apps, you may want to improve the app's performance by isolating the compute resources. You can do it by moving the app into a separate App Service plan.

You can potentially save money by putting multiple apps into one App Service plan. However, since apps in the same App Service plan all share the same compute resources you need to understand the capacity of the existing App Service plan and the expected load for the new app.

Isolate your app into a new App Service plan when:

- The app is resource-intensive.
- You want to scale the app independently from the other apps in the existing plan.
- The app needs resource in a different geographical region.

This way you can allocate a new set of resources for your app and gain greater control of your apps.

### 📒 Deploy to App Service <a name="1012"></a>

Every development team has unique requirements that can make implementing an efficient deployment pipeline difficult on any cloud service. App Service supports both automated and manual deployment.

<b>Automated deployment</b>

`Automated deployment`, or `continuous deployment`, is a process used to push out new features and bug fixes in a fast and repetitive pattern with minimal effect on end users.

Azure supports automated deployment directly from several sources. The following options are available:

- `Azure DevOps Services`: You can push your code to Azure DevOps Services, build your code in the cloud, run the tests, generate a release from the code, and finally, push your code to an Azure Web App.
- `GitHub`: Azure supports automated deployment directly from GitHub. When you connect your GitHub repository to Azure for automated deployment, any changes you push to your production branch on GitHub are automatically deployed for you.
- `Bitbucket`: With its similarities to GitHub, you can configure an automated deployment with Bitbucket.

<b>Manual deployment</b>

There are a few options that you can use to manually push your code to Azure:

- `Git`: App Service web apps feature a Git URL that you can add as a remote repository. Pushing to the remote repository deploys your app.
- `CLI`: 'webapp up' is a feature of the 'az command-line interface' that packages your app and deploys it. Unlike other deployment methods,'az webapp' up can create a new App Service web app for you if you haven't already created one.
- `Zip deploy`: Use curl or a similar HTTP utility to send a ZIP of your application files to App Service.
- `FTP/S`: FTP or FTPS is a traditional way of pushing your code to many hosting environments, including App Service.

<b>Use deployment slots</b>

Whenever possible, use deployment slots when deploying a new production build. When using a Standard App Service Plan tier or better, you can deploy your app to a staging environment and then swap your staging and production slots. The swap operation warms up the necessary worker instances to match your production scale, thus eliminating downtime.

<b>Continuously deploy code</b>
If your project has designated branches for testing, QA, and staging, then each of those branches should be continuously deployed to a staging slot. This allows your stakeholders to easily assess and test the deployed branch.

<b>Continuously deploy containers</b>
For custom containers from Azure Container Registry or other container registries, deploy the image into a staging slot and swap into production to prevent downtime. The automation is more complex than code deployment because you must push the image to a container registry and update the image tag on the webapp.

- `Build and tag the image`: As part of the build pipeline, tag the image with the git commit ID, timestamp, or other identifiable information. It’s best not to use the default “latest” tag. Otherwise, it’s difficult to trace back what code is currently deployed, which makes debugging far more difficult.
- `Push the tagged image`: Once the image is built and tagged, the pipeline pushes the image to our container registry. In the next step, the deployment slot will pull the tagged image from the container registry.
- `Update the deployment slot with the new image tag`: When this property is updated, the site will automatically restart and pull the new container image.

### 📒 Authentication and authorization <a name="1013"></a>

Azure App Service provides built-in authentication and authorization support, so you can sign in users and access data by writing minimal, or no code in your web app, RESTful API, mobile back end, and Azure Functions.

<b>Why use the built-in authentication?</b>

You're not required to use App Service for authentication and authorization. Many web frameworks are bundled with security features, and you can use them if you like. If you need more flexibility than App Service provides, you can also write your own utilities.

The built-in authentication feature for App Service and Azure Functions can save you time and effort by providing out-of-the-box authentication with federated identity providers, allowing you to focus on the rest of your application.

- Azure App Service allows you to integrate various auth capabilities into your web app or API without implementing them yourself.
- It’s built directly into the platform and doesn’t require any particular language, SDK, security expertise, or code.
- You can integrate with multiple login providers. For example, Microsoft Entra ID, Facebook, Google, Twitter.

<b>Identity providers</b>

App Service uses federated identity, in which a third-party identity provider manages the user identities and authentication flow for you. The following identity providers are available by default:

![](images/1.png)

When you enable authentication and authorization with one of these providers, its sign-in endpoint is available for user authentication and for validation of authentication tokens from the provider. You can provide your users with any number of these sign-in options.

<b>How it works</b>

The authentication and authorization module runs in the same sandbox as your application code. When it's enabled, every incoming HTTP request passes through it before being handled by your application code. This module handles several things for your app:

- Authenticates users and clients with the specified identity provider(s)
- Validates, stores, and refreshes OAuth tokens issued by the configured identity provider(s)
- Manages the authenticated session
- Injects identity information into HTTP request headers

The module runs separately from your application code and can be configured using Azure Resource Manager settings or using a configuration file. No SDKs, specific programming languages, or changes to your application code are required.

In Linux and containers the authentication and authorization module runs in a separate container, isolated from your application code. Because it does not run in-process, no direct integration with specific language frameworks is possible.

<b>Authentication flow</b>

The authentication flow is the same for all providers, but differs depending on whether you want to sign in with the provider's SDK.

- `Without provider SDK`: The application delegates federated sign-in to App Service. This is typically the case with browser apps, which can present the provider's login page to the user. The server code manages the sign-in process, so it's also called `server-directed flow` or `server flow`.

- `With provider SDK`: The application signs users in to the provider manually and then submits the authentication token to App Service for validation. This is typically the case with browser-less apps, which can't present the provider's sign-in page to the user. The application code manages the sign-in process, so it's also called client-directed flow or client flow. This applies to REST APIs, Azure Functions, JavaScript browser clients, and native mobile apps that sign users in using the provider's SDK.

The following table shows the steps of the authentication flow

![](images/2.png)

For client browsers, App Service can automatically direct all unauthenticated users to `/.auth/login/<provider>`. You can also present users with one or more `/.auth/login/<provider>` links to sign in to your app using their provider of choice.

<b>Authorization behavior</b>

In the Azure portal, you can configure App Service with many behaviors when an incoming request isn't authenticated.

- `Allow unauthenticated requests`: This option defers authorization of unauthenticated traffic to your application code. For authenticated requests, App Service also passes along authentication information in the HTTP headers. This option provides more flexibility in handling anonymous requests. It lets you present multiple sign-in providers to your users.

- `Require authentication`: This option rejects any unauthenticated traffic to your application. This rejection can be a redirect action to one of the configured identity providers. In these cases, a browser client is redirected to /.auth/login/<provider> for the provider you choose. If the anonymous request comes from a native mobile app, the returned response is an HTTP 401 Unauthorized. You can also configure the rejection to be an HTTP 401 Unauthorized or HTTP 403 Forbidden for all requests.

Restricting access in this way applies to all calls to your app, which may not be desirable for apps wanting a publicly available home page, as in many single-page applications.

App Service provides a built-in `token store`, which is a repository of tokens that are associated with the users of your web apps, APIs, or native mobile apps. When you enable authentication with any provider, this token store is immediately available to your app.

If you enable application logging, authentication and authorization traces are collected directly in your log files. If you see an authentication error that you didn't expect, you can conveniently find all the details by looking in your existing application logs.

### 📒 Networking features <a name="1014"></a>

By default, apps hosted in App Service are accessible directly through the internet and can reach only internet-hosted endpoints. But for many applications, you need to control the inbound and outbound network traffic.

There are two main deployment types for Azure App Service. The multitenant public service hosts App Service plans in the Free, Shared, Basic, Standard, Premium, PremiumV2, and PremiumV3 pricing SKUs. There's also the single-tenant `App Service Environment` (ASE) hosts Isolated SKU App Service plans directly in your Azure virtual network.

<b>Multi-tenant App Service networking features</b>

Azure App Service is a distributed system. The roles that handle incoming HTTP or HTTPS requests are called `front ends`. The roles that host the customer workload are called `workers`. All the roles in an App Service deployment exist in a multi-tenant network. Because there are many different customers in the same App Service scale unit, you can't connect the App Service network directly to your network.

Instead of connecting the networks, you need `features` to handle the various aspects of application communication. The features that handle requests to your app can't be used to solve problems when you're making calls from your app. Likewise, the features that solve problems for calls from your app can't be used to solve problems to your app.

![](images/3.png)

You can mix the features to solve your problems with a few exceptions. The following inbound use cases are examples of how to use App Service networking features to control traffic inbound to your app.

![](images/4.png)

<b>Default networking </b>

Azure App Service scale units support many customers in each deployment. The Free and Shared SKU plans host customer workloads on multitenant workers. The Basic and higher plans host customer workloads that are dedicated to only one App Service plan. If you have a Standard App Service plan, all the apps in that plan run on the same worker. If you scale out the worker, all the apps in that App Service plan are replicated on a new worker for each instance in your App Service plan.

<b>Outbound addresses</b>

The worker VMs are broken down in large part by the App Service plans. The Free, Shared, Basic, Standard, and Premium plans all use the same worker VM type. The PremiumV2 plan uses another VM type. PremiumV3 uses yet another VM type. When you change the VM family, you get a different set of outbound addresses.

There are many addresses that are used for outbound calls. The outbound addresses used by your app for making outbound calls are listed in the properties for your app. These addresses are shared by all the apps running on the same worker VM family in the App Service deployment. If you want to see all the addresses that your app might use in a scale unit, there's a property called `possibleOutboundIpAddresses` that lists them.

<b>Find outbound IPs </b>

To find the outbound IP addresses currently used by your app in the Azure portal, select Properties in your app's left-hand navigation.

You can find the same information by running the following Azure CLI command in the Cloud Shell. They're listed in the Additional Outbound IP Addresses field.

```
az webapp show \
    --resource-group <group_name> \
    --name <app_name> \
    --query outboundIpAddresses \
    --output tsv
```

To find all possible outbound IP addresses for your app, regardless of pricing tiers, run the following command in the Cloud Shell.

```
az webapp show \
    --resource-group <group_name> \
    --name <app_name> \
    --query possibleOutboundIpAddresses \
    --output tsv
```

### 📒 Examples <a name="1015"></a>

> Deploy a basic HTML+CSS site to Azure App Service by using the Azure CLI `az webapp up` command. Then update the code and redeploy it by using the same command.

The `az webapp up` command makes it easy to create and update web apps. When executed it performs the following actions:

- Create a default resource group if one isn't specified.
- Create a default app service plan.
- Create an app with the specified name.
- Zip deploy files from the current working directory to the web app.

Exercise: Create a static HTML web app by using Azure Cloud Shell
https://learn.microsoft.com/en-us/training/modules/introduction-to-azure-app-service/7-create-html-web-app

---

### 📒 Configure web app settings <a name="102"></a>

In App Service, app settings are variables passed as environment variables to the application code. For Linux apps and custom containers, App Service passes app settings to the container using the --env flag to set the environment variable in the container.

Application settings can be accessed by navigating to your app's management page and selecting `Environment variables > Application settings`.

<b>Adding and editing settings</b>

To add a new app setting, select `+ Add`. If you're using deployment slots you can specify if your setting is swappable or not. In the dialog, you can stick the setting to the current slot.

To edit app settings in bulk, select the Advanced edit button. When finished, select OK. Don't forget to select Apply back in the Environment variables page.

Adding and editing connection strings follow the same principles as other app settings and they can also be tied to deployment slots.

```
// Note

In a default Linux app service or a custom Linux container, any nested JSON key structure in the app setting name like ApplicationInsights:InstrumentationKey needs to be configured in App Service as ApplicationInsights__InstrumentationKey for the key name. In other words, any : should be replaced by __ (double underscore). Any periods in the app setting name will be replaced with a _ (single underscore).
```

<b>Configure environment variables for custom containers</b>
Your custom container might use environment variables that need to be supplied externally. You can pass them in via the Cloud Shell.

```
// In Bash:
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings key1=value1 key2=value2

// In PowerShell:
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"DB_HOST"="myownserver.mysql.database.azure.com"}
```

When your app runs, the App Service app settings are injected into the process as environment variables automatically. You can verify container environment variables with the URL
`https://<app-name>.scm.azurewebsites.net/Env.`

### 📒 Configure general settings <a name="1021"></a>

In the `Configuration > General settings` section you can configure some common settings for your app. Some settings require you to scale up to higher pricing tiers.

A list of the currently available settings:

- `Stack settings`: The software stack to run the app, including the language and SDK versions. For Linux apps and custom container apps, you can also set an optional start-up command or file.
- `Platform settings`: Lets you configure settings for the hosting platform, including:

  - Platform bitness: 32-bit or 64-bit. For Windows apps only.
  - FTP state: Allow only FTPS or disable FTP altogether.
  - HTTP version: Set to 2.0 to enable support for HTTPS/2 protocol.

    ```
    Note
    Most modern browsers support HTTP/2 protocol over TLS only, while non-encrypted traffic continues to use HTTP/1.1. To ensure that client browsers connect to your app with HTTP/2, secure your custom DNS name.
    ```

  - Web sockets: For ASP.NET SignalR or socket.io, for example.
    Always On: Keeps the app loaded even when there's no traffic. When Always On isn't turned on (default), the app is unloaded after 20 minutes without any incoming requests. The unloaded app can cause high latency for new requests because of its warm-up time. When Always On is turned on, the front-end load balancer sends a GET request to the application root every five minutes. The continuous ping prevents the app from being unloaded.
    Always On is required for continuous WebJobs or for WebJobs that are triggered using a CRON expression.
  - ARR affinity: In a multi-instance deployment, ensure that the client is routed to the same instance for the life of the session. You can set this option to Off for stateless applications.
  - HTTPS Only: When enabled, all HTTP traffic is redirected to HTTPS.
  - Minimum TLS version: Select the minimum TLS encryption version required by your app.

- `Debugging`: Enable remote debugging for ASP.NET, ASP.NET Core, or Node.js apps. This option turns off automatically after 48 hours.
- `Incoming client certificates`: Require client certificates in mutual authentication. TLS mutual authentication is used to restrict access to your app by enabling different types of authentication for it.

### 📒 Configure path mappings <a name="1022"></a>

In the `Configuration > Path mappings` section you can configure handler mappings, and virtual application and directory mappings. The Path mappings page displays different options based on the OS type.

<b>Windows apps (uncontainerized)</b>

For Windows apps, you can customize the IIS handler mappings and virtual applications and directories.

Handler mappings let you add custom script processors to handle requests for specific file extensions. To add a custom handler, select `New handler mapping`. Configure the handler as follows:

- Extension: The file extension you want to handle, such as \*.php or handler.fcgi.
- Script processor: The absolute path of the script processor. Requests to files that match the file extension are processed by the script processor. Use the path D:\home\site\wwwroot to refer to your app's root directory.
- Arguments: Optional command-line arguments for the script processor.

Each app has the default root path (/) mapped to D:\home\site\wwwroot, where your code is deployed by default. If your app root is in a different folder, or if your repository has more than one application, you can edit or add virtual applications and directories.

You can configure virtual applications and directories by specifying each virtual directory and its corresponding physical path relative to the website root (D:\home). To mark a virtual directory as a web application, clear the Directory check box.

<b>Linux and containerized apps</b>

You can add custom storage for your containerized app. Containerized apps include all Linux apps and also the Windows and Linux custom containers running on App Service. Select `New Azure Storage Mount` and configure your custom storage as follows:

- Name: The display name.
- Configuration options: Basic or Advanced. Select Basic if the storage account isn't using service endpoints, private endpoints, or Azure Key Vault. Otherwise, select Advanced.
- Storage accounts: The storage account with the container you want.
- Storage type: Azure Blobs or Azure Files. Windows container apps only support Azure Files. Azure Blobs only supports read-only access.
- Storage container: For basic configuration, the container you want.
- Share name: For advanced configuration, the file share name.
- Access key: For advanced configuration, the access key.
- Mount path: The absolute path in your container to mount the custom storage.
- Deployment slot setting: When checked, the storage mount settings also apply to deployment slots.

### 📒 Enable diagnostic logging <a name="1023"></a>

There are built-in diagnostics to assist with debugging an App Service app. In this lesson, you learn how to enable diagnostic logging and add instrumentation to your application, and how to access the information logged by Azure.

The following table shows the types of logging, the platforms supported, and where the logs can be stored and located for accessing the information.

![](images/5.png)

<b>Enable application logging (Windows)</b>

1. To enable application logging for Windows apps in the Azure portal, navigate to your app and select App Service logs.

2. Select On for either Application Logging (Filesystem) or Application Logging (Blob), or both. The Filesystem option is for temporary debugging purposes, and turns itself off in 12 hours. The Blob option is for long-term logging, and needs a blob storage container to write logs to.

If you regenerate your storage account's access keys, you must reset the respective logging configuration to use the updated access keys. To do this turn the logging feature off and then on again.

3. You can also set the Level of details included in the log as shown in the following table.

| Level       | Included categories                                           |
| ----------- | ------------------------------------------------------------- |
| Disabled    | None                                                          |
| Error       | Error, Critical                                               |
| Warning     | Warning, Error, Critical                                      |
| Information | Info, Warning, Error, Critical                                |
| Verbose     | Trace, Debug, Info, Warning, Error, Critical (all categories) |

4. When finished, select Save.

<b>Enable application logging (Linux/Container)</b>

1. In App Service logs set the Application logging option to File System.

2. In Quota (MB), specify the disk quota for the application logs. In Retention Period (Days), set the number of days the logs should be retained.

3. When finished, select Save.

<b>Enable web server logging</b>

1. For Web server logging, select Storage to store logs on blob storage, or File System to store logs on the App Service file system.

2. In Retention Period (Days), set the number of days the logs should be retained.

3. When finished, select Save.

<b>Stream logs</b>

Before you stream logs in real time, enable the log type that you want. Any information written to files ending in .txt, .log, or .htm that are stored in the /LogFiles directory (d:/home/logfiles) is streamed by App Service.

Some types of logging buffer write to the log file, which can result in out of order events in the stream. For example, an application log entry that occurs when a user visits a page may be displayed in the stream before the corresponding HTTP log entry for the page request.

- Azure portal - To stream logs in the Azure portal, navigate to your app and select Log stream.

- Azure CLI - To stream logs live in Cloud Shell, use the following command:

```
az webapp log tail --name appname --resource-group myResourceGroup
```

- Local console - To stream logs in the local console, install Azure CLI and sign in to your account. Once signed in, follow the instructions shown for Azure CLI.

<b>Access log files</b>
If you configure the Azure Storage blobs option for a log type, you need a client tool that works with Azure Storage.

For logs stored in the App Service file system, the easiest way is to download the ZIP file in the browser at:

- Linux/container apps: https://<app-name>.scm.azurewebsites.net/api/logs/docker/zip
- Windows apps: https://<app-name>.scm.azurewebsites.net/api/dump

For Linux/container apps, the ZIP file contains console output logs for both the docker host and the docker container. For a scaled-out app, the ZIP file contains one set of logs for each instance. In the App Service file system, these log files are the contents of the /home/LogFiles directory.

### 📒 Configure security certificates <a name="1024"></a>

You've been asked to help secure information being transmitted between your company’s app and the customer. Azure App Service has tools that let you create, upload, or import a private certificate or a public certificate into App Service.

A certificate uploaded into an app is stored in a deployment unit that is bound to the app service plan's resource group and region combination (internally called a webspace). This makes the certificate accessible to other apps in the same resource group and region combination.

The table below details the options you have for adding certificates in App Service:

![](images/6.png)

<b>Private certificate requirements</b>

The free App Service managed certificate and the App Service certificate already satisfy the requirements of App Service. If you want to use a private certificate in App Service, your certificate must meet the following requirements:

- Exported as a password-protected PFX file, encrypted using triple DES.
- Contains private key at least 2048 bits long.
- Contains all intermediate certificates and the root certificate in the certificate chain.

To secure a custom domain in a TLS binding, the certificate has other requirements:

- Contains an Extended Key Usage for server authentication (OID = 1.3.6.1.5.5.7.3.1)
- Signed by a trusted certificate authority

<b>Creating a free managed certificate</b>

To create custom TLS/SSL bindings or enable client certificates for your App Service app, your App Service plan must be in the Basic, Standard, Premium, or Isolated tier.

The free App Service managed certificate is a turn-key solution for securing your custom DNS name in App Service. It's a TLS/SSL server certificate that's fully managed by App Service and renewed continuously and automatically in six-month increments, 45 days before expiration. You create the certificate and bind it to a custom domain, and let App Service do the rest.

Before you create a free managed certificate, make sure you have met the prerequisites for your app. Free certificates are issued by DigiCert. For some domains, you must explicitly allow DigiCert as a certificate issuer by creating a CAA domain record with the value: 0 issue digicert.com. Azure fully manages the certificates on your behalf, so any aspect of the managed certificate, including the root issuer, can change at anytime. These changes are outside your control. Make sure to avoid hard dependencies and "pinning" practice certificates to the managed certificate or any part of the certificate hierarchy.

The free certificate comes with the following limitations:

- Doesn't support wildcard certificates.
- Doesn't support usage as a client certificate by using certificate thumbprint, which is planned for deprecation and removal.
- Doesn't support private DNS.
- Isn't exportable.
- Isn't supported in an App Service Environment (ASE).
- Only supports alphanumeric characters, dashes (-), and periods (.).
- Only custom domains of length up to 64 characters are supported.

<b>Import an App Service Certificate</b>

If you purchase an App Service Certificate from Azure, Azure manages the following tasks:

- Takes care of the purchase process from certificate provider.
- Performs domain verification of the certificate.
- Maintains the certificate in Azure Key Vault.
- Manages certificate renewal.
- Synchronize the certificate automatically with the imported copies in App Service apps.

If you already have a working App Service certificate, you can:

- Import the certificate into App Service.
- Manage the certificate, such as renew, rekey, and export it.

App Service Certificates are not supported in Azure National Clouds at this time.

### 📒 Scale apps in Azure App Service <a name="103"></a>

Azure App Service supports two options for scaling out your web apps automatically:

- `Autoscaling with Azure autoscale`. Autoscaling makes scaling decisions based on rules that you define.
- `Azure App Service automatic scaling`. Automatic scaling makes scaling decisions for you based on the parameters that you select.

`Autoscaling` is a cloud system or process that adjusts available resources based on the current demand. Autoscaling performs scaling in and out, as opposed to scaling up and down.

Autoscaling can be triggered according to a schedule, or by assessing whether the system is running short on resources. For example, autoscaling could be triggered if CPU utilization grows, memory occupancy increases, the number of incoming requests to a service appears to be surging, or some combination of factors.

<b>Azure App Service autoscaling</b>

Autoscaling in Azure App Service monitors the resource metrics of a web app as it runs. It detects situations where other resources are required to handle an increasing workload, and ensures those resources are available before the system becomes overloaded.

Autoscaling responds to changes in the environment by adding or removing web servers and balancing the load between them. Autoscaling doesn't have any effect on the CPU power, memory, or storage capacity of the web servers powering the app, it only changes the number of web servers.

Autoscaling makes its decisions based on rules that you define. A rule specifies the threshold for a metric, and triggers an autoscale event when this threshold is crossed. Autoscaling can also deallocate resources when the workload has diminished.

Define your autoscaling rules carefully. For example, a Denial of Service attack will likely result in a large-scale influx of incoming traffic. Trying to handle a surge in requests caused by a DoS attack would be fruitless and expensive. These requests aren't genuine, and should be discarded rather than processed. A better solution is to implement detection and filtering of requests that occur during such an attack before they reach your service.

Autoscaling provides elasticity for your services. For example, you might expect increased/reduced activity for a business app during holidays.

Autoscaling improves availability and fault tolerance. It can help ensure that client requests to a service won't be denied because an instance is either not able to acknowledge the request in a timely manner, or because an overloaded instance has crashed.

Autoscaling works by adding or removing web servers. If your web apps perform resource-intensive processing as part of each request, then autoscaling might not be an effective approach. In these situations, manually scaling up may be necessary. For example, if a request sent to a web app involves performing complex processing over a large dataset, depending on the instance size, this single request could exhaust the processing and memory capacity of the instance.

Autoscaling isn't the best approach to handling long-term growth. You might have a web app that starts with a few users, but increases in popularity over time. Autoscaling has an overhead associated with monitoring resources and determining whether to trigger a scaling event. In this scenario, if you can anticipate the rate of growth, manually scaling the system over time may be a more cost effective approach.

The number of instances of a service is also a factor. You might expect to run only a few instances of a service most of the time. However, in this situation, your service is susceptible to downtime or lack of availability whether autoscaling is enabled or not. The fewer the number of instances initially, the less capacity you have to handle an increasing workload while autoscaling spins up more instances.

<b>Azure App Service automatic scaling</b>

`Automatic scaling` is a new scale-out option that automatically handles scaling decisions for your web apps and App Service Plans. It's different from the pre-existing Azure autoscale, which lets you define scaling rules based on schedules and resources. With automatic scaling, you can adjust scaling settings to improve your app's performance and avoid cold start issues. The platform prewarms instances to act as a buffer when scaling out, ensuring smooth performance transitions. You're charged per second for every instance, including prewarmed instances.

Here are a few scenarios where you should scale-out automatically:

- You don't want to set up autoscale rules based on resource metrics.
- You want your web apps within the same App Service Plan to scale differently and independently of each other.
- Your web app is connected to a database or legacy system, which may not scale as fast as the web app. Scaling automatically allows you to set the maximum number of instances your App Service Plan can scale to. This setting helps the web app to not overwhelm the backend.

### 📒 Identify autoscale factors <a name="1031"></a>

Autoscaling enables you to specify the conditions under which a web app should be scaled out, and back in again. Effective autoscaling ensures sufficient resources are available to handle large volumes of requests at peak times, while managing costs when the demand drops.

You can configure autoscaling to detect when to scale in and out according to a combination of factors, based on resource usage. You can also configure autoscaling to occur according to a schedule.

Autoscaling is a feature of the App Service Plan used by the web app. When the web app scales out, Azure starts new instances of the hardware defined by the App Service Plan to the app.

To prevent runaway autoscaling, an App Service Plan has an instance limit. Plans in more expensive pricing tiers have a higher limit. Autoscaling can't create more instances than this limit. Not all App Service Plan pricing tiers support autoscaling.

<b>Autoscale conditions</b>

You indicate how to autoscale by creating autoscale conditions. Azure provides two options for autoscaling:

- Scale based on a metric, such as the length of the disk queue, or the number of HTTP requests awaiting processing.
- Scale to a specific instance count according to a schedule. For example, you can arrange to scale out at a particular time of day, or on a specific date or day of the week. You also specify an end date, and the system scales back in at this time.

Scaling to a specific instance count only enables you to scale out to a defined number of instances. If you need to scale out incrementally, you can combine metric and schedule-based autoscaling in the same autoscale condition. So, you could arrange for the system to scale out if the number of HTTP requests exceeds some threshold, but only between certain hours of the day.

You can create multiple autoscale conditions to handle different schedules and metrics. Azure autoscales your service when any of these conditions apply. An App Service Plan also has a default condition that is used if none of the other conditions are applicable. This condition is always active and doesn't have a schedule.

<b> Metrics for autoscale rules </b>

Autoscaling by metric requires that you define one or more autoscale rules. An autoscale rule specifies a metric to monitor, and how autoscaling should respond when this metric crosses a defined threshold. The metrics you can monitor for a web app are:

- CPU Percentage. This metric is an indication of the CPU utilization across all instances. A high value shows that instances are becoming CPU-bound, which could cause delays in processing client requests.
- Memory Percentage. This metric captures the memory occupancy of the application across all instances. A high value indicates that free memory could be running low, and could cause one or more instances to fail.
- Disk Queue Length. This metric is a measure of the number of outstanding I/O requests across all instances. A high value means that disk contention could be occurring.
- Http Queue Length. This metric shows how many client requests are waiting for processing by the web app. If this number is large, client requests might fail with HTTP 408 (Timeout) errors.
- Data In. This metric is the number of bytes received across all instances.
- Data Out. This metric is the number of bytes sent by all instances.

You can also scale based on metrics for other Azure services. For example, if the web app processes requests received from a Service Bus Queue, you might want to spin up more instances of a web app if the number of items held in an Azure Service Bus Queue exceeds a critical length.

<b>How an autoscale rule analyzes metrics</b>

Autoscaling works by analyzing trends in metric values over time across all instances. Analysis is a multi-step process.

In the first step, an autoscale rule aggregates the values retrieved for a metric for all instances across a period of time known as the time grain. Each metric has its own intrinsic time grain, but in most cases this period is 1 minute. The aggregated value is known as the time aggregation. The options available are Average, Minimum, Maximum, Sum, Last, and Count.

An interval of one minute is a short interval in which to determine whether any change in metric is long-lasting enough to make autoscaling worthwhile. So, an autoscale rule performs a second step that performs a further aggregation of the value calculated by the time aggregation over a longer, user-specified period, known as the Duration. The minimum Duration is 5 minutes. If the Duration is set to 10 minutes for example, the autoscale rule aggregates the 10 values calculated for the time grain.

The aggregation calculation for the Duration can be different from the time grain. For example, if the time aggregation is Average and the statistic gathered is CPU Percentage across a one-minute time grain, each minute the average CPU percentage utilization across all instances for that minute is calculated. If the time grain statistic is set to Maximum, and the Duration of the rule is set to 10 minutes, the maximum of the 10 average values for the CPU percentage utilization is to determine whether the rule threshold has been crossed.

<b>Autoscale actions</b>

When an autoscale rule detects that a metric has crossed a threshold, it can perform an autoscale action. An autoscale action can be scale-out or scale-in. A scale-out action increases the number of instances, and a scale-in action reduces the instance count. An autoscale action uses an operator (such as less than, greater than, equal to, and so on) to determine how to react to the threshold. Scale-out actions typically use the greater than operator to compare the metric value to the threshold. Scale-in actions tend to compare the metric value to the threshold with the less than operator. An autoscale action can also set the instance count to a specific level, rather than incrementing or decrementing the number available.

An autoscale action has a cool down period, specified in minutes. During this interval, the scale rule won't be triggered again. This is to allow the system to stabilize between autoscale events. Remember that it takes time to start up or shut down instances, and so any metrics gathered might not show any significant changes for several minutes. The minimum cool down period is five minutes.

<b>Pairing autoscale rules</b>

You should plan for scaling-in when a workload decreases. Consider defining autoscale rules in pairs in the same autoscale condition. One autoscale rule should indicate how to scale the system out when a metric exceeds an upper threshold. Then other rule should define how to scale the system back in again when the same metric drops below a lower threshold.

<b>Combining autoscale rules </b>

A single autoscale condition can contain several autoscale rules (for example, a scale-out rule and the corresponding scale-in rule). However, the autoscale rules in an autoscale condition don't have to be directly related. You could define the following four rules in the same autoscale condition:

- If the HTTP queue length exceeds 10, scale out by 1
- If the CPU utilization exceeds 70%, scale out by 1
- If the HTTP queue length is zero, scale in by 1
- If the CPU utilization drops below 50%, scale in by 1

When determining whether to scale out, the autoscale action is performed if any of the scale-out rules are met (HTTP queue length exceeds 10 or CPU utilization exceeds 70%). When scaling in, the autoscale action runs only if all of the scale-in rules are met (HTTP queue length drops to zero and CPU utilization falls below 50%). If you need to scale in if only one of the scale-in rules are met, you must define the rules in separate autoscale conditions.

### 📒 Enable autoscale <a name="1032"></a>

To get started with autoscaling navigate to your App Service plan in the Azure portal and select `Scale out (App Service plan)` in the `Settings` group in the left navigation pane.

Not all pricing tiers support autoscaling. The development pricing tiers are either limited to a single instance (the F1 and D1 tiers), or they only provide manual scaling (the B1 tier). If you've selected one of these tiers, you must first scale up to the S1 or any of the P level production tiers.

By default, an App Service Plan only implements manual scaling. Selecting `Custom autoscale` reveals condition groups you can use to manage your scale settings.

<b>Add scale conditions</b>

Once you enable autoscaling, you can edit the automatically created default scale condition, and you can add your own custom scale conditions. Remember that each scale condition can either scale based on a metric, or scale to a specific instance count. The Default scale condition is executed when none of the other scale conditions are active.

A metric-based scale condition can also specify the minimum and maximum number of instances to create. The maximum number can't exceed the limits defined by the pricing tier. Additionally, all scale conditions other than the default may include a schedule indicating when the condition should be applied.

<b>Create scale rules</b>

A metric-based scale condition contains one or more scale rules. You use the Add a rule link to add your own custom rules. You define the criteria that indicate when a rule should trigger an autoscale action, and the autoscale action to be performed (scale out or scale in) using the metrics, aggregations, operators, and thresholds described earlier.

<b>Monitor autoscaling activity</b>

The Azure portal enables you to track when autoscaling has occurred through the Run history chart. This chart shows how the number of instances varies over time, and which autoscale conditions caused each change.
You can use the Run history chart with the metrics shown on the Overview page to correlate the autoscaling events with resource utilization.

### 📒 Autoscale best practices <a name="1033"></a>

If you're not following good practices when creating autoscale settings, you can create conditions that lead to undesirable results. In this unit, you'll learn how to avoid creating rules that conflict with each other.

<b>Autoscale concepts</b>

- An autoscale setting scales instances horizontally, which is out by increasing the instances and in by decreasing the number of instances. An autoscale setting has a maximum, minimum, and default value of instances.

- An autoscale job always reads the associated metric to scale by, checking if it has crossed the configured threshold for scale-out or scale-in.

- All thresholds are calculated at an instance level. For example, "scale out by one instance when average CPU > 80% when instance count is 2", means to scale out when the average CPU across all instances is greater than 80%.

- All autoscale successes and failures are logged to the Activity Log. You can then configure an activity log alert so that you can be notified via email, SMS, or webhooks whenever there's activity.

<b>Autoscale best practices</b>

Use the following best practices as you create your autoscale rules.

> Ensure the maximum and minimum values are different and have an adequate margin between them

If you have a setting that has minimum=two, maximum=two and the current instance count is two, no scale action can occur. Keep an adequate margin between the maximum and minimum instance counts, which are inclusive. Autoscale always scales between these limits.

> Choose the appropriate statistic for your diagnostics metric

For diagnostics metrics, you can choose among Average, Minimum, Maximum and Total as a metric to scale by. The most common statistic is Average.

> Choose the thresholds carefully for all metric types

We recommend carefully choosing different thresholds for scale-out and scale-in based on practical situations.

We don't recommend autoscale settings like the following examples with the same or similar threshold values for out and in conditions:

- Increase instances by one count when Thread Count >= 600
- Decrease instances by one count when Thread Count <= 600
  Let's look at an example of what can lead to a behavior that may seem confusing. Consider the following sequence.

1. Assume there are two instances to begin with and then the average number of threads per instance grows to 625.
2. Autoscale scales out adding a third instance.
3. Next, assume that the average thread count across instance falls to 575.
4. Before scaling in, autoscale tries to estimate what the final state will be if it scaled in. For example, 575 x 3 (current instance count) = 1,725 / 2 (final number of instances when scaled in) = 862.5 threads. This means autoscale would have to immediately scale out again even after it scaled in, if the average thread count remains the same or even falls only a small amount. However, if it scaled out again, the whole process would repeat, leading to an infinite loop.
5. To avoid this situation (termed "flapping"), autoscale doesn't scale in at all. Instead, it skips and reevaluates the condition again the next time the service's job executes. This can confuse many people because autoscale wouldn't appear to work when the average thread count was 575.

Estimation during a scale-in is intended to avoid "flapping" situations, where scale-in and scale out actions continually go back and forth. Keep this behavior in mind when you choose the same thresholds for scale-out and in.

We recommend choosing an adequate margin between the scale-out and in thresholds. As an example, consider the following better rule combination.

- Increase instances by 1 count when CPU% >= 80
- Decrease instances by 1 count when CPU% <= 60

In this case

1. Assume there are 2 instances to start with.
2. If the average CPU% across instances goes to 80, autoscale scales out adding a third instance.
3. Now assume that over time the CPU% falls to 60.
4. Autoscale's scale-in rule estimates the final state if it were to scale-in. For example, 60 x 3 (current instance count) = 180 / 2 (final number of instances when scaled in) = 90. So autoscale doesn't scale-in because it would have to scale out again immediately. Instead, it skips scaling in.
5. The next time autoscale checks, the CPU continues to fall to 50. It estimates again - 50 x 3 instance = 150 / 2 instances = 75, which is below the scale-out threshold of 80, so it scales in successfully to 2 instances.

> Considerations for scaling when multiple rules are configured in a profile

There are cases where you may have to set multiple rules in a profile. The following set of autoscale rules are used by services when multiple rules are set.

On scale-out, autoscale runs if any rule is met. On scale-in, autoscale require all rules to be met.

To illustrate, assume that you have the following four autoscale rules:

- If CPU < 30%, scale-in by 1
- If Memory < 50%, scale-in by 1
- If CPU > 75%, scale out by 1
- If Memory > 75%, scale out by 1

Then the following occurs:

- If CPU is 76% and Memory is 50%, we scale out.
- If CPU is 50% and Memory is 76% we scale out.

On the other hand, if CPU is 25% and memory is 51% autoscale doesn't scale-in. An automatic scale-in would occur if the CPU is 29% and the Memory is 49% since both of the scale-in rules would be true.

> Always select a safe default instance count

The default instance count is important because autoscale scales your service to that count when metrics aren't available. Therefore, select a default instance count that's safe for your workloads.

> Configure autoscale notifications

Autoscale posts to the Activity Log if any of the following conditions occur:

- Autoscale issues a scale operation
- Autoscale service successfully completes a scale action
- Autoscale service fails to take a scale action.
- Metrics aren't available for autoscale service to make a scale decision.
- Metrics are available (recovery) again to make a scale decision.

You can also use an Activity Log alert to monitor the health of the autoscale engine. In addition to using activity log alerts, you can also configure email or webhook notifications to get notified for successful scale actions via the notifications tab on the autoscale setting.

### 📒 Azure App Service deployment slots <a name="104"></a>

<b>Explore staging environments</b>

The Standard, Premium, and Isolated App Service plan tiers support deployment to a specified deployment slot instead of the default production slot. `Deployment slots` are live apps with their own host names. You can deploy your web app, web app on Linux, mobile back end, or API app to a staging environment. App content and configurations elements can be swapped between two deployment slots, including the production slot.

Deploying your application to a non-production slot has the following benefits:

- You can validate app changes in a staging deployment slot before swapping it with the production slot.
- Deploying an app to a slot first and swapping it into production makes sure that all instances of the slot are warmed up before being swapped into production. This eliminates downtime when you deploy your app. The traffic redirection is seamless, and no requests are dropped because of swap operations. You can automate this entire workflow by configuring auto swap when pre-swap validation isn't needed.
- After a swap, the previous production app is located in the staging slot. If the changes swapped into the production slot aren't as you expect, you can perform the same swap immediately to get your "last known good site" back.

Each App Service plan tier supports a different number of deployment slots. There's no extra charge for using deployment slots. To find out the number of slots your app's tier supports, visit App Service limits.

To scale your app to a different tier, make sure that the target tier supports the number of slots your app already uses. For example, if your app has more than five slots, you can't scale it down to the Standard tier, because the Standard tier supports only five deployment slots.

When you create a new deployment slot the new slot has no content, even if you clone the settings from a different slot. You can deploy to the slot from a different repository branch or a different repository.

<b>Examine slot swapping</b>

When you swap two slots (for example, from a staging slot to the production slot), App Service completes the following process to ensure that the target slot doesn't experience downtime:

1. Apply the following settings from the target slot (for example, the production slot) to all instances of the source slot:

- Slot-specific app settings and connection strings, if applicable.
- Continuous deployment settings, if enabled.
- App Service authentication settings, if enabled.

Any of these cases trigger all instances in the source slot to restart. During swap with preview, this marks the end of the first phase. The swap operation is paused, and you can validate that the source slot works correctly with the target slot's settings.

2. Wait for every instance in the source slot to complete its restart. If any instance fails to restart, the swap operation reverts all changes to the source slot and stops the operation.

3. If local cache is enabled, trigger local cache initialization by making an HTTP request to the application root ("/") on each instance of the source slot. Wait until each instance returns any HTTP response. Local cache initialization causes another restart on each instance.

4. If auto swap is enabled with custom warm-up, trigger Application Initiation by making an HTTP request to the application root ("/") on each instance of the source slot.

- If applicationInitialization isn't specified, trigger an HTTP request to the application root of the source slot on each instance.

- If an instance returns any HTTP response, it's considered to be warmed up.

If all instances on the source slot are warmed up successfully, swap the two slots by switching the routing rules for the two slots. After this step, the target slot (for example, the production slot) has the app that's previously warmed up in the source slot.

Now that the source slot has the pre-swap app previously in the target slot, perform the same operation by applying all settings and restarting the instances.

At any point of the swap operation, all work of initializing the swapped apps happens on the source slot. The target slot remains online while the source slot is being prepared and warmed up, regardless of where the swap succeeds or fails. To swap a staging slot with the production slot, make sure that the production slot is always the target slot. This way, the swap operation doesn't affect your production app.

When you clone configuration from another deployment slot, the cloned configuration is editable. Some configuration elements follow the content across a swap (not slot specific), whereas other configuration elements stay in the same slot after a swap (slot specific). The following table shows the settings that change when you swap slots.

![](images/7.png)

Features marked with an asterisk (\*) are planned to be unswapped.

To make settings swappable, add the app setting WEBSITE_OVERRIDE_PRESERVE_DEFAULT_STICKY_SLOT_SETTINGS in every slot of the app and set its value to 0 or false. These settings are either all swappable or not at all. You can't make just some settings swappable and not the others. Managed identities are never swapped and are not affected by this override app setting.

To configure an app setting or connection string to stick to a specific slot (not swapped), go to the Configuration page for that slot. Add or edit a setting, and then select Deployment slot setting. Selecting this check box tells App Service that the setting isn't swappable.

<b>Swap deployment slots</b>

You can swap deployment slots on your app's Deployment slots page and the Overview page. Before you swap an app from a deployment slot into production, make sure that production is your target slot and that all settings in the source slot are configured exactly as you want to have them in production.

<b>Manually swapping deployment slots </b>

To swap deployment slots:

1. Go to your app's Deployment slots page and select Swap. The Swap dialog box shows settings in the selected source and target slots that will be changed.

2. Select the desired Source and Target slots. Usually, the target is the production slot. Also, select the Source Changes and Target Changes tabs and verify that the configuration changes are expected. When you're finished, you can swap the slots immediately by selecting Swap.

To see how your target slot would run with the new settings before the swap actually happens, don't select Swap, but follow the instructions in Swap with preview below.

3. When you're finished, close the dialog box by selecting Close.

<b> Swap with preview (multi-phase swap) </b>

Before you swap into production as the target slot, validate that the app runs with the swapped settings. The source slot is also warmed up before the swap completion, which is desirable for mission-critical applications.

When you perform a swap with preview, App Service performs the same swap operation but pauses after the first step. You can then verify the result on the staging slot before completing the swap.

If you cancel the swap, App Service reapplies configuration elements to the source slot.

To swap with preview:

1. Follow the steps above in Swap deployment slots but select the Perform swap with preview checkbox. The dialog box shows you how the configuration in the source slot changes in phase 1, and how the source and target slot change in phase 2.

2. When you're ready to start the swap, select Start Swap.

When phase 1 finishes, you're notified in the dialog box. Preview the swap in the source slot by going to https://<app_name>-<source-slot-name>.azurewebsites.net.

3. When you're ready to complete the pending swap, select Complete Swap in Swap action and select Complete Swap.

To cancel a pending swap, select Cancel Swap instead.

4. When you're finished, close the dialog box by selecting Close.

<b>Configure auto </b>

Auto swap streamlines Azure DevOps Services scenarios where you want to deploy your app continuously with zero cold starts and zero downtime for customers of the app. When auto swap is enabled from a slot into production, every time you push your code changes to that slot, App Service automatically swaps the app into production after it's warmed up in the source slot.

Auto swap isn't currently supported in web apps on Linux and Web App for Containers.

To configure auto swap:

1. Go to your app's resource page and select the deployment slot you want to configure to auto swap. The setting is on the Configuration > General settings page.

2. Set Auto swap enabled to On. Then select the desired target slot for Auto swap deployment slot, and select Save on the command bar.

3. Execute a code push to the source slot. Auto swap happens after a short time, and the update is reflected at your target slot's URL.

<b>Specify custom warm-</b>

Some apps might require custom warm-up actions before the swap. The applicationInitialization configuration element in web.config lets you specify custom initialization actions. The swap operation waits for this custom warm-up to finish before swapping with the target slot. Here's a sample web.config fragment.

```
<system.webServer>
    <applicationInitialization>
        <add initializationPage="/" hostName="[app hostname]" />
        <add initializationPage="/Home/About" hostName="[app hostname]" />
    </applicationInitialization>
</system.webServer>
```

For more information on customizing the applicationInitialization element, see Most common deployment slot swap failures and how to fix them.

You can also customize the warm-up behavior with one or both of the following app settings:

- WEBSITE_SWAP_WARMUP_PING_PATH: The path to ping to warm up your site. Add this app setting by specifying a custom path that begins with a slash as the value. An example is /statuscheck. The default value is /.
- WEBSITE_SWAP_WARMUP_PING_STATUSES: Valid HTTP response codes for the warm-up operation. Add this app setting with a comma-separated list of HTTP codes. An example is 200,202 . If the returned status code isn't in the list, the warmup and swap operations are stopped. By default, all response codes are valid.
- WEBSITE_WARMUP_PATH: A relative path on the site that should be pinged whenever the site restarts (not only during slot swaps). Example values include /statuscheck or the root path, /.

<b>Roll back and monitor a swap </b>

If any errors occur in the target slot (for example, the production slot) after a slot swap, restore the slots to their pre-swap states by swapping the same two slots immediately.

If the swap operation takes a long time to complete, you can get information on the swap operation in the activity log.

1. On your app's resource page in the portal, in the left pane, select Activity log.

2. A swap operation appears in the log query as Swap Web App Slots. You can expand it and select one of the suboperations or errors to see the details.

<b>Route traffic</b>

By default, all client requests to the app's production URL `(http://<app_name>.azurewebsites.net)` are routed to the production slot. You can route a portion of the traffic to another slot. This feature is useful if you need user feedback for a new update, but you're not ready to release it to production.

<b>Route production traffic automatically</b>

To route production traffic automatically:

1. Go to your app's resource page and select Deployment slots.

2. In the Traffic % column of the slot you want to route to, specify a percentage (between 0 and 100) to represent the amount of total traffic you want to route. Select Save.

After the setting is saved, the specified percentage of clients is randomly routed to the non-production slot.

After a client is automatically routed to a specific slot, it's "pinned" to that slot for the life of that client session. On the client browser, you can see which slot your session is pinned to by looking at the x-ms-routing-name cookie in your HTTP headers. A request that's routed to the "staging" slot has the cookie x-ms-routing-name=staging. A request that's routed to the production slot has the cookie x-ms-routing-name=self.

<b>Route production traffic manually</b>

In addition to automatic traffic routing, App Service can route requests to a specific slot. This is useful when you want your users to be able to opt in to or opt out of your beta app. To route production traffic manually, you use the x-ms-routing-name query parameter.

To let users opt out of your beta app, for example, you can put this link on your webpage:

```
<a href="<webappname>.azurewebsites.net/?x-ms-routing-name=self">Go back to production app</a>
```

The string x-ms-routing-name=self specifies the production slot. After the client browser accesses the link, it's redirected to the production slot. Every subsequent request has the x-ms-routing-name=self cookie that pins the session to the production slot.

To let users opt in to your beta app, set the same query parameter to the name of the non-production slot. Here's an example:

```
<webappname>.azurewebsites.net/?x-ms-routing-name=staging
```

By default, new slots are given a routing rule of 0%, a default value is displayed in grey. When you explicitly set the routing rule value to 0% it's displayed in black, your users can access the staging slot manually by using the x-ms-routing-name query parameter. But they won't be routed to the slot automatically because the routing percentage is set to 0. This is an advanced scenario where you can "hide" your staging slot from the public while allowing internal teams to test changes on the slot.

---

## 2. Functions <a name="2"></a>

### 📒 Explore Azure Function <a name="201"></a>

`Azure Functions` is a serverless solution that allows you to write less code, maintain less infrastructure, and save on costs. Instead of worrying about deploying and maintaining servers, the cloud infrastructure provides all the up-to-date resources needed to keep your applications running.

We often build systems to react to a series of critical events. Whether you're building a web API, responding to database changes, processing IoT data streams, or even managing message queues - every application needs a way to run some code as these events occur.

Azure Functions supports `triggers`, which are ways to start execution of your code, and `bindings`, which are ways to simplify coding for input and output data. There are other integration and automation services in Azure and they all can solve integration problems and automate business processes. They can all define input, actions, conditions, and output.

<b>Compare Azure Functions and Azure Logic Apps</b>

Both Functions and Logic Apps are Azure Services that enable serverless workloads. Azure Functions is a serverless compute service, whereas Azure Logic Apps is a serverless workflow integration platform. Both can create complex orchestrations. An `orchestration` is a collection of functions or steps, called actions in Logic Apps, that are executed to accomplish a complex task.

For Azure Functions, you develop orchestrations by writing code and using the `Durable Functions extension`. For Logic Apps, you create orchestrations by using a GUI or editing configuration files.

The following table lists some of the key differences between Functions and Logic Apps:
![](images/8.png)

<b>Compare Azure Functions and WebJobs</b>

Like Azure Functions, Azure App Service WebJobs with the WebJobs SDK is a code-first integration service that is designed for developers. Both are built on Azure App Service and support features such as source control integration, authentication, and monitoring with Application Insights integration.

Azure Functions is built on the WebJobs SDK, so it shares many of the same event triggers and connections to other Azure services. Here are some factors to consider when you're choosing between Azure Functions and WebJobs with the WebJobs SDK:

![](images/9.png)

Azure Functions offers more developer productivity than Azure App Service WebJobs does. It also offers more options for programming languages, development environments, Azure service integration, and pricing. For most scenarios, it's the best choice.

<b>Compare Azure Functions hosting options</b>

When you create a function app in Azure, you must choose a `hosting plan` for your app. Azure provides you with these hosting options for your function code:

![](images/10.png)

Azure App Service infrastructure facilitates Azure Functions hosting on both Linux and Windows virtual machines. The hosting option you choose dictates the following behaviors:

- How your function app is scaled.
- The resources available to each function app instance.
- Support for advanced functionality, such as Azure Virtual Network connectivity.
- Support for Linux containers.

The plan you choose also impacts the costs for running your function code.

Following is a summary of the benefits of the various hosting options:

> Consumption plan

The Consumption plan is the default hosting plan. Pay for compute resources only when your functions are running (pay-as-you-go) with automatic scale. On the Consumption plan, instances of the Functions host are dynamically added and removed based on the number of incoming events.

> Flex Consumption plan

Get high scalability with compute choices, virtual networking, and pay-as-you-go billing. On the Flex Consumption plan, instances of the Functions host are dynamically added and removed based on the configured per instance concurrency and the number of incoming events. You can reduce cold starts by specifying the number of pre-provisioned (always ready) instances. Scales automatically based on demand.

> Premium plan

Automatically scales based on demand using prewarmed workers, which run applications with no delay after being idle, runs on more powerful instances, and connects to virtual networks.

Consider the Azure Functions Premium plan in the following situations:

- Your function apps run continuously, or nearly continuously.
- You want more control of your instances and want to deploy multiple function apps on the same plan with event-driven scaling.
- You have a high number of small executions and a high execution bill, but low GB seconds in the Consumption plan.
- You need more CPU or memory options than are provided by consumption plans.
- Your code needs to run longer than the maximum execution time allowed on the Consumption plan.
- You require virtual network connectivity.
- You want to provide a custom Linux image in which to run your functions.

> Dedicated plan

Run your functions within an App Service plan at regular App Service plan rates. Best for long-running scenarios where Durable Functions can't be used.

Consider an App Service plan in the following situations:

- You must have fully predictable billing, or you need to manually scale instances.
- You want to run multiple web apps and function apps on the same plan
- You need access to larger compute size choices.
- Full compute isolation and secure network access provided by an App Service Environment (ASE).
- High memory usage and high scale (ASE).

> Container Apps

Create and deploy containerized function apps in a fully managed environment hosted by Azure Container Apps.

Use the Azure Functions programming model to build event-driven, serverless, cloud native function apps. Run your functions alongside other microservices, APIs, websites, and workflows as container-hosted programs.

Consider hosting your functions on Container Apps in the following situations:

- You want to package custom libraries with your function code to support line-of-business apps.
- You need to migration code execution from on-premises or legacy apps to cloud native microservices running in containers.
- You want to avoid the overhead and complexity of managing Kubernetes clusters and dedicated compute.
- You need the high-end processing power provided by dedicated CPU compute resources for your functions.

<b>Function app timeout duration</b>

The `functionTimeout` property in the `host.json` project file specifies the timeout duration for functions in a function app. This property applies specifically to function executions. After the trigger starts function execution, the function needs to return/respond within the timeout duration.

The following table shows the default and maximum values (in minutes) for specific plans:

![](images/11.png)

1. Regardless of the function app timeout setting, 230 seconds is the maximum amount of time that an HTTP triggered function can take to respond to a request.
2. The default timeout for version 1.x of the Functions runtime is unlimited.
3. Guaranteed for up to 60 minutes. OS and runtime patching, vulnerability patching, and scale in behaviors can still cancel function executions.
4. In a Flex Consumption plan, the host doesn't enforce an execution time limit. However, there are currently no guarantees because the platform might need to terminate your instances during scale-in, deployments, or to apply updates.
5. When the minimum number of replicas is set to zero, the default timeout depends on the specific triggers used in the app.

<b>Scale Azure Functions</b>

The following table compares the scaling behaviors of the various hosting plans. Maximum instances are given on a per-function app (Consumption) or per-plan (Premium/Dedicated) basis, unless otherwise indicated.

![](images/12.png)

1. During scale-out, there's currently a limit of 500 instances per subscription per hour for Linux 1. apps on a Consumption plan.
2. In some regions, Linux apps on a Premium plan can scale to 100 instances.
3. For specific limits for the various App Service plan options, see the App Service plan limits.
4. On Container Apps, you can set the maximum number of replicas, which is honored as long as there's enough cores quota available

### 📒 Develop Azure Function <a name="202"></a>

A function app provides an `execution context` in Azure in which your functions run. As such, it's the unit of deployment and management for your functions. A `function app` is composed of one or more individual functions that are managed, deployed, and scaled together. All of the functions in a function app share the same pricing plan, deployment method, and runtime version. Think of a function app as a way to organize and collectively manage your functions. In Functions 2.x all functions in a function app must be authored in the same language. In previous versions of the Azure Functions runtime, this wasn't required.

<b>Develop and test Azure Functions locally</b>

Functions make it easy to use your favorite code editor and development tools to create and test functions on your local computer. Your local functions can connect to live Azure services, and you can debug them on your local computer using the full Functions runtime.
The way in which you develop functions on your local computer depends on your language and tooling preferences. For more information, see <a href="https://learn.microsoft.com/en-us/azure/azure-functions/functions-develop-local">Code and test Azure Functions locally</a>.
Because of limitations on editing function code in the Azure portal, you should develop your functions locally and publish your code project to a function app in Azure.

A Functions project directory contains the following files in the project root folder, regardless of language:

- host.json
- local.settings.json
- Other files in the project depend on your language and specific functions.

The `host.json` metadata file contains configuration options that affect all functions in a function app instance. Other function app configuration options are managed depending on where the function app runs:

- Deployed to Azure: in your application settings
- On your local computer: in the `local.settings.json` file.

Configurations in `host.json` related to bindings are applied equally to each function in the function app. You can also override or apply settings per environment using application settings. To learn more, see <a href="https://learn.microsoft.com/en-us/azure/azure-functions/functions-host-json">the host.json reference.</a>

The `local.settings.json` file stores app settings, and settings used by local development tools. Settings in the `local.settings.json` file are used only when you're running your project locally. When you publish your project to Azure, be sure to also add any required settings to the app settings for the function app. Because the `local.settings.json` may contain secrets, such as connection strings, you should never store it in a remote repository.

When you develop your functions locally, any local settings required by your app must also be present in the app settings of the deployed function app. You can also download current settings from the function app to your local project.

<b>Create triggers and bindings</b>

A `trigger` defines how a function is invoked and a function must have exactly one trigger. Triggers have associated data, which is often provided as the payload of the function.

`Binding to a function` is a way of declaratively connecting another resource to the function; bindings might be connected as `input bindings`, `output bindings`, or both. Data from bindings is provided to the function as parameters. You can mix and match different bindings to suit your needs. Bindings are optional and a function might have one or multiple input and/or output bindings. Triggers and bindings let you avoid hardcoding access to other services. Your function receives data (for example, the content of a queue message) in function parameters. You send data (for example, to create a queue message) by using the return value of the function.

Triggers and bindings are defined differently depending on the development language. For languages that rely on `function.json` (JavaScript/PowerShell/Python/TypeScript), the portal provides a UI for adding bindings in the Integration tab. You can also edit the file directly in the portal in the `Code + test` tab of your function.

In .NET and Java, the parameter type defines the data type for input data. For instance, use `string` to bind to the text of a queue trigger, a byte array to read as binary, and a custom type to deserialize to an object. Since .NET class library functions and Java functions don't rely on function.json for binding definitions, they can't be created and edited in the portal. C# portal editing is based on C# script, which uses function.json instead of attributes.

For languages that are dynamically typed such as JavaScript, use the `dataType` property in the function.json file. For example, to read the content of an HTTP request in binary format, set `dataType` to `binary`:

```
{
    "dataType": "binary",
    "type": "httpTrigger",
    "name": "req",
    "direction": "in"
}
```

Other options for `dataType` are `stream` and `string`.

<b>Binding direction</b>

All triggers and bindings have a direction property in the function.json file:

- For triggers, the direction is always `in`
- Input and output bindings use `in` and `out`
- Some bindings support a special direction `inout`. If you use inout, only the `Advanced editor` is available via the `Integrate` tab in the portal.

When you use attributes in a class library to configure triggers and bindings, the direction is provided in an attribute constructor or inferred from the parameter type.

<b>Connect functions to Azure services</b>

As a security best practice, Azure Functions takes advantage of the application settings functionality of Azure App Service to help you more securely store strings, keys, and other tokens required to connect to other services. Application settings in Azure are stored encrypted and can be accessed at runtime by your app as environment variable `name value` pairs. For triggers and bindings that require a connection property, you set the application setting name instead of the actual connection string. You can't configure a binding directly with a connection string or key.

The default configuration provider uses environment variables. These variables are defined in application settings when running in the Azure and in the local settings file when developing locally.

Some connections in Azure Functions are configured to use an identity instead of a secret. Support depends on the extension using the connection. In some cases, a connection string may still be required in Functions even though the service to which you're connecting supports identity-based connections.

When running in a Consumption or Elastic Premium plan, your app uses the WEBSITE_AZUREFILESCONNECTIONSTRING and WEBSITE_CONTENTSHARE settings when connecting to Azure Files on the storage account used by your function app. Azure Files doesn't support using managed identity when accessing the file share.

When hosted in the Azure Functions service, identity-based connections use a managed identity. The system-assigned identity is used by default, although a user-assigned identity can be specified with the `credential` and `clientID` properties. Configuring a user-assigned identity with a resource ID is not supported. When run in other contexts, such as local development, your developer identity is used instead, although this can be customized.

Identities must have permissions to perform the intended actions. This is typically done by assigning a role in Azure RBAC or specifying the identity in an access policy, depending on the service to which you're connecting.

Some permissions might be exposed by the target service that are not necessary for all contexts. Where possible, adhere to the principle of least privilege, granting the identity only required privileges.

### 📒 Example <a name="2021"></a>

> 1.

Suppose you want to write a new row to Azure Table storage whenever a new message appears in Azure Queue storage. This scenario can be implemented using an Azure Queue storage trigger and an Azure Table storage output binding.

Here's a function.json file for this scenario.

```
{
  "disabled": false,
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        },
        {
          "tableName": "Person",
          "connection": "MyStorageConnectionAppSetting",
          "name": "tableBinding",
          "type": "table",
          "direction": "out"
        }
  ]
}
```

The first element in the bindings array is the `Queue storage trigger`. The `type` and `direction` properties identify the trigger. The `name` property identifies the function parameter that receives the queue message content. The name of the queue to monitor is in `queueName`, and the connection string is in the app setting identified by `connection`.

The second element in the bindings array is the `Azure Table Storage` output binding. The `type` and `direction` properties identify the binding. The `name` property specifies how the function provides the new table row, in this case by using the function return value. The name of the table is in `tableName`, and the connection string is in the app setting identified by `connection`.

> 2. <a href="https://learn.microsoft.com/en-us/training/modules/develop-azure-functions/5-create-function-visual-studio-code">Create an Azure Function by using VS Code and C#</a>

---

## 3. Blob storage <a name="3"></a>

### 📒 Explore Azure Blob storage <a name="301"></a>

`Azure Blob storage` is Microsoft's object storage solution for the cloud. Blob storage is optimized for storing massive amounts of unstructured data. Unstructured data is data that doesn't adhere to a particular data model or definition, such as text or binary data.

Blob storage is designed for:

- Serving images or documents directly to a browser.
- Storing files for distributed access.
- Streaming video and audio.
- Writing to log files.
- Storing data for backup and restore, disaster recovery, and archiving.
- Storing data for analysis by an on-premises or Azure-hosted service.

Users or client applications can access objects in Blob storage via HTTP/HTTPS, from anywhere in the world. Objects in Blob storage are accessible via the Azure Storage REST API, Azure PowerShell, Azure CLI, or an Azure Storage client library.

An Azure Storage account is the top-level container for all of your Azure Blob storage. The storage account provides a unique namespace for your Azure Storage data that is accessible from anywhere in the world over HTTP or HTTPS.

<b>Types of storage accounts</b>

Azure Storage offers two performance levels of storage accounts, standard and premium. Each performance level supports different features and has its own pricing model.

- `Standard`: This is the standard general-purpose v2 account and is recommended for most scenarios using Azure Storage.
- `Premium`: Premium accounts offer higher performance by using solid-state drives. If you create a premium account you can choose between three account types, block blobs, page blobs, or file shares.

The following table describes the types of storage accounts recommended by Microsoft for most scenarios using Blob storage.

![](images/13.png)

<b>Access tiers for block blob data</b>

Azure Storage provides different options for accessing block blob data based on usage patterns. Each access tier in Azure Storage is optimized for a particular pattern of data usage. By selecting the right access tier for your needs, you can store your block blob data in the most cost-effective manner.

The available access tiers are:

- The `Hot` access tier, which is optimized for frequent access of objects in the storage account. The Hot tier has the highest storage costs, but the lowest access costs. New storage accounts are created in the hot tier by default.

- The `Cool` access tier, which is optimized for storing large amounts of data that is infrequently accessed and stored for a minimum of 30 days. The Cool tier has lower storage costs and higher access costs compared to the Hot tier.

- The `Cold` access tier, which is optimized for storing data that is infrequently accessed and stored for a minimum of 90 days. The cold tier has lower storage costs and higher access costs compared to the cool tier.

- The `Archive` tier, which is available only for individual block blobs. The archive tier is optimized for data that can tolerate several hours of retrieval latency and remains in the Archive tier for a minimum 180 days. The archive tier is the most cost-effective option for storing data, but accessing that data is more expensive than accessing data in the hot or cool tiers.

If there's a change in the usage pattern of your data, you can switch between these access tiers at any time.

<b>Blob storage resource types</b>

Blob storage offers three types of resources:

- The `storage account`.
- A `container` in the storage account
- A `blob` in a container

A `storage account` provides a unique namespace in Azure for your data. Every object that you store in Azure Storage has an address that includes your unique account name. The combination of the account name and the Azure Storage blob endpoint forms the base address for the objects in your storage account.

For example, if your storage account is named mystorageaccount, then the default endpoint for Blob storage is:

```
http://mystorageaccount.blob.core.windows.net
```

A `container` organizes a set of blobs, similar to a directory in a file system. A storage account can include an unlimited number of containers, and a container can store an unlimited number of blobs.

A container name must be a valid DNS name, as it forms part of the unique URI (Uniform resource identifier) used to address the container or its blobs. Follow these rules when naming a container:

- Container names can be between 3 and 63 characters long.
- Container names must start with a letter or number, and can contain only lowercase letters, numbers, and the dash (-) character.
- Two or more consecutive dash characters aren't permitted in container names.

The URI for a container is similar to:

```
https://myaccount.blob.core.windows.net/mycontainer
```

Azure Storage supports three types of `blobs`:

- `Block blobs` store text and binary data. Block blobs are made up of blocks of data that can be managed individually. Block blobs can store up to about 190.7 TiB.
- `Append blobs` are made up of blocks like block blobs, but are optimized for append operations. Append blobs are ideal for scenarios such as logging data from virtual machines.
- `Page blobs` store random access files up to 8 TB in size. Page blobs store virtual hard drive (VHD) files and serve as disks for Azure virtual machines.

The URI for a blob is similar to:

```
https://myaccount.blob.core.windows.net/mycontainer/myblob

https://myaccount.blob.core.windows.net/mycontainer/myvirtualdirectory/myblob

```

Azure Storage uses service-side encryption (SSE) to automatically encrypt your data when it's persisted to the cloud. Azure Storage encryption protects your data and to help you to meet your organizational security and compliance commitments.

Microsoft recommends using service-side encryption to protect your data for most scenarios. However, the Azure Storage client libraries for Blob Storage and Queue Storage also provide client-side encryption for customers who need to encrypt data on the client.

### 📒 Azure Storage security features <a name="3011"></a>

Azure Storage uses `service-side encryption` (SSE) to automatically encrypt your data when it's persisted to the cloud. Azure Storage encryption protects your data and to help you to meet your organizational security and compliance commitments.

Microsoft recommends using service-side encryption to protect your data for most scenarios. However, the Azure Storage client libraries for Blob Storage and Queue Storage also provide client-side encryption for customers who need to encrypt data on the client.

Azure Storage automatically encrypts your data when persisting it to the cloud. Encryption protects your data and helps you meet your organizational security and compliance commitments. Data in Azure Storage is encrypted and decrypted transparently using 256-bit Advanced Encryption Standard (AES) encryption, one of the strongest block ciphers available, and is Federal Information Processing Standards (FIPS) 140-2 compliant. Azure Storage encryption is similar to BitLocker encryption on Windows.

Azure Storage encryption is enabled for all storage accounts and can't be disabled. Because your data is secured by default, you don't need to modify your code or applications to take advantage of Azure Storage encryption.

Data in a storage account is encrypted regardless of performance tier, access tier, or deployment model. All new and existing block blobs, append blobs, and page blobs are encrypted, including blobs in the archive tier. All Azure Storage redundancy options support encryption, and all data in both the primary and secondary regions is encrypted when geo-replication is enabled. All Azure Storage resources are encrypted, including blobs, disks, files, queues, and tables. All object metadata is also encrypted. There's no extra cost for Azure Storage encryption.

Data in a new storage account is encrypted with `Microsoft-managed keys` by default. You can continue to rely on Microsoft-managed keys for the encryption of your data, or you can manage encryption with your own keys. If you choose to manage encryption with your own keys, you have two options. You can use either type of key management, or both:

- You can specify a `customer-managed key` to use for encrypting and decrypting data in Blob Storage and in Azure Files.Customer-managed keys must be stored in Azure Key Vault or Azure Key Vault Managed Hardware Security Model (HSM).

- You can specify a `customer-provided key` on Blob Storage operations. A client can include an encryption key on a read/write request for granular control over how blob data is encrypted and decrypted.

The following table compares key management options for Azure Storage encryption.

![](images/14.png)

<b>Client-side encryption</b>

The Azure Blob Storage client libraries for .NET, Java, and Python support encrypting data within client applications before uploading to Azure Storage, and decrypting data while downloading to the client. The Queue Storage client libraries for .NET and Python also support client-side encryption.

The Blob Storage and Queue Storage client libraries uses AES in order to encrypt user data. There are two versions of client-side encryption available in the client libraries:

- Version 2 uses Galois/Counter Mode (GCM) mode with AES. The Blob Storage and Queue Storage SDKs support client-side encryption with v2.
- Version 1 uses Cipher Block Chaining (CBC) mode with AES. The Blob Storage, Queue Storage, and Table Storage SDKs support client-side encryption with v1.

### 📒 Manage the Azure Blob storage lifecycle <a name="302"></a>

Data sets have unique lifecycles. Early in the lifecycle, people access some data often. But the need for access drops drastically as the data ages. Some data stays idle in the cloud and is rarely accessed once stored. Some data expires days or months after creation, while other data sets are actively read and modified throughout their lifetimes.

Azure storage offers different `access tiers`, allowing you to store blob object data in the most cost-effective manner. Available access tiers include:

- `Hot` - An online tier optimized for storing data that is accessed frequently.
- `Cool` - An online tier optimized for storing data that is infrequently accessed and stored for a minimum of 30 days.
- `Cold tier` - An online tier optimized for storing data that is infrequently accessed and stored for a minimum of 90 days. The cold tier has lower storage costs and higher access costs compared to the cool tier.
- `Archive` - An offline tier optimized for storing data that is rarely accessed and stored for at least 180 days with flexible latency requirements, on the order of hours.

Data storage limits are set at the account level and not per access tier. You can choose to use all of your limit in one tier or across all three tiers.

Azure Blob Storage `lifecycle management` offers a rule-based policy that you can use to transition blob data to the appropriate access tiers or to expire data at the end of the data lifecycle.

With the lifecycle management policy, you can:

- Transition blobs from cool to hot immediately when accessed, to optimize for performance.
- Transition current versions of a blob, previous versions of a blob, or blob snapshots to a cooler storage tier if these objects aren't accessed or modified for a period of time, to optimize for cost.
- Delete current versions of a blob, previous versions of a blob, or blob snapshots at the end of their lifecycles.
- Apply rules to an entire storage account, to select containers, or to a subset of blobs using name prefixes or blob index tags as filters.

Consider a scenario where data is frequently accessed during the early stages of the lifecycle, but only occasionally after two weeks. Beyond the first month, the data set is rarely accessed. In this scenario, hot storage is best during the early stages. Cool storage is most appropriate for occasional access. Archive storage is the best tier option after the data ages over a month. By moving data to the appropriate storage tier based on its age with lifecycle management policy rules, you can design the least expensive solution for your needs.

<b>Blob storage lifecycle policies</b>

A lifecycle management policy is a collection of rules in a JSON document. Each rule definition within a policy includes a filter set and an action set. The filter set limits rule actions to a certain set of objects within a container or objects names. The action set applies the tier or delete actions to the filtered set of objects:

```
{
  "rules": [
    {
      "name": "rule1",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {...}
    },
    {
      "name": "rule2",
      "type": "Lifecycle",
      "definition": {...}
    }
  ]
}
```

A policy is a collection of rules. Each rule within the policy has several parameters.

Each rule definition includes a `filter set` and an `action set`. The filter set limits rule actions to a certain set of objects within a container or objects names. The action set applies the tier or delete actions to the filtered set of objects.

The following sample rule filters the account to run the actions on objects that exist inside sample-container and start with blob1.

- Tier blob to cool tier 30 days after last modification
- Tier blob to archive tier 90 days after last modification
- Delete blob 2,555 days (seven years) after last modification
- Delete blob snapshots 90 days after snapshot creation

```
{
  "rules": [
    {
      "enabled": true,
      "name": "sample-rule",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "version": {
            "delete": {
              "daysAfterCreationGreaterThan": 90
            }
          },
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90,
              "daysAfterLastTierChangeGreaterThan": 7
            },
            "delete": {
              "daysAfterModificationGreaterThan": 2555
            }
          }
        },
        "filters": {
          "blobTypes": [
            "blockBlob"
          ],
          "prefixMatch": [
            "sample-container/blob1"
          ]
        }
      }
    }
  ]
}
```

`Rule filters` limit rule actions to a subset of blobs within the storage account. If more than one filter is defined, a logical AND runs on all filters. Filters include:
![](images/15.png)

`Rule actions` are applied to the filtered blobs when the run condition is met. Lifecycle management supports tiering and deletion of blobs and deletion of blob snapshots. Define at least one action for each rule on blobs or blob snapshots.

![](images/16.png)

If you define more than one action on the same blob, lifecycle management applies the least expensive action to the blob. For example, action delete is cheaper than action tierToArchive. Action tierToArchive is cheaper than action tierToCool.

The run conditions are based on age. Base blobs use the last modified time to track age, and blob snapshots use the snapshot creation time to track age.

![](images/17.png)

You can add, edit, or remove a policy by using any of the following methods:

- Azure portal
- Azure PowerShell
- Azure CLI
- REST APIs

The following are the steps and some examples for the Portal and Azure CLI.

There are two ways to add a policy through the Azure portal: Azure portal List view, and Azure portal Code view. Following is an example of how to add a policy in the Azure portal Code view.

`Azure portal Code view`

1. In the Azure portal, navigate to your storage account.
2. Under Data management, select Lifecycle Management to view or change lifecycle management policies.
3. Select the Code View tab. On this tab, you can define a lifecycle management policy in JSON.

The following JSON is an example of a policy that moves a block blob whose name begins with log to the cool tier if it has been more than 30 days since the blob was modified.

```
{
  "rules": [
    {
      "enabled": true,
      "name": "move-to-cool",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            }
          }
        },
        "filters": {
          "blobTypes": [
            "blockBlob"
          ],
          "prefixMatch": [
            "sample-container/log"
          ]
        }
      }
    }
  ]
}
```

To add a lifecycle management policy with `Azure CLI`, write the policy to a JSON file, then call the az storage account management-policy create command to create the policy.

```
az storage account management-policy create
     --account-name <storage-account>
     --policy @policy.json
     --resource-group <resource-group>
```

A lifecycle management policy must be read or written in full. Partial updates aren't supported.

<b>Rehydrate blob data from the archive tier</b>

While a blob is in the archive access tier, it's considered to be offline and can't be read or modified. In order to read or modify data in an archived blob, you must first rehydrate the blob to an online tier, either the hot or cool tier. There are two options for rehydrating a blob that is stored in the archive tier:

- `Copy an archived blob to an online tier`: You can rehydrate an archived blob by copying it to a new blob in the hot or cool tier with the `Copy Blob` or `Copy Blob from URL` operation. Microsoft recommends this option for most scenarios.

- `Change a blob's access tier to an online tier`: You can rehydrate an archived blob to hot or cool by changing its tier using the `Set Blob Tier` operation.

Rehydrating a blob from the archive tier can take several hours to complete. Microsoft recommends rehydrating larger blobs for optimal performance. Rehydrating several small blobs concurrently might require extra time.

When you rehydrate a blob, you can set the priority for the rehydration operation via the optional `x-ms-rehydrate-priority` header on a `Set Blob Tier` or `Copy Blob/Copy Blob From URL` operation. Rehydration priority options include:

- `Standard priority`: The rehydration request is processed in the order it was received and might take up to 15 hours.
- `High priority`: The rehydration request is prioritized over standard priority requests and might complete in under one hour for objects under 10 GB in size.

To check the rehydration priority while the rehydration operation is underway, call Get Blob Properties to return the value of the `x-ms-rehydrate-priority` header. The rehydration priority property returns either ``Standard` or `High`.

Copy an archived blob to an online tier
The first option for moving a blob from the archive tier to an online tier is to copy the archived blob to a new destination blob that is in either the hot or cool tier. You can use the Copy Blob operation to copy the blob. When you copy an archived blob to a new blob in an online tier, the source blob remains unmodified in the archive tier. You must copy the archived blob to a new blob with a different name or to a different container. You can't overwrite the source blob by copying to the same blob.

Rehydrating an archived blob by copying it to an online destination tier is supported within the same storage account only for service versions earlier than 2021-02-12. Beginning with service version 2021-02-12, you can rehydrate an archived blob by copying it to a different storage account, as long as the destination account is in the same region as the source account.

Change a blob's access tier to an online tier
The second option for rehydrating a blob from the archive tier to an online tier is to change the blob's tier by calling Set Blob Tier. With this operation, you can change the tier of the archived blob to either hot or cool.

Once a Set Blob Tier request is initiated, it can't be canceled. During the rehydration operation, the blob's access tier setting continues to show as archived until the rehydration process is complete.

To learn how to rehydrate a blob by changing its tier to an online tier, see Rehydrate a blob by changing its tier.

Caution

Changing a blob's tier doesn't affect its last modified time. If there is a lifecycle management policy in effect for the storage account, then rehydrating a blob with Set Blob Tier can result in a scenario where the lifecycle policy moves the blob back to the archive tier after rehydration because the last modified time is beyond the threshold set for the policy.

### 📒 Azure Blob storage client library for .NET <a name="303"></a>

The Azure Storage client libraries for .NET offer a convenient interface for making calls to Azure Storage. The latest version of the Azure Storage client library is version 12.x. Microsoft recommends using version 12.x for new applications.

The following table lists the basic classes, along with a brief description:

![](images/18.png)

The following packages contain the classes used to work with Blob Storage data resources:

- Azure.Storage.Blobs: Contains the primary classes (client objects) that you can use to operate on the service, containers, and blobs.
- Azure.Storage.Blobs.Specialized: Contains classes that you can use to perform operations specific to a blob type, such as block blobs.
- Azure.Storage.Blobs.Models: All other utility classes, structures, and enumeration types.

> <a href="https://learn.microsoft.com/en-us/training/modules/work-azure-blob-storage/3-create-client-object">Create a client object with C#</a>

> <a href="https://learn.microsoft.com/en-us/training/modules/work-azure-blob-storage/4-develop-blob-storage-dotnet">Create Blob storage resources by using the .NET client library</a>

> <a href="https://learn.microsoft.com/en-us/training/modules/work-azure-blob-storage/5-manage-container-properties-metadata-dotnet">Manage container properties and metadata by using .NET</a>

### 📒 Set and retrieve properties and metadata for blob resources by using REST <a name="304"></a>

Containers and blobs support custom metadata, represented as HTTP headers. Metadata headers can be set on a request that creates a new container or blob resource, or on a request that explicitly creates a property on an existing resource.

Metadata headers are name/value pairs. The format for the header is:

```
x-ms-meta-name:string-value
```

Beginning with version 2009-09-19, metadata names must adhere to the naming rules for C# identifiers. Names are case-insensitive. Metadata names preserve the case with which they were created, but are case-insensitive when set or read. If two or more metadata headers with the same name are submitted for a resource, the Blob service returns status code 400 (Bad Request). The metadata consists of name/value pairs. The total size of all metadata pairs can be up to 8 KB in size. Metadata name/value pairs are valid HTTP headers, and so they adhere to all restrictions governing HTTP headers.

Metadata on a blob or container resource can be retrieved or set directly, without returning or altering the content of the resource. Metadata values can only be read or written in full; partial updates aren't supported. Setting metadata on a resource overwrites any existing metadata values for that resource.

<b>Retrieving properties and metadata</b>

The GET and HEAD operations both retrieve metadata headers for the specified container or blob. These operations return headers only; they don't return a response body. The URI syntax for retrieving metadata headers on a container is as follows:

```
GET/HEAD https://myaccount.blob.core.windows.net/mycontainer?restype=container
```

The URI syntax for retrieving metadata headers on a blob is as follows:

```
GET/HEAD https://myaccount.blob.core.windows.net/mycontainer/myblob?comp=metadata
```

<b>Setting Metadata Headers</b>

The PUT operation sets metadata headers on the specified container or blob, overwriting any existing metadata on the resource. Calling PUT without any headers on the request clears all existing metadata on the resource.

The URI syntax for setting metadata headers on a container is as follows:

```
PUT https://myaccount.blob.core.windows.net/mycontainer?comp=metadata&restype=container
```

The URI syntax for setting metadata headers on a blob is as follows:

```
PUT https://myaccount.blob.core.windows.net/mycontainer/myblob?comp=metadata
```

<b>Standard HTTP properties for containers and blobs</b>

Containers and blobs also support certain standard HTTP properties. Properties and metadata are both represented as standard HTTP headers; the difference between them is in the naming of the headers. Metadata headers are named with the header prefix x-ms-meta- and a custom name. Property headers use standard HTTP header names, as specified in the Header Field Definitions section 14 of the HTTP/1.1 protocol specification.

The standard HTTP headers supported on containers include:

- ETag
- Last-Modified

The standard HTTP headers supported on blobs include:

- ETag
- Last-Modified
- Content-Length
- Content-Type
- Content-MD5
- Content-Encoding
- Content-Language
- Cache-Control
- Origin
- Range

### 📒 Examples <a name="3041"></a>

> 1. <a href="https://learn.microsoft.com/en-us/training/modules/work-azure-blob-storage/4-develop-blob-storage-dotnet">Create Blob storage resources by using the .NET client library</a>

---

## 4. Cosmos DB <a name="4"></a>

### 📒 Explore Azure Cosmos DB <a name="401"></a>

`Azure Cosmos DB` is a globally distributed database system that allows you to read and write data from the local replicas of your database and it transparently replicates the data to all the regions associated with your Cosmos account. The Azure Cosmos DB account is the fundamental unit of global distribution and high availability. Your Azure Cosmos DB account contains a unique `Domain Name System` (DNS) name and you can manage an account by using the Azure portal or the Azure CLI, or by using different language-specific SDKs. For globally distributing your data and throughput across multiple Azure regions, you can add and remove Azure regions to your account at any time.

An `Azure Cosmos DB container` is the fundamental unit of scalability. You can virtually have an unlimited provisioned throughput (RU/s) and storage on a container. Azure Cosmos DB transparently partitions your container using the logical partition key that you specify in order to elastically scale your provisioned throughput and storage. Currently, you can create a maximum of 50 Azure Cosmos DB accounts under an Azure subscription (can be increased via support request). After you create an account under your Azure subscription, you can manage the data in your account by creating databases, containers, and items.

You can create one or multiple Azure Cosmos DB databases under your account. A database is analogous to a namespace. A database is the unit of management for a set of Azure Cosmos DB containers.

An Azure Cosmos DB container is where data is stored. Unlike most relational databases, which scale up with larger sizes of virtual machines, Azure Cosmos DB scales out. Data is stored on one or more servers called `partitions`. To increase partitions, you increase throughput, or they grow automatically as storage increases. This relationship provides a virtually unlimited amount of throughput and storage for a container. When you create a container, you need to supply a partition key. The `partition key` is a property that you select from your items to help Azure Cosmos DB distribute the data efficiently across partitions. Azure Cosmos DB uses the value of this property to route data to the appropriate partition to be written, updated, or deleted. You can also use the partition key in the `WHERE` clause in queries for efficient data retrieval.

The underlying storage mechanism for data in Azure Cosmos DB is called a `physical partition`. Physical partitions can have a throughput amount up to 10,000 Request Units per second, and they can store up to 50 GB of data. Azure Cosmos DB abstracts this partitioning concept with a logical partition, which can store up to 20 GB of data.

When you create a container, you configure throughput in one of the following modes:

- `Dedicated throughput`: The throughput on a container is exclusively reserved for that container. There are two types of dedicated throughput: standard and autoscale.

- `Shared throughput`: Throughput is specified at the database level and then shared with up to 25 containers within the database. Sharing of throughput excludes containers that are configured with their own dedicated throughput.

Depending on which API you use, individual data entities can be represented in various ways:

| Azure Cosmos DB entity | API for NoSQL | API for Cassandra | API for MongoDB | API for Gremlin | API for Table |
| ---------------------- | ------------- | ----------------- | --------------- | --------------- | ------------- |
| Azure Cosmos DB item   | Item          | Row               | Document        | Node or edge    | Item          |

### 📒 Consistency levels <a name="4011"></a>

Azure Cosmos DB approaches data consistency as a spectrum of choices instead of two extremes. Strong consistency and eventual consistency are at the ends of the spectrum, but there are many consistency choices along the spectrum. Developers can use these options to make precise choices and granular tradeoffs with respect to high availability and performance.

Azure Cosmos DB offers five well-defined levels. From strongest to weakest, the levels are:

- Strong
- Bounded staleness
- Session
- Consistent prefix
- Eventual

Each level provides availability and performance tradeoffs.

The consistency levels are region-agnostic and are guaranteed for all operations, regardless of:

- The region where the reads and writes are served
- The number of regions associated with your Azure Cosmos DB account
- Whether your account is configured with a single or multiple write regions.

Read consistency applies to a single read operation scoped within a partition-key range or a logical partition.

<b> Choose the right consistency level</b>

Each of the consistency models can be used for specific real-world scenarios. Each provides precise availability and performance tradeoffs backed by comprehensive SLAs. The following simple considerations help you make the right choice in many common scenarios. You can configure the `default consistency level` on your Azure Cosmos DB account at any time. The default consistency level configured on your account applies to all Azure Cosmos DB databases and containers under that account. All reads and queries issued against a container or a database use the specified consistency level by default. Read consistency applies to a single read operation scoped within a logical partition. The read operation can be issued by a remote client or a stored procedure.

Azure Cosmos DB guarantees that 100 percent of read requests meet the consistency guarantee for the consistency level chosen. The precise definitions of the five consistency levels in Azure Cosmos DB using the TLA+ specification language are provided in the azure-cosmos-tla GitHub repo.

`Strong consistency` offers a linearizability guarantee. Linearizability refers to serving requests concurrently. The reads are guaranteed to return the most recent committed version of an item. A client never sees an uncommitted or partial write. Users are always guaranteed to read the latest committed write.

In `bounded staleness consistency`, the lag of data between any two regions is always less than a specified amount. The amount can be "K" versions (that is, "updates") of an item or by "T" time intervals, whichever is reached first. In other words, when you choose bounded staleness, the maximum "staleness" of the data in any region can be configured in two ways:

- The number of versions (K) of the item
- The time interval (T) reads might lag behind the writes

Bounded Staleness is beneficial primarily to single-region write accounts with two or more regions. If the data lag in a region (determined per physical partition) exceeds the configured staleness value, writes for that partition are throttled until staleness is back within the configured upper bound. For a single-region account, Bounded Staleness provides the same write consistency guarantees as Session and Eventual Consistency. With Bounded Staleness, data is replicated to a local majority (three replicas in a four replica set) in the single region.

In `session consistency`, within a single client session, reads are guaranteed to honor the read-your-writes, and write-follows-reads guarantees. This guarantee assumes a single “writer” session or sharing the session token for multiple writers. Like all consistency levels weaker than Strong, writes are replicated to a minimum of three replicas (in a four replica set) in the local region, with asynchronous replication to all other regions.

In `consistent prefix`, updates made as single document writes see eventual consistency. Updates made as a batch within a transaction, are returned consistent to the transaction in which they were committed. Write operations within a transaction of multiple documents are always visible together. Assume two write operations are performed on documents Doc 1 and Doc 2, within transactions T1 and T2. When client does a read in any replica, the user sees either “Doc 1 v1 and Doc 2 v1” or “Doc 1 v2 and Doc 2 v2”, but never “Doc 1 v1 and Doc 2 v2” or “Doc 1 v2 and Doc 2 v1” for the same read or query operation.

In `eventual consistency`, there's no ordering guarantee for reads. In the absence of any further writes, the replicas eventually converge.Eventual consistency is the weakest form of consistency because a client might read the values that are older than the ones it read before. Eventual consistency is ideal where the application doesn't require any ordering guarantees. Examples include count of Retweets, Likes, or nonthreaded comments

### 📒 Supported APIs <a name="4012"></a>

Azure Cosmos DB offers multiple database APIs, which include NoSQL, MongoDB, PostgreSQL, Cassandra, Gremlin, and Table. By using these APIs, you can model real world data using documents, key-value, graph, and column-family data models. These APIs allow your applications to treat Azure Cosmos DB as if it were various other databases technologies, without the overhead of management, and scaling approaches. Azure Cosmos DB helps you to use the ecosystems, tools, and skills you already have for data modeling and querying with its various APIs.

All the APIs offer automatic scaling of storage and throughput, flexibility, and performance guarantees. There's no one best API, and you may choose any one of the APIs to build your application.

API for NoSQL is native to Azure Cosmos DB.

API for MongoDB, PostgreSQL, Cassandra, Gremlin, and Table implement the wire protocol of open-source database engines. These APIs are best suited if the following conditions are true:

- If you have existing MongoDB, PostgreSQL Cassandra, or Gremlin applications
- If you don't want to rewrite your entire data access layer
- If you want to use the open-source developer ecosystem, client-drivers, expertise, and resources for your database

<b>API for NoSQL</b>

The Azure Cosmos DB API for NoSQL stores data in document format. It offers the best end-to-end experience as we have full control over the interface, service, and the SDK client libraries. Any new feature that is rolled out to Azure Cosmos DB is first available on API for NoSQL accounts. NoSQL accounts provide support for querying items using the Structured Query Language (SQL) syntax.

<b>API for MongoDB</b>

The Azure Cosmos DB API for MongoDB stores data in a document structure, via BSON format. It's compatible with MongoDB wire protocol; however, it doesn't use any native MongoDB related code. The API for MongoDB is a great choice if you want to use the broader MongoDB ecosystem and skills, without compromising on using Azure Cosmos DB features.

<b>API for PostgreSQL</b>

Azure Cosmos DB for PostgreSQL is a managed service for running PostgreSQL at any scale, with the Citus open source superpower of distributed tables. It stores data either on a single node, or distributed in a multi-node configuration.

<b>API for Apache Cassandra</b>

The Azure Cosmos DB API for Cassandra stores data in column-oriented schema. Apache Cassandra offers a highly distributed, horizontally scaling approach to storing large volumes of data while offering a flexible approach to a column-oriented schema. API for Cassandra in Azure Cosmos DB aligns with this philosophy to approaching distributed NoSQL databases. This API for Cassandra is wire protocol compatible with native Apache Cassandra.

<b>API for Apache Gremlin</b>

The Azure Cosmos DB API for Gremlin allows users to make graph queries and stores data as edges and vertices.

Use the API for Gremlin for scenarios:

- Involving dynamic data
- Involving data with complex relations
- Involving data that is too complex to be modeled with relational databases
- If you want to use the existing Gremlin ecosystem and skills

<b>API for Table</b>

The Azure Cosmos DB API for Table stores data in key/value format. If you're currently using Azure Table storage, you might see some limitations in latency, scaling, throughput, global distribution, index management, low query performance. API for Table overcomes these limitations and the recommendation is to migrate your app if you want to use the benefits of Azure Cosmos DB. API for Table only supports OLTP scenarios.

### 📒 Request units <a name="4013"></a>

With Azure Cosmos DB, you pay for the throughput you provision and the storage you consume on an hourly basis. Throughput must be provisioned to ensure that sufficient system resources are available for your Azure Cosmos database always. The cost of all database operations is normalized in Azure Cosmos DB and expressed by request units (or RUs, for short). A `request unit` represents the system resources such as CPU, IOPS, and memory that are required to perform the database operations supported by Azure Cosmos DB. The cost to do a point read, which is fetching a single item by its ID and partition key value, for a 1-KB item is 1RU. All other database operations are similarly assigned a cost using RUs. No matter which API you use to interact with your Azure Cosmos container, costs are measured by RUs. Whether the database operation is a write, point read, or query, costs are measured in RUs.

The following image shows the high-level idea of RUs:

![](images/19.png)

The type of Azure Cosmos DB account you're using determines the way consumed RUs get charged. There are three modes in which you can create an account:

- `Provisioned throughput mode`: In this mode, you provision the number of RUs for your application on a per-second basis in increments of 100 RUs per second. To scale the provisioned throughput for your application, you can increase or decrease the number of RUs at any time in increments or decrements of 100 RUs. You can make your changes either programmatically or by using the Azure portal. You can provision throughput at container and database granularity level.

- `Serverless mode`: In this mode, you don't have to provision any throughput when creating resources in your Azure Cosmos DB account. At the end of your billing period, you get billed for the number of request units that have been consumed by your database operations.

- `Autoscale mode`: In this mode, you can automatically and instantly scale the throughput (RU/s) of your database or container based on its usage. This scaling operation doesn't affect the availability, latency, throughput, or performance of the workload. This mode is well suited for mission-critical workloads that have variable or unpredictable traffic patterns, and require SLAs on high performance and scale.

### 📒 Examples <a name="4014"></a>

> 1. <a href="https://learn.microsoft.com/en-us/training/modules/explore-azure-cosmos-db/8-create-cosmos-db-resources-portal">Create Azure Cosmos DB resources by using the Azure portal</a>

### 📒 Develop solutions that use Cosmos DB <a name="402"></a>

### 📒 Explore Microsoft .NET SDK v3 <a name="4021"></a>

> <a href="https://learn.microsoft.com/en-us/training/modules/work-with-cosmos-db/2-cosmos-db-dotnet-overview">Explore Microsoft .NET SDK v3</a>

### 📒 Examples <a name="4022"></a>

> <a href="https://learn.microsoft.com/en-us/training/modules/work-with-cosmos-db/3-exercise-work-cosmos-db-dotnet">Create resources by using the Microsoft .NET SDK v3</a>

### 📒 Create stored procedures <a name="4023"></a>

> <a href="https://learn.microsoft.com/en-us/training/modules/work-with-cosmos-db/4-cosmos-db-stored-procedures">Create stored procedures</a>

### 📒 Create triggers and user-defined functions <a name="4024"></a>

> <a href="https://learn.microsoft.com/en-us/training/modules/work-with-cosmos-db/5-cosmos-db-triggers-user-defined-functions">Create triggers and user-defined functions</a>

### 📒 Change feed in Azure Cosmos DB <a name="4024"></a>

> <a href="https://learn.microsoft.com/en-us/training/modules/work-with-cosmos-db/6-cosmos-db-change-feed">Change feed in Azure Cosmos DB</a>

## 5. Implement containerized solutions <a name="5"></a>

### 📒 Discover the Azure Container Registry <a name="501"></a>

`Azure Container Registry` (ACR) is a managed registry service based on the open-source `Docker Registry 2.0`. Create and maintain Azure container registries to store and manage your container images and related artifacts. Use the ACR service with your existing container development and deployment pipelines, or use Azure Container Registry Tasks to build container images in Azure. Build on demand, or fully automate builds with triggers such as source code commits and base image updates.

Pull images from an Azure container registry to various deployment targets:

- `Scalable orchestration systems` that manage containerized applications across clusters of hosts, including Kubernetes, DC/OS, and Docker Swarm.
- `Azure services` that support building and running applications at scale, including Azure Kubernetes Service (AKS), App Service, Batch, and Service Fabric.

Developers can also push to a container registry as part of a container development workflow. For example, target a container registry from a continuous integration and delivery tool such as Azure Pipelines or Jenkins. Configure ACR Tasks to automatically rebuild application images when their base images are updated, or automate image builds when your team commits code to a Git repository. Create multi-step tasks to automate building, testing, and patching multiple container images in parallel in the cloud.

Azure Container Registry is available in multiple service tiers. These tiers provide predictable pricing and several options for aligning to the capacity and usage patterns of your private Docker registry in Azure.

- `Basic`. A cost-optimized entry point for developers learning about Azure Container Registry. Basic registries have the same programmatic capabilities as Standard and Premium (such as Microsoft Entra authentication integration, image deletion, and webhooks). However, the included storage and image throughput are most appropriate for lower usage scenarios.
- `Standard`. Standard registries offer the same capabilities as Basic, with increased included storage and image throughput. Standard registries should satisfy the needs of most production scenarios.
- `Premium`. Premium registries provide the highest amount of included storage and concurrent operations, enabling high-volume scenarios. In addition to higher image throughput, Premium adds features such as: geo-replication for managing a single registry across multiple regions, content trust for image tag signing, and private link with private endpoints to restrict access to the registry.

When images are grouped in a repository, each image is a read-only snapshot of a Docker-compatible container. Azure container registries can include both Windows and Linux images. In addition to Docker container images, Azure Container Registry stores related content formats such as Helm charts and images built to the <a href="https://github.com/opencontainers/image-spec/blob/main/spec.md">Open Container Initiative (OCI) Image Format Specification</a>. Use <a href="https://learn.microsoft.com/en-us/azure/container-registry/container-registry-tasks-overview">Azure Container Registry Tasks</a> (ACR Tasks) to streamline building, testing, pushing, and deploying images in Azure. Configure build tasks to automate your container OS and framework patching pipeline, and build images automatically when your team commits code to source control.

All Azure Container Registry tiers benefit from advanced Azure storage features like encryption-at-rest for image data security and geo-redundancy for image data protection.

- `Encryption-at-rest`: All container images and other artifacts in your registry are encrypted at rest. Azure automatically encrypts an image before storing it, and decrypts it on-the-fly when you or your applications and services pull the image. Optionally apply an extra encryption layer with a customer-managed key.

- `Regional storage`: Azure Container Registry stores data in the region where the registry is created, to help customers meet data residency and compliance requirements. In all regions except Brazil South and Southeast Asia, Azure might also store registry data in a paired region in the same geography. In the Brazil South and Southeast Asia regions, registry data is always confined to the region, to accommodate data residency requirements for those regions. If a regional outage occurs, the registry data might become unavailable and isn't automatically recovered. Customers who wish to have their registry data stored in multiple regions for better performance across different geographies, or who wish to have resiliency in a regional outage event, should enable geo-replication.

- `Geo-replication`: For scenarios requiring high-availability assurance, consider using the geo-replication feature of Premium registries. Geo-replication helps guard against losing access to your registry in a regional failure event. Geo-replication provides other benefits, too, like network-close image storage for faster pushes and pulls in distributed development or deployment scenarios.

- `Zone redundancy`: A feature of the Premium service tier, zone redundancy uses Azure availability zones to replicate your registry to a minimum of three separate zones in each enabled region.

- `Scalable storage`: Azure Container Registry allows you to create as many repositories, images, layers, or tags as you need, up to the registry storage limit. High numbers of repositories and tags can impact the performance of your registry. Periodically delete unused repositories, tags, and images as part of your registry maintenance routine. Deleted registry resources like repositories, images, and tags can't be recovered after deletion.

### 📒 Build and manage containers with tasks <a name="502"></a>

Azure Container Registry (ACR) tasks are a suite of features that:

- Provide cloud-based container image building for platforms like Linux, Windows, and Advanced RISC Machines (Arm).
- Extend the early parts of an application development cycle to the cloud with on-demand container image builds.
- Enable automated builds triggered by source code updates, updates to a container's base image, or timers.

<b>Task scenarios</b>

ACR Tasks supports several scenarios to build and maintain container images and other artifacts.

- `Quick task` - Build and push a single container image to a container registry on-demand, in Azure, without needing a local Docker Engine installation. Think docker build, docker push in the cloud.

- `Automatically triggered tasks` - Enable one or more triggers to build an image:

  - Trigger on source code update
  - Trigger on base image update
  - Trigger on a schedule

- `Multi-step task` - Extend the single image build-and-push capability of ACR Tasks with multi-step, multi-container-based workflows.

Each ACR Task has an associated source code context - the location of a set of source files used to build a container image or other artifact. Example contexts include a Git repository or a local filesystem.

`Quick task`
The inner-loop development cycle is the iterative process of writing code, building, and testing your application before committing to source control. It's really the beginning of container lifecycle management.

Before you commit your first line of code, ACR Tasks's quick task feature can provide an integrated development experience by offloading your container image builds to Azure. With quick tasks, you can verify your automated build definitions and catch potential problems before committing your code.

Using the familiar docker build format, the az acr build command in the Azure CLI takes a context (the set of files to build), sends it to ACR Tasks and, by default, pushes the built image to its registry upon completion.

`Trigger task on source code update`
Trigger a container image build or multi-step task when code is committed, or a pull request is made or updated, to a Git repository in GitHub or Azure DevOps Services. For example, configure a build task with the Azure CLI command az acr task create by specifying a Git repository and optionally a branch and Dockerfile. When your team updates code in the repository, an ACR Tasks-created webhook triggers a build of the container image defined in the repo.

`Trigger on base image update`
You can set up an ACR task to track a dependency on a base image when it builds an application image. When the updated base image is pushed to your registry, or a base image is updated in a public repo such as in Docker Hub, ACR Tasks can automatically build any application images based on it.

`Schedule a task`
Optionally schedule a task by setting up one or more timer triggers when you create or update the task. Scheduling a task is useful for running container workloads on a defined schedule, or running maintenance operations or tests on images pushed regularly to your registry.

`Multi-step tasks`
Multi-step tasks, defined in a YAML file specify individual build and push operations for container images or other artifacts. They can also define the execution of one or more containers, with each step using the container as its execution environment. For example, you can create a multi-step task that automates the following:

1. Build a web application image
2. Run the web application container
3. Build a web application test image
4. Run the web application test container, which performs tests against the running application container
5. If the tests pass, build a Helm chart archive package
6. Perform a helm upgrade using the new Helm chart archive package

`Image platforms`
By default, ACR Tasks builds images for the Linux OS and the amd64 architecture. Specify the --platform tag to build Windows images or Linux images for other architectures. Specify the OS and optionally a supported architecture in OS/architecture format (for example, --platform Linux/arm). For ARM architectures, optionally specify a variant in OS/architecture/variant format (for example, --platform Linux/arm64/v8)

### 📒 Elements of a Dockerfile <a name="503"></a>

A `Dockerfile` is a script that contains a series of instructions that are used to build a Docker image. Dockerfiles typically include the following information:

- The base or parent image we use to create the new image
- Commands to update the base OS and install other software
- Build artifacts to include, such as a developed application
- Services to expose, such a storage and network configuration
- Command to run when the container is launched

<b>Create a Dockerfile</b>

The first step in creating a Dockerfile is choosing a base image that serves as the foundation for your application. For example, if you're building a .NET application, you might choose a Microsoft .NET image as your base.

```
# Use the .NET 6 runtime as a base image
FROM mcr.microsoft.com/dotnet/runtime:6.0

# Set the working directory to /app
WORKDIR /app

# Copy the contents of the published app to the container's /app directory
COPY bin/Release/net6.0/publish/ .

# Expose port 80 to the outside world
EXPOSE 80

# Set the command to run when the container starts
CMD ["dotnet", "MyApp.dll"]
```

Let's go through each line to see what it does:

- FROM mcr.microsoft.com/dotnet/runtime:6.0: This command sets the base image to the .NET 6 runtime, which is needed to run .NET 6 apps.
- WORKDIR /app: Sets the working directory to /app, which is where app files are copied.
- COPY bin/Release/net6.0/publish/ .: Copies the contents of the published app to the container's /app directory. We assume that the .NET 6 app is built and published to the bin/Release/net6.0/publish directory.
- EXPOSE 80: Exposes port 80, which is the default HTTP port, to the outside world. Change this line accordingly if your app listens on a different port.
- CMD ["dotnet", "MyApp.dll"]: The command to run when the container starts. In this case, we're running the dotnet command with the name of our app's DLL file (MyApp.dll). Change this line to match your apps name and entry point.

We're not going to cover the Dockerfile file specification. Visit the Dockerfile reference for more information. Each of these steps creates a cached container image as we build the final container image. These temporary images are layered on top of the previous and presented as single image once all steps complete.

Resources
Docker run reference (CLI)
https://docs.docker.com/engine/reference/run/
Docker build reference
https://docs.docker.com/engine/reference/commandline/build/

### 📒 Example <a name="5031"></a>

> 1. <a href="https://learn.microsoft.com/en-us/training/modules/publish-container-image-to-azure-container-registry/6-build-run-image-azure-container-registry">Build and run a container image by using Azure Container Registry Tasks</a>

### 📒 Explore Azure Container Instances <a name="504"></a>

`Azure Container Instances` (ACI) is a great solution for any scenario that can operate in isolated containers, including simple applications, task automation, and build jobs. Here are some of the benefits:

- `Fast startup`: ACI can start containers in Azure in seconds, without the need to provision and manage a virtual machine (VM)
- `Container access`: ACI enables exposing your container groups directly to the internet with an IP address and a fully qualified domain name (FQDN)
- `Hypervisor-level security`: Isolate your application as completely as it would be in a VM
- `Customer data`: The ACI service stores the minimum customer data required to ensure your container groups are running as expected
- `Custom sizes`: ACI provides optimum utilization by allowing exact specifications of CPU cores and memory
- `Persistent storage`: Mount Azure Files shares directly to a container to retrieve and persist state
- `Linux and Windows`: Schedule both Windows and Linux containers using the same API.

For scenarios where you need full container orchestration, including service discovery across multiple containers, automatic scaling, and coordinated application upgrades, we recommend `Azure Kubernetes Service` (AKS).

The top-level resource in Azure Container Instances is the `container group`. A container group is a collection of containers that get scheduled on the same host machine. The containers in a container group share a lifecycle, resources, local network, and storage volumes. It's similar in concept to a `pod` in Kubernetes.

There are two common ways to deploy a multi-container group: use a Resource Manager template or a YAML file. A Resource Manager template is recommended when you need to deploy more Azure service resources (for example, an Azure Files share) when you deploy the container instances. Due to the YAML format's more concise nature, a YAML file is recommended when your deployment includes only container instances.

Azure Container Instances allocates resources such as CPUs, memory, and optionally GPUs (preview) to a container group by adding the resource requests of the instances in the group. Using CPU resources as an example, if you create a container group with two instances, each requesting one CPU, then the container group is allocated two CPUs.

Container groups share an IP address and a port namespace on that IP address. To enable external clients to reach a container within the group, you must expose the port on the IP address and from the container. Because containers within the group share a port namespace, port mapping isn't supported. Containers within a group can reach each other via localhost on the ports that they exposed, even if those ports aren't exposed externally on the group's IP address.

You can specify external volumes to mount within a container group. You can map those volumes into specific paths within the individual containers in a group. Supported volumes include:

- Azure file share
- Secret
- Empty directory
- Cloned git repo

Multi-container groups are useful in cases where you want to divide a single functional task into a few container images. These images might be delivered by different teams and have separate resource requirements.

Example usage could include:

- A container serving a web application and a container pulling the latest content from source control.
- An application container and a logging container. The logging container collects the logs and metrics output by the main application and writes them to long-term storage.
- An application container and a monitoring container. The monitoring container periodically makes a request to the application to ensure that it's running and responding correctly, and raises an alert if it's not.
- A front-end container and a back-end container. The front end might serve a web application, with the back end running a service to retrieve data.

<b> Run containerized tasks with restart policies </b>

The ease and speed of deploying containers in Azure Container Instances provides a compelling platform for executing run-once tasks like build, test, and image rendering in a container instance. With a configurable restart policy, you can specify that your containers are stopped when their processes are completed. Because container instances are billed by the second, you're charged only for the compute resources used while the container executing your task is running.

When you create a container group in Azure Container Instances, you can specify one of three restart policy settings.

- `Always`. Containers in the container group are always restarted. This is the default setting applied when no restart policy is specified at container creation.
- `Never`. Containers in the container group are never restarted. The containers run at most once.
- `OnFailure`. Containers in the container group are restarted only when the process executed in the container fails (when it terminates with a nonzero exit code). The containers are run at least once.

Specify the `--restart-policy` parameter when you call az container create.

```
az container create \
    --resource-group myResourceGroup \
    --name mycontainer \
    --image mycontainerimage \
    --restart-policy OnFailure
```

Azure Container Instances starts the container, and then stops it when its application, or script, exits. When Azure Container Instances stops a container whose restart policy is Never or OnFailure, the container's status is set to `Terminated`.

<b>Set environment variables in container instances</b>

Setting environment variables in your container instances allows you to provide dynamic configuration of the application or script run by the container. These environment variables are similar to the `--env` command-line argument to `docker run`.

If you need to pass secrets as environment variables, Azure Container Instances supports secure values for both Windows and Linux containers.

In the following example, two variables are passed to the container when it's created. The example is assuming you're running the CLI in a Bash shell or Cloud Shell, if you use the Windows Command Prompt, specify the variables with double-quotes, such as `--environment-variables "NumWords"="5" "MinLength"="8"`.

```
az container create \
    --resource-group myResourceGroup \
    --name mycontainer2 \
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest
    --restart-policy OnFailure \
    --environment-variables 'NumWords'='5' 'MinLength'='8'\
```

Objects with secure values are intended to hold sensitive information like passwords or keys for your application. Using secure values for environment variables is both safer and more flexible than including it in your container's image.

Environment variables with secure values aren't visible in your container's properties. Their values can be accessed only from within the container. For example, container properties viewed in the Azure portal or Azure CLI display only a secure variable's name, not its value.

Set a secure environment variable by specifying the secureValue property instead of the regular value for the variable's type. The two variables defined in the following YAML demonstrate the two variable types.

```
apiVersion: 2018-10-01
location: eastus
name: securetest
properties:
  containers:
  - name: mycontainer
    properties:
      environmentVariables:
        - name: 'NOTSECRET'
          value: 'my-exposed-value'
        - name: 'SECRET'
          secureValue: 'my-secret-value'
      image: nginx
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  osType: Linux
  restartPolicy: Always
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

You would run the following command to deploy the container group with YAML:

```
az container create --resource-group myResourceGroup \
    --file secure-env.yaml \
```

<b>Mount an Azure file share in Azure Container Instances</b>

By default, Azure Container Instances are stateless. If the container crashes or stops, all of its state is lost. To persist state beyond the lifetime of the container, you must mount a volume from an external store. As shown in this unit, Azure Container Instances can mount an Azure file share created with Azure Files. Azure Files offers fully managed file shares in the cloud that are accessible via the industry standard Server Message Block (SMB) protocol. Using an Azure file share with Azure Container Instances provides file-sharing features similar to using an Azure file share with Azure virtual machines.

Limitations:

- You can only mount Azure Files shares to Linux containers.
- Azure file share volume mount requires the Linux container run as root.
- Azure File share volume mounts are limited to CIFS support.

<b>Deploy container and mount volume</b>

To mount an Azure file share as a volume in a container by using the Azure CLI, specify the share and volume mount point when you create the container with `az container create`. Following is an example of the command:

```
az container create
--resource-group $ACI_PERS_RESOURCE_GROUP
--name hellofiles
--image mcr.microsoft.com/azuredocs/aci-hellofiles
--dns-name-label aci-demo
--ports 80
--azure-file-volume-account-name $ACI_PERS_STORAGE_ACCOUNT_NAME --azure-file-volume-account-key $STORAGE_KEY --azure-file-volume-share-name $ACI_PERS_SHARE_NAME --azure-file-volume-mount-path /aci/logs/
```

The `--dns-name-label` value must be unique within the Azure region where you create the container instance. Update the value in the preceding command if you receive a DNS name label error message when you execute the command.

<b> Deploy container and mount volume - YAML</b>

You can also deploy a container group and mount a volume in a container with the Azure CLI and a YAML template. Deploying by YAML template is the preferred method when deploying container groups consisting of multiple containers.

The following YAML template defines a container group with one container created with the aci-hellofiles image. The container mounts the Azure file share acishare created previously as a volume. Following is an example YAML file.

```
apiVersion: '2019-12-01'
location: eastus
name: file-share-demo
properties:
  containers:
  - name: hellofiles
    properties:
      environmentVariables: []
      image: mcr.microsoft.com/azuredocs/aci-hellofiles
      ports:
      - port: 80
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
      volumeMounts:
      - mountPath: /aci/logs/
        name: filesharevolume
  osType: Linux
  restartPolicy: Always
  ipAddress:
    type: Public
    ports:
      - port: 80
    dnsNameLabel: aci-demo
  volumes:
  - name: filesharevolume
    azureFile:
      sharename: acishare
      storageAccountName: <Storage account name>
      storageAccountKey: <Storage account key>
tags: {}
type: Microsoft.ContainerInstance/containerGroups
```

<b>Mount multiple volumes</b>

To mount multiple volumes in a container instance, you must deploy using an Azure Resource Manager template or a YAML file. To use a template or YAML file, provide the share details and define the volumes by populating the volumes array in the properties section of the template.

For example, if you created two Azure Files shares named share1 and share2 in storage account myStorageAccount, the volumes array in a Resource Manager template would appear similar to the following:

```
JSON

"volumes": [{
  "name": "myvolume1",
  "azureFile": {
    "shareName": "share1",
    "storageAccountName": "myStorageAccount",
    "storageAccountKey": "<storage-account-key>"
  }
},
{
  "name": "myvolume2",
  "azureFile": {
    "shareName": "share2",
    "storageAccountName": "myStorageAccount",
    "storageAccountKey": "<storage-account-key>"
  }
}]

```

Next, for each container in the container group in which you'd like to mount the volumes, populate the volumeMounts array in the properties section of the container definition. For example, this mounts the two volumes, myvolume1 and myvolume2, previously defined:

```
JSON

"volumeMounts": [{
  "name": "myvolume1",
  "mountPath": "/mnt/share1/"
},
{
  "name": "myvolume2",
  "mountPath": "/mnt/share2/"
}]

```

### 📒 Example <a name="5041"></a>

> 1. <a href="https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/3-run-azure-container-instances-cloud-shell">Deploy a container instance by using the Azure CLI</a>

### 📒 Implement Azure Container Apps <a name="505"></a>

`Azure Container Apps` enables you to run microservices and containerized applications on a serverless platform that runs on top of Azure Kubernetes Service. Common uses of Azure Container Apps include:

- Deploying API endpoints
- Hosting background processing applications
- Handling event-driven processing
- Running microservices

Applications built on Azure Container Apps can dynamically scale based on: HTTP traffic, event-driven processing, CPU or memory load, and any KEDA-supported scaler.

With Azure Container Apps, you can:

- Run multiple container revisions and manage the container app's application lifecycle.
- Autoscale your apps based on any KEDA-supported scale trigger. Most applications can scale to zero. (Applications that scale on CPU or memory load can't scale to zero.)
- Enable HTTPS ingress without having to manage other Azure infrastructure.
- Split traffic across multiple versions of an application for Blue/Green deployments and A/B testing scenarios.
- Use internal ingress and service discovery for secure internal-only endpoints with built-in DNS-based service discovery.
- Build microservices with Dapr and access its rich set of APIs.
- Run containers from any registry, public or private, including Docker Hub and Azure Container Registry (ACR).
- Use the Azure CLI extension, Azure portal or ARM templates to manage your applications.
- Provide an existing virtual network when creating an environment for your container apps.
- Securely manage secrets directly in your application.
- Monitor logs using Azure Log Analytics.

Individual container apps are deployed to a single Container Apps environment, which acts as a secure boundary around groups of container apps. Container Apps in the same environment are deployed in the same virtual network and write logs to the same Log Analytics workspace. You might provide an existing virtual network when you create an environment.

Reasons to deploy container apps to the same environment include situations when you need to:

- Manage related services
- Deploy different applications to the same virtual network
- nstrument Dapr applications that communicate via the Dapr service invocation API
- Have applications to share the same Dapr configuration
- Have applications share the same log analytics workspace

Reasons to deploy container apps to different environments include situations when you want to ensure:

- Two applications never share the same compute resources
- Two Dapr applications can't communicate via the Dapr service invocation API

`Microservice architectures` allow you to independently develop, upgrade, version, and scale core areas of functionality in an overall system. Azure Container Apps provides the foundation for deploying microservices featuring:

- Independent scaling, versioning, and upgrades
- Service discovery
- Native Dapr integration

When you implement a system composed of microservices, function calls are spread across the network. To support the distributed nature of microservices, you need to account for failures, retries, and timeouts. While Container Apps features the building blocks for running microservices, use of `Dapr` provides an even richer microservices programming model. `Dapr` includes features like observability, pub/sub, and service-to-service invocation with mutual TLS, retries, and more.

### 📒 Example <a name="5051"></a>

> 1. <a href="https://learn.microsoft.com/en-us/training/modules/implement-azure-container-apps/3-exercise-deploy-app">Deploy a container app</a>

### 📒 Explore containers in Azure Container Apps <a name="506"></a>

Azure Container Apps manages the details of Kubernetes and container orchestration for you. Containers in Azure Container Apps can use any runtime, programming language, or development stack of your choice.

![](images/20.png)

Azure Container Apps supports any Linux-based x86-64 (linux/amd64) container image. There's no required base container image, and if a container crashes it automatically restarts.

<b>Configuration</b>

The following code is an example of the containers array in the properties.template section of a container app resource template. The excerpt shows some of the available configuration options when setting up a container when using Azure Resource Manager (ARM) templates. Changes to the template ARM configuration section trigger a new container app revision.

```
JSON

"containers": [
  {
       "name": "main",
       "image": "[parameters('container_image')]",
    "env": [
      {
        "name": "HTTP_PORT",
        "value": "80"
      },
      {
        "name": "SECRET_VAL",
        "secretRef": "mysecret"
      }
    ],
    "resources": {
      "cpu": 0.5,
      "memory": "1Gi"
    },
    "volumeMounts": [
      {
        "mountPath": "/myfiles",
        "volumeName": "azure-files-volume"
      }
    ]
    "probes":[
        {
            "type":"liveness",
            "httpGet":{
            "path":"/health",
            "port":8080,
            "httpHeaders":[
                {
                    "name":"Custom-Header",
                    "value":"liveness probe"
                }]
            },
            "initialDelaySeconds":7,
            "periodSeconds":3
// file is truncated for brevity

```

You can define multiple containers in a single container app to implement the sidecar pattern. The containers in a container app share hard disk and network resources and experience the same application lifecycle.

Examples of sidecar containers include:

- An agent that reads logs from the primary app container on a shared volume and forwards them to a logging service.
- A background process that refreshes a cache used by the primary app container in a shared volume.

Running multiple containers in a single container app is an advanced use case. In most situations where you want to run multiple containers, such as when implementing a microservice architecture, deploy each service as a separate container app. To run multiple containers in a container app, add more than one container in the containers array of the container app template.

You can deploy images hosted on private registries by providing credentials in the Container Apps configuration. To use a container registry, you define the required fields in registries array in the properties.configuration section of the container app resource template. The passwordSecretRef field identifies the name of the secret in the secrets array name where you defined the password.

```
JSON

{
  ...
  "registries": [{
    "server": "docker.io",
    "username": "my-registry-user-name",
    "passwordSecretRef": "my-password-secret-name"
  }]
}

```

With the registry information added, the saved credentials can be used to pull a container image from the private registry when your app is deployed.

Azure Container Apps has the following limitations:

- Privileged containers: Azure Container Apps can't run privileged containers. If your program attempts to run a process that requires root access, the application inside the container experiences a runtime error.
- Operating system: Linux-based (linux/amd64) container images are required.

### 📒 Implement authentication and authorization in Azure Container Apps <a name="507"></a>

Azure Container Apps provides built-in authentication and authorization features to secure your external ingress-enabled container app with minimal or no code. The built-in authentication feature for Container Apps can save you time and effort by providing out-of-the-box authentication with federated identity providers, allowing you to focus on the rest of your application.

- Azure Container Apps provides access to various built-in authentication providers.
- The built-in auth features don’t require any particular language, SDK, security expertise, or even any code that you have to write.

This feature should only be used with HTTPS. Ensure allowInsecure is disabled on your container app's ingress configuration. You can configure your container app for authentication with or without restricting access to your site content and APIs.

- To restrict app access only to authenticated users, set its Restrict access setting to Require authentication.
- To authenticate but not restrict access, set its Restrict access setting to Allow unauthenticated access.

Container Apps uses federated identity, in which a third-party identity provider manages the user identities and authentication flow for you. The following identity providers are available by default: Microsoft Identity Platform, Facebook, GitHub, Google, Twitter, Any OpenID Connect provider. When you use one of these providers, the sign-in endpoint is available for user authentication and authentication token validation from the provider. You can provide your users with any number of these provider options.

The authentication and authorization middleware component is a feature of the platform that runs as a sidecar container on each replica in your application. When enabled, every incoming HTTP request passes through the security layer before being handled by your application.
The platform middleware handles several things for your app:

- Authenticates users and clients with the specified identity providers
- Manages the authenticated session
- Injects identity information into HTTP request headers

The authentication and authorization module runs in a separate container, isolated from your application code. As the security container doesn't run in-process, no direct integration with specific language frameworks is possible. However, relevant information your app needs is provided in request headers.

The `authentication flow` is the same for all providers, but differs depending on whether you want to sign in with the provider's SDK:

- Without provider SDK (server-directed flow or server flow): The application delegates federated sign-in to Container Apps. Delegation is typically the case with browser apps, which presents the provider's sign-in page to the user.

- With provider SDK (client-directed flow or client flow): The application signs users in to the provider manually and then submits the authentication token to Container Apps for validation. This approach is typical for browser-less apps that don't present the provider's sign-in page to the user. An example is a native mobile app that signs users in using the provider's SDK.

### 📒 Manage revisions and secrets in Azure Container Apps<a name="508"></a>

Azure Container Apps implements container app versioning by creating revisions. A revision is an immutable snapshot of a container app version. You can use revisions to release a new version of your app, or quickly revert to an earlier version of your app. New revisions are created when you update your application with revision-scope changes. You can also update your container app based on a specific revision.

You can control which revisions are active, and the external traffic that is routed to each active revision. Revision names are used to identify a revision, and in the revision's URL. You can customize the revision name by setting the revision suffix.

By default, Container Apps creates a unique revision name with a suffix consisting of a semi-random string of alphanumeric characters. For example, for a container app named album-api, setting the revision suffix name to 1st-revision would create a revision with the name album-api--1st-revision. You can set the revision suffix in the ARM template, through the Azure CLI az containerapp create and az containerapp update commands, or when creating a revision via the Azure portal.

With the az containerapp update command you can modify environment variables, compute resources, scale parameters, and deploy a different image. If your container app update includes revision-scope changes, a new revision is generated.

```
az containerapp update \
  --name <APPLICATION_NAME> \
  --resource-group <RESOURCE_GROUP_NAME> \
  --image <IMAGE_NAME>
```

You can list all revisions associated with your container app with the az containerapp revision list command.

```
az containerapp revision list \
  --name <APPLICATION_NAME> \
  --resource-group <RESOURCE_GROUP_NAME> \
  -o table
```

Azure Container Apps allows your application to securely store sensitive configuration values. Once secrets are defined at the application level, secured values are available to container apps. Specifically, you can reference secured values inside scale rules.

- Secrets are scoped to an application, outside of any specific revision of an application.
- Adding, removing, or changing secrets doesn't generate new revisions.
- Each application revision can reference one or more secrets.
- Multiple revisions can reference the same secrets.
  An updated or deleted secret doesn't automatically affect existing revisions in your app. When a secret is updated or deleted, you can respond to changes in one of two ways:

- Deploy a new revision.
- Restart an existing revision.

Before you delete a secret, deploy a new revision that no longer references the old secret. Then deactivate all revisions that reference the secret.

Container Apps doesn't support Azure Key Vault integration. Instead, enable managed identity in the container app and use the Key Vault SDK in your app to access secrets.

When you create a container app, secrets are defined using the --secrets parameter.

- The parameter accepts a space-delimited set of name/value pairs.
- Each pair is delimited by an equals sign (=).

In the example below, a connection string to a queue storage account is declared in the --secrets parameter. The value for queue-connection-string comes from an environment variable named $CONNECTION_STRING.

```
az containerapp create \
  --resource-group "my-resource-group" \
  --name queuereader \
  --environment "my-environment-name" \
  --image demos/queuereader:v1 \
  --secrets "queue-connection-string=$CONNECTION_STRING"
```

After declaring secrets at the application level, you can reference them in environment variables when you create a new revision in your container app. When an environment variable references a secret, its value is populated with the value defined in the secret. To reference a secret in an environment variable in the Azure CLI, set its value to secretref:, followed by the name of the secret.

The following example shows an application that declares a connection string at the application level. This connection is referenced in a container environment variable.

```
az containerapp create \
  --resource-group "my-resource-group" \
  --name myQueueApp \
  --environment "my-environment-name" \
  --image demos/myQueueApp:v1 \
  --secrets "queue-connection-string=$CONNECTIONSTRING" \
  --env-vars "QueueName=myqueue" "ConnectionString=secretref:queue-connection-string"
```

### 📒 Explore Dapr integration with Azure Container Apps <a name="509"></a>

The Distributed Application Runtime (Dapr) is a set of incrementally adoptable features that simplify the authoring of distributed, microservice-based applications. Dapr provides capabilities for enabling application intercommunication through messaging via pub/sub or reliable and secure service-to-service calls.

Dapr is an open source, Cloud Native Computing Foundation (CNCF) project. The CNCF is part of the Linux Foundation and provides support, oversight, and direction for fast-growing, cloud native projects. As an alternative to deploying and managing the Dapr OSS project yourself, the Container Apps platform:

- Provides a managed and supported Dapr integration
- Handles Dapr version upgrades seamlessly
- Exposes a simplified Dapr interaction model to increase developer productivity

![](images/21.png)

The following example based on the Pub/sub API is used to illustrate core concepts related to Dapr in Azure Container Apps.

![](images/22.png)

You can configure Dapr using various arguments and annotations based on the runtime context. Azure Container Apps provides three channels through which you can configure Dapr:

- Container Apps CLI
- Infrastructure as Code (IaC) templates, as in Bicep or Azure Resource Manager (ARM) templates
- The Azure portal

Dapr uses a modular design where functionality is delivered as a component. The use of Dapr components is optional and dictated exclusively by the needs of your application.

Dapr components in container apps are environment-level resources that:

- Can provide a pluggable abstraction model for connecting to supporting external services.
- Can be shared across container apps or scoped to specific container apps.
- Can use Dapr secrets to securely retrieve configuration metadata.

By default, all Dapr-enabled container apps within the same environment load the full set of deployed components. To ensure components are loaded at runtime by only the appropriate container apps, application scopes should be used.

---

# 6. Implement user authentication and authorization <a name="6"></a>

`The Microsoft identity platform` helps you build applications your users and customers can sign in to using their Microsoft identities or social accounts, and provide authorized access to your own APIs or Microsoft APIs like Microsoft Graph.

There are several components that make up the Microsoft identity platform:

- OAuth 2.0 and OpenID Connect standard-compliant authentication service enabling developers to authenticate several identity types, including:

  - Work or school accounts, provisioned through Microsoft Entra ID
  - Personal Microsoft account, like Skype, Xbox, and Outlook.com
  - Social or local accounts, by using Azure Active Directory B2C
  - Social or local customer accounts, by using Microsoft Entra External ID

- Open-source libraries: Microsoft Authentication Libraries (MSAL) and support for other standards-compliant libraries

- Microsoft identity platform endpoint: Works with the Microsoft Authentication Libraries (MSAL) or any other standards-compliant library. It implements human readable scopes, in accordance with industry standards.

- Application management portal: A registration and configuration experience in the Azure portal, along with the other Azure management capabilities.

- Application configuration API and PowerShell: Programmatic configuration of your applications through the Microsoft Graph API and PowerShell so you can automate your DevOps tasks.

<b> Service principals</b>

For developers, the Microsoft identity platform offers integration of modern innovations in the identity and security space like passwordless authentication, step-up authentication, and Conditional Access. You don’t need to implement such functionality yourself: applications integrated with the Microsoft identity platform natively take- [Microsoft Graph](#603)

If you register an application in the portal, an application object (the globally unique instance of the app) and a service principal object are automatically created in your home tenant. You also have a globally unique ID for your app (the app or client ID). In the portal, you can then add secrets or certificates and scopes to make your app work, customize the branding of your app in the sign-in dialog, and more. You can also create service principal objects in a tenant using Azure PowerShell, Azure CLI, Microsoft Graph, and other tools.

A Microsoft Entra application is scoped to its one and only `application object`. The application object resides in the Microsoft Entra tenant where the application was registered (known as the application's "home" tenant). An application object is used as a template or blueprint to create one or more service principal objects. A service principal is created in every tenant where the application is used. Similar to a class in object-oriented programming, the application object has some static properties that are applied to all the created service principals (or application instances).

The application object describes three aspects of an application:

- How the service can issue tokens in order to access the application.
- Resources that the application might need to access.
- The actions that the application can take.

The Microsoft Graph Application entity defines the schema for an application object's properties.

To access resources secured by a Microsoft Entra tenant, the entity that is requesting access must be represented by a `security principal`. This is true for both users (user principal) and applications (service principal).

The security principal defines the access policy and permissions for the user/application in the Microsoft Entra tenant. This enables core features such as authentication of the user/application during sign-in, and authorization during resource access.

There are three types of service principal:

- Application - This type of service principal is the local representation, or application instance, of a global application object in a single tenant or directory. A service principal is created in each tenant where the application is used, and references the globally unique app object. The service principal object defines what the app can actually do in the specific tenant, who can access the app, and what resources the app can access.

- Managed identity - This type of service principal is used to represent a managed identity. Managed identities provide an identity for applications to use when connecting to resources that support Microsoft Entra authentication. When a managed identity is enabled, a service principal representing that managed identity is created in your tenant. Service principals representing managed identities can be granted access and permissions, but can't be updated or modified directly.

- Legacy - This type of service principal represents a legacy app, which is an app created before app registrations were introduced or an app created through legacy experiences. A legacy service principal can have:

  - credentials
  - service principal names
  - reply URLs
  - and other properties that an authorized user can edit, but doesn't have an associated app registration.

The application object is the global representation of your application for use across all tenants, and the service principal is the local representation for use in a specific tenant. The application object serves as the template from which common and default properties are derived for use in creating corresponding service principal objects.

An application object has:

- A one to one relationship with the software application, and
- A one to many relationships with its corresponding service principal objects.

A service principal must be created in each tenant where the application is used to establish an identity for sign-in and/or access to resources being secured by the tenant. A single-tenant application has only one service principal (in its home tenant), created and consented for use during application registration. A multitenant application also has a service principal created in each tenant where a user from that tenant consented to its use.

<b>Permissions and consent</b>

Applications that integrate with the Microsoft identity platform follow an authorization model that gives users and administrators control over how data can be accessed.

The Microsoft identity platform implements the OAuth 2.0 authorization protocol. OAuth 2.0 is a method through which a third-party app can access web-hosted resources on behalf of a user. Any web-hosted resource that integrates with the Microsoft identity platform has a resource identifier, or application ID URI.

Here are some examples of Microsoft web-hosted resources:

- Microsoft Graph: https://graph.microsoft.com
- Microsoft 365 Mail API: https://outlook.office.com
- Azure Key Vault: https://vault.azure.net

The same is true for any third-party resources that are integrated with the Microsoft identity platform. Any of these resources also can define a set of permissions that can be used to divide the functionality of that resource into smaller chunks. When a resource's functionality is chunked into small permission sets, third-party apps can be built to request only the permissions that they need to perform their function. Users and administrators can know what data the app can access.

In OAuth 2.0, these types of permission sets are called scopes. They're also often referred to as permissions. In the Microsoft identity platform, a permission is represented as a string value. An app requests the permissions it needs by specifying the permission in the scope query parameter. Identity platform supports several well-defined OpenID Connect scopes and resource-based permissions (each permission is indicated by appending the permission value to the resource's identifier or application ID URI). For example, the permission string https://graph.microsoft.com/Calendars.Read is used to request permission to read users calendars in Microsoft Graph.

An app most commonly requests these permissions by specifying the scopes in requests to the Microsoft identity platform authorize endpoint. However, some high-privilege permissions can be granted only through administrator consent. They can be requested or granted by using the administrator consent endpoint.

In requests to the authorization, token or consent endpoints for the Microsoft Identity platform, if the resource identifier is omitted in the scope parameter, the resource is assumed to be Microsoft Graph. For example, scope=User.Read is equivalent to https://graph.microsoft.com/User.Read.

Permission typeshe Microsoft identity platform supports two `types of permissions`: delegated access and app-only access.

- Delegated access are used by apps that have a signed-in user present. For these apps, either the user or an administrator consents to the permissions that the app requests. The app is delegated with the permission to act as a signed-in user when it makes calls to the target resource.

- App-only access permissions are used by apps that run without a signed-in user present, for example, apps that run as background services or daemons. Only an administrator can consent to app-only access permissions.

Consent types:
Applications in Microsoft identity platform rely on consent in order to gain access to necessary resources or APIs. There are many kinds of consent that your app might need to know about in order to be successful. If you're defining permissions, you'll also need to understand how your users gain access to your app or API.

There are three consent types: static user consent, incremental and dynamic user consent, and admin consent.

In the `static user consent` scenario, you must specify all the permissions it needs in the app's configuration in the Azure portal. If the user (or administrator, as appropriate) hasn't granted consent for this app, then Microsoft identity platform prompts the user to provide consent at this time. Static permissions also enable administrators to consent on behalf of all users in the organization.

While static permissions of the app defined in the Azure portal keep the code nice and simple, it presents some possible issues for developers:

- The app needs to request all the permissions it would ever need upon the user's first sign-in. This can lead to a long list of permissions that discourages end users from approving the app's access on initial sign-in.
- The app needs to know all of the resources it would ever access ahead of time. It's difficult to create apps that could access an arbitrary number of resources.

Incremental and dynamic user consent:
With the Microsoft identity platform endpoint, you can ignore the static permissions defined in the app registration information in the Azure portal and request permissions incrementally instead. You can ask for a minimum set of permissions upfront and request more over time as the customer uses more app features.

To do so, you can specify the scopes your app needs at any time by including the new scopes in the scope parameter when requesting an access token - without the need to predefine them in the application registration information. If the user hasn't yet consented to new scopes added to the request, they're prompted to consent only to the new permissions. Incremental, or dynamic consent, only applies to delegated permissions and not to app-only access permissions. Dynamic consent can be convenient, but presents a big challenge for permissions that require admin consent, since the admin consent experience doesn't know about those permissions at consent time. If you require admin privileged permissions or if your app uses dynamic consent, you must register all of the permissions in the Azure portal (not just the subset of permissions that require admin consent). This enables tenant admins to consent on behalf of all their users.

`Admin consent` is required when your app needs access to certain high-privilege permissions. Admin consent ensures that administrators have some other controls before authorizing apps or users to access highly privileged data from the organization. Admin consent done on behalf of an organization still requires the static permissions registered for the app. Set those permissions for apps in the app registration portal if you need an admin to give consent on behalf of the entire organization. This reduces the cycles required by the organization admin to set up the application.

In an OpenID Connect or OAuth 2.0 authorization request, an app can request the permissions it needs by using the scope query parameter. For example, when a user signs in to an app, the app sends a request like the following example. Line breaks are added for legibility.

```
HTTP

Copy
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=query
&scope=
https%3A%2F%2Fgraph.microsoft.com%2Fcalendars.read%20
https%3A%2F%2Fgraph.microsoft.com%2Fmail.send
&state=12345
```

The scope parameter is a space-separated list of delegated permissions that the app is requesting. Each permission is indicated by appending the permission value to the resource's identifier (the application ID URI). In the request example, the app needs permission to read the user's calendar and send mail as the user. After the user enters their credentials, the Microsoft identity platform checks for a matching record of user consent. If the user hasn't consented to any of the requested permissions in the past, and if the administrator hasn't consented to these permissions on behalf of the entire organization, the Microsoft identity platform asks the user to grant the requested permissions.

<b>Conditional access</b>

The Conditional Access feature in Microsoft Entra ID offers one of several ways that you can use to secure your app and protect a service. Conditional Access enables developers and enterprise customers to protect services in a multitude of ways including:

- Multifactor authentication
- Allowing only Intune enrolled devices to access specific services
- Restricting user locations and IP ranges

In most common cases, Conditional Access doesn't change an app's behavior or require any changes from the developer. Only in certain cases when an app indirectly or silently requests a token for a service does an app require code changes to handle Conditional Access challenges. It may be as simple as performing an interactive sign-in request.

Specifically, the following scenarios require code to handle Conditional Access challenges:

- Apps performing the on-behalf-of flow
- Apps accessing multiple services/resources
- Single-page apps using MSAL.js
- Web apps calling a resource

Conditional Access policies can be applied to the app and also a web API your app accesses. Depending on the scenario, an enterprise customer can apply and remove Conditional Access policies at any time. For your app to continue functioning when a new policy is applied, implement challenge handling.

Some scenarios require code changes to handle Conditional Access whereas others work as is. Here are a few scenarios using Conditional Access to do multifactor authentication that gives some insight into the difference.

- You're building a single-tenant iOS app and apply a Conditional Access policy. The app signs in a user and doesn't request access to an API. When the user signs in, the policy is automatically invoked and the user needs to perform multifactor authentication.

- You're building an app that uses a middle tier service to access a downstream API. An enterprise customer at the company using this app applies a policy to the downstream API. When an end user signs in, the app requests access to the middle tier and sends the token. The middle tier performs on-behalf-of flow to request access to the downstream API. At this point, a claims "challenge" is presented to the middle tier. The middle tier sends the challenge back to the app, which needs to comply with the Conditional Access policy.

### 📒 Implement authentication by using the Microsoft Authentication Library <a name="601"></a>

`The Microsoft Authentication Library` (MSAL) enables developers to acquire security tokens from the Microsoft identity platform to authenticate users and access secured web APIs. It can be used to provide secure access to Microsoft Graph, other Microsoft APIs, third-party web APIs, or your own web API MSAL supports many different application architectures and platforms including .NET, JavaScript, Java, Python, Android, and iOS.

MSAL gives you many ways to get tokens, with a consistent API for many platforms. Using MSAL provides the following benefits:

- No need to directly use the OAuth libraries or code against the protocol in your application.
- Acquires tokens on behalf of a user or on behalf of an application (when applicable to the platform).
- Maintains a token cache and refreshes tokens for you when they're close to expire. You don't need to handle token expiration on your own.
- Helps you specify which audience you want your application to sign in.
- Helps you set up your application from configuration files.
- Helps you troubleshoot your app by exposing actionable exceptions, logging, and telemetry.

Within MSAL, a token can be acquired from many application types: web applications, web APIs, single-page apps (JavaScript), mobile and native applications, and daemons and server-side applications. MSAL currently supports the platforms and frameworks listed in the following table.

| Library                | Supported platforms and frameworks                                                               |
| ---------------------- | ------------------------------------------------------------------------------------------------ |
| MSAL for Android       | Android                                                                                          |
| MSAL Angular           | Single-page apps with Angular and Angular.js frameworks                                          |
| MSAL for iOS and macOS | iOS and macOS                                                                                    |
| MSAL Go (Preview)      | Windows, macOS, Linux                                                                            |
| MSAL Java              | Windows, macOS, Linux                                                                            |
| MSAL.js                | JavaScript/TypeScript frameworks such as Vue.js, Ember.js, or Durandal.js                        |
| MSAL.NET               | .NET Framework, .NET, .NET MAUI, WINUI, Xamarin Android, Xamarin iOS, Universal Windows Platform |
| MSAL Node              | Web apps with Express, desktop apps with Electron, Cross-platform console apps                   |
| MSAL Python            | Windows, macOS, Linux                                                                            |
| MSAL React             | Single-page apps with React and React-based libraries (Next.js, Gatsby.js)                       |

The following table shows some of the different authentication flows provided by Microsoft Authentication Library (MSAL). These flows can be used in various application scenarios.

![](images/23.png)

`The Microsoft Authentication Library` (MSAL) defines two types of clients; public clients and confidential clients. A client is a software entity that has a unique identifier assigned by an identity provider. The client types differ based their ability to authenticate securely with the authorization server and to hold sensitive, identity proving information so that it can't be accessed or known to a user within the scope of its access. When examining the public or confidential nature of a given client, we're evaluating the ability of that client to prove its identity to the authorization server. This is important because the authorization server must be able to trust the identity of the client in order to issue access tokens.

- Public client applications run on devices, such as desktop, browserless APIs, mobile or client-side browser apps. They can't be trusted to safely keep application secrets, so they can only access web APIs on behalf of the user. Anytime the source, or compiled bytecode of a given app, is transmitted anywhere it can be read, disassembled, or otherwise inspected by untrusted parties. As they also only support public client flows and can't hold configuration-time secrets, they can't have client secrets.

- Confidential client applications run on servers, such as web apps, web API apps, or service/daemon apps. They're considered difficult to access by users or attackers, and therefore can adequately hold configuration-time secrets to assert proof of its identity. The client ID is exposed through the web browser, but the secret is passed only in the back channel and never directly exposed.

<b>Initialize client applications</b>

With MSAL.NET 3.x, the recommended way to instantiate an application is by using the application builders: PublicClientApplicationBuilder and ConfidentialClientApplicationBuilder. They offer a powerful mechanism to configure the application either from the code, or from a configuration file, or even by mixing both approaches.

Before initializing an application, you first need to register it so that your app can be integrated with the Microsoft identity platform. After registration, you may need the following information (which can be found in the Azure portal):

- Application (client) ID - This is a string representing a GUID.
- Directory (tenant) ID - Provides identity and access management (IAM) capabilities to applications and resources used by your organization. It can specify if you're writing a line of business application solely for your organization (also named single-tenant application).
- The identity provider URL (named the instance) and the sign-in audience for your application. These two parameters are collectively known as the authority.
- Client credentials - which can take the form of an application secret (client secret string) or certificate (of type X509Certificate2) if it's a confidential client app.
- For web apps, and sometimes for public client apps (in particular when your app needs to use a broker), you need to set the Redirect URI where the identity provider will contact back your application with the security tokens.

The following code instantiates a public client application, signing-in users in the Microsoft Azure public cloud, with their work and school accounts, or their personal Microsoft accounts.

```
C#
IPublicClientApplication app = PublicClientApplicationBuilder.Create(clientId).Build();
```

In the same way, the following code instantiates a confidential application (a Web app located at https://myapp.azurewebsites.net) handling tokens from users in the Microsoft Azure public cloud, with their work and school accounts, or their personal Microsoft accounts. The application is identified with the identity provider by sharing a client secret:

```
C#
string redirectUri = "https://myapp.azurewebsites.net";
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(clientId)
    .WithClientSecret(clientSecret)
    .WithRedirectUri(redirectUri )
    .Build();

```

In the code snippets using application builders, .With methods can be applied as modifiers (for example, .WithAuthority and .WithRedirectUri).

- .WithAuthority modifier: The .WithAuthority modifier sets the application default authority to a Microsoft Entra authority, with the possibility of choosing the Azure Cloud, the audience, the tenant (tenant ID or domain name), or providing directly the authority URI.

```
C#
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, tenantId)
    .Build();

```

- .WithRedirectUri modifier: The .WithRedirectUri modifier overrides the default redirect URI.

```
C#

IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(client_id)
    .WithAuthority(AzureCloudInstance.AzurePublic, tenant_id)
    .WithRedirectUri("http://localhost")
    .Build();
```

The table below lists some of the modifiers you can set on a public, or confidential client.

![](images/24.png)

The modifiers specific to a confidential client application builder can be found in the ConfidentialClientApplicationBuilder class. The different methods can be found in the Azure SDK for .NET documentation. Modifiers such as .WithCertificate(X509Certificate2 certificate) and .WithClientSecret(string clientSecret) are mutually exclusive. If you provide both, MSAL throws a meaningful exception.

### 📒 Example <a name="6011"></a>

> 1. <a href="https://learn.microsoft.com/en-us/training/modules/implement-authentication-by-using-microsoft-authentication-library/4-interactive-authentication-msal">Implement interactive authentication by using MSAL.NET</a>

### 📒 Implement shared access signatures <a name="602"></a>

`A shared access signature` (SAS) is a signed URI that points to one or more storage resources and includes a token that contains a special set of query parameters. The token indicates how the resources might be accessed by the client. One of the query parameters, the signature, is constructed from the SAS parameters and signed with the key that was used to create the SAS. This signature is used by Azure Storage to authorize access to the storage resource.

Azure Storage supports three types of shared access signatures:

- `User delegation SAS`: A user delegation SAS is secured with Microsoft Entra credentials and also by the permissions specified for the SAS. A user delegation SAS applies to Blob storage only.

- `Service SAS`: A service SAS is secured with the storage account key. A service SAS delegates access to a resource in the following Azure Storage services: Blob storage, Queue storage, Table storage, or Azure Files.

- `Account SAS`: An account SAS is secured with the storage account key. An account SAS delegates access to resources in one or more of the storage services. All of the operations available via a service or user delegation SAS are also available via an account SAS.

Microsoft recommends that you use Microsoft Entra credentials when possible as a security best practice, rather than using the account key, which can be more easily compromised. When your application design requires shared access signatures for access to Blob storage, use Microsoft Entra credentials to create a user delegation SAS when possible for superior security.

How shared access signatures work:
When you use a SAS to access data stored in Azure Storage, you need two components. The first is a URI to the resource you want to access. The second part is a SAS token that you've created to authorize access to that resource. In a single URI, such as https://medicalrecords.blob.core.windows.net/patient-images/patient-116139-nq8z7f.jpg?sp=r&st=2020-01-20T11:42:32Z&se=2020-01-20T19:42:32Z&spr=https&sv=2019-02-02&sr=b&sig=SrW1HZ5Nb6MbRzTbXCaPm%2BJiSEn15tC91Y4umMPwVZs%3D, you can separate the URI from the SAS token as follows:

- URI: https://medicalrecords.blob.core.windows.net/patient-images/patient-116139-nq8z7f.jpg?
- SAS token: sp=r&st=2020-01-20T11:42:32Z&se=2020-01-20T19:42:32Z&spr=https&sv=2019-02-02&sr=b&sig=SrW1HZ5Nb6MbRzTbXCaPm%2BJiSEn15tC91Y4umMPwVZs%3D

The SAS token itself is made up of several components.

![](images/25.png)

To reduce the potential risks of using a SAS, Microsoft provides some guidance:

- To securely distribute a SAS and prevent man-in-the-middle attacks, always use HTTPS.
- The most secure SAS is a user delegation SAS. Use it wherever possible because it removes the need to store your storage account key in code. You must use Microsoft Entra ID to manage credentials. This option might not be possible for your solution.
- Try to set your expiration time to the smallest useful value. If a SAS key becomes compromised, it can be exploited for only a short time.
- Apply the rule of minimum-required privileges. Only grant the access that's required. For example, in your app, read-only access is sufficient.
- There are some situations where a SAS isn't the correct solution. When there's an unacceptable risk of using a SAS, create a middle-tier service to manage users and their access to storage.

Use a SAS when you want to provide secure access to resources in your storage account to any client who doesn't otherwise have permissions to those resources.

A common scenario where a SAS is useful is a service where users read and write their own data to your storage account. In a scenario where a storage account stores user data, there are two typical design patterns:

- Clients upload and download data via a front-end proxy service, which performs authentication. This front-end proxy service has the advantage of allowing validation of business rules, but for large amounts of data or high-volume transactions, creating a service that can scale to match demand may be expensive or difficult.

- A lightweight service authenticates the client as needed and then generates a SAS. Once the client application receives the SAS, they can access storage account resources directly with the permissions defined by the SAS and for the interval allowed by the SAS. The SAS mitigates the need for routing all data through the front-end proxy service.

Many real-world services might use a hybrid of these two approaches. For example, some data might be processed and validated via the front-end proxy, while other data is saved and/or read directly using SAS.

Additionally, a SAS is required to authorize access to the source object in a copy operation in certain scenarios:

- When you copy a blob to another blob that resides in a different storage account, you must use a SAS to authorize access to the source blob. You can optionally use a SAS to authorize access to the destination blob as well.

- When you copy a file to another file that resides in a different storage account, you must use a SAS to authorize access to the source file. You can optionally use a SAS to authorize access to the destination file as well.

- When you copy a blob to a file, or a file to a blob, you must use a SAS to authorize access to the source object, even if the source and destination objects reside within the same storage account.

`A stored access policy` provides an extra level of control over service-level shared access signatures (SAS) on the server side. Establishing a stored access policy groups SAS and provides more restrictions for signatures that bound by the policy. You can use a stored access policy to change the start time, expiry time, or permissions for a signature, or to revoke it after it is issued.

The following storage resources support stored access policies:

- Blob containers
- File shares
- Queues
- Tables

The access policy for a SAS consists of the start time, expiry time, and permissions for the signature. You can specify all of these parameters on the signature URI and none within the stored access policy; all on the stored access policy and none on the URI; or some combination of the two. However, you can't specify a given parameter on both the SAS token and the stored access policy. To create or modify a stored access policy, call the Set ACL operation for the resource (see Set Container ACL, Set Queue ACL, Set Table ACL, or Set Share ACL) with a request body that specifies the terms of the access policy. The body of the request includes a unique signed identifier of your choosing, up to 64 characters in length, and the optional parameters of the access policy, as follows:

```
C#
BlobSignedIdentifier identifier = new BlobSignedIdentifier
{
    Id = "stored access policy identifier",
    AccessPolicy = new BlobAccessPolicy
    {
        ExpiresOn = DateTimeOffset.UtcNow.AddHours(1),
        Permissions = "rw"
    }
};

blobContainer.SetAccessPolicy(permissions: new BlobSignedIdentifier[] { identifier });
```

```
Azure CLI
az storage container policy create --name <stored access policy identifier> --container-name <container name> --start <start time UTC datetime> --expiry <expiry time UTC datetime> --permissions <(a)dd, (c)reate, (d)elete, (l)ist, (r)ead, or (w)rite> --account-key <storage account key> --account-name <storage account name>
```

When you establish a stored access policy on a container, table, queue, or share, it may take up to 30 seconds to take effect. During this time requests against a SAS associated with the stored access policy may fail with status code 403 (Forbidden), until the access policy becomes active. Table entity range restrictions (startpk, startrk, endpk, and endrk) cannot be specified in a stored access policy.

To modify the parameters of the stored access policy you can call the access control list operation for the resource type to replace the existing policy. For example, if your existing policy grants read and write permissions to a resource, you can modify it to grant only read permissions for all future requests.

To revoke a stored access policy you can delete it, rename it by changing the signed identifier, or change the expiry time to a value in the past. Changing the signed identifier breaks the associations between any existing signatures and the stored access policy. Changing the expiry time to a value in the past causes any associated signatures to expire. Deleting or modifying the stored access policy immediately affects all of the SAS associated with it.

To remove a single access policy, call the resource's Set ACL operation, passing in the set of signed identifiers that you wish to maintain on the container. To remove all access policies from the resource, call the Set ACL operation with an empty request body.

### 📒 Microsoft Graph <a name="603"></a>

`Microsoft Graph` is the gateway to data and intelligence in Microsoft 365. It provides a unified programmability model that you can use to access the tremendous amount of data in Microsoft 365, Windows 10, and Enterprise Mobility + Security.

In the Microsoft 365 platform, three main components facilitate the access and flow of data:

- The Microsoft Graph API offers a single endpoint, https://graph.microsoft.com. You can use REST APIs or SDKs to access the endpoint. Microsoft Graph also includes a powerful set of services that manage user and device identity, access, compliance, security, and help protect organizations from data leakage or loss.
- Microsoft Graph connectors work in the incoming direction, delivering data external to the Microsoft cloud into Microsoft Graph services and applications, to enhance Microsoft 365 experiences such as Microsoft Search. Connectors exist for many commonly used data sources such as Box, Google Drive, Jira, and Salesforce.
- Microsoft Graph Data Connect provides a set of tools to streamline secure and scalable delivery of Microsoft Graph data to popular Azure data stores. The cached data serves as data sources for Azure development tools that you can use to build intelligent applications.

Microsoft Graph is a RESTful web API that enables you to access Microsoft Cloud service resources. After you register your app and get authentication tokens for a user or service, you can make requests to the Microsoft Graph API. The Microsoft Graph API defines most of its resources, methods, and enumerations in the OData namespace, microsoft.graph, in the Microsoft Graph metadata. A few API sets are defined in their sub-namespaces, such as the call records API which defines resources like callRecord in microsoft.graph.callRecords. Unless explicitly specified in the corresponding topic, assume types, methods, and enumerations are part of the microsoft.graph namespace.

To read from or write to a resource such as a user or an email message, construct a request that looks like the following:

```
{HTTP method} https://graph.microsoft.com/{version}/{resource}?{query-parameters}
```

The components of a request include:

- {HTTP method} - The HTTP method used on the request to Microsoft Graph.
- {version} - The version of the Microsoft Graph API your application is using.
- {resource} - The resource in Microsoft Graph that you're referencing.
- {query-parameters} - Optional OData query options or REST method parameters that customize the response.

After you make a request, a response is returned that includes:

- Status code - An HTTP status code that indicates success or failure.
- Response message - The data that you requested or the result of the operation. The response message can be empty for some operations.
- nextLink - If your request returns numerous data, you need to page through it by using the URL returned in @odata.nextLink.

Microsoft Graph uses the HTTP method on your request to determine what your request is doing. The API supports the following methods: GET, POST, PATCH, PUT, DELETE.

- For the CRUD methods GET and DELETE, no request body is required.
- The POST, PATCH, and PUT methods require a request body specified in JSON format that contains additional information. Such as the values for properties of the resource.

Microsoft Graph currently supports two versions: v1.0 and beta.

- v1.0 includes generally available APIs. Use the v1.0 version for all production apps.
- beta includes APIs that are currently in preview. Because we might introduce breaking changes to our beta APIs, we recommend that you use the beta version only to test apps that are in development; don't use beta APIs in your production apps.

A resource can be an entity or complex type, commonly defined with properties. Entities differ from complex types by always including an id property. Your URL includes the resource you're interacting with in the request, such as me, user, group, drive, and site. Often, top-level resources also include relationships, which you can use to access other resources, like me/messages or me/drive. You can also interact with resources using methods; for example, to send an email, use me/sendMail Each resource might require different permissions to access it. You often need a higher level of permissions to create or update a resource than to read it. For details about required permissions, see the method reference topic.

Query parameters can be OData system query options, or other strings that a method accepts to customize its response. You can use optional OData system query options to include more or fewer properties than the default response, filter the response for items that match a custom query, or provide another parameters for a method. For example, adding the following filter parameter restricts the messages returned to only those with the emailAddress property of jon@contoso.com.

```
GET https://graph.microsoft.com/v1.0/me/messages?filter=emailAddress eq 'jon@contoso.com'
```

<b> Query Microsoft Graph by using SDKs</b>

The Microsoft Graph SDKs are designed to simplify building high-quality, efficient, and resilient applications that access Microsoft Graph. The SDKs include two components: a service library and a core library. The service library contains models and request builders that are generated from Microsoft Graph metadata to provide a rich and discoverable experience. The core library provides a set of features that enhance working with all the Microsoft Graph services. Embedded support for retry handling, secure redirects, transparent authentication, and payload compression, improve the quality of your application's interactions with Microsoft Graph, with no added complexity, while leaving you completely in control. The core library also provides support for common tasks such as paging through collections and creating batch requests. In this unit, you learn about the available SDKs and see some code examples of some of the most common operations.

The Microsoft Graph .NET SDK is included in the following NuGet packages:

- Microsoft.Graph - Contains the models and request builders for accessing the v1.0 endpoint with the fluent API. Microsoft.Graph has a dependency on Microsoft.Graph.Core.
- Microsoft.Graph.Beta - Contains the models and request builders for accessing the beta endpoint with the fluent API. Microsoft.Graph.Beta has a dependency on Microsoft.Graph.Core.
- Microsoft.Graph.Core - The core library for making calls to Microsoft Graph.

The Microsoft Graph client is designed to make it simple to make calls to Microsoft Graph. You can use a single client instance for the lifetime of the application. The following code examples show how to create an instance of a Microsoft Graph client. The authentication provider handles acquiring access tokens for the application. The different application providers support different client scenarios. For details about which provider and options are appropriate for your scenario, see Choose an Authentication Provider.

```
C#
var scopes = new[] { "User.Read" };

// Multi-tenant apps can use "common",
// single-tenant apps must use the tenant ID from the Azure portal
var tenantId = "common";

// Value from app registration
var clientId = "YOUR_CLIENT_ID";

// using Azure.Identity;
var options = new TokenCredentialOptions
{
    AuthorityHost = AzureAuthorityHosts.AzurePublicCloud
};

// Callback function that receives the user prompt
// Prompt contains the generated device code that you must
// enter during the auth process in the browser
Func<DeviceCodeInfo, CancellationToken, Task> callback = (code, cancellation) => {
    Console.WriteLine(code.Message);
    return Task.FromResult(0);
};

// /dotnet/api/azure.identity.devicecodecredential
var deviceCodeCredential = new DeviceCodeCredential(
    callback, tenantId, clientId, options);

var graphClient = new GraphServiceClient(deviceCodeCredential, scopes);
```

To `read` information from Microsoft Graph, you first need to create a request object and then run the GET method on the request.

```
C#
// GET https://graph.microsoft.com/v1.0/me

var user = await graphClient.Me
    .GetAsync();
```

`Retrieving` a list of entities is similar to retrieving a single entity except there are other options for configuring the request. The $filter query parameter can be used to reduce the result set to only those rows that match the provided condition. The $orderBy query parameter requests that the server provides the list of entities sorted by the specified properties.

```
C#
// GET https://graph.microsoft.com/v1.0/me/messages?$select=subject,sender&$filter=<some condition>&orderBy=receivedDateTime

var messages = await graphClient.Me.Messages
    .Request()
    .Select(m => new {
        m.Subject,
        m.Sender
    })
    .Filter("<filter condition>")
    .OrderBy("receivedDateTime")
    .GetAsync();
```

`Delete` requests are constructed in the same way as requests to retrieve an entity, but use a DELETE request instead of a GET.

```
C#
// DELETE https://graph.microsoft.com/v1.0/me/messages/{message-id}

string messageId = "AQMkAGUy...";
var message = await graphClient.Me.Messages[messageId]
    .Request()
    .DeleteAsync();
```

`Create a new entity`. For SDKs that support a fluent style, new items can be added to collections with an Add method. For template-based SDKs, the request object exposes a post method.

```
// POST https://graph.microsoft.com/v1.0/me/calendars

var calendar = new Calendar
{
    Name = "Volunteer"
};

var newCalendar = await graphClient.Me.Calendars
    .Request()
    .AddAsync(calendar);

```

<b>Apply best practices to Microsoft Graph</b>

This unit describes best practices that you can apply to get the most out of Microsoft Graph and make your application more reliable for end users.

To access the data in Microsoft Graph, your application needs to acquire an OAuth 2.0 access token, and presents it to Microsoft Graph in either of the following methods:

- The HTTP Authorization request header, as a Bearer token
- The graph client constructor, when using a Microsoft Graph client library

Use the Microsoft Authentication Library API, MSAL to acquire the access token to Microsoft Graph.

Apply the following best practices for consent and authorization in your app:

- Use least privilege. Only request permissions that are necessary, and only when you need them. For the APIs, your application calls check the permissions section in the method topics. For example, see creating a user and choose the least privileged permissions.
- Use the correct permission type based on scenarios. If you're building an interactive application where a signed in user is present, your application should use delegated permissions. If, however, your application runs without a signed-in user, such as a background service or daemon, your application should use application permissions. Using application permissions for interactive scenarios can put your application at compliance and security risk. Be sure to check user's privileges to ensure they don't have undesired access to information, or are circumnavigating policies configured by an administrator.
- Consider the end user and admin experience. Directly affects end user and admin experiences. For example:
  - Consider who will be consenting to your application, either end users or administrators, and configure your application to request permissions appropriately.
  - Ensure that you understand the difference between static, dynamic and incremental consent.
- Consider multi-tenant applications. Expect customers to have various application and consent controls in different states. For example:
  - Tenant administrators can disable the ability for end users to consent to applications. In this case, an administrator would need to consent on behalf of their users.
  - Tenant administrators can set custom authorization policies such as blocking users from reading other user's profiles, or limiting self-service group creation to a limited set of users. In this case, your application should expect to handle 403 error response when acting on behalf of a user.

Depending on the requests you make to Microsoft Graph, your applications should be prepared to handle different types of responses. The following are some of the most important practices to follow to ensure that your application behaves reliably and predictably for your end users. For example:

- `Pagination`: When querying resource collections, you should expect that Microsoft Graph returns the result set in multiple pages, due to server-side page size limits. Your application should always handle the possibility that the responses are paged in nature, and use the @odata.nextLink property to obtain the next paged set of results, until all pages of the result set are read. The final page doesn't include an @odata.nextLink property. For more information, visit paging.
- `Evolvable enumerations`: Adding members to existing enumerations can break applications already using these enums. Evolvable enums are a mechanism that Microsoft Graph API uses to add new members to existing enumerations without causing a breaking change for applications. By default, a GET operation returns only known members for properties of evolvable enum types and your application needs to handle only the known members. If you design your application to handle unknown members as well, you can opt in to receive those members by using an HTTP Prefer request header.

`Storing data locally`. Your application should ideally make calls to Microsoft Graph to retrieve data in real time as necessary. You should only cache or store data locally necessary for a specific scenario, and if that use case is covered by your terms of use and privacy policy, and doesn't violate the Microsoft APIs Terms of Use. Your application should also implement proper retention and deletion policies.

# 7. Implement secure Azure solutions <a name="7"></a>

### 📒 Azure Key Vault <a name="701"></a>

`The Azure Key Vault` service supports two types of containers: vaults and managed hardware security module(HSM) pools. Vaults support storing software and HSM-backed keys, secrets, and certificates. Managed HSM pools only support HSM-backed keys.

Azure Key Vault helps solve the following problems:

- `Secrets Management`: Azure Key Vault can be used to Securely store and tightly control access to tokens, passwords, certificates, API keys, and other secrets
- `Key Management`: Azure Key Vault can also be used as a Key Management solution. Azure Key Vault makes it easy to create and control the encryption keys used to encrypt your data.
- `Certificate Management`: Azure Key Vault is also a service that lets you easily provision, manage, and deploy public and private Secure Sockets Layer/Transport Layer Security (SSL/TLS) certificates for use with Azure and your internal connected resources.

Azure Key Vault has two service tiers: Standard, which encrypts with a software key, and a Premium tier, which includes hardware security module(HSM)-protected keys. To see a comparison between the Standard and Premium tiers, see the Azure Key Vault pricing page.

Key benefits of using Azure Key :

- `Centralized application secrets`: Centralizing storage of application secrets in Azure Key Vault allows you to control their distribution. For example, instead of storing the connection string in the app's code you can store it securely in Key Vault. Your applications can securely access the information they need by using URIs. These URIs allow the applications to retrieve specific versions of a secret.
- `Securely store secrets and keys`: Access to a key vault requires proper authentication and authorization before a caller (user or application) can get access. Authentication is done via Microsoft Entra ID. Authorization may be done via Azure role-based access control (Azure RBAC) or Key Vault access policy. Azure RBAC can be used for both management of the vaults and to access data stored in a vault, while key vault access policy can only be used when attempting to access data stored in a vault. Azure Key Vaults may be either software-protected or, with the Azure Key Vault Premium tier, hardware-protected by hardware security modules (HSMs).
- `Monitor access and use`: You can monitor activity by enabling logging for your vaults. You have control over your logs and you may secure them by restricting access and you may also delete logs that you no longer need. Azure Key Vault can be configured to:

  - Archive to a storage account.
  - Stream to an event hub.
  - Send the logs to Azure Monitor logs.

- `Simplified administration of application secrets`: Security information must be secured, it must follow a life cycle, and it must be highly available. Azure Key Vault simplifies the process of meeting these requirements by:

  - Removing the need for in-house knowledge of Hardware Security Modules
  - Scaling up on short notice to meet your organization’s usage spikes.
  - Replicating the contents of your Key Vault within a region and to a secondary region. Data replication ensures high availability and takes away the need of any action from the administrator to trigger the failover.
  - Providing standard Azure administration options via the portal, Azure CLI and PowerShell.
  - Automating certain tasks on certificates that you purchase from Public CAs, such as enrollment and renewal.

Azure Key Vault is a tool for securely storing and accessing secrets. A secret is anything that you want to tightly control access to, such as API keys, passwords, or certificates. A vault is logical group of secrets. To do any operations with Key Vault, you first need to authenticate to it. There are three ways to authenticate to Key Vault:

- `Managed identities for Azure resources`: When you deploy an app on a virtual machine in Azure, you can assign an identity to your virtual machine that has access to Key Vault. You can also assign identities to other Azure resources. The benefit of this approach is that the app or service isn't managing the rotation of the first secret. Azure automatically rotates the service principal client secret associated with the identity. We recommend this approach as a best practice.
- `Service principal and certificate`: You can use a service principal and an associated certificate that has access to Key Vault. We don't recommend this approach because the application owner or developer must rotate the certificate.
- `Service principal and secret`: Although you can use a service principal and a secret to authenticate to Key Vault, we don't recommend it. It's hard to automatically rotate the bootstrap secret that's used to authenticate to Key Vault.

Azure Key Vault enforces `Transport Layer Security` (TLS) protocol to protect data when it’s traveling between Azure Key Vault and clients. Clients negotiate a TLS connection with Azure Key Vault. TLS provides strong authentication, message privacy, and integrity (enabling detection of message tampering, interception, and forgery), interoperability, algorithm flexibility, and ease of deployment and use.
`Perfect Forward Secrecy` (PFS) protects connections between customers’ client systems and Microsoft cloud services by unique keys. Connections also use RSA-based 2,048-bit encryption key lengths. This combination makes it difficult for someone to intercept and access data that is in transit.

Azure Key Vault best practices:

- Use separate key vaults: Recommended using a vault per application per environment (Development, Pre-Production and Production). This pattern helps you not share secrets across environments and also reduces the threat if there is a breach.
- Control access to your vault: Key Vault data is sensitive and business critical, you need to secure access to your key vaults by allowing only authorized applications and users.
- Backup: Create regular back ups of your vault on update/delete/create of objects within a Vault.
- Logging: Be sure to turn on logging and alerts.
- Recovery options: Turn on soft-delete and purge protection if you want to guard against force deletion of the secret.

`Authentication with Key Vault` works with Microsoft Entra ID, which is responsible for authenticating the identity of any given security principal.

For applications, there are two ways to obtain a service principal:

- Enable a system-assigned managed identity for the application. With managed identity, Azure internally manages the application's service principal and automatically authenticates the application with other Azure services. Managed identity is available for applications deployed to various services.
- If you can't use managed identity, you instead register the application with your Microsoft Entra tenant. Registration also creates a second application object that identifies the app across all tenants.

It is recommended to use a system-assigned managed identity. The following is information on authenticating to Key Vault without using a managed identity.

Key Vault SDK is using Azure Identity client library, which allows seamless authentication to Key Vault across environments with same code. The table below provides information on the Azure Identity client libraries: Azure Identity SDK .NET, Azure Identity SDK Python, Azure Identity SDK Java, Azure Identity SDK JavaScript.

Access tokens must be sent to the service using the HTTP Authorization header:

```
HTTP

PUT /keys/MYKEY?api-version=<api_version>  HTTP/1.1
Authorization: Bearer <access_token>
```

When an access token isn't supplied, or when a token isn't accepted by the service, an HTTP 401 error is returned to the client and will include the WWW-Authenticate header, for example:

```
HTTP

401 Not Authorized
WWW-Authenticate: Bearer authorization="…", resource="…"
```

The parameters on the WWW-Authenticate header are:

- authorization: The address of the OAuth2 authorization service that may be used to obtain an access token for the request.
- resource: The name of the resource (https://vault.azure.net) to use in the authorization request.

### 📒 Example <a name="7011"></a>

> 1. <a href="https://learn.microsoft.com/en-us/training/modules/implement-azure-key-vault/5-set-retrieve-secret-azure-key-vault">Set and retrieve a secret from Azure Key Vault by using Azure CLI</a>

### 📒 Implement managed identities <a name="702"></a>

A common challenge for developers is the management of secrets, credentials, certificates, and keys used to secure communication between services. `Managed identities` eliminate the need for developers to manage these credentials. While developers can securely store the secrets in Azure Key Vault, services need a way to access Azure Key Vault. Managed identities provide an automatically managed identity in Microsoft Entra ID for applications to use when connecting to resources that support Microsoft Entra authentication. Applications can use managed identities to obtain Microsoft Entra tokens without having to manage any credentials.

There are two types of managed identities:

- `A system-assigned managed identity` is enabled directly on an Azure service instance. When the identity is enabled, Azure creates an identity for the instance in the Microsoft Entra tenant trusted by the subscription of the instance. After the identity is created, the credentials are provisioned onto the instance. The lifecycle of a system-assigned identity is directly tied to the Azure service instance that it's enabled on. If the instance is deleted, Azure automatically cleans up the credentials and the identity in Microsoft Entra ID.
- `A user-assigned managed identity` is created as a standalone Azure resource. Through a create process, Azure creates an identity in the Microsoft Entra tenant that's trusted by the subscription in use. After the identity is created, the identity can be assigned to one or more Azure service instances. The lifecycle of a user-assigned identity is managed separately from the lifecycle of the Azure service instances to which it's assigned.

Internally, managed identities are service principals of a special type, which are locked to only be used with Azure resources. When the managed identity is deleted, the corresponding service principal is automatically removed.

The following table highlights some of the key differences between the two types of managed identities.

| Property                       | System-assigned managed identity                                                                                                                                  | User-assigned managed                                                                                       |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Creation                       | Created as part of an Azure resource (for example, an Azure virtual machine or Azure App Service)                                                                 | Created as a stand-alone Azure resource                                                                     |
| Lifecycle                      | Shared lifecycle with the Azure resource that the managed identity is created with. When the parent resource is deleted, the managed identity is deleted as well. | Independent life-cycle. Must be explicitly deleted.                                                         |
| Sharing across Azure resources | Can't be shared, it can only be associated with a single Azure resource.                                                                                          | Can be shared. The same user-assigned managed identity can be associated with more than one Azure resource. |

Following are common use cases for managed identities:

- System-assigned managed identity
  - Workloads contained within a single Azure resource.
  - Workloads needing independent identities.
  - For example, an application that runs on a single virtual machine.
- User-assigned managed identity
  - Workloads that run on multiple resources and can share a single identity.
  - Workloads needing preauthorization to a secure resource, as part of a provisioning flow.
  - Workloads where resources are recycled frequently, but permissions should stay consistent.
  - For example, a workload where multiple virtual machines need to access the same resource.

Managed identities for Azure resources can be used to authenticate to services that support Microsoft Entra authentication. For a list of Azure services that support the managed identities for Azure resources feature, visit Services that support managed identities for Azure resources. The rest of this module uses Azure virtual machines in the examples, but the same concepts and similar actions can be applied to any resource in Azure that supports Microsoft Entra authentication.

<b>How a system-assigned managed identity works with an Azure virtual machine</b>

1. Azure Resource Manager receives a request to enable the system-assigned managed identity on a virtual machine.
2. Azure Resource Manager creates a service principal in Microsoft Entra ID for the identity of the virtual machine. The service principal is created in the Microsoft Entra tenant that's trusted by the subscription.
3. Azure Resource Manager configures the identity on the virtual machine by updating the Azure Instance Metadata Service identity endpoint with the service principal client ID and certificate.
4. After the virtual machine has an identity, use the service principal information to grant the virtual machine access to Azure resources. To call Azure Resource Manager, use role-based access control in Microsoft Entra ID to assign the appropriate role to the virtual machine service principal. To call Key Vault, grant your code access to the specific secret or key in Key Vault.
5. Your code that's running on the virtual machine can request a token from the Azure Instance Metadata service endpoint, accessible only from within the virtual machine: http://169.254.169.254/metadata/identity/oauth2/token
6. A call is made to Microsoft Entra ID to request an access token (as specified in step 5) by using the client ID and certificate configured in step 3. Microsoft Entra ID returns a JSON Web Token (JWT) access token.
7. Your code sends the access token on a call to a service that supports Microsoft Entra authentication.

<b>How a user-assigned managed identity works with an Azure virtual machine</b>

1. Azure Resource Manager receives a request to create a user-assigned managed identity.
2. Azure Resource Manager creates a service principal in Microsoft Entra ID for the user-assigned managed identity. The service principal is created in the Microsoft Entra tenant that's trusted by the subscription.
3. Azure Resource Manager receives a request to configure the user-assigned managed identity on a virtual machine and updates the Azure Instance Metadata Service identity endpoint with the user-assigned managed identity service principal client ID and certificate.
4. After the user-assigned managed identity is created, use the service principal information to grant the identity access to Azure resources. To call Azure Resource Manager, use role-based access control in Microsoft Entra ID to assign the appropriate role to the service principal of the user-assigned identity. To call Key Vault, grant your code access to the specific secret or key in Key Vault. You can also do this step before step 3.
5. Your code that's running on the virtual machine can request a token from the Azure Instance Metadata Service identity endpoint, accessible only from within the virtual machine: http://169.254.169.254/metadata/identity/oauth2/token
6. A call is made to Microsoft Entra ID to request an access token (as specified in step 5) by using the client ID and certificate configured in step 3. Microsoft Entra ID returns a JSON Web Token (JWT) access token.
7. Your code sends the access token on a call to a service that supports Microsoft Entra authentication.

<b> Configure managed identities </b>

You can configure an Azure virtual machine with a managed identity during, or after, the creation of the virtual machine. CLI examples showing the commands for both system- and user-assigned identities are used in this unit. To create, or enable, an Azure virtual machine with the system-assigned managed identity your account needs the Virtual Machine Contributor role assignment. No other Microsoft Entra directory role assignments are required.

The following example creates a virtual machine named myVM with a system-assigned managed identity, as requested by the --assign-identity parameter, with the specified --role and --scope. The --admin-username and --admin-password parameters specify the administrative user name and password account for virtual machine sign-in. Update these values as appropriate for your environment:

```
az vm create --resource-group myResourceGroup \
    --name myVM --image win2016datacenter \
    --generate-ssh-keys \
    --assign-identity \
    --role contributor \
    --scope mySubscription \
    --admin-username azureuser \
    --admin-password myPassword12
```

Use the az vm identity assign command to assign the system-assigned identity to an existing virtual machine:

```
az vm identity assign -g myResourceGroup -n myVm
```

To assign a user-assigned identity to a virtual machine during its creation, your account needs the Virtual Machine Contributor and Managed Identity Operator role assignments. No other Microsoft Entra directory role assignments are required.

Enabling user-assigned managed identities is a two-step process:

1. Create the user-assigned identity
2. Assign the identity to a virtual machine

Create a user-assigned managed identity using az identity create. The -g parameter specifies the resource group where the user-assigned managed identity is created, and the -n parameter specifies its name.

```
az identity create -g myResourceGroup -n myUserAssignedIdentity
```

The following example creates a virtual machine associated with the new user-assigned identity, as specified by the --assign-identity parameter, with the given --role and --scope.

```
az vm create \
--resource-group <RESOURCE GROUP> \
--name <VM NAME> \
--image Ubuntu2204 \
--admin-username <USER NAME> \
--admin-password <PASSWORD> \
--assign-identity <USER ASSIGNED IDENTITY NAME> \
--role <ROLE> \
--scope <SUBSCRIPTION>
```

Assign the user-assigned identity to your virtual machine using az vm identity assign.

```
az vm identity assign \
    -g <RESOURCE GROUP> \
    -n <VM NAME> \
    --identities <USER ASSIGNED IDENTITY>
```

Azure supports multiple programming platforms through a series of Azure SDKs. Several of them have been updated to support managed identities for Azure resources, and provide corresponding samples to demonstrate usage.

| SDK     | Sample                                                                                             |
| ------- | -------------------------------------------------------------------------------------------------- |
| .NET    | Manage resource from a virtual machine enabled with managed identities for Azure resources enabled |
| Java    | Manage storage from a virtual machine enabled with managed identities for Azure resources          |
| Node.js | Create a virtual machine with system-assigned managed identity enabled                             |
| Python  | Create a virtual machine with system-assigned managed identity enabled                             |
| Ruby    | Create Azure virtual machine with an system-assigned identity enabled                              |

<b> Acquire an access token</b>

A client application can request managed identities for Azure resources app-only access token for accessing a given resource. The token is based on the managed identities for Azure resources service principal. The recommended method is to use the DefaultAzureCredential. The Azure Identity library supports a DefaultAzureCredential type. DefaultAzureCredential automatically attempts to authenticate via multiple mechanisms, including environment variables or an interactive sign-in. The credential type can be used in your development environment using your own credentials. It can also be used in your production Azure environment using a managed identity. No code changes are required when you deploy your application. DefaultAzureCredential is intended to simplify getting started with the SDK by handling common scenarios with reasonable default behaviors. Developers who want more control or whose scenario isn't served by the default settings should use other credential types.

The DefaultAzureCredential attempts to authenticate via the following mechanisms, in this order, stopping when one succeeds:

1. Environment - The DefaultAzureCredential reads account information specified via environment variables and use it to authenticate.
2. Managed Identity - If the application is deployed to an Azure host with Managed Identity enabled, the DefaultAzureCredential authenticates with that account.
3. Visual Studio - If the developer authenticated via Visual Studio, the DefaultAzureCredential authenticates with that account.
4. Azure CLI - If the developer authenticated an account via the Azure CLI az login command, the DefaultAzureCredential authenticates with that account. Visual Studio Code users can authenticate their development environment using the Azure CLI.
5. Azure PowerShell - If the developer authenticated an account via the Azure PowerShell Connect-AzAccount command, the DefaultAzureCredential authenticates with that account.
6. Interactive browser - If enabled, the DefaultAzureCredential interactively authenticates the developer via the current system's default browser. By default, this credential type is disabled.

### 📒 Example <a name="7021"></a>

> 1. <a href="https://learn.microsoft.com/en-us/training/modules/implement-managed-identities/5-acquire-access-token">Acquire an access token</a>

### 📒 Implement Azure App Configuration <a name="703"></a>

Azure App Configuration provides a service to centrally manage application settings and feature flags. Modern programs, especially programs running in a cloud, generally have many components that are distributed in nature. Spreading configuration settings across these components can lead to hard-to-troubleshoot errors during an application deployment. Use App Configuration to store all the settings for your application and secure their accesses in one place.

App Configuration offers the following benefits:

- A fully managed service that can be set up in minutes
- Flexible key representations and mappings
- Tagging with labels
- Point-in-time replay of settings
- Dedicated UI for feature flag management
- Comparison of two sets of configurations on custom-defined dimensions
- Enhanced security through Azure-managed identities
- Encryption of sensitive information at rest and in transit
- Native integration with popular frameworks

App Configuration complements Azure Key Vault, which is used to store application secrets. App Configuration makes it easier to implement the following scenarios:

- Centralize management and distribution of hierarchical configuration data for different environments and geographies
- Dynamically change application settings without the need to redeploy or restart an application
- Control feature availability in real-time

The easiest way to add an App Configuration store to your application is through a client library that Microsoft provides. Based on the programming language and framework, the following best methods are available to you: App Configuration provider for .NET, App Configuration builder for .NET, pp Configuration provider for Spring Cloud, App Configuration provider for JavaScript, App Configuration provider for Python, App Configuration REST API.

`Keys` serve as the name for key-value pairs and are used to store and retrieve corresponding values. It's a common practice to organize keys into a hierarchical namespace by using a character delimiter, such as / or :. Use a convention that's best suited for your application. App Configuration treats keys as a whole. It doesn't parse keys to figure out how their names are structured or enforce any rule on them.

Here's an example of key names structured into a hierarchy based on component services:

```
AppName:Service1:ApiEndpoint
AppName:Service2:ApiEndpoint
```

The use of configuration data within application frameworks might dictate specific naming schemes for key-values. For example, Java's Spring Cloud framework defines Environment resources that supply settings to a Spring application. These resources are parameterized by variables that include application name and profile. Keys for Spring Cloud-related configuration data typically start with these two elements separated by a delimiter.Keys stored in App Configuration are case-sensitive, unicode-based strings. The keys app1 and App1 are distinct in an App Configuration store. Keep this in mind when you use configuration settings within an application because some frameworks handle configuration keys case-insensitively. You can use any unicode character in key names entered into App Configuration except for \*, ,, and \. These characters are reserved. If you need to include a reserved character, you must escape it by using \{Reserved Character}. There's a combined size limit of 10,000 characters on a key-value pair. This limit includes all characters in the key, its value, and all associated optional attributes. Within this limit, you can have many hierarchical levels for keys.

There are two general approaches to naming keys used for configuration data: flat or hierarchical. These methods are similar from an application usage standpoint, but hierarchical naming offers many advantages:

- Easier to read. Instead of one long sequence of characters, delimiters in a hierarchical key name function as spaces in a sentence.
- Easier to manage. A key name hierarchy represents logical groups of configuration data.
- Easier to use. It's simpler to write a query that pattern-matches keys in a hierarchical structure and retrieves only a portion of configuration data.

Key-values in App Configuration can optionally have a label attribute. Labels are used to differentiate key-values with the same key. A key app1 with labels A and B forms two separate keys in an App Configuration store. By default, a key-value has no label. To explicitly reference a key-value without a label, use \0 (URL encoded as %00). Label provides a convenient way to create variants of a key. A common use of labels is to specify multiple environments for the same key:

```
Key = AppName:DbEndpoint & Label = Test
Key = AppName:DbEndpoint & Label = Staging
Key = AppName:DbEndpoint & Label = Production
```

App Configuration doesn't version key values automatically as they're modified. Use labels as a way to create multiple versions of a key value. For example, you can input an application version number or a Git commit ID in labels to identify key values associated with a particular software build.

Each key-value is uniquely identified by its key plus a label that can be \0. You query an App Configuration store for key-values by specifying a pattern. The App Configuration store returns all key-values that match the pattern including their corresponding values and attributes.

Values assigned to keys are also unicode strings. You can use all unicode characters for values. There's an optional user-defined content type associated with each value. Use this attribute to store information, for example an encoding scheme, about a value that helps your application to process it properly. Configuration data stored in an App Configuration store, which includes all keys and values, is encrypted at rest and in transit. App Configuration isn't a replacement solution for Azure Key Vault. Don't store application secrets in it.

<b> Manage application features</b>

Feature management is a modern software-development practice that decouples feature release from code deployment and enables quick changes to feature availability on demand. It uses a technique called feature flags (also known as feature toggles, feature switches, and so on) to dynamically administer a feature's lifecycle.

Basic concepts
Here are several new terms related to feature management:

- `Feature flag`: A feature flag is a variable with a binary state of on or off. The feature flag also has an associated code block. The state of the feature flag triggers whether the code block runs or not.
- `Feature manager`: A feature manager is an application package that handles the lifecycle of all the feature flags in an application. The feature manager typically provides extra functionality, such as caching feature flags and updating their states.
- `Filter`: A filter is a rule for evaluating the state of a feature flag. A user group, a device or browser type, a geographic location, and a time window are all examples of what a filter can represent.

An effective implementation of feature management consists of at least two components working in concert:

- An application that makes use of feature flags.
- A separate repository that stores the feature flags and their current states.

How these components interact is illustrated in the following examples.

The basic pattern for implementing `feature flags` in an application is simple. You can think of a feature flag as a Boolean state variable used with an if conditional statement in your code:

```
if (featureFlag) {
    // Run the following code
}

```

In this case, if featureFlag is set to True, the enclosed code block is executed; otherwise, it's skipped. You can set the value of featureFlag statically, as in the following code example:

```
bool featureFlag = true;
```

You can also evaluate the flag's state based on certain rules:

```
bool featureFlag = isBetaUser();
```

You can extend the conditional to set application behavior for either state:

```
if (featureFlag) {
    // This following code will run if the featureFlag value is true
} else {
    // This following code will run if the featureFlag value is false
}
```

Each `feature flag` has two parts: a name and a list of one or more filters that are used to evaluate if a feature's state is on (that is, when its value is True). A filter defines a use case for when a feature should be turned on. When a feature flag has multiple filters, the filter list is traversed in order until one of the filters determines the feature should be enabled. At that point, the feature flag is on, and any remaining filter results are skipped. If no filter indicates the feature should be enabled, the feature flag is off. The feature manager supports appsettings.json as a configuration source for feature flags. The following example shows how to set up feature flags in a JSON file:

```
"FeatureManagement": {
    "FeatureA": true, // Feature flag set to on
    "FeatureB": false, // Feature flag set to off
    "FeatureC": {
        "EnabledFor": [
            {
                "Name": "Percentage",
                "Parameters": {
                    "Value": 50
                }
            }
        ]
    }
}
```

To use feature flags effectively, you need to externalize all the feature flags used in an application. This approach allows you to change feature flag states without modifying and redeploying the application itself.

Azure App Configuration is designed to be a centralized repository for feature flags. You can use it to define different kinds of feature flags and manipulate their states quickly and confidently. You can then use the App Configuration libraries for various programming language frameworks to easily access these feature flags from your application.

<b> Secure app configuration data</b>

In this unit you learn how to secure your apps configuration data by using:

- Customer-managed keys
- Private endpoints
- Managed

Azure App Configuration encrypts sensitive information at rest using a 256-bit AES encryption key provided by Microsoft. Every App Configuration instance has its own encryption key managed by the service and used to encrypt sensitive information. Sensitive information includes the values found in key-value pairs. When customer-managed key capability is enabled, App Configuration uses a managed identity assigned to the App Configuration instance to authenticate with Microsoft Entra ID. The managed identity then calls Azure Key Vault and wraps the App Configuration instance's encryption key. The wrapped encryption key is then stored and the unwrapped encryption key is cached within App Configuration for one hour. App Configuration refreshes the unwrapped version of the App Configuration instance's encryption key hourly. This ensures availability under normal operating conditions.

The following components are required to successfully enable the customer-managed key capability for Azure App Configuration:

- Standard tier Azure App Configuration instance
- Azure Key Vault with soft-delete and purge-protection features enabled
- An RSA or RSA-HSM key within the Key Vault: The key must not be expired, it must be enabled, and it must have both wrap and unwrap capabilities enabled

Once these resources are configured, two steps remain to allow Azure App Configuration to use the Key Vault key:

1. Assign a managed identity to the Azure App Configuration instance
2. Grant the identity GET, WRAP, and UNWRAP permissions in the target Key Vault's access policy.

You can use private endpoints for Azure App Configuration to allow clients on a virtual network to securely access data over a private link. The private endpoint uses an IP address from the virtual network address space for your App Configuration store. Network traffic between the clients on the virtual network and the App Configuration store traverses over the virtual network using a private link on the Microsoft backbone network, eliminating exposure to the public internet.

Using private endpoints for your App Configuration store enables you to:

- Secure your application configuration details by configuring the firewall to block all connections to App Configuration on the public endpoint.
- Increase security for the virtual network ensuring data doesn't escape.
- Securely connect to the App Configuration store from on-premises networks that connect to the virtual network using VPN or ExpressRoutes with private-peering.

A managed identity from Microsoft Entra ID allows Azure App Configuration to easily access other Microsoft Entra ID-protected resources, such as Azure Key Vault. The identity is managed by the Azure platform. It doesn't require you to provision or rotate any secrets.

Your application can be granted two types of identities:

- A system-assigned identity is tied to your configuration store. It's deleted if your configuration store is deleted. A configuration store can only have one system-assigned identity.
- A user-assigned identity is a standalone Azure resource that can be assigned to your configuration store. A configuration store can have multiple user-assigned identities.

To set up a managed identity using the Azure CLI, use the az appconfig identity assign command against an existing configuration store. The following Azure CLI example creates a system-assigned identity for an Azure App Configuration store named myTestAppConfigStore.

```
az appconfig identity assign \
    --name myTestAppConfigStore \
    --resource-group myResourceGroup
```

Creating an App Configuration store with a user-assigned identity requires that you create the identity and then assign its resource identifier to your store. The following Azure CLI examples create a user-assigned identity called myUserAssignedIdentity and assign it to an Azure App Configuration store named myTestAppConfigStore.

Create an identity using the az identity create command:

```
az identity create --resource-group myResourceGroup --name myUserAssignedIdentity
```

Assign the new user-assigned identity to the myTestAppConfigStore configuration store:

```
az appconfig identity assign --name myTestAppConfigStore \
    --resource-group myResourceGroup \
    --identities /subscriptions/[subscription id]/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserAssignedIdentity
```

# 8. Implement API Management <a name="8"></a>

Your solution may contain several front- and back-end services. In this scenario, how does a client know what endpoints to call? What happens when new services are introduced, or existing services are refactored? How do services handle SSL termination, authentication, and other concerns?

`The API Management gateway` (also called data plane or runtime) is the service component that's responsible for proxying API requests, applying policies, and collecting telemetry. An API gateway sits between clients and services. It acts as a reverse proxy, routing requests from clients to services. It may also perform various cross-cutting tasks such as authentication, SSL termination, and rate limiting. If you don't deploy a gateway, clients must send requests directly to back-end services. However, there are some potential problems with exposing services directly to clients:

- It can result in complex client code. The client must keep track of multiple endpoints, and handle failures in a resilient way.
- It creates coupling between the client and the backend. The client needs to know how the individual services are decomposed. That makes it harder to maintain the client and also harder to refactor services.
- A single operation might require calls to multiple services.
  Each public-facing service must handle concerns such as authentication, SSL, and client rate limiting.
- Services must expose a client-friendly protocol such as HTTP or WebSocket. This limits the choice of communication protocols.
- Services with public endpoints are a potential attack surface, and must be hardened.

A gateway helps to address these issues by decoupling clients from services.

API Management offers both managed and self-hosted gateways:

- `Managed` - The managed gateway is the default gateway component that is deployed in Azure for every API Management instance in every service tier. With the managed gateway, all API traffic flows through Azure regardless of where backends implementing the APIs are hosted.

- `Self-hosted` - The self-hosted gateway is an optional, containerized version of the default managed gateway. It's useful for hybrid and multicloud scenarios where there's a requirement to run the gateways off of Azure in the same environments where API backends are hosted. The self-hosted gateway enables customers with hybrid IT infrastructure to manage APIs hosted on-premises and across clouds from a single API Management service in Azure.

<b> API Management policies </b>

In Azure API Management, policies allow the publisher to change the behavior of the API through configuration. Policies are a collection of Statements that are executed sequentially on the request or response of an API. Policies are applied inside the gateway that sits between the API consumer and the managed API. The gateway receives all requests and usually forwards them unaltered to the underlying API. However a policy can apply changes to both the inbound request and outbound response. Policy expressions can be used as attribute values or text values in any of the API Management policies, unless the policy specifies otherwise.

The `policy definition` is a simple XML document that describes a sequence of inbound and outbound statements. The XML can be edited directly in the definition window. The configuration is divided into inbound, backend, outbound, and on-error. The series of specified policy statements is executed in order for a request and a response.

```
XML

<policies>
  <inbound>
    <!-- statements to be applied to the request go here -->
  </inbound>
  <backend>
    <!-- statements to be applied before the request is forwarded to
         the backend service go here -->
  </backend>
  <outbound>
    <!-- statements to be applied to the response go here -->
  </outbound>
  <on-error>
    <!-- statements to be applied if there is an error condition go here -->
  </on-error>
</policies>

```

If there's an error during the processing of a request, any remaining steps in the inbound, backend, or outbound sections are skipped and execution jumps to the statements in the on-error section. By placing policy statements in the on-error section you can review the error by using the context.LastError property, inspect and customize the error response using the set-body policy, and configure what happens if an error occurs.

Unless the policy specifies otherwise, `policy expressions` can be used as attribute values or text values in any of the API Management policies. A policy expression is either:

- a single C# statement enclosed in @(expression), or
- a multi-statement C# code block, enclosed in @{expression}, that returns a value

Each expression has access to the implicitly provided context variable and an allowed subset of .NET Framework types.

`Policy expressions`provide a sophisticated means to control traffic and modify API behavior without requiring you to write specialized code or modify backend services. The following example uses policy expressions and the set-header policy to add user data to the incoming request. The added header includes the user ID associated with the subscription key in the request, and the region where the gateway processing the request is hosted.

```
XML

<policies>
    <inbound>
        <base />
        <set-header name="x-request-context-data" exists-action="override">
            <value>@(context.User.Id)</value>
            <value>@(context.Deployment.Region)</value>
      </set-header>
    </inbound>
</policies>
```

If you have a policy at the global level and a policy configured for an API, then whenever that particular API is used both policies are applied. API Management allows for deterministic ordering of combined policy statements via the base element.

```
<policies>
    <inbound>
        <cross-domain />
        <base />
        <find-and-replace from="xyz" to="abc" />
    </inbound>
</policies>
```

In the previous example policy definition, The cross-domain statement would execute first. The find-and-replace policy would execute after any policies at a broader scope.

The policy defined in following example demonstrates how to filter data elements from the response payload based on the product associated with the request. The snippet assumes that response content is formatted as JSON and contains root-level properties named### 📒 Example <a name="7011"></a>

  </backend>
  <outbound>
    <base />
    <choose>
      <when condition="@(context.Response.StatusCode == 200 && context.Product.Name.Equals("Starter"))">
        <!-- NOTE that we are not using preserveContent=true when deserializing response body stream into a JSON object since we don't intend to access it again. See details on /azure/api-management/api-management-transformation-policies#SetBody -->
        <set-body>
          @{
            var response = context.Response.Body.As<JObject>();
            foreach (var key in new [] {"minutely", "hourly", "daily", "flags"}) {
            response.Property (key).Remove ();
           }
          return response.ToString();
          }
    </set-body>
      </when>
    </choose>
  </outbound>
  <on-error>
    <base />
  </on-error>
</policies>

```

<b> Create advanced policies </b>

This unit provides a reference for the following API Management policies:

- Control flow - Conditionally applies policy statements based on the results of the evaluation of Boolean expressions.
- Forward request - Forwards the request to the backend service.
- Limit concurrency - Prevents enclosed policies from executing by more than the specified number of requests at a time.
- Log to Event Hub - Sends messages in the specified format to an Event Hub defined by a Logger entity.
- Mock response - Aborts pipeline execution and returns a mocked response directly to the caller.
- Retry - Retries execution of the enclosed policy statements, if and until the condition is met. Execution will repeat at the specified time intervals and up to the specified retry count.

`Control flow`
The choose policy applies enclosed policy statements based on the outcome of evaluation of boolean expressions, similar to an if-then-else or a switch construct in a programming language.

```

XML
<choose>
<when condition="Boolean expression | Boolean constant">
<!— one or more policy statements to be applied if the above condition is true -->
</when>
<when condition="Boolean expression | Boolean constant">
<!— one or more policy statements to be applied if the above condition is true -->
</when>
<otherwise>
<!— one or more policy statements to be applied if none of the above conditions are true -->
</otherwise>
</choose>

```

The control flow policy must contain at least one <when/> element. The <otherwise/> element is optional. Conditions in <when/> elements are evaluated in order of their appearance within the policy. Policy statement(s) enclosed within the first <when/> element with condition attribute equals true will be applied. Policies enclosed within the <otherwise/> element, if present, will be applied if all of the <when/> element condition attributes are false.

`Forward request`
The forward-request policy forwards the incoming request to the backend service specified in the request context. The backend service URL is specified in the API settings and can be changed using the set backend service policy. Removing this policy results in the request not being forwarded to the backend service and the policies in the outbound section are evaluated immediately upon the successful completion of the policies in the inbound section.

```

XML
<forward-request timeout="time in seconds" follow-redirects="true | false"/>

```

`Limit concurrency`
The limit-concurrency policy prevents enclosed policies from executing by more than the specified number of requests at any time. Upon exceeding that number, new requests will fail immediately with a 429 Too Many Requests status code.

```

XML
<limit-concurrency key="expression" max-count="number">
<!— nested policy statements -->
</limit-concurrency>

```

`Log to Event Hub`
The log-to-eventhub policy sends messages in the specified format to an Event Hub defined by a Logger entity. As its name implies, the policy is used for saving selected request or response context information for online or offline analysis.

```

XML
<log-to-eventhub logger-id="id of the logger entity" partition-id="index of the partition where messages are sent" partition-key="value used for partition assignment">
Expression returning a string to be logged
</log-to-eventhub>

```

`Mock response`
The mock-response, as the name implies, is used to mock APIs and operations. It aborts normal pipeline execution and returns a mocked response to the caller. The policy always tries to return responses of highest fidelity. It prefers response content examples, whenever available. It generates sample responses from schemas, when schemas are provided and examples are not. If neither examples or schemas are found, responses with no content are returned.

```

<mock-response status-code="code" content-type="media type"/>

```

`Retry`
The retry policy executes its child policies once and then retries their execution until the retry condition becomes false or retry count is exhausted.

```

XML
<retry
    condition="boolean expression or literal"
    count="number of retry attempts"
    interval="retry interval in seconds"
    max-interval="maximum retry interval in seconds"
    delta="retry interval delta in seconds"
    first-fast-retry="boolean expression or literal">

<!-- One or more child policies. No restrictions -->
</retry>

```

`Return response`
The return-response policy aborts pipeline execution and returns either a default or custom response to the caller. Default response is 200 OK with no body. Custom response can be specified via a context variable or policy statements. When both are provided, the response contained within the context variable is modified by the policy statements before being returned to the caller.

```

XML
<return-response response-variable-name="existing context variable">
<set-header/>
<set-body/>
<set-status/>
</return-response>

```

<b>Secure APIs by using subscriptions</b>

When you publish APIs through API Management, it's easy and common to secure access to those APIs by using subscription keys. Developers who need to consume the published APIs must include a valid subscription key in HTTP requests when they make calls to those APIs. Otherwise, the calls are rejected immediately by the API Management gateway. They aren't forwarded to the back-end services. To get a subscription key for accessing APIs, a subscription is required. A subscription is essentially a named container for a pair of subscription keys. Developers who need to consume the published APIs can get subscriptions. And they don't need approval from API publishers. API publishers can also create subscriptions directly for API consumers. API Management also supports other mechanisms for securing access to APIs, including: OAuth2.0, Client certificates, and IP allow listing.

A `subscription key` is a unique auto-generated key that can be passed through in the headers of the client request or as a query string parameter. The key is directly related to a subscription, which can be scoped to different areas. Subscriptions give you granular control over permissions and policies.

The three main subscription scopes are:

- `All APIs`. Applies to every API accessible from the gateway
- `Single API`. This scope applies to a single imported API and all of its endpoints
- `Product`. A product is a collection of one or more APIs that you configure in API Management. You can assign APIs to more than one product. Products can have different access rules, usage quotas, and terms of use.

Applications that call a protected API must include the key in every request. You can regenerate these subscription keys at any time, for example, if you suspect that a key has been shared with unauthorized users. Every subscription has two keys, a primary and a secondary. Having two keys makes it easier when you do need to regenerate a key. For example, if you want to change the primary key and avoid downtime, use the secondary key in your apps. For products where subscriptions are enabled, clients must supply a key when making calls to APIs in that product. Developers can obtain a key by submitting a subscription request. If you approve the request, you must send them the subscription key securely, for example, in an encrypted message. This step is a core part of the API Management workflow.

Applications must include a valid key in all HTTP requests when they make calls to API endpoints that are protected by a subscription. Keys can be passed in the request header, or as a query string in the URL. The default header name is Ocp-Apim-Subscription-Key, and the default query string is subscription-key. To test out your API calls, you can use the developer portal, or command-line tools, such as curl. Here's an example of a GET request using the developer portal, which shows the subscription key header.

Here's how you can pass a key in the request header using curl:

```

Bash
curl --header "Ocp-Apim-Subscription-Key: <key string>" https://<apim gateway>.azure-api.net/api/path

```

Here's an example curl command that passes a key in the URL as a query string:

```

Bash
curl https://<apim gateway>.azure-api.net/api/path?subscription-key=<key string>

```

If the key is not passed in the header, or as a query string in the URL, you'll get a 401 Access Denied response from the API gateway.

`Certificates` can be used to provide Transport Layer Security (TLS) mutual authentication between the client and the API gateway. You can configure the API Management gateway to allow only requests with certificates containing a specific thumbprint. The authorization at the gateway level is handled through inbound policies.

With TLS client authentication, the API Management gateway can inspect the certificate contained within the client request and check for properties like:

- `Certificate Authority (CA)` Only allow certificates signed by a particular CA
- `Thumbprint` Allow certificates containing a specified thumbprint
- `Subject` Only allow certificates with a specified subject
- `Expiration Date` Only allow certificates that have not expired

These properties are not mutually exclusive and they can be mixed together to form your own policy requirements. For instance, you can specify that the certificate passed in the request is signed by a certain certificate authority and hasn't expired. Client certificates are signed to ensure that they are not tampered with. When a partner sends you a certificate, verify that it comes from them and not an imposter. There are two common ways to verify a certificate:

- Check who issued the certificate. If the issuer was a certificate authority that you trust, you can use the certificate. You can configure the trusted certificate authorities in the Azure portal to automate this process.
- If the certificate is issued by the partner, verify that it came from them. For example, if they deliver the certificate in person, you can be sure of its authenticity. These are known as self-signed certificates.

The Consumption tier in API Management is designed to conform with serverless design principals. If you build your APIs from serverless technologies, such as Azure Functions, this tier is a good fit. In the Consumption tier, you must explicitly enable the use of client certificates, which you can do on the Custom domains page. This step is not necessary in other tiers.

Create `Certificate Authorization Policies` in the inbound processing policy file within the API Management gateway. Every client certificate includes a thumbprint, which is a hash, calculated from other certificate properties. The thumbprint ensures that the values in the certificate have not been altered since the certificate was issued by the certificate authority. You can check the thumbprint in your policy. The following example checks the thumbprint of the certificate passed in the request:

```

XML
<choose>
<when condition="@(context.Request.Certificate == null || context.Request.Certificate.Thumbprint != "desired-thumbprint")" >
<return-response>
<set-status code="403" reason="Invalid client certificate" />
</return-response>
</when>
</choose>

```

In the previous example, only one thumbprint would work so only one certificate would be validated. Usually, each customer or partner company would pass a different certificate with a different thumbprint. To support this scenario, obtain the certificates from your partners and use the Client certificates page in the Azure portal to upload them to the API Management resource. Then add this code to your policy:

```

XML

<choose>
    <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Verify()  || !context.Deployment.Certificates.Any(c => c.Value.Thumbprint == context.Request.Certificate.Thumbprint))" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

This example checks the issuer and subject of the certificate passed in the request:

```
XML
<choose>
    <when condition="@(context.Request.Certificate == null || context.Request.Certificate.Issuer != "trusted-issuer" || context.Request.Certificate.SubjectName.Name != "expected-subject-name")" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

### 📒 Example <a name="801"></a>

> 1. <a href="https://learn.microsoft.com/en-us/training/modules/explore-api-management/8-exercise-import-api">Create a backend API</a>

1 - 4 - 2,30 - v
2 - 2 - 0,53 - v
3 - 3 - 1,19 - v
4 - 2 - 1,19 - v
5 - 3 - 1,42 - v
6 - 4 - 1,25 - v
7 - 3 - 1,05 - v
8 - 1 - 0,42 - v
9 - 2 - 0,56 - 26
10 - 1 - 0,57 - 27
11 - 1 - 0,24 - 27
12 - 2 - 0,55 - 28

```

```
