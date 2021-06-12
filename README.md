# Sample of an ARM deployment

In this example we are going to deploy an ARM template, assisted by an UI definition file.

The first step is creating a button to deploy:

[![Deploy To Azure](https://docs.microsoft.com/en-us/azure/templates/media/deploy-to-azure.svg)](https://portal.azure.com/#blade/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2Fvmss.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2FCreateUiDefinition.json)

The button above is created by this markdown:

 <div style="white-space: pre-wrap; font-family:monospace">
[![Deploy To Azure](https://docs.microsoft.com/en-us/azure/templates/media/deploy-to-azure.svg)](https://portal.azure.com/#blade/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/<bold>https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2Fvmss.json</bold>/createUIDefinitionUri/<bold>https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2FCreateUiDefinition.json</bold>)
</div>
</br>

As you can see, the hyperlink goes to the Azure portal and embeds in the path both the template and the UI definition file (both in bold in the previous paragraph).