# Sample of an ARM deployment

In this example we are going to deploy an ARM template, assisted by an UI definition file.

The first step is creating a button to deploy:

[![Deploy To Azure](https://docs.microsoft.com/en-us/azure/templates/media/deploy-to-azure.svg)](https://portal.azure.com/#blade/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2Fvmss.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2FCreateUiDefinition.json)

The button above is created by this markdown:

```[![Deploy To Azure](https://docs.microsoft.com/en-us/azure/templates/media/deploy-to-azure.svg)](https://portal.azure.com/#blade/Microsoft_Azure_CreateUIDef/CustomDeploymentBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2Fvmss.json/createUIDefinitionUri/<https%3A%2F%2Fraw.githubusercontent.com%2Ferjosito%2Fsample-arm-deployment%2Fmaster%2FCreateUiDefinition.json)```

As you can see, the hyperlink goes to the Azure portal and embeds in the path both the template and the UI definition file.

To test your CreateUiDefinition file, the best thing is using the [sandbox](https://portal.azure.com/?feature.customPortal=false&#blade/Microsoft_Azure_CreateUIDef/SandboxBlade). You can of course commit and test the results, but the sandbox will be much quicker. First, CDN doesn't get in your way (sometimes you commit a file to Github, but you are not sure if the Azure portal is downloading the latest version or a cached stale one). And second, the error messages will be more meaningful in the sandbox.

VMSS cloud init file calculated dynamically as output:

```"cloudInitScript": "[concat('#cloud-config__break____break__runcmd:__break__- apt update && apt install -y jq__break__- export hostname=$(curl -s -H Metadata:true \"http://169.254.169.254/metadata/instance?api-version=2020-09-01\" | jq -r \".compute.name\")__break__- echo $hostname > /tmp/hostname.txt__break__- export instance_id=$(echo -n $hostname | tail -c 1)__break__- echo $instance_id > /tmp/instance_id.txt__break__- if [ \"$instance_id\" = \"0\" ]; then echo \"Hello ', steps('InstanceNames').instance1name, ' from cloudinit at $hostname\" > /tmp/helloworld.txt; else echo \"Hello ', steps('InstanceNames').instance2name, ' from cloudinit at $hostname\" > /tmp/helloworld.txt; fi')]"```

Some notes:

* I have observed that a CreateUiDefinition output cannot contain a line break (`\n`), that gives a weird error (something like `[] should only contain a function`). So I am sending a special string (`__break__`) that will be substituted in the JSON template by `\n` characters.
* To get the hostname you can use the Azure metadata endpoint: ```export hostname=$(curl -s -H Metadata:true "http://169.254.169.254/metadata/instance?api-version=2020-09-01" | jq -r ".compute.name")```
* Hence you need to install the package `jq` in advance
* You can get the instance ID taking the last character of the hostname. I use `(echo -n $hostname | tail -c 1)` (and not something like `${hostname: -1}`) because IMHO it is compatible with more shells
* Finally, the one-line if compares the last character of the hostname and decides what to put in the file. Note the single quote comparison, this this command is run per default in `sh` and not `bash`: ```if [ "$instance_id" = "0" ]; then echo "Hello Tom from cloudinit at $hostname" > /tmp/helloworld.txt; else echo "Hello Jerry from cloudinit at $hostname" > /tmp/helloworld.txt; fi```
* You could probably do the script in fewer lines, but for debugging I am putting every intermediate step into a file in `/tmp/`
