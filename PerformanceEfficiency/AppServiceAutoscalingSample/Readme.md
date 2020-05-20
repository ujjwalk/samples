## Application Service Autoscaling Sample

In this sample you create an Azure App Service plan which includes an Azure App Service. Then you deploy a basic Asp.Net Core MVC application that you can use to simulate a CPU spike. The App Service plan is configured with a basic S1 SKU (1 core, 1.75 GB) to easily create the conditions for the autoscaling scenario. You can deploy the azure resources by publishing the Web Application, but it's recommended to use the provided ARM template since it has a set of custom autoscale rules.

### Autoscale rules

For this scenario, the App Service plan Scale-out custom setting is configured with the following rule combination:

Increase instances by 1 count when CPU% > 80
Decrease instances by 1 count when CPU% <= 60


This margin between the scale-out and in and the threshold is recommended, consider this case:

Assuming there is 1 instance to start with. If the average CPU% across instances is 81 (the CPU% usage of the only instance), autoscale scales out adding a second instance. Then over time the CPU% falls to 60. Autoscale's scale-in rule estimates the final state if it were to scale-in. For example, 60 x 2 (current instance count) = 120 / 1 (final number of instances when scaled down) = 120. So autoscale does not scale-in because it would have to scale-out again immediately. Instead, it skips scaling down. The next time autoscale checks, the CPU continues to fall to 30%. It estimates again - 30 x 2 instance = 60 / 1 instance = 60, which is below the scale-out threshold of 80, so it scales in successfully to 1 instance.

The duration is set to 5 minutes. This is amount of time that Autoscale engine will look back for metrics. So in this case, 5 minutes means that every time autoscale runs, it will query metrics for the past 5 minutes. This allows your metrics to stabilize and avoids reacting to transient spikes. 

 The instance limits are:  max 5 instances, min 1 instance. and the cool down setting is set to 5 minutes (The cool down setting is the amount of time to wait after a scale operation before scaling again. In this case, since the cooldown is 5 minutes and a scale operation just occurred, Autoscale will not attempt to scale again until after 5 minutes. This is to allow the metrics to stabilize first.)

 These settings may not be valid for a real scenario, but are intentionally set (in conjunction with a small SKU for the app service plan) to easily reproduce the autoscale conditions.

 ### Deployment instructions


#### Run these commands by using the Azure CLI from your computer. You need to run az login to log in to Azure. Make sure that you have a subscription associated with your Azure Account If the CLI can open your default browser, it will do so and load an Azure sign-in page. Otherwise, open a browser page at https://aka.ms/devicelogin and enter the authorization code displayed in your terminal.
<br><br>
1 - Log in to Azure.
<br><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;az login
<br><br>
2 - Deploy the ARM template provided by the sample, you will need to have a resource group already created.
<br><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;az deployment group create --resource-group <resource-group-name> --template-file .\deploymentTemplate\AppServiceAutoScale.json
<br><br>
3 - Once the deployment has successfully completed go to the Azure Portal, you will see three new resources created under the resource group.

4 - Find the App Service called "CPUstressWebApp",  go to overview.

5 - Click on "Get Publish Profile" option in the upper toolbar, that will download the publish profile to your computer.

6 - Open the solution in Visual Studio, right click on the project named "CPUstressWebApp", select "Publish".

7 - Select the option "New", then click on the "Import Profile" button located at the bottom of the dialog.

8 - Find and select the publish profile file that you downloaded in step 5.

9 - Click on "Publish", that will publish the CPUstress Web App to the App Service.

10 - Once the Web App has been successfully published, a browser's window will show up with the web application's home page.

11 - The UI is simple, by clicking the "trigger CPU Spike" button, a 100% CPU usage spike will happen for a period of time of one minute by default, or the number of minutes selected in the "minutes to run" text box (ten minutes is recommended to have a spike long enough to see the scaling effect). This is a fire and forget action.

12 - Go back to the Azure portal, select the App Service Plan (CPUstressWebAppPlan), in the overview, view you will see the CPU percentage chart.

13 - In 5 minutes the CPU spike will be reflected in the chart.

14 - Once you see the CPU spike, select the setting "Scale Out (App Service Plan)" on the settings Menu on the left.

15 - Select "Run History" in the upper toolbar.

16 - In the run history view, you will see the number of instances increased to two, also you will see the operation called "Autoscale scale up completed" in the autoscale events.

17 - Now you need to wait for the CPU spike to finish, and then after the cool period has passed (5 more minutes), the "Autoscale scale down" operation will appear in the list of autoscale events, and the number of instances will decrease to one again.