# VMSS ARM deployment with distinct cloud init per instance

## Problem definition

The goal I am trying to reach here is having a cloud init file being dynamically generated upon user input on a graphical UI, so that a Virtual Machine Scale Set (VMSS) is generated that will run some specific logic. Additionally, the generated cloud init script should do something different for each instance of the VMSS: for example, installing different license files in each of the instances. This is simulated in this example by creating a slightly different `/tmp/helloworld.txt` file in the instances.

The VMSS will be generated with an ARM template, and the UI will be the actual Azure portal. This Github repo will explore how to use UI definition files for plain ARM templates, and not only for managed applications (as the [docs](https://docs.microsoft.com/azure/azure-resource-manager/managed-applications/create-uidefinition-overview) seem to imply).

Credits to my esteemed colleague [Guy Bowerman](https://github.com/gbowerman), I took his [ARM VMSS template](https://github.com/gbowerman/azure-minecraft/blob/main/azure-marketplace/minecraft-server-ubuntu/mainTemplate.json) as base for mine.

## ARM templates with UI definition files

In this example we are going to deploy an ARM template, assisted by an UI definition file.

The first step is creating a button to deploy:

[![Deploy To Azure](https://docs.microsoft.com/en-us/azure/templates/media/deploy-to-azure.svg)](https://portal.azure.com/#blade/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2Fvmss.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2FCreateUiDefinition.json)

The button above is created by this markdown:

```[![Deploy To Azure](https://docs.microsoft.com/en-us/azure/templates/media/deploy-to-azure.svg)](https://portal.azure.com/#blade/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2Fvmss.json/createUIDefinitionUri/<https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2FCreateUiDefinition.json)```

As you can see, the hyperlink goes to the Azure portal and embeds in the path links to both the template and the UI definition file.

More information about how to create a `createUIdefinition` file can be found [here](https://docs.microsoft.com/azure/azure-resource-manager/managed-applications/create-uidefinition-overview).

To test your CreateUiDefinition file, the best thing is using the [sandbox](https://portal.azure.com/?feature.customPortal=false&#blade/Microsoft_Azure_CreateUIDef/SandboxBlade). You can of course commit and test the results by clicking the button, but the sandbox will be much quicker. Firstly, CDN doesn't get in your way (sometimes you commit a file to Github, but you are not sure if the Azure portal is downloading the latest version or a cached, stale one). And secondly, the error messages will be more meaningful in the sandbox.

## Sending cloud init data from the UI

In the outputs of the UI definition file, the VMSS cloud init string is calculated dynamically:

```"cloudInitScript": "[concat('#cloud-config__break____break__runcmd:__break__- apt update && apt install -y jq__break__- export hostname=$(curl -s -H Metadata:true \"http://169.254.169.254/metadata/instance?api-version=2020-09-01\" | jq -r \".compute.name\")__break__- echo $hostname > /tmp/hostname.txt__break__- export instance_id=$(echo -n $hostname | tail -c 1)__break__- echo $instance_id > /tmp/instance_id.txt__break__- if [ \"$instance_id\" = \"0\" ]; then echo \"Hello ', steps('InstanceNames').instance1name, ' from cloudinit at $hostname\" > /tmp/helloworld.txt; else echo \"Hello ', steps('InstanceNames').instance2name, ' from cloudinit at $hostname\" > /tmp/helloworld.txt; fi')]"```

Some notes about this long line:

* I have observed that a CreateUiDefinition output cannot contain a line break (`\n`), that gives a weird error (something like `[] should only contain a function`). So I am sending a special string (`__break__`) that will be substituted in the target JSON template by `\n` characters (ARM function [replace](https://docs.microsoft.com/azure/azure-resource-manager/templates/template-functions-string#replace)).
* To get the hostname you can use the Azure metadata endpoint: ```export hostname=$(curl -s -H Metadata:true "http://169.254.169.254/metadata/instance?api-version=2020-09-01" | jq -r ".compute.name")```
* Hence I need to install the package `jq` in advance
* You can get the instance ID taking the last character of the hostname. I use `(echo -n $hostname | tail -c 1)` (and not something like `${hostname: -1}`) because IMHO it is compatible with more shells
* Finally, the one-line if compares the last character of the hostname and decides what to put in the file. Note the single quote comparison, since this command is run per default in `sh` and not `bash`: ```if [ "$instance_id" = "0" ]; then echo "Hello Tom from cloudinit at $hostname" > /tmp/helloworld.txt; else echo "Hello Jerry from cloudinit at $hostname" > /tmp/helloworld.txt; fi```
* You could probably do the script in fewer lines, but for debugging I am putting every intermediate step into a file in `/tmp/`

## Verifying the generated cloud init

The ARM template includes as output the input cloud init file (after replacing `__break__` with `\n`), so you can verify the provided cloud init file easily. In the next command I use `jq` instead of the `--query` argument of the Azure CLI, because `jq` will interpret the line breaks correctly, for easier readability:

```
$ az deployment group list -g vmsstest --query '[0]' -o json | jq -r '.properties.outputs.cloudinit.value'
#cloud-config

runcmd:
- apt update && apt install -y jq
- export hostname=$(curl -s -H Metadata:true "http://169.254.169.254/metadata/instance?api-version=2020-09-01" | jq -r ".compute.name")
- echo $hostname > /tmp/hostname.txt
- export instance_id=$(echo -n $hostname | tail -c 1)
- echo $instance_id > /tmp/instance_id.txt
- if [ "$instance_id" = "0" ]; then echo "Hello Tom from cloudinit at $hostname" > /tmp/helloworld.txt; else echo "Hello Jerry  from cloudinit at $hostname" > /tmp/helloworld.txt; fi
```

As you can see, the first VMSS instance (instance 0) will contain the string "Hello Tom", while the second one (and every one else too, in this particular example) will contain the string "Hello Jerry". `Tom` and `Jerry` are the default values of the each instance string variable in the UI definition file above, this could be different licenses to be installed in each VMSS instance, for example.
