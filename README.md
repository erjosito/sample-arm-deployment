# Sample of an ARM deployment

In this example we are going to deploy an ARM template, assisted by an UI definition file.

The first step is creating a button to deploy:

[![Deploy To Azure](https://docs.microsoft.com/en-us/azure/templates/media/deploy-to-azure.svg)](https://portal.azure.com/#blade/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2Fvmss.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2FCreateUiDefinition.json)

The botton above is created by this markdown:

 <div style="white-space: pre-wrap; font-family:monospace">
[![Deploy To Azure](https://docs.microsoft.com/en-us/azure/templates/media/deploy-to-azure.svg)](https://portal.azure.com/#blade/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/<span style="color:blue;">https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2Fvmss.json</span>/createUIDefinitionUri/<span style="color:orange;">https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2FCreateUiDefinition.json</span>)
<br>
</div>

As you can see, the hyperlink goes to the Azure portal and embeds in the path both the template (in blue in the previous paragraph) and the UI definition file (in green in the previous paragraph).