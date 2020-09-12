# IoT-RaspberryPi-Scale
Implementing a RaspberryPi as an IoT device to weigh objects and show the weight on a web app
<img src="https://drive.google.com/uc?export=view&id=1EDToCJYf-1djUHyYib_CJt84bjaeWEHZ" width="500">

<img src="https://drive.google.com/uc?export=view&id=1P3COD9YsuKv7YA0wJdXE4lCbruYsp3Fv" width="600">



## Table of Contents
* [About the project](#about-the-project)
* **Step 1** [Setup Raspberry Pi](#setup-raspberry-pi)
* **Step 2** [Stream data to Azure IoT Hub](#stream-data-to-azure-iot-hub)
* **Step 3** [Show data on Dotnet web app](#show-data-on-dotnet-web-app)

## About the project
The **aim** of this project is to present a way to use a Raspberry Pi Model 1 and a load cell as a scale. The Pi should then be configured as an IoT device via Azure IoT Hub. All the data streamed to the IoT Hub should then be presented on a ASP.NET Web App by using Azure Function.
All the source code referenced in this project can be found either in this repository or in one of the following repositories:
* [Azure Function](https://github.com/jekolbe/IoT-RaspberryPi-Scale-Azure-Function)
* [ASP.NET Web App](https://github.com/jekolbe/IoT-RaspberryPi-Scale-WebApp)

## Setup Raspberry Pi
First of all we have to setup the Pi to weigh objects. To do so I am referring to this [tutorial](https://tutorials-raspberrypi.de/raspberry-pi-waage-bauen-gewichtssensor-hx711/).
### Needed materials
We need the following:
* [Load Cell & HX711 sensor](https://www.amazon.de/gp/product/B07L82YWPV/ref=ppx_yo_dt_b_asin_title_o05_s01?ie=UTF8&psc=1)
* [Jumper cable & breadboard](https://www.amazon.de/gp/product/B078JGQKWP/ref=ppx_yo_dt_b_asin_title_o05_s00?ie=UTF8&psc=1)
* 2 boards
* M4 & M5 screws & nuts with about 5 cm length
* Raspberry Pi (Model 1 or newer)

### Mount the load cell
You should screw the load cell to the bottom part of your setup.

<img src="https://drive.google.com/uc?export=view&id=1vsJ1lDNEGJMJXQbsmKYEYWTfHEFyitwR" width="300">

After this you can also mount the top part and it should look something like this.

<img src="https://drive.google.com/uc?export=view&id=1vs3QUzlrAaI5DuxFxXhAz5Sk0sxeEGcF" width="300">

### Connecting the HX711
Now we somehow have to connect the four cables of the HX711 to the Pi via the breadboard. Just follow these connection pattern:
* Red: E+
* Black: E-
* Green: A-
* White: A+

We can leave B- and B+ empty.

<img src="https://drive.google.com/uc?export=view&id=1w2gtAXfPY_oxsHO4_tTr6H4D04rnA-_2" width="300">

At this point you have to use four jumper cables connecting the HX711 and the pi. You can also use different kind of colors.
* Black: GND to pin 6 (Ground)
* Orange: DT to pin 12 (GPIO 18)
* Yellow: SCK to pin 16 (GPIO 23)
* Red: VCC to pin 2 (+5V)
You can find a scheme of the pins for the Pi 1 [here](https://developer-blog.net/wp-content/uploads/2013/09/raspberry-pi-rev2-gpio-pinout.jpg). If you are using a newer Pi with more Pins then just look up and connect to orange (DT) and yellow (SCK) to empty pins. Later on you have to change the pin number in the code.

### Configuring the software
In order to weigh an object we are going to use a python library. First of all you have to clone this library on the Pi:
```
git clone https://github.com/tatobari/hx711py
```
You will find a `example.py` which you can also find as a final version in this repository in order to assist you.
You can execute this script on your raspberry pi by doing the following in the command line.
```
sudo python example.py
```
Before adding some weight you will see some values. You can just ignore them and add a object of which you know the weight. As soon as you add the weight to the board you can divide the appearing values by your weight in gram. In my case I used a glas of honey with about 400 gram
For example: -310000 ÷ 400 = -775
Now I know that -775 is my reference unit. We can now go on by editing the `example.py`
```
cd hx711py
sudo nano example.py
```
Search for the line which says `referenceunit = 1` and change the value to your reference unit.
```
referenceunit = -775
```
You also have to pay attention to line which says `hx = HX711(5, 6)`. The numbers 5 and 6 indicate which GPIO are used. In my case I had to change the numbers to 18 and 23 as I described earlier. If setup up correctly you will now see some values which should be the weight of the objects in gram.

## Stream data to Azure IoT Hub
The next step is to stream the data to the [Azure IoT Hub](https://azure.microsoft.com/de-de/services/iot-hub/). In this step I am using this [YouTube video](https://www.youtube.com/watch?v=7T91jbtm95A) as a reference. Feel free to check out the steps over there.

### Install dependencies
We have to install the following dependencies on the pi. Make sure you are using at least Jessie as debian version on your pi. Even better would be Stretch or Buster. Otherwise you will run into issues with Python 3.
```
sudo pip3 install azure-iot-device
sudo pip3 install azure-iot-hub
sudo pip3 install azure-iothub-service-client
sudo pip3 install azure-iothub-device-client
```

### Create Azure IoT Hub
Next you will need some kind of Azure subscription in order to use the services from the Azure portal. You can register for a trial [here](https://azure.microsoft.com/de-de/free/)

Navigate to the [Azure Portal](https://portal.azure.com/) and click on "Create new ressource". Search for "IoT Hub" and click on "Create". Next you should fill in the form. I also recommend to Create a new ressource group and call it something like `iot-prototyp`. In this step you also have to give your IoT Hub a name, in my case `iot-waage`. Click on the tab "Size and scale" at the top and choose "F1: Free tier" in the dropdown. Click on "Review and create".
<img src="https://drive.google.com/uc?export=view&id=1nF4-7FDvSqHR-NOZSqGazoCKS1pif6AW" width="500">
                                                                                                  
Now we have to add the Raspberry Pi as new device to the hub. Go to "IoT devices" and click on "New" as seen below.
<img src="https://drive.google.com/uc?export=view&id=1SV4cFxvCWMdKnJVbBZndKG60Tlk01mEk" width="500">

Chosse "raspberrypi" as Device Id on the next screen and leave anything else as it is. Click on "Save"

### Use python script for data transfer
Next we need to run a script on the pi to transfer telemetry data to Azure IoT Hub. You will find the script in this repository with the name `SimulatedDevice.py`. Copy this file to your pi. In this file you need to replace the connection string on line 15. Navigate to the list of IoT devices in the IoT Hub which is shown in the picture above. Click on the device you have just created. Copy the "Primary Connection String" on the next screen and paste it in the `SimulatedDevice.py`. The line will look something like:
```
CONNECTION_STRING = "HostName=iot-waage.azure-devices.net;DeviceId=raspberrypi;SharedAccessKey=<YOUR KEY>"
```

Open the cloud shell in Azure Portal and paste in the following line.
<img src="https://i2.wp.com/www.ntweekly.com/wp-content/uploads/2019/05/052019_0051_ConnectToEx1.png?w=525&ssl=1" width="200">

```
az extension add --name azure-cli-iot-ext
```

After the execution we can listen for incoming events by using the following line. Make sure to use the correct names of the hub and device in case you chose different names.
```
az iot hub monitor-events --hub-name iot-waage --device-id raspberrypi
```

Go to your pi and execute the script.
```
sudo python3 SimulatedDevice.py
```

After some seconds you should see the first incoming events on the cloud shell in the Azure Portal. The sample data will just present random temperature and humidity values. In the next step we will include the weight of objects from step 1.

If this works then you can use the script `SendWeight.py` I have provided in this repository to send the weight of an object which is placed on the scale. Make sure to replace the right GPIO numbers at `hx = HX711(18,23)` and the reference unit at `REFERENCE_UNIT = -775` with your own values from step 1.

## Show data on Dotnet web app
In this step we want to show the weight of objects on the scale at the web app. I am referring to this [tutorial](https://www.theta.co.nz/news-blogs/tech-blog/exploring-digital-twins-part-2-iot-hub-azure-functions-and-signalr/)
If you have problems please see the code in this repository --> [ASP.NET Web App](https://github.com/jekolbe/IoT-RaspberryPi-Scale-WebApp)

### Creating a SignalR application
To create a SignalR application in Visual Studio you should follow these [documentation](https://docs.microsoft.com/en-us/aspnet/core/tutorials/signalr?view=aspnetcore-2.2&tabs=visual-studio) In this case it is sufficient to create the project and add the client library. In the next step the documentation describes a way to set up a ChatHub. What we need is a message hub. So create a new folder `Hubs` in your dotnet project and create a `MessageHub.cs` with the following content.
```csharp
using Microsoft.AspNetCore.SignalR;
using System.Threading.Tasks;
 
namespace IotSignalRApp.Hubs
{
    public class MessageHub : Hub
    {
        //The SendMessage method can be called by a connected client to send a message to all clients
        public async Task SendMessage(string device, string message)
        {
            await Clients.All.SendAsync("ReceiveMessage", device, message);
        }
    }
}
```

Next configure SignalR as described in the documentation. For this you have to change the `startup.cs` like the following:

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using SignalRChat.Hubs;

namespace SignalRChat
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.Configure<CookiePolicyOptions>(options =>
            {
                // This lambda determines whether user consent for non-essential cookies is needed for a given request.
                options.CheckConsentNeeded = context => true;
                options.MinimumSameSitePolicy = SameSiteMode.None;
            });


            services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

            services.AddSignalR();
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Error");
                app.UseHsts();
            }

            app.UseHttpsRedirection();
            app.UseStaticFiles();
            app.UseCookiePolicy();
            app.UseSignalR(routes =>
            {
                routes.MapHub<MessageHub>("/messagehub");
            });
            app.UseMvc();
        }
    }
}
```

You now have to add some JavaScript functionality. Therefore crate a file `message.js` in `wwwroot/js` with the following content
```javascript
"use strict";

var connection = new signalR.HubConnectionBuilder().withUrl("/messageHub").build();

connection.start().then(function () {
    console.log("connected");
});

connection.on("ReceiveMessage", function (device, message) {
    var msg = message.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
    var encodedMsg = device + " says " + msg;
    var json = JSON.parse(msg);
    var li = document.createElement("li");
    li.textContent = encodedMsg;
    document.getElementById("weight").innerHTML = Math.round(json.weight) + " Gramm";
    //document.getElementById("messagesList").appendChild(li);
});
```

Go to `Pages` and change `index.cshtml`
```html
@page
@model IndexModel
@{
    ViewData["Title"] = "Home page";
}


<div class="container">
    <div class="row">&nbsp;</div>
    <div class="row">
        <div class="col-6">&nbsp;</div>
        <div class="col-6">
            <ul id="messagesList"></ul>
        </div>
    </div>
    <div class="row">
        <div class="col-6">
            <h1>Gewicht Honig - Regal 1 - Fach 1</h1>
            <div id="weight"></div>
        </div>
    </div>
</div>
<script src="~/lib/signalr/dist/browser/signalr.js"></script>
<script src="~/js/messages.js"></script>
```

Please note that in `message.js` the last line ist commented out. If you comment it in, you will see all the events as a list in the web app. But for now we only want to see one value.

You can test the web app by hitting F5. If everything looks good (you won't see any events by now) you can deploy the app according to the [documentation](https://docs.microsoft.com/en-us/aspnet/core/tutorials/publish-to-azure-webapp-using-vs?view=aspnetcore-2.2#deploy-the-app-to-azure) It is recommended that you are using the sames ressource group as before. You can stop the set up as soon as it comes to SQL database as we don't need a database.

### Obtaining name from Built-In Event Hub
Azure provides the option to create a new service called Event Hub. In this case we can use the built-in endpoints and event hub from the IoT Hub. To do so navigate to the following page. At this point copy the name and endpoint string to some temporary editor file.

<img src="https://www.theta.co.nz/media/3641/image-0001.png" width="300">

### Setup Azure Function
To develop the serverless Azure function we will use Visual Studio Code. In order to start with the code we need to prepare VS Code first.
* Install the Azure Function Core Tools as described [here](https://docs.microsoft.com/de-de/azure/developer/javascript/tutorial-vscode-serverless-node-01?tabs=bash#install-the-azure-functions-core-tools)
* Install [C# extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* Install [NuGet Package Manager extension](https://marketplace.visualstudio.com/items?itemName=jmrog.vscode-nuget-package-manager)

Now we can create a new function by hitting F1 and type `Azure Functions: Create Function…`
* Chose `EventHubTrigger` as template
* Chose a function name e.g. `IoTEventHubFunction`
* Chose a namespace e.g. `IoTPrototyp.IoTEventHubFunction`
* Chose `Create new local app settings`
* Select your Azure subscription
* Chose a namespace for the event hub e.g. `iot-waage-eventhub`
* Enter the event hub name you have copied earlier from the Azure Portal
* Chose Storage Account or configure new one according to these [instructions](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal)

After configuring chose `In current window`.

The project should look something like this.
<img src="https://www.theta.co.nz/media/3646/image-004.png" width="300">

Now you need to change the `local.seetings.json` by replacing the endpoint with the string you have copied earlier.
```
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=<account_name>;AccountKey=<account_key>;EndpointSuffix=core.windows.net",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet",
    "innolab-iot-EVENTHUB": "Endpoint=<your_enpoint>.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=<access_key>;EntityPath=<event_hub_name>"
  }
}
```

You now have to change the `IoTEventHubFunction.cs` with the following information. The event hub name is the name you have copied from the Azure Portal.
```csharp
namespace Innolab.IoTEventHubFunction
{
    public static class IoTEventHubFunction
    {
        [FunctionName("IoTEventHubFunction")]
        public static async Task Run([EventHubTrigger("<event_hub_name>", Connection="iot-waage")] EventData[] events, ILogger log)
        {
```

Exectue these two commands in VS Code
```
dotnet add package Microsoft.AspNetCore.SignalR.Client
dotnet restore
```

Add the logic to `IoTEventHubFunction.cs` and change the values at `<event-hub-name>` and `<signalr-app>`. The event hub name is the name you have copied earlier. The signalr app url can be accessed by navigating to your Azure web app and copying the url from there in the overview.
```chsarp
using System; 
using System.Collections.Generic; 
using System.Linq; 
using System.Text; 
using System.Threading.Tasks; 
using Microsoft.Azure.EventHubs; 
using Microsoft.Azure.WebJobs; 
using Microsoft.AspNetCore.SignalR.Client; 
using Microsoft.Extensions.Logging; 
using Newtonsoft.Json; 
 
namespace Innolab.IoTEventHubFunction 
{ 
    public static class IoTEventHubFunction 
    { 
        [FunctionName("IoTEventHubFunction")] 
        public static async Task Run([EventHubTrigger("<event-hub-name>", Connection="iot-waage")] EventData[] events, ILogger log) 
        { 
            var exceptions = new List<Exception>(); 
 
            foreach (EventData eventData in events) 
            { 
                try 
                { 
                    string messageBody = Encoding.UTF8.GetString(eventData.Body.Array, eventData.Body.Offset, eventData.Body.Count); 
 
                    Dictionary<string, string> messageObj = JsonConvert.DeserializeObject<Dictionary<string, string>>(messageBody); 
                    string device = messageObj.TryGetValue("device", out device) ? device : "unknown"; 
                     
                    var connection = new HubConnectionBuilder() 
                        .WithUrl("https://<signalr-app>.azurewebsites.net/messageHub") 
                        .Build(); 
 
                    connection.StartAsync().Wait(); 
 
                    connection.InvokeAsync("SendMessage", device, messageBody).Wait(); 
 
                    connection.DisposeAsync().Wait(); 
 
                    log.LogInformation($"C# Event Hub trigger function processed a message: {messageBody}"); 
 
                    await Task.Yield(); 
                } 
                catch (Exception e) 
                { 
                    // We need to keep processing the rest of the batch - capture this exception and continue. 
                    // Also, consider capturing details of the message that failed processing so it can be processed again later. 
                    exceptions.Add(e); 
                } 
            } 
 
            // Once processing of the batch is complete, if any messages in the batch failed processing throw an exception so that there is a record of the failure. 
 
            if (exceptions.Count > 1) 
                throw new AggregateException(exceptions); 
 
            if (exceptions.Count == 1) 
                throw exceptions.Single(); 
        } 
    } 
} 
```

By hitting F5 you can test the Azure function locally. If you execute the `SendWeight.py` script on your pi by entering `sudo python3 SendWeight.py` you should be see a value when you go to your SignalR Web App. 

Deploy your Azure function by hitting F1 and provide the following:
* Chose a name e.g. `IoTAzureFunction`
* Select Windows as OS
* Chose consumption plan
* Chose .NET as runtime
* Chose ressource group you have defined in step 2
* Chose storage account

You have to click `Upload Seetings` when your deployment has finished. Find details [here](https://docs.microsoft.com/de-de/azure/azure-functions/functions-develop-vs-code?tabs=csharp#publish-application-settings)
<img src="https://www.theta.co.nz/media/3640/screen-shot-2019-07-04-at-22744-pm.png" width="300">

**Hopefully it works!**
