# Sample of an ARM deployment

In this example we are going to deploy an ARM template, assisted by an UI definition file.

The first step is creating a button to deploy:

[![Deploy To Azure](https://docs.microsoft.com/en-us/azure/templates/media/deploy-to-azure.svg)](https://portal.azure.com/#blade/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2Fvmss.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2FCreateUiDefinition.json)

The button above is created by this markdown:

```[![Deploy To Azure](https://docs.microsoft.com/en-us/azure/templates/media/deploy-to-azure.svg)](https://portal.azure.com/#blade/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2Fvmss.json/createUIDefinitionUri/<https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2FCreateUiDefinition.json)```

As you can see, the hyperlink goes to the Azure portal and embeds in the path both the template and the UI definition file.

To test your CreateUiDefinition file, the best thing is using the [sandbox](https://portal.azure.com/?feature.customPortal=false&#blade/Microsoft_Azure_CreateUIDef/SandboxBlade). You can of course commit and test the results, but the sandbox will be much quicker. First, CDN doesn't get in your way (sometimes you commit a file to Github, but you are not sure if the Azure portal is downloading the latest version or a cached stale one). And second, the error messages will be more meaningful in the sandbox.

VMSS cloud init file calculated dynamically as output:

```"cloudInitScript": "[concat('#cloud-config\n\nruncmd:\n- if [[ \"${HOSTNAME: -1}\" == \"0\" ]]; then echo \"Hello ', steps('InstanceNames').instance1name, ' from cloudinit at ${HOSTNAME}!\" > /tmp/helloworld.txt; else echo \"Hello ', steps('InstanceNames').instance2name, '  from cloudinit at ${HOSTNAME}!\" > /tmp/helloworld.txt; fi')]"```

Some notes:

* I have observed that a CreateUiDefinition output cannot contain a line break (`\n`), that gives a weird error (something like `[] should only contain a function`). So I am sending a special string (`__break__`) that will be substituted in the JSON template by `\n` characters.
* To get the hostname you can use the Azure metadata endpoint: ```export hostname=$(curl -s -H Metadata:true "http://169.254.169.254/metadata/instance?api-version=2020-09-01" | jq -r '.compute.name')```
* Hence you need to install the package `jq` in advance
* Finally, the one-line if compares the last character of the hostname and decides what to put in the file. Note the single quote comparison, this this command is run per default in `sh` and not `bash`: ```if [ "$(echo -n $hostname | tail -c 1)" = "0" ]; then echo "Hello Tom from cloudinit at $hostname" > /tmp/helloworld.txt; else echo "Hello Jerry from cloudinit at $hostname" > /tmp/helloworld.txt; fi```
