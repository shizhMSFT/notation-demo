# Build and Sign using Azure Container Registry (ACR) Tasks

In this demo, we will use docker and notation to build and sign an image via ACR Tasks.

Before following the below demo steps,
- An ACR instance should be provisioned and set `ACR_NAME` to the instance name
- Choose a name `ACR_TASK_NAME` for the ACR Task to be created
- Create a managed identity and record the resource ID of the identity in `IDENTITY_RESOURCE_ID`
  - User-assigned identity demonstrated in this example
  - Optionally, system-assigned identity can also be used
  - Related doc: [Use an Azure-managed identity in ACR Tasks](https://learn.microsoft.com/azure/container-registry/container-registry-tasks-authentication-managed-identity)
- Provision an Azure Key Value, create a certificate for signing, and grant the managed identity permission according to the [Getting started](https://github.com/Azure/notation-azure-kv#getting-started) section of the `notation-azure-kv` doc
  - Self-signed certificate is demonstrated in this example
  - Record the key ID of the certificate in `SIGNING_KEY_ID`

```sh
ACR_NAME=myregistry
ACR_TASK_NAME=notation-sign
IDENTITY_RESOURCE_ID=/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserAssignedIdentity
SIGNING_KEY_ID=https://myvalue.vault.azure.net/keys/keyname/version
```

Step 1: Build `notation` container and the `notation-azure-kv` plugin installer.

```sh
az acr build -t notation -r $ACR_NAME notation
az acr build -t notation-azure-kv-installer -r $ACR_NAME notation-azure-kv-installer
```

Step 2: Create a build-and-sign task
```sh
az acr task create -r $ACR_NAME -n $ACR_TASK_NAME -f sign.yaml -c /dev/null --set SIGNING_KEY_ID=$SIGNING_KEY_ID --assign-identity $IDENTITY_RESOURCE_ID --commit-trigger-enabled false --base-image-trigger-enabled false
```

Step 3: Run the task
```sh
az acr task run -r $ACR_NAME -n $ACR_TASK_NAME
```

Example output:
```
2023/10/12 16:16:37 Alias support enabled for version >= 1.1.0, please see https://aka.ms/acr/tasks/task-aliases for more information.
2023/10/12 16:16:37 Creating Docker network: acb_default_network, driver: 'bridge'
2023/10/12 16:16:37 Successfully set up Docker network: acb_default_network
2023/10/12 16:16:37 Setting up Docker configuration...
2023/10/12 16:16:37 Successfully set up Docker configuration
2023/10/12 16:16:37 Logging in to registry: <detracted>.azurecr.io
2023/10/12 16:16:38 Successfully logged into <detracted>.azurecr.io
2023/10/12 16:16:38 Executing step ID: acb_step_0. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2023/10/12 16:16:38 Launching container with name: acb_step_0
Unable to find image '<detracted>.azurecr.io/notation-azure-kv-installer:latest' locally
latest: Pulling from notation-azure-kv-installer
96526aa774ef: Pulling fs layer
800e37483346: Pulling fs layer
25b6a3ec37b4: Pulling fs layer
25b6a3ec37b4: Verifying Checksum
25b6a3ec37b4: Download complete
96526aa774ef: Verifying Checksum
96526aa774ef: Download complete
96526aa774ef: Pull complete
800e37483346: Verifying Checksum
800e37483346: Download complete
800e37483346: Pull complete
25b6a3ec37b4: Pull complete
Digest: sha256:ccd315d4b85c0b8dce19f0f4d47466133a645eb43465c6943a95c2671f792123
Status: Downloaded newer image for <detracted>.azurecr.io/notation-azure-kv-installer:latest
Installing notation-azure-kv plugin...
Target path: /acb/home/.config/notation/plugins/azure-kv/notation-azure-kv
2023/10/12 16:16:40 Successfully executed container: acb_step_0
2023/10/12 16:16:40 Executing step ID: acb_step_1. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2023/10/12 16:16:40 Launching container with name: acb_step_1
Unable to find image '<detracted>.azurecr.io/notation:latest' locally
latest: Pulling from notation
707e32e9fc56: Pulling fs layer
fef0f3746d28: Pulling fs layer
b165baa9c267: Pulling fs layer
b165baa9c267: Verifying Checksum
b165baa9c267: Download complete
707e32e9fc56: Verifying Checksum
707e32e9fc56: Download complete
fef0f3746d28: Verifying Checksum
fef0f3746d28: Download complete
707e32e9fc56: Pull complete
fef0f3746d28: Pull complete
b165baa9c267: Pull complete
Digest: sha256:ab062aaf56e309a28fd8c3033c526d10640731392f60a157b6b10e9a6e73c869
Status: Downloaded newer image for <detracted>.azurecr.io/notation:latest
NAME       DESCRIPTION                       VERSION   CAPABILITIES                ERROR
azure-kv   Notation Azure Key Vault plugin   1.0.1     [SIGNATURE_GENERATOR.RAW]   <nil>
2023/10/12 16:16:44 Successfully executed container: acb_step_1
2023/10/12 16:16:44 Executing step ID: acb_step_2. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2023/10/12 16:16:44 Launching container with name: acb_step_2
akv-key: marked as default
2023/10/12 16:16:45 Successfully executed container: acb_step_2
2023/10/12 16:16:45 Executing step ID: acb_step_3. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2023/10/12 16:16:45 Launching container with name: acb_step_3
NAME        KEY PATH   CERTIFICATE PATH   ID                                                                                 PLUGIN NAME
* akv-key                                 https://<detracted>.vault.azure.net/keys/notation/cbd7e852c5584d0caebade3de7a47db1   azure-kv
2023/10/12 16:16:45 Successfully executed container: acb_step_3
2023/10/12 16:16:45 Executing step ID: acb_step_4. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2023/10/12 16:16:45 Launching container with name: acb_step_4
Unable to find image 'mcr.microsoft.com/acr/bash:b689300' locally
b689300: Pulling from acr/bash
d3009a26dfab: Pulling fs layer
f977210aef55: Pulling fs layer
4238a4795d74: Pulling fs layer
f977210aef55: Verifying Checksum
f977210aef55: Download complete
4238a4795d74: Download complete
d3009a26dfab: Verifying Checksum
d3009a26dfab: Download complete
d3009a26dfab: Pull complete
f977210aef55: Pull complete
4238a4795d74: Pull complete
Digest: sha256:a732e04dc9e858c8989fd8957e48d49a6d1d72fd66d616674a2c351bd715bd5d
Status: Downloaded newer image for mcr.microsoft.com/acr/bash:b689300
2023/10/12 16:16:47 Successfully executed container: acb_step_4
2023/10/12 16:16:47 Executing step ID: acb_step_5. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2023/10/12 16:16:47 Scanning for dependencies...
2023/10/12 16:16:48 Successfully scanned dependencies
2023/10/12 16:16:48 Launching container with name: acb_step_5
Sending build context to Docker daemon  2.048kB
Step 1/1 : FROM hello-world
 ---> 9c7a54a9a43c
Successfully built 9c7a54a9a43c
Successfully tagged <detracted>.azurecr.io/demo/hello:latest
2023/10/12 16:16:48 Successfully executed container: acb_step_5
2023/10/12 16:16:48 Executing step ID: acb_step_6. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2023/10/12 16:16:48 Pushing image: <detracted>.azurecr.io/demo/hello:latest, attempt 1
The push refers to repository [<detracted>.azurecr.io/demo/hello]
01bb4fce3eb1: Preparing
01bb4fce3eb1: Pushed
latest: digest: sha256:7e9b6e7ba2842c91cf49f3e214d04a7a496f8214356f41d81a6e6dcad11f11e3 size: 525
2023/10/12 16:17:00 Successfully pushed image: <detracted>.azurecr.io/demo/hello:latest
2023/10/12 16:17:00 Executing step ID: acb_step_7. Timeout(sec): 600, Working directory: '', Network: 'acb_default_network'
2023/10/12 16:17:00 Launching container with name: acb_step_7
Warning: Always sign the artifact using digest(@sha256:...) rather than a tag(:latest) because tags are mutable and a tag reference can point to a different artifact than the one signed.
Successfully signed <detracted>.azurecr.io/demo/hello@sha256:7e9b6e7ba2842c91cf49f3e214d04a7a496f8214356f41d81a6e6dcad11f11e3
2023/10/12 16:17:08 Successfully executed container: acb_step_7
2023/10/12 16:17:08 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 1.921239)
2023/10/12 16:17:08 Step ID: acb_step_1 marked as successful (elapsed time in seconds: 4.496956)
2023/10/12 16:17:08 Step ID: acb_step_2 marked as successful (elapsed time in seconds: 0.461398)
2023/10/12 16:17:08 Step ID: acb_step_3 marked as successful (elapsed time in seconds: 0.439995)
2023/10/12 16:17:08 Step ID: acb_step_4 marked as successful (elapsed time in seconds: 2.060022)
2023/10/12 16:17:08 Step ID: acb_step_5 marked as successful (elapsed time in seconds: 0.939890)
2023/10/12 16:17:08 Populating digests for step ID: acb_step_5...
2023/10/12 16:17:09 Successfully populated digests for step ID: acb_step_5
2023/10/12 16:17:09 Step ID: acb_step_6 marked as successful (elapsed time in seconds: 11.875840)
2023/10/12 16:17:09 Step ID: acb_step_7 marked as successful (elapsed time in seconds: 7.413247)
2023/10/12 16:17:09 The following dependencies were found:
2023/10/12 16:17:09
- image:
    registry: <detracted>.azurecr.io
    repository: demo/hello
    tag: latest
    digest: sha256:7e9b6e7ba2842c91cf49f3e214d04a7a496f8214356f41d81a6e6dcad11f11e3
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/hello-world
    tag: latest
    digest: sha256:4f53e2564790c8e7856ec08e384732aa38dc43c52f02952483e3f003afbf23db
  git: {}


Run ID: cm2aw was successful after 33s
```