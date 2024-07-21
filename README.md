# Developing Solutions for Microsoft Azure

1. [Azure App Service](#1)

- [Explore Azure App Service](#11)
  - [Service plans](#1101)
  - [Deploy to App Service](#1102)
  - [Authentication and authorization](#1103)
  - [Networking features](#1104)
    - [Examples](105)
- [Configure web app settings](#12)
  - [Configure general settings](#1201)
  - [Configure path mappings](#1202)
  - [Enable diagnostic logging](#1203)
  - [Configure security certificates](#1204)
- [Scale apps in Azure App Service](#13)
  - [Identify autoscale factors](#1301)
  - [Enable autoscale](#1302)
  - [Autoscale best practices](#1303)
- [Azure App Service deployment slots](#14)

2. [Implement Azure Functions](#15)

- [Explore Azure Function](#16)
- [Develop Azure Function](#17)

3. [Develop solutions that use Blob storage](#18)

### 📒 Azure App Service <a name="1"></a>

### 📒 Explore Azure App Service <a name="11"></a>

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

### 📒 Service plans <a name="1101"></a>

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

### 📒 Deploy to App Service <a name="1102"></a>

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

### 📒 Authentication and authorization <a name="1103"></a>

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

### 📒 Networking features <a name="1104"></a>

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

### 📒 Examples <a name="1105"></a>

> Deploy a basic HTML+CSS site to Azure App Service by using the Azure CLI `az webapp up` command. Then update the code and redeploy it by using the same command.

The `az webapp up` command makes it easy to create and update web apps. When executed it performs the following actions:

- Create a default resource group if one isn't specified.
- Create a default app service plan.
- Create an app with the specified name.
- Zip deploy files from the current working directory to the web app.

Exercise: Create a static HTML web app by using Azure Cloud Shell
https://learn.microsoft.com/en-us/training/modules/introduction-to-azure-app-service/7-create-html-web-app

---

### 📒 Configure web app settings <a name="12"></a>

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

### 📒 Configure general settings <a name="1201"></a>

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

### 📒 Configure path mappings <a name="1202"></a>

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

### 📒 Enable diagnostic logging <a name="1203"></a>

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

### 📒 Configure security certificates <a name="1204"></a>

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

### 📒 Scale apps in Azure App Service <a name="13"></a>

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

### 📒 Identify autoscale factors <a name="1301"></a>

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

### 📒 Enable autoscale <a name="1302"></a>

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

### 📒 Autoscale best practices <a name="1303"></a>

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

### 📒 Azure App Service deployment slots <a name="14"></a>

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
