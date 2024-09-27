---
title: Troubleshoot Azure Arc resource bridge issues
description: This article tells how to troubleshoot and resolve issues with the Azure Arc resource bridge when trying to deploy or connect to the service.
ms.date: 09/20/2024
ms.topic: troubleshooting
---

# Troubleshoot Azure Arc resource bridge issues

This article provides information on troubleshooting and resolving issues that could occur while attempting to deploy, use, or remove the Azure Arc resource bridge. The resource bridge is a packaged virtual machine, which hosts a *management* Kubernetes cluster. For general information, see [Azure Arc resource bridge overview](./overview.md).

## General issues

### Logs collection

For issues encountered with Arc resource bridge, collect logs for further investigation using the Azure CLI [`az arcappliance logs`](/cli/azure/arcappliance/logs) command. This command needs to be run from the management machine used to deploy the Arc resource bridge. If you're using a different machine, the machine must meet the [management machine requirements](system-requirements.md#management-machine-requirements).

If there's a problem collecting logs, most likely the management machine is unable to reach the Appliance VM. Contact your network administrator to allow SSH communication from the management machine to the Appliance VM on TCP port 22.

You can collect the Arc resource bridge logs by passing either the appliance VM IP or the kubeconfig in the logs command.

To collect Arc resource bridge logs on VMware using the appliance VM IP address:

   ```azurecli
   az arcappliance logs vmware --ip <appliance VM IP> --username <vSphere username> --password <vSphere password> --address <vCenter address> --out-dir <path to output directory>
   ```

To collect Arc resource bridge logs for Azure Stack HCI using the appliance VM IP address:

   ```azurecli
   az arcappliance logs hci --ip <appliance VM IP> --cloudagent <cloud agent service IP/FQDN> --loginconfigfile <file path of kvatoken.tok> 
   ```

If you're unsure of your appliance VM IP, there's also the option to use the kubeconfig. You can retrieve the kubeconfig by running the [get-credentials command](/cli/azure/arcappliance) then run the logs command.

To retrieve the kubeconfig and log key then collect logs for Arc-enabled VMware from a different machine than the one used to deploy Arc resource bridge for Arc-enabled VMware:

   ```azurecli
az account set -s <subscription id>
az arcappliance get-credentials -n <Arc resource bridge name> -g <resource group name> 
az arcappliance logs vmware --kubeconfig kubeconfig --out-dir <path to specified output directory>
   ```

### Download/upload connectivity was not successful

If your network speed is slow, you might not be able to successfully download the Arc resource bridge VM image, resulting in this error: `ErrorCode: ValidateKvaError, Error: Pre-deployment validation of your download/upload connectivity was not successful. Timeout error occurred during download and preparation of appliance image to the on-premises fabric storage. Common causes of this timeout error are slow network download/upload speeds, a proxy limiting the network speed or slow storage performance.`

As a workaround, try creating a VM directly on the on-premises private cloud, and then run the Arc resource bridge deployment script from that VM. Doing this should result in a faster upload of the image to the datastore.

### Context timed out during phase `ApplyingKvaImageOperator`

When you deploy Arc resource bridge, you might see this error: `Deployment of the Arc resource bridge appliance VM timed out. Collect logs with _az arcappliance logs_ and create a support ticket for help. To troubleshoot the error, refer to aka.ms/arc-rb-error   { _errorCode_: _ContextError_, _errorResponse_: _{\n\_message\_: \_Context timed out during phase _ApplyingKvaImageOperator_\_\n}_ }`

This error typically occurs when trying to download the `KVAIO` image (400 MB compressed) over a network that is slow or experiencing intermittent connectivity. The `KVAIO` controller manager waits for the image download to complete, and times out.

Check that your network speed between the Arc resource bridge VM and Microsoft Container Registry (`mcr.microsoft.com`) is stable and at least 2 Mbps. If your network connectivity and speed are stable, and you're still getting this error, wait at least 30 minutes before you retry, as it could be due to Microsoft Container Registry receiving a high volume of traffic.

### Context timed out during phase `WaitingForAPIServer`

When you deploy Arc resource bridge, you might see this error: `Deployment of the Arc resource bridge appliance VM timed out. Collect logs with _az arcappliance logs_ and create a support ticket for help. To troubleshoot the error, refer to aka.ms/arc-rb-error   { _errorCode_: _ContextError_, _errorResponse_: _{\n\_message\_: \_Context timed out during phase _WaitingForAPIServer`

This error indicates that the deployment machine can't contact the control plane IP for Arc resource bridge within the time limit. Common causes of the error are often networking related, such as communication between the deployment machine and control plane IP being routed through a proxy. Traffic from the deployment machine to the control plane and the appliance VM IPs must not pass through proxy. If traffic is being proxied, configure the proxy settings on your network or deployment machine to not proxy traffic between the deployment machine to the control plane IP and appliance VM IPs. Another cause for this error is if a firewall is closing access to port 6443 and port 22 between the deployment machine and control plane IP or the deployment machine and appliance VM IPs.

### `UploadError` 403 Forbidden or 404 Site Not Found

When you deploy Arc resource bridge, you might see this error: `{ _errorCode_: _UploadError_, _errorResponse_: _{\n\_message\_: \_Pre-deployment validation of your download/upload connectivity was not successful. {\\n  \\\_code\\\_: \\\_ImageProvisionError\\\_,\\n  \\\_message\\\_: \\\_403 Forbidden` or `{ _errorCode_: _UploadError_, _errorResponse_: _{\n\_message\_: \_Pre-deployment validation of your download/upload connectivity was not successful. {\\n  \\\_code\\\_: \\\_ImageProvisionError\\\_,\\n  \\\_message\\\_: \\\_404 Site Not Found`

This error occurs when images need to be downloaded from Microsoft registries to the deployment machine, but a proxy or firewall blocks the download. Review the [network requirements](network-requirements.md#general-network-requirements) and verify that all required URLs are reachable. You may need to update your no proxy settings to ensure that traffic from your deployment machine to Microsoft required URLs aren't going through a proxy.

### SSH folder access denied

The CLI requires permission to access the SSH folder during deployment or operations that involve accessing files within the folder. This folder contains essential files such as the kubeconfig and logs key for the appliance VM. For instance, the CLI needs to access the logs key stored in the SSH folder to collect logs from the appliance VM.

You might see this error: `Access to the file in the SSH folder was denied. This may occur if the CLI doesn't have permission to the SSH folder or if another CLI instance is using the file`. There are two common causes for this issue:

- Insufficient permissions: The CLI lacks the necessary permissions to access the SSH folder. Ensure that the user account running the CLI has appropriate permissions to access the SSH folder.
- Concurrent file access: Another instance of the CLI might be using the file in the SSH folder. This often happens on workstations with shared profiles. Ensure that any other CLI instance completes or terminates its operation before you proceed.

### Arc resource bridge is offline

Networking changes in the infrastructure, environment or cluster can stop the appliance VM from being able to communicate with its counterpart Azure resource. If you're unable to determine what changed, you can reboot the appliance VM, collect logs and submit a support ticket for further investigation.

### Remote PowerShell isn't supported

If you run `az arcappliance` CLI commands for Arc resource bridge via remote PowerShell, you might see an [authentication handshake failure error when trying to install the resource bridge on an Azure Stack HCI cluster](#authentication-handshake-failure) or another type of error.

Using `az arcappliance` commands from remote PowerShell isn't currently supported. Instead, sign in to the node through Remote Desktop Protocol (RDP) or use a console session.

### Resource bridge configurations can't be updated

In this release, all the parameters are specified at time of creation. To update Arc resource bridge, you must delete it and redeploy it again.

For example, if you specify the wrong location or subscription during deployment, resource creation fails. If you only try to recreate the resource without redeploying the resource bridge VM, the status gets stuck at `WaitForHeartBeat`.

To resolve this issue, delete the appliance and update the appliance YAML file. After that, redeploy and create the resource bridge.

### Appliance Network Unavailable

If Arc resource bridge experiences network problems, you might see an `Appliance Network Unavailable` error. In general, any network or infrastructure connectivity issue to the appliance VM may cause this error. This error can also surface as `Error while dialing dial tcp xx.xx.xxx.xx:55000: connect: no route to host`. The problem could be that communication from the host to the Arc resource bridge VM needs to be opened over TCP port 22 with the help of your network administrator. A temporary network issue may not allow the host to reach the Arc resource bridge VM. Once the network issue is resolved, you can retry the operation. You can also check that the appliance VM for Arc resource bridge isn't stopped or offline. With Azure Stack HCI, this error can be caused when the host storage is full.

### Token refresh error

When you run Azure CLI commands, you may see the following error: `The refresh token has expired or is invalid due to sign-in frequency checks by conditional access.`

This error occurs because when you sign in to Azure, the token has a maximum lifetime. When that lifetime is exceeded, you need to sign in to Azure again by using the `az login` command.

### Default host resource pools are unavailable for deployment

When you use the `az arcappliance createconfig` or `az arcappliance run` command, an interactive experience shows the list of VMware entities which you can select to deploy the virtual appliance. This list shows all user-created resource pools along with default cluster resource pools, but the default host resource pools aren't listed. When the appliance is deployed to a host resource pool, there's no high availability if the host hardware fails. We recommend that you don't deploy the appliance in a host resource pool.

### Resource bridge status `Offline` and `provisioningState` `Failed`

When you deploy Arc resource bridge, the bridge might appear to be successfully deployed because no errors were encountered when running `az arcappliance deploy` or `az arcappliance create`. However, when viewing the bridge in Azure portal, you might see status showing as `Offline`, and `az arcappliance show` might show the `provisioningState` as `Failed`. This issue happens when required providers aren't registered before the bridge is deployed.

To resolve this problem, delete the resource bridge, register the providers, then redeploy the resource bridge.

1. Delete the resource bridge:

   ```azurecli
   az arcappliance delete <fabric> --config-file <path to appliance.yaml>
   ```

1. Register the providers:

   ```azurecli
   az provider register --namespace Microsoft.ExtendedLocation –-wait
   az provider register --namespace Microsoft.ResourceConnector –-wait
   ```

1. Redeploy the resource bridge.

> [!NOTE]
> Partner products (such as Arc-enabled VMware vSphere) might have their own required providers to register. For information about these additional providers, see the product's documentation.

### Expired credentials in the appliance VM

Arc resource bridge consists of an appliance VM that is deployed to the on-premises infrastructure. The appliance VM maintains a connection to the management endpoint of the on-premises infrastructure using locally stored credentials. If these credentials aren't updated, the resource bridge is no longer able to communicate with the management endpoint. This can cause problems when trying to upgrade the resource bridge or manage VMs through Azure.

To fix this problem, the credentials in the appliance VM need to be updated. For more information, see [Update credentials in the appliance VM](maintenance.md#update-credentials-in-the-appliance-vm).

### Private link is unsupported

Arc resource bridge doesn't support private link. Calls coming from the appliance VM shouldn't be going through your private link setup. Private link IPs may conflict with the appliance IP pool range, which isn't configurable on the resource bridge. Arc resource bridge reaches out to [required URLs](network-requirements.md#firewallproxy-url-allowlist) that shouldn't go through a private link connection. You must deploy Arc resource bridge on a separate network segment unrelated to the private link setup.

## Networking issues

### Back-off pulling image error

When trying to deploy Arc resource bridge, you might see an error that contains `back-off pulling image \\\"url"\\\: FailFastPodCondition`. This error is caused when the appliance VM can't reach the URL specified in the error. To resolve this issue, make sure the appliance VM meets system requirements, including internet access connectivity to [required allowlist URLs](network-requirements.md).

### Management machine unable to reach appliance

When trying to deploy Arc resource bridge, you might receive an error message similar to: 

`{ _errorCode_: _PostOperationsError_, _errorResponse_: _{\n\_message\_: \_Timeout occurred due to management machine being unable to reach the appliance VM IP, 10.2.196.170.  Ensure that the requirements are met: https://aka.ms/arb-machine-reqs: dial tcp 10.2.196.170:22: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.\_\n}_, _errorMetadata_: { _errorCategory_: __ }`

This error occurs when the management machine can't reach the Arc resource bridge VM IP by SSH (Port 22) or API Server (Port 6443). It could also occur if the Arc resource bridge API server is being proxied; the Arc resource bridge API server needs to be added to the noproxy settings. For more information, see [Azure Arc resource bridge network requirements](network-requirements.md#inbound-connectivity-requirements).

### Not able to connect to URL

If you receive an error that contains `Not able to connect to https://example.url.com`, check with your network administrator to ensure your network allows all of the required firewall and proxy URLs to deploy Arc resource bridge. For more information, see [Azure Arc resource bridge network requirements](network-requirements.md).

### Not able to connect - network and internet connectivity validation failed

When you deploy Arc resource bridge, you may receive an error with `errorCode` as `PostOperationsError`, `errorResponse` as code `GuestInternetConnectivityError` with a URL specifying port 53 (DNS). This error may be due to the appliance VM IPs being unable to reach DNS servers, so they can't resolve the endpoint specified in the error.

Error examples:

`{ _errorCode_: _PostOperationsError_, _errorResponse_: _{\n\_message\_: \_{\\n  \\\_code\\\_:\\\_GuestInternetConnectivityError\\\_,\\n\\\_message\\\_:\\\_Not able to connect to http://aszhcitest01.company.org:55000. Error returned: action failed after 5 attempts: Get \\\\\\\_http://aszhcitest01.company.org:55000\\\\\\\_: dial tcp: lookup aszhcitest01.company.org on 127.0.0.53:53: read udp 127.0.0.1:32975-\\u003e127.0.0.53:53: i/o timeout. Arc Resource Bridge network and internet connectivity validation failed: cloud-agent-connectivity-test. 1.  check your networking setup and ensure the URLs mentioned in : https://aka.ms/AAla73m are reachable from the Appliance VM.   2. Check firewall/proxy settings\\\_\\n }\_\n}_ }`

`{ _errorCode_: _PostOperationsError_, _errorResponse_: _{\n\_message\_: \_{\\n  \\\_code\\\_: \\\_GuestInternetConnectivityError\\\_,\\n  \\\_message\\\_: \\\_Not able to connect to https://linuxgeneva-microsoft.azurecr.io. Error returned: action failed after 5 attempts: Get \\\\\\\_https://linuxgeneva-microsoft.azurecr.io\\\\\\\_: dial tcp: lookup linuxgeneva-microsoft.azurecr.io on 127.0.0.53:53: server misbehaving. Arc Resource Bridge network and internet connectivity validation failed: http-connectivity-test-arc. 1. Please check your networking setup and ensure the URLs mentioned in : https://aka.ms/AAla73m are reachable from the Appliance VM.   2. Check firewall/proxy settings\\\_\\n }\_\n}_ }`

To resolve these errors, work with your network administrator to allow the appliance VM IPs to reach the DNS servers. For more information, see [Azure Arc resource bridge network requirements](network-requirements.md).

### Http2 server sent GOAWAY

When trying to deploy Arc resource bridge, you might receive an error message similar to:

`"errorResponse": "{\n\"message\": \"Post \\\"https://region.dp.kubernetesconfiguration.azure.com/azure-arc-appliance-k8sagents/GetLatestHelmPackagePath?api-version=2019-11-01-preview\\u0026releaseTrain=stable\\\": http2: server sent GOAWAY and closed the connection; LastStreamID=1, ErrCode=NO_ERROR, debug=\\\"\\\"\"\n}"`

This error occurs when a firewall or proxy has SSL/TLS inspection enabled and blocks http2 calls from the machine used to deploy the resource bridge. To confirm the problem, run the following PowerShell cmdlet to invoke the web request with http2 (requires PowerShell version 7 or above), replacing the region in the URL and `api-version` (for example, `2019-11-01`) with values from the error:

`Invoke-WebRequest -HttpVersion 2.0 -UseBasicParsing -Uri https://region.dp.kubernetesconfiguration.azure.com/azure-arc-appliance-k8sagents/GetLatestHelmPackagePath?api-version=2019-11-01-preview"&"releaseTrain=stable -Method Post -Verbose`

If the result is `The response ended prematurely while waiting for the next frame from the server`, then the http2 call is being blocked and needs to be allowed. Work with your network administrator to disable the SSL/TLS inspection to allow http2 calls from the machine used to deploy the bridge.

### No such host - `.local` not supported

When trying to set the configuration for Arc resource bridge, you might receive an error message similar to:

`"message": "Post \"https://esx.lab.local/52c-acac707ce02c/disk-0.vmdk\": dial tcp: lookup esx.lab.local: no such host"`

This error occurs when a `.local` path is provided for a configuration setting, such as proxy, dns, datastore, or management endpoint (such as vCenter). Arc resource bridge appliance VM uses Azure Linux OS, which doesn't support `.local` by default. A workaround could be to provide the IP address where applicable.

### Azure Arc resource bridge is unreachable

Azure Arc resource bridge runs a Kubernetes cluster, and its control plane requires a static IP address. The IP address is specified in the `infra.yaml` file. If the IP address is assigned from a DHCP server, the address can change if it's not reserved. Rebooting the Azure Arc resource bridge or VM can trigger an IP address change and result in failing services.

Arc resource bridge may intermittently lose the reserved IP configuration. This loss is due to the behavior described in [loss of VIPs when `systemd-networkd` is restarted](https://github.com/acassen/keepalived/issues/1385). When the IP address isn't assigned to the Azure Arc resource bridge VM, any call to the resource bridge API server fails. Core operations, such as creating a new resource, connecting to your private cloud from Azure, or creating a custom location, won't function as expected.

To resolve this issue, reboot the resource bridge VM, and it should recover its IP address. If the address is assigned from a DHCP server, reserve the IP address associated with the resource bridge.

The Arc resource bridge may also be unreachable due to slow disk access. Azure Arc resource bridge uses Kubernetes extended configuration tree (ETCD), which requires [latency of 10 ms or less](https://docs.openshift.com/container-platform/4.6/scalability_and_performance/recommended-host-practices.html#recommended-etcd-practices_). If the underlying disk has low performance, operations are impacted and failures can occur.

### SSL proxy configuration issues

Be sure that the proxy server on your management machine trusts both the SSL certificate for your SSL proxy and the SSL certificate of the Microsoft download servers. For more information, see [SSL proxy configuration](network-requirements.md#ssl-proxy-configuration).

### No such host - dp.kubernetesconfiguration.azure.com

An error that contains `dial tcp: lookup westeurope.dp.kubernetesconfiguration.azure.com: no such host` while deploying Arc resource bridge means that the configuration data plane is currently unavailable in the specified region. The service may be temporarily unavailable. Wait for the service to be available, then retry the deployment.

### Proxy connect tcp - No such host for Arc resource bridge required URL

An error that contains an Arc resource bridge required URL with the message `proxyconnect tcp: dial tcp: lookup http: no such host` indicates that DNS is unable to resolve the URL. The error may look similar to this example, where the required URL is `https://msk8s.api.cdp.microsoft.com`:

`Error:  { _errorCode_: _InvalidEntityError_, _errorResponse_: _{\n\_message\_: \_Post \\\_https://msk8s.api.cdp.microsoft.com/api/v1.1/contents/default/namespaces/default/names/arc-appliance-stable-catalogs-ext/versions/latest?action=select\\\_: POST https://msk8s.api.cdp.microsoft.com/api/v1.1/contents/default/namespaces/default/names/arc-appliance-stable-catalogs-ext/versions/latest?action=select giving up after 6 attempt(s): Post \\\_https://msk8s.api.cdp.microsoft.com/api/v1.1/contents/default/namespaces/default/names/arc-appliance-stable-catalogs-ext/versions/latest?action=select\\\_: proxyconnect tcp: dial tcp: lookup http: no such host\_\n}_ }`

This error can occur if the DNS settings provided during deployment aren't correct or there's a problem with the DNS servers. You can check if your DNS server is able to resolve the url by running the following command from the management machine or a machine that has access to the DNS servers:

```
nslookup
> set debug
> <hostname> <DNS server IP>
```

To resolve the error, configure your DNS servers to resolve all Arc resource bridge required URLs. The DNS servers must be correctly provided when you deploy Arc resource bridge.

### KVA timeout error

The KVA timeout error is a generic error caused by various network misconfigurations that involve the management machine, For instance, the appliance VM or Control Plane IP may not have communication with each other, to the internet, or required URLs. These communication failures are often due to issues with DNS resolution, proxy settings, network configuration, or internet access.

For clarity, management machine refers to the machine where deployment CLI commands are being run. Appliance VM is the VM that hosts Arc resource bridge. Control Plane IP is the IP of the control plane for the Kubernetes management cluster in the Appliance VM.

#### Top causes of the KVA timeout error  

- Management machine is unable to communicate with Control Plane IP and Appliance VM IP.
- Appliance VM is unable to communicate with the management machine, vCenter endpoint (for VMware), or MOC cloud agent endpoint (for Azure Stack HCI).  
- Appliance VM doesn't have internet access.
- Appliance VM has internet access, but connectivity to one or more required URLs is being blocked, possibly due to a proxy or firewall.
- Appliance VM is unable to reach a DNS server that can resolve internal names, such as vCenter endpoint for vSphere or cloud agent endpoint for Azure Stack HCI. The DNS server must also be able to resolve external addresses, such as Azure service addresses and container registry names.  
- Proxy server configuration on the management machine or Arc resource bridge configuration files is incorrect. This can impact both the management machine and the Appliance VM. When the `az arcappliance prepare` command is run and the host proxy isn't correctly configured, the management machine can't connect and download OS images. Internet access on the Appliance VM might be broken by incorrect or missing proxy configuration, which impacts the VM's ability to pull container images.  

#### Troubleshoot KVA timeout error

To resolve the error, one or more network misconfigurations might need to be addressed.

- The first step is to collect logs by Appliance VM IP (not by kubeconfig, as the kubeconfig could be empty if the deploy command didn't complete). Problems collecting logs are most likely due to the management machine being unable to reach the Appliance VM.

   Once logs are collected, extract the folder and open `kva.log`. Review the log for information that might help pinpoint the cause of the KVA timeout error.

- The management machine must be able to communicate with the Appliance VM IP and Control Plane IP. Ping the Control Plane IP and Appliance VM IP from the management machine and verify that there's a response from both IPs.

   If a request times out, the management machine can't communicate with the IPs. This issue might be caused by a closed port, network misconfiguration, or firewall block. Work with your network administrator to allow communication between the management machine to the Control Plane IP and Appliance VM IP.

- Appliance VM IP and Control Plane IP must be able to communicate with the management machine and vCenter endpoint (for VMware) or MOC cloud agent endpoint (for Azure Stack HCI). Work with your network administrator to ensure the network is configured to permit this communication. You might need to add a firewall rule to open port 443 from the Appliance VM IP and Control Plane IP to vCenter, or to open port 65000 and 55000 for Azure Stack HCI MOC cloud agent. Review [network requirements for Azure Stack HCI](/azure-stack/hci/manage/azure-arc-vm-management-prerequisites#network-port-requirements) and [VMware](../vmware-vsphere/quick-start-connect-vcenter-to-arc-using-script.md) for Arc resource bridge.

- Appliance VM IP and Control Plane IP need internet access to [these required URLs](#not-able-to-connect-to-url). Azure Stack HCI requires [additional URLs](/azure-stack/hci/manage/azure-arc-vm-management-prerequisites). Work with your network administrator to ensure that the IPs can access the required URLs.

- In a non-proxy environment, the management machine must have external and internal DNS resolution. The management machine must be able to reach a DNS server that can resolve internal names such as vCenter endpoint for vSphere or cloud agent endpoint for Azure Stack HCI. The DNS server also needs to be able to [resolve external addresses](#not-able-to-connect-to-url), such as Azure URLs and OS image download URLs. Work with your system administrator to ensure that the management machine has internal and external DNS resolution. In a proxy environment, the DNS resolution on the proxy server should resolve internal endpoints and [required external addresses](#not-able-to-connect-to-url).

   To test DNS resolution to an internal address from the management machine in a non-proxy scenario, open a command prompt and run `nslookup <vCenter endpoint or HCI MOC cloud agent IP>`. You should receive an answer if the management machine has internal DNS resolution in a non-proxy scenario.  

1. Appliance VM needs to be able to reach a DNS server that can resolve internal names, such as vCenter endpoint for vSphere or cloud agent endpoint for Azure Stack HCI. The DNS server also needs to be able to resolve external/internal addresses, such as Azure service addresses and container registry names for download of the Arc resource bridge container images from the cloud.

   Verify that the DNS server IP used to create the configuration files has internal and external address resolution. If not, [delete the appliance](/cli/azure/arcappliance/delete), recreate the Arc resource bridge configuration files with the correct DNS server settings, and then deploy Arc resource bridge using the new configuration files.

### Move Arc resource bridge location

Resource move of Arc resource bridge isn't currently supported. Instead, delete the Arc resource bridge and redeploy it to the desired location.

## Azure Arc-enabled VMs on Azure Stack HCI issues

For general help resolving issues related to Azure Arc-enabled VMs on Azure Stack HCI, see [Troubleshoot Azure Arc-enabled virtual machines](/azure-stack/hci/manage/troubleshoot-arc-enabled-vms).

### Action failed - no such host

When you deploy Arc resource bridge, if you receive an error with `errorCode` as `PostOperationsError`, `errorResponse` as code `GuestInternetConnectivityError` and `no such host`, the appliance VM IPs may not be able to reach the endpoint specified in the error.

Error example:

`{ _errorCode_: _PostOperationsError_, _errorResponse_: _{\n\_message\_: \_{\\n  \\\_code\\\_: \\\_GuestInternetConnectivityError\\\_,\\n  \\\_message\\\_: \\\_Not able to connect to http://aszhcitest01.company.org:55000. Error returned: action failed after 5 attempts: Get \\\\\\\_http://aszhcitest01.company.org:55000\\\\\\\_: dial tcp: lookup aszhcitest01.company.org: on 127.0.0.53:53: no such host. Arc Resource Bridge network and internet connectivity validation failed: cloud-agent-connectivity-test. 1.  check your networking setup and ensure the URLs mentioned in : https://aka.ms/AAla73m are reachable from the Appliance VM.   2. Check firewall/proxy settings`

In the example, the appliance VM IPs are unable to access `http://aszhcitest01.company.org:55000`, which is the MOC endpoint. Work with your network administrator to make sure that the DNS server is able to resolve the required URLs.

To test connectivity to the DNS server:

```ping <dns-server.com>```

To check if the DNS server is able to resolve an address, run this command from a machine that can reach the DNS servers:

```Resolve-DnsName -Name "http://aszhcitest01.company.org:55000" -Server "<dns-server.com>"```

### Authentication handshake failure

When running an `az arcappliance` command, you might see a connection error: `authentication handshake failed: x509: certificate signed by unknown authority`

This error is often caused when trying to run commands from remote PowerShell, which Azure Arc resource bridge doesn't support.

To install Arc resource bridge on an Azure Stack HCI cluster, `az arcappliance` commands must be run locally on a node in the cluster. Sign in to the node through Remote Desktop Protocol (RDP) or use a console session to run these commands.

## Azure Arc-enabled VMware VCenter issues

### `errorResponse: error getting the vsphere sdk client`

Errors with `errorCode: CreateConfigKvaCustomerError` and `errorResponse: error getting the vsphere sdk client` occur when your deployment machine is trying to establish a TCP connection to your vCenter address but encounters a problem. This can happen when your vCenter address is incorrect (403 or 404 error), or because a network/proxy/firewall configuration blocks it (connection attempt failed).

If you enter your vCenter address as a hostname and receive the error `no such host`, then your deployment machine isn't able to resolve the vCenter hostname via the client DNS. This may occur when the deployment machine is able to resolve the vCenter hostname, but the deployment machine can't reach the IP address it received from DNS. You might also see this error if the endpoint returned by DNS isn't your vCenter address, or if the traffic was intercepted by proxy. If your deployment machine is able to communicate with your vCenter address, confirm that your username and password are correct.

### vSphere SDK client - Connection attempt failed

If you receive an error during deployment that states: `errorCode_: _CreateConfigKvaCustomerError_, _errorResponse_: _error getting the vsphere sdk client: Post \_https://ip.address/sdk\_: dial tcp ip.address:443: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond._ }` then your management machine is unable to communicate with your vCenter server.

To resolve this issue, ensure that your management machine meets the [management machine requirements](system-requirements.md#management-machine-requirements) and that there's not a firewall or proxy blocking communication.

### vSphere SDK client - 403 Forbidden or 404 not found

Errors that contain `errorCode_: _CreateConfigKvaCustomerError_, _errorResponse_: _error getting the vsphere sdk client: POST \_/sdk\_: 403 Forbidden` or `404 not found` while deploying Arc resource bridge are most likely due to an incorrect vCenter address. This address is provided during configuration file creation, when you're prompted to enter the vCenter address as either a hostname or IP address.

There are different ways to find your vCenter address. One option is to access the vSphere client via its web interface. The vCenter hostname or IP address is typically what you use in the browser to access the vSphere client. If you're already logged in, you can look at the browser's address bar, where the URL you use to access vSphere is your vCenter server's hostname or IP address. Verify your vCenter address, then try the deployment again.

### vSphere SDK client - no such host

The error`{ _errorCode_: _CreateConfigKvaCustomerError_, _errorResponse_: _error getting the vsphere sdk client: Post \_https://your.vcenter.hostname/sdk\_: dial tcp: lookup your.vcenter.hostname: no such host_ }` can occur during deployment when the deployment machine can't resolve the vCenter hostname to an IP address. This issue arises because the deployment process is attempting to establish a TCP connection from your deployment machine to the vCenter hostname, but the connection fails due to DNS resolution problems.

To fix this error, ensure the DNS configuration on your deployment machine is correct, verify that the DNS server is online, and check for a missing DNS entry for the vCenter hostname. You can test the DNS resolution by running `nslookup your.vcenter.hostname` or `ping your.vcenter.hostname` from the deployment machine. If you specified your vCenter address as a hostname, consider using the IP address directly instead.

### Predeployment validation errors

When you deploy Arc resource bridge, you might see various `pre-deployment validation of your download\upload connectivity wasn't successful` errors, such as:

`Pre-deployment validation of your download/upload connectivity wasn't successful. {\\n  \\\_code\\\_: \\\_ImageProvisionError\\\_,\\n  \\\_message\\\_: \\\_Post \\\\\\\_https://vcenter-server.com/nfc/unique-identifier/disk-0.vmdk\\\\\\\_: Service Unavailable`

`Pre-deployment validation of your download/upload connectivity wasn't successful. {\\n  \\\_code\\\_: \\\_ImageProvisionError\\\_,\\n  \\\_message\\\_: \\\_Post \\\\\\\_https://vcenter-server.com/nfc/unique-identifier/disk-0.vmdk\\\\\\\_: dial tcp 172.16.60.10:443: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.`

`Pre-deployment validation of your download/upload connectivity wasn't successful. {\\n  \\\_code\\\_: \\\_ImageProvisionError\\\_,\\n  \\\_message\\\_: \\\_Post \\\\\\\_https://vcenter-server.com/nfc/unique-identifier/disk-0.vmdk\\\\\\\_: use of closed network connection.`

`Pre-deployment validation of your download/upload connectivity wasn't successful. {\\n  \\\_code\\\_: \\\_ImageProvisionError\\\_,\\n  \\\_message\\\_: \\\_Post \\\\\\\_https://vcenter-server.com/nfc/unique-identifier/disk-0.vmdk\\\\\\\_: dial tcp: lookup hostname.domain: no such host`

A combination of these errors usually indicates that the management machine has lost connection to the datastore, or that there's a networking issue causing the datastore to be unreachable. This connection is needed in order to upload the OVA from the management machine used to build the appliance VM in vCenter.

To fix the issue, reestablish the connection between the management machine and datastore, then try deploying Arc resource bridge again.

### x509 certificate has expired or isn't yet valid

When you deploy Arc resource bridge, you may encounter the error:

`Error: { _errorCode_: _PostOperationsError_, _errorResponse_: _{\n\_message\_: \_{\\n  \\\_code\\\_: \\\_GuestInternetConnectivityError\\\_,\\n  \\\_message\\\_: \\\_Not able to connect to https://msk8s.api.cdp.microsoft.com. Error returned: action failed after 3 attempts: Get \\\\\\\_https://msk8s.api.cdp.microsoft.com\\\\\\\_: x509: certificate has expired or isn't yet valid: current time 2022-01-18T11:35:56Z is before 2023-09-07T19:13:21Z. Arc Resource Bridge network and internet connectivity validation failed: http-connectivity-test-arc. 1.  check your networking setup and ensure the URLs mentioned in : https://aka.ms/AAla73m are reachable from the Appliance VM.   2. Check firewall/proxy settings`

This error is caused when there's a clock/time difference between ESXi hosts and the management machine running the deployment commands for Arc resource bridge. To resolve this issue, turn on NTP time sync on the ESXi hosts, confirm that the management machine is also synced to NTP, then try the deployment again.

### Resolves to multiple networks

When you deploy or upgrade Arc resource bridge, you may encounter an error similar to:

`{ "ErrorCode": "PreflightcheckErrorOnPrem", 
"ErrorDetails": "Upgrade Operation Failed with error: \"{\\n \\\"code\\\": \\\"PreflightcheckError\\\",\\n \\\"message\\\": \\\"{\\\\n \\\\\\\"code\\\\\\\": \\\\\\\"InvalidEntityError\\\\\\\",\\\\n \\\\\\\"message\\\\\\\": \\\\\\\"Cannot retrieve vSphere Network 'vmware-azure-arc-01': path 'vmware-azure-arc-01' resolves to multiple networks\\\\\\\",\\\\n \\\\\\\"category\\\\\\\": \\\\\\\"\\\\\\\"\\\\n }\\\",\\n \\\"category\\\": \\\"\\\"\\n }\"" }`

This error occurs when the vSphere network segment resolves to multiple networks, due to multiple vSphere network segments using the same name that is specified in the error. To fix this error, change the duplicate network name in vCenter (not the network with the appliance VM) or deploy Arc resource bridge on a different network.

### Arc resource bridge status is disconnected

When running the initial Arc-enabled VMware onboarding script, you're prompted to provide a vSphere account. This account is stored locally within the Arc resource bridge as an encrypted Kubernetes secret. The account is used to allow the Arc resource bridge to interact with vCenter.

If the vSphere account stored locally within the resource bridge expires, your Arc resource bridge status can become disconnected. Update the credentials within Arc resource bridge and for Arc-enabled VMware by [following the updating vSphere account credentials instructions](/azure/azure-arc/vmware-vsphere/administer-arc-vmware#updating-the-vsphere-account-credentials-using-a-new-password-or-a-new-vsphere-account-after-onboarding).

### Error during host configuration

If you use the same template to deploy and delete the Arc resource bridge multiple times, you might encounter the following error:

`Appliance cluster deployment failed with error: Error: An error occurred during host configuration`

To resolve this issue, manually delete the existing template. Then run [`az arcappliance prepare`](/cli/azure/arcappliance/prepare) to download a new template for deployment.

### Unable to find folders

When you deploy Arc resource bridge on VMware, you specify the folder in which the template and VM are created. The selected folder must be a VM and template folder type. Other types of folder, such as storage folders, network folders, or host and cluster folders, can't be used for the resource bridge deployment.

### Cannot retrieve resource - not found or does not exist

When you deploy Arc resource bridge, you specify where the appliance VM is deployed. The appliance VM can't be moved from that location path. If the appliance VM moves location and you try to upgrade, you might see errors similar the following:

`{\n  \"code\": \"PreflightcheckError\",\n  \"message\": \"{\\n  \\\"code\\\": \\\"InvalidEntityError\\\",\\n  \\\"message\\\": \\\"Cannot retrieve <resource> 'resource-name': <resource> 'resource-name' not found\\\"\\n }\"\n }"`

`{\n  \"code\": \"PreflightcheckError\",\n  \"message\": \"{\\n  \\\"code\\\": \\\"InvalidEntityError\\\",\\n  \\\"message\\\": \\\"The specified vSphere Datacenter '/VxRail-Datacenter' does not exist\\\"\\n }\"\n }"`

To fix these errors, use one of these options:

- Move the appliance VM back to its original location and ensure RBAC credentials are updated for the location change.
- Create a resource with the same name, then move Arc resource bridge to that new resource.
- For Arc-enabled VMware, [run the Arc-enabled VMware disaster recovery script](../vmware-vsphere/disaster-recovery.md). The script deletes the appliance, deploys a new appliance, and reconnects the appliance with the previously deployed custom location, cluster extension, and Arc-enabled VMs.
- Delete and [redeploy the Arc resource bridge](../vmware-vsphere/quick-start-connect-vcenter-to-arc-using-script.md).

### Insufficient privileges

When you deploy or upgrade the resource bridge on VMware vCenter, you might see an error similar to:

`{  ""code"": ""PreflightcheckError"", ""message"": ""{\n  \""code\"": \""InsufficientPrivilegesError\"",\n  \""message\"": \""The provided vCenter account is missing required vSphere privileges on the resource 'root folder (MoRefId: Folder:group-d1)'. Missing privileges: [Sessions.ValidateSession].  add the privileges to the vCenter account and try again. To review the full list of required privileges, go to https://aka.ms/ARB-vsphere-privilege.\""\n }`

When you deploy Arc resource bridge, you provide vCenter credentials. Arc resource bridge stores these vCenter credentials locally to interact with vCenter. To resolve the missing privileges issue, the vCenter account used by the resource bridge needs the following privileges in VMware vCenter:

**Datastore**:

- Allocate space
- Browse datastore
- Low level file operations

**Folder**:

- Create folder

**vSphere Tagging**:

- Assign or Unassign vSphere Tag

**Network**:

- Assign network

**Resource**:

- Assign virtual machine to resource pool
- Migrate powered off virtual machine
- Migrate powered on virtual machine

**Sessions**:

- Validate session

**vApp**:

- Assign resource pool
- Import

**Virtual machine**:

- Change Configuration
  - Acquire disk lease
  - Add existing disk
  - Add new disk
  - Add or remove device
  - Advanced configuration
  - Change CPU count
  - Change Memory
  - Change Settings
  - Change resource
  - Configure managedBy
  - Display connection settings
  - Extend virtual disk
  - Modify device settings
  - Query Fault Tolerance compatibility
  - Query unowned files
  - Reload from path
  - Remove disk
  - Rename
  - Reset guest information
  - Set annotation
  - Toggle disk change tracking
  - Toggle fork parent
  - Upgrade virtual machine compatibility
- Edit Inventory
  - Create from existing
  - Create new
  - Register
  - Remove
  - Unregister
- Guest operations
  - Guest operation alias modification
  - Guest operation modifications
  - Guest operation program execution
  - Guest operation queries
- Interaction
  - Connect devices
  - Console interaction
  - Guest operating system management by VIX API
  - Install VMware Tools
  - Power off
  - Power on
  - Reset
  - Suspend
- Provisioning
  - Allow disk access
  - Allow file access
  - Allow read-only disk access
  - Allow virtual machine download
  - Allow virtual machine files upload
  - Clone virtual machine
  - Deploy template
  - Mark as template
  - Mark as virtual machine
  - Customize guest
- Snapshot management
  - Create snapshot
  - Remove snapshot
  - Revert to snapshot

## Next steps

[Understand recovery operations for resource bridge in Azure Arc-enabled VMware vSphere disaster scenarios](../vmware-vsphere/disaster-recovery.md)

If you don't see your problem here or you can't resolve your issue, try one of the following channels for support:

- Get answers from Azure experts through [Microsoft Q&A](/answers/topics/azure-arc.html).
- Connect with [@AzureSupport](https://x.com/azuresupport), the official Microsoft Azure account for improving customer experience. Azure Support connects the Azure community to answers, support, and experts.
- [Open an Azure support request](../../azure-portal/supportability/how-to-create-azure-support-request.md).