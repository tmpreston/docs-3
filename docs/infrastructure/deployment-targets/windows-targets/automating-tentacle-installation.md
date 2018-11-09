---
title: Automating Tentacle Installation
description: Information on how to install and configure an Octopus Tentacle in a fully automated way from the command line.
position: 60
---

The Tentacle agent can be installed fully automatically from the command line. This is very useful if you're deploying to a large number of servers, or you'll be provisioning servers automatically.

:::warning
**Cloning Tentacle VMs**
In a virtualized environment, it may be desirable to install Tentacle on a base virtual machine image, and clone this image to create multiple machines.

If you choose to do this, please **do not complete the configuration wizard** before taking the snapshot. The configuration wizard generates a unique per-machine cryptographic certificate that should not be duplicated. Instead, use PowerShell to [automate configuration](/docs/infrastructure/deployment-targets/windows-targets/automating-tentacle-installation.md) after the clone has been materialized.
:::

## Tentacle Installers {#AutomatingTentacleinstallation-Tentacleinstallers}

Tentacle comes in an MSI that can be deployed via group policy or other means.

:::success
**Download the Tentacle MSI**
The latest Tentacle MSI can always be [downloaded from the Octopus Deploy downloads page](https://octopus.com/downloads).

Permalinks to always get the latest MSIs are:

- 32-bit: [https://octopus.com/downloads/latest/WindowsX86/OctopusTentacle](https://octopus.com/downloads/latest/WindowsX86/OctopusTentacle)
- 64-bit: [https://octopus.com/downloads/latest/WindowsX64/OctopusTentacle](https://octopus.com/downloads/latest/WindowsX64/OctopusTentacle)
  :::

To install the MSI silently:

```bash
msiexec /i Octopus.Tentacle.<version>.msi /quiet
```

By default, the Tentacle files are installed under **%programfiles(x86)%**. To change the installation directory, you can specify:

```bash
msiexec INSTALLLOCATION=C:\YourDirectory /i Octopus.Tentacle.<version>.msi /quiet
```

:::problem
While you can set a custom INSTALLLOCATION for the Tentacle, please be aware that upgrades initiated by Octopus Server will currently install the upgraded Tentacle back into the default location. This may have an impact if you are using the [Service Watchdog](/docs/administration/service-watchdog.md).
:::

## Configuration {#AutomatingTentacleinstallation-Configuration}

The MSI installer simply extracts files and adds some shortcuts and event log sources. The actual configuration of Tentacle is done later, and this can automated too.

To configure the Tentacle in listening or polling mode, it's easiest to run the installation wizard once, and at the end, use the Show Script option in the setup wizard. This will show you the command-line equivalent to configure a Tentacle.

![Tentacle installation wizard](tentacle-setup.png)

:::success
**Advanced configuration options**
When configuring your Tentacle you can configure advanced options, like [proxies](/docs/infrastructure/deployment-targets/windows-targets/proxy-support.md), [machine policies](/docs/infrastructure/machine-policies.md) and [tenants](/docs/deployment-patterns/multi-tenant-deployments/multi-tenant-deployment-guide/designing-a-multi-tenant-hosting-model.md), which can also be automated. Use the setup wizard to configure the Tentacle, and click the **Show Script** link which will show you the command-line equivalent to configure the Tentacle.
:::

## Example: Listening Tentacle {#AutomatingTentacleinstallation-Example:ListeningTentacle}

The following example configures a [listening Tentacle](/docs/infrastructure/deployment-targets/windows-targets/tentacle-communication.md#listening-tentacles-recommended), and registers it with an Octopus Deploy Server:

**Using Tentacle.exe to create Listening Tentacle instance**

```bash
cd "C:\Program Files\Octopus Deploy\Tentacle"

Tentacle.exe create-instance --instance "Tentacle" --config "C:\Octopus\Tentacle.config" --console
Tentacle.exe new-certificate --instance "Tentacle" --if-blank --console
Tentacle.exe configure --instance "Tentacle" --reset-trust --console
Tentacle.exe configure --instance "Tentacle" --home "C:\Octopus" --app "C:\Octopus\Applications" --port "10933" --console
Tentacle.exe configure --instance "Tentacle" --trust "YOUR_OCTOPUS_THUMBPRINT" --console
"netsh" advfirewall firewall add rule "name=Octopus Deploy Tentacle" dir=in action=allow protocol=TCP localport=10933
Tentacle.exe register-with --instance "Tentacle" --server "http://YOUR_OCTOPUS" --apiKey="API-YOUR_API_KEY" --role "web-server" --environment "Staging" --comms-style TentaclePassive --console
Tentacle.exe service --instance "Tentacle" --install --start --console
```

You can also register a Tentacle with the Octopus Server after it has been installed by using Octopus.Client (i.e. register-with could be omitted above and the following could be used after the instance has started.  See below for how to obtain the Tentacle's thumbprint):

**Using Octopus.Client to register a Tentacle in an Octopus Server**

```powershell
Add-Type -Path 'Newtonsoft.Json.dll'
Add-Type -Path 'Octopus.Client.dll'

$octopusApiKey = 'API-ABCXYZ'
$octopusURI = 'http://YOUR_OCTOPUS'

$endpoint = new-object Octopus.Client.OctopusServerEndpoint $octopusURI, $octopusApiKey
$repository = new-object Octopus.Client.OctopusRepository $endpoint

$tentacle = New-Object Octopus.Client.Model.MachineResource

$tentacle.name = "Tentacle registered from client"
$tentacle.EnvironmentIds.Add("Environments-1")
$tentacle.Roles.Add("WebServer")

$tentacleEndpoint = New-Object Octopus.Client.Model.Endpoints.ListeningTentacleEndpointResource
$tentacle.EndPoint = $tentacleEndpoint
$tentacle.Endpoint.Uri = "https://YOUR_TENTACLE:10933"
$tentacle.Endpoint.Thumbprint = "YOUR_TENTACLE_THUMBPRINT"

$repository.machines.create($tentacle)
```

:::success
Want to register your Tentacles another way? Take a look at the examples in our [sample repository](https://github.com/OctopusDeploy/OctopusDeploy-Api) using the Octopus API to register Tentacles, and do a whole lot more!
:::

## Example: Polling Tentacle {#AutomatingTentacleinstallation-Example:PollingTentacle}

The following example configures a [polling Tentacle](/docs/infrastructure/deployment-targets/windows-targets/tentacle-communication.md#polling-tentacles), and registers it with an Octopus Deploy Server:

**Polling Tentacle**

```bash
cd "C:\Program Files\Octopus Deploy\Tentacle"

Tentacle.exe create-instance --instance "Tentacle" --config "C:\Octopus\Tentacle.config" --console
Tentacle.exe new-certificate --instance "Tentacle" --if-blank --console
Tentacle.exe configure --instance "Tentacle" --reset-trust --console
Tentacle.exe configure --instance "Tentacle" --home "C:\Octopus" --app "C:\Octopus\Applications" --noListen "True" --console
Tentacle.exe register-with --instance "Tentacle" --server "http://YOUR_OCTOPUS" --name "YOUR_TENTACLE_NAME" --apiKey "API-YOUR_API_KEY" --comms-style "TentacleActive" --server-comms-port "10943" --force --environment "YOUR_TENTACLE_ENVIRONMENTS" --role "YOUR_TENTACLE_ROLES" --console
Tentacle.exe service --instance "Tentacle" --install --start --console
```

:::warning
If you are running this from a PowerShell remote session, make sure to add `--console` at the end of each command to force Tentacle.exe not to run as a service.
:::

:::success
Want to register your Tentacles another way? Take a look at the examples in our [sample repository](https://github.com/OctopusDeploy/OctopusDeploy-Api) using the Octopus API to register Tentacles, and do a whole lot more!
:::

## Obtaining the Tentacle Thumbprint {#AutomatingTentacleinstallation-tentaclethumbprintObtainingtheTentacleThumbprint}

If you don't know the thumbprint for the above PowerShell scripts, it can be obtained with the following command line option:

**Obtaining Thumbprint**

```bash
Tentacle.exe show-thumbprint --instance "Tentacle" --nologo
```

## Export and Import Tentacle Certificates Without a Profile

When the Tentacle agent is configured, the default behavior is to generate a new X.509 certificate. When automating the provisioning of Tentacles on a machine, however, you may run into problems when trying to generate a certificate when running as a user without a profile loaded.

A simple workaround is to generate a certificate on one machine (such as your workstation), export it to a file, and then import that certificate when provisioning Tentacles.

## Generating and Exporting a Certificate

Install the Tentacle agent on a computer, and run the following command:

```powershell
tentacle.exe new-certificate -e MyFile.txt
```

The output file will now contain a base-64 encoded version of a PKCS#12 export of the X.509 certificate and corresponding private key. This file is now ready to be used in your setup scripts.

## Importing a Certificate

When automatically provisioning your Tentacle, the commands typically look something like this:

```powershell
Tentacle.exe create-instance --instance "Tentacle" --config "C:\Octopus\Tentacle\Tentacle.config" --console
Tentacle.exe new-certificate --instance "Tentacle" --console
Tentacle.exe configure --instance "Tentacle" --home "C:\Octopus" --console
...
```

Instead, replace the `new-certificate` command with `import-certificate`. For example:

```powershell
Tentacle.exe create-instance --instance "Tentacle" --config "C:\Octopus\Tentacle\Tentacle.config" --console
Tentacle.exe import-certificate --instance "Tentacle" -f MyFile.txt --console
Tentacle.exe configure --instance "Tentacle" --home "C:\Octopus" --console
...
```

## Desired State Configuration

Tentacles can also be installed via [Desired State Configuration](https://msdn.microsoft.com/en-us/powershell/dsc/overview) (DSC). Using the module from the [OctopusDSC GitHub repository](https://www.powershellgallery.com/packages/OctopusDSC), you can add, remove, start and stop Tentacles in either polling or listening mode.

The following PowerShell script will install a Tentacle listening on port `10933` against the Octopus Server at `https://YOUR_OCTOPUS`, add it to the `Development` environment and assign the `web-server` and `app-server` roles:

**DSC Configuration**

```powershell
Configuration SampleConfig
{
    param ($ApiKey, $OctopusServerUrl, $Environments, $Roles, $ListenPort)

    Import-DscResource -Module OctopusDSC

    Node "localhost"
    {
        cTentacleAgent OctopusTentacle
        {
            Ensure = "Present"
            State = "Started"

            # Tentacle instance name. Leave it as 'Tentacle' unless you have more
            # than one instance
            Name = "Tentacle"

            # Registration - all parameters required
            ApiKey = $ApiKey
            OctopusServerUrl = $OctopusServerUrl
            Environments = $Environments
            Roles = $Roles

            # Optional settings
            ListenPort = $ListenPort
            DefaultApplicationDirectory = "C:\Applications"
        }
    }
}

# Execute the configuration above to create a mof file
SampleConfig -ApiKey "API-YOUR_API_KEY" -OctopusServerUrl "https://YOUR_OCTOPUS/" -Environments @("Development") -Roles @("web-server", "app-server") -ListenPort 10933

# Run the configuration
Start-DscConfiguration .\SampleConfig -Verbose -wait

# Test the configuration ran successfully
Test-DscConfiguration
```
### Settings and Properties

To review the latest available settings and properties, refer to the [OctopusDSC Tentacle readme.md](https://github.com/OctopusDeploy/OctopusDSC/blob/master/README-cTentacleAgent.md) in the GitHub repository.


DSC can be applied in various ways, such as [Group Policy](https://sdmsoftware.com/group-policy-blog/desired-state-configuration/desired-state-configuration-and-group-policy-come-together/), a [DSC Pull Server](https://msdn.microsoft.com/en-us/powershell/dsc/pullserver), [Azure Automation](https://msdn.microsoft.com/en-us/powershell/dsc/azuredsc), or even via configuration management tools such as [Chef](https://docs.chef.io/resource_dsc_resource.html) or [Puppet](https://github.com/puppetlabs/puppetlabs-dsc). A good resource to learn more about DSC is the [Microsoft Virtual Academy training course](http://www.microsoftvirtualacademy.com/training-courses/getting-started-with-powershell-desired-state-configuration-dsc-).

For an in depth look, check out the [sample walkthrough](docs/infrastructure/deployment-targets/windows-targets/azure-virtual-machines/via-an-arm-template-with-dsc.md) of how to use DSC with an Azure ARM template to deploy and configure the Tentacle on an Azure VM.