# Install

Install the `sourcegraph-server-gen` tool.

On macOS:
```bash
wget https://storage.googleapis.com/sourcegraph-assets/sourcegraph-server-gen/darwin_amd64/sourcegraph-server-gen && chmod +x ./sourcegraph-server-gen && sudo mv sourcegraph-server-gen /usr/local/bin/
```

On Linux:
```bash
wget https://storage.googleapis.com/sourcegraph-assets/sourcegraph-server-gen/linux_amd64/sourcegraph-server-gen && chmod +x ./sourcegraph-server-gen && sudo mv sourcegraph-server-gen /usr/local/bin/
```

### Configure Sourcegraph Server

Create a `config.yaml` file with the following contents (substituting in your Sourcegraph Server access token):
```yaml
appURL: http://localhost:3080
appID: ${YOUR_ACCESS_TOKEN}              # Set this to be your Sourcegraph Server download access token
licenseKey: file!sourcegraph-server.sgl  # Make sure you copy your license key file to the same directory as config.yaml
repoListUpdateInterval: 1
httpNodePort: 30080
deploymentOverrides:
  sourcegraph-frontend:
    replicas: 1
  searcher:
    replicas: 1
  alertmanager:
    replicas: 0
  prometheus:
    replicas: 0
  load-testing:
    replicas: 0
  bi-logger:
    replicas: 0
  lsp-proxy:
    replicas: 0
storageClass: default
```

**Note:** If you are using Minikube, change the `storageClass` line to:
```yaml
storageClass: standard
```

Set the `sourcegraph-frontend` and `searcher` replica counts according to the following table:

| Seats      | sourcegraph-frontend | searcher |
|------------|----------------------|----------|
| 10-500     | 1                    | 1        |
| 500-2000   | 2                    | 2        |
| 2000-4000  | 6                    | 6        |
| 4000-10000 | 18                   | 18       |
| 10000+     | 28                   | 28       |


### Provision a Kubernetes cluster

#### Option 1: Provision a cluster via Minikube

1. Ensure you have [VirtualBox](https://www.virtualbox.org/wiki/Downloads) or another Minikube-compatible VM monitor installed.

1. `minikube start --cpus 10 --memory 24576 --feature-gates=DynamicVolumeProvisioning=true`. This starts up a Kubernetes cluster running in a VM.

#### Option 2: Provision a cluster on a cloud infastructure provider

Follow the instructions for provisioning a Kubernetes cluster on your infrastructure. Ensure that your cluster meets the recommended CPU and memory requirements based on your seat count (see table above). Refer to the table below for the recommended node type.

**Security note:** If you intend to set this up as a production instance, we recommend you create the cluster in a VPC or other secure network that disallows unauthenticated access from the public Internet. You can later expose the necessary ports via an [Internet Gateway](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Internet_Gateway.html) or equivalent mechanism. VPC is the default networking option on some providers, but you should verify this is true for your cluster. It is your responsibility to secure your Sourcegraph Server instance in a manner that meets the security requirements of your organization.

<div class="resources">
<table>
  <tr>
    <th colspan="3">Compute nodes</th>
  </tr>
  <tr><th>Provider</th><th>Node type</th><th>Attached storage</th></tr>
  <tr><td><a href="https://kubernetes.io/docs/getting-started-guides/aws/">AWS EC2</a></td><td>m4.4xlarge</td><td>100 GB SSD</td></tr>
  <tr><td><a href="https://cloud.google.com/container-engine/docs/quickstart">Google Compute Engine</a></td><td>n1-standard-16</td><td>100 GB SSD</td></tr>
  <tr><td><a href="https://azure.microsoft.com/en-us/services/container-service/kubernetes/">Azure VM</a></td><td>D16 v3</td><td>100 GB SSD</td></tr>
  <tr><td><a href="https://kubernetes.io/docs/setup/pick-right-solution/">Other</a></td><td>16 vCPU, 60 GiB memory per node</td><td>100 GB SSD</td></tr>
</table>
</div>

Set up [Dynamic Provisioning](http://blog.kubernetes.io/2017/03/dynamic-provisioning-and-storage-classes-kubernetes.html) for persistent volumes. For this, you need to add a storage class named "default" to your cluster. To do this, create a file called `storage-class.yaml`. If you're using AWS or GCP, set the contents of this file to be the snippet in the table below, with the proper substitutions. For other providers, refer to the [Kubernetes storage class documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#storageclasses) for snippets.

<div class="resources">
<table>
  <tr>
    <th colspan="3">Storage (<tt>storage-class.yaml</tt>)</th>
  </tr>
  <tr>
    <td>AWS Elastic Block Store (EBS)</td>
<td colspan="2">

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: default
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gcp2
  zones: $YOUR_ZONE
```
</td>
  </tr>
  <tr>
    <td>Google Compute Engine (GCE)</td>
    <td colspan="2">

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: default
  annotations:
    storageclass.beta.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  zone: $YOUR_ZONE
```
</td>
  </tr>
  <tr>
    <td>Azure Disk storage</td><td colspan="2">Premium Managed Disk (SSD), Premium_LRS</td>
  </tr>
  <tr>
    <td>Other</td><td colspan="2">SSD, See <a target="_blank" href="https://kubernetes.io/docs/concepts/storage/persistent-volumes/">docs</a></td>
  </tr>
</table>
</div>

**IMPORTANT:** Replace `$YOUR_ZONE` in your `storage-class.yaml` with whatever zone your cluster was created in (e.g., "us-east-1d" or "us-central1-f"). If you choose a different zone, Kubernetes will be unable to automatically provision Persistent Volumes, which will prevent Sourcegraph Server from starting.

Once you've created `storage-class.yaml`, run `kubectl apply -f ./storage-class.yaml`. Verify that the storage class has been created with `kubectl get storageclass`.



### Install Sourcegraph Server onto your cluster via Helm

<div class="alert-info">

**Note:** Sourcegraph Server sends performance and usage data to Sourcegraph to help us make our product better for you. The data sent does NOT include any source code or file data (including URLs that might implicitly contain this information). If you would prefer to opt out of sending telemetry, email us at <support@sourcegraph.com>.

</div>


1. Enable your Kubernetes cluster to download the Sourcegraph Server Docker images. Run
```
kubectl edit serviceaccounts default
```
and add the following lines:
    ```
    imagePullSecrets:
    - name: docker-registry
    ```
Now run:
```
kubectl create secret docker-registry docker-registry --docker-server=docker.sourcegraph.com --docker-username=none --docker-password=$ACCESS_TOKEN --docker-email=none
````
where `$ACCESS_TOKEN` is the token you obtained by emailing <server@sourcegraph.com>.

1. Run `sourcegraph-server-gen config.yaml ./helm-chart`<br />
This generates a Helm Chart that describes the configuration for your cluster.

1. `helm init`. This installs Tiller (the server-side counterpart to Helm) onto your cluster.
If you set up your Kubernetes cluster on external infrastructure, make sure you have [configured `kubectl` to access your cluster](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/).

1. `helm install --name sourcegraph ./helm-chart/`. (If you see an error, "could not find a ready tiller pod", wait for a minute and try again.)<br />
Check that your instance is spinning up with `kubectl get pods`. If pods get stuck in "Pending" status, run `kubectl get pv` to check if the necessary volumes have been provisioned (you should see at least 4). GCP users: you may have to [request an increase in your storage quota](https://cloud.google.com/compute/quotas).

1. If you're running Minikube, run `minikube service sourcegraph-frontend --url` to obtain the URL for your instance. Note that the cluster may take a few minutes before Sourcegraph Server is available, even after Minikube claims the service is ready. You can run `kubectl get pods` to check on the health and initialization status of the cluster.<br />

    If you're running Sourcegraph Server on cloud infrastructure, your instance is likely inaccessible over the Internet at this point. You'll need to connect port 30080 (or whatever you set as the value of `httpNodePort`) on the nodes in your cluster to the Internet. The easiest way to do this is to add a network rule that allows ingress traffic to port 30080 on at least one node (e.g., [AWS Security Group rules](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_SecurityGroups.html), [GCP Firewall rules](https://cloud.google.com/compute/docs/vpc/using-firewalls)). Sourcegraph Server should then be accessible at `$EXTERNAL_IP_OF_YOUR_NODE:30080`. For full production instances, we recommend using an [Internet Gateway](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Internet_Gateway.html) (or equivalent) and configuring a load balancer in Kubernetes. Note that your instance is now accessible on the public web, so make sure you secure it before granting it access to private code (instructions below).

You should now see the Sourcegraph Server landing page when you visit the address of your instance.



### Configure Sourcegraph Server to index your code host

<div class="alert-warn">

**CAUTION:** Before you configure Sourcegraph Server to access your code host, take the necessary actions to ensure the security of your instance. Ensure at least one of the following is true before granting access to your private code:

* Sourcegraph Server is running via Minikube and your VM is not accessible on an insecure network. (By default, the Minikube VM is only accessible from the host machine.)
* Sourcegraph Server is running in a secure private network inaccessible to the public web.
* Sourcegraph Server is accessible to the public web, but you've set up TLS and authentication (see the "Production configuration" section below).


</div>

Sourcegraph Server can be configured to index any git-based code host, including GitHub Enterprise, Bitbucket Server, GitLab, private GitHub.com or Bitbucket.org, or a standalone git server. The preferred method of authenticating is via SSH key. We provide instructions for GitHub Enterprise here, but steps for other code hosts are very similar.

1. Obtain a SSH key with clone access to your GitHub Enterprise instance. For production, we recommend [creating a new key](https://help.github.com/enterprise/2.11/user/articles/adding-a-new-ssh-key-to-your-github-account/) and adding it to a [machine user](https://developer.github.com/v3/guides/managing-deploy-keys/#https-cloning-with-oauth-tokens). For non-production settings, you can use a SSH key associated with your personal account, such as the one you have on your development machine in your `~/.ssh` folder.
1. Copy your private key to the file `id_rsa` in the same file that contains your `config.yaml`.
1. Copy `~/.ssh/known_hosts` to the directory that contains your `config.yaml`. This is to ensure Sourcegraph Server trusts the identity of your code host when cloning. (Alternatively, you can use `ssh-keyscan ${HOST} >> known_hosts` to create a new `known_hosts` file containing only the hosts you wish to make accessible to your Sourcegraph Server instance.)
1. Add the following lines to the end of your `config.yaml`:
```yaml
gitOriginMap: ghe.mycompany.com/!git@ghe.mycompany.com:%.git
gitserverSSH:
      id_rsa: file!id_rsa
      known_hosts: file!known_hosts
```
Change the value of `gitOriginMap` to correspond to the URL of your code host. This value tells Sourcegraph Server how to map its URLs to your code host's SSH clone URLs. The above value would map `http(s)://my-sourcegraph-server-domain/ghe.mycompany.com/path/to/repo` to `git@ghe.mycompany.com:path/to/repo.git`. You can separate multiple mappings with a space to enable multiple code hosts simultaneously. For instance, the following would let Sourcegraph Server index your repositories on both GitHub Enterprise and GitHub.com:
```yaml
gitOriginMap: "ghe.mycompany.com/!git@ghe.mycompany.com:%.git github.com/!git@github.com:%.git"
````
1. Live-update your Sourcegraph Server configuration with:
```bash
rm -rf ./helm-chart && sourcegraph-server-gen config.yaml ./helm-chart && helm upgrade sourcegraph ./helm-chart/
```
1. Wait for the change to take effect (you can check via `kubectl get pods`). Then navigate to a URL that corresponds to a private repository. If configured correctly, Sourcegraph Server will automatically clone it from your code host. For instance, if you sent `gitOriginMap: ghe.mycompany.com/!git@ghe.mycompany.com:%s.git`, you can set the URL path to `/ghe.mycompany.com/path/to/my/repo` and Sourcegraph Server will attempt to clone from `git@ghe.company.com:path/to/my/repo.git`.



## Production configuration

After initial setup, you may want to configure your Sourcegraph Server instance for production. Sourcegraph Server supports many configuration options to satisfy your specific production environment requirements. This section describes these configuration points. To deploy any config changes to Sourcegraph Server, make the necessary changes to `config.yaml` and then run:

```bash
sourcegraph-server-gen config.yaml ./helm-chart && helm upgrade sourcegraph ./helm-chart/
```

### Search scopes

The default set of search scopes allows users to search across all repositories or only in a few predefined subsets, such as vendor files or text documents. You can configure the search scopes for all users by setting the `searchScopes` field in `config.yaml` to a string of a JSON array of `{name, value}` objects.

The `value` of a search scope can be any valid query and can include fields (such as `repo:`, `file:`, etc.).

For example:

```yaml
searchScopes: '[{"name":"Test code","value":"file:(test|spec)"},{"name":"Non-vendor code","value":"-file:vendor/ -file:node_modules/"}]'
```

You must surround the JSON array in single quotes.

### TLS/SSL

If you intend to make your Sourcegraph Server instance accessible on the Internet or another untrusted network, you should add a TLS certificate so that all traffic will be served over HTTPS.

Add the following lines to `config.yaml`:
```yaml
tlsCert: file!path/to/my/cert
tlsKey: file!path/to/my/tls/key
httpsNodePort: 30443
```
You can also change the value of `appURL` to `https` to force-redirect to a HTTPS URL.

### Custom Domain

It is highly recommended that you do NOT make any of the nodes running Sourcegraph Server directly accessible to the Internet. Instead, configure an [Internet Gateway](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Internet_Gateway.html) or equivalent to forward traffic to `httpNodePort` or `httpsNodePort` on any of the nodes in your cluster.

Once that is done, update your DNS records to point to your gateway's external IP, and change the following line in your `config.yaml`:
```yaml
appURL: https://your.domain.com
```

### Single sign-on (SSO)

Sourcegraph Server supports SSO authentication via OpenID Connect (OIDC) and SAML 2.0.

#### OpenID Connect

OpenID Connect is supported by most major SSO providers, including but not limited to:
- [Okta](https://developer.okta.com/docs/api/resources/oidc.html)
- [OneLogin](https://www.onelogin.com/openid-connect)
- [Ping Identity](https://www.pingidentity.com/en/resources/client-library/articles/openid-connect.html)
- [Auth0](https://auth0.com/docs/protocols/oidc)
- [Salesforce Identity](https://developer.salesforce.com/page/Inside_OpenID_Connect_on_Force.com)
- [Microsoft Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-protocols-openid-connect-code)

To configure Sourcegraph Server to require SSO via OpenID Connect, add the following lines to your `config.yaml`:
```yaml
appURL: https://${your-sourcegraph-server-url}  # set this explicitly, even if it is just an IP address
oidcProvider: https://${your-sso-provider-url}
oidcClientID: your-oidc-client-id
oidcClientSecret: your-oidc-client-secret
```

In your SSO provider settings, set the OIDC callback URL of your client application to `https://${your-sourcegraph-server-url}/.auth/callback`.

#### SAML

SAML 2.0 is supported by most major SSO providers, including but not limited to:
- [Okta](https://developer.okta.com/standards/SAML/index)
- [OneLogin](https://www.onelogin.com/saml)
- [Ping Identity](https://www.pingidentity.com/en/resources/client-library/articles/saml.html)
- [Auth0](https://auth0.com/docs/protocols/saml)
- [Salesforce Identity](https://help.salesforce.com/articleView?id=sso_saml_setting_up.htm&type=0)
- [Microsoft Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-single-sign-on-protocol-reference)

To configure Sourcegraph Server to require SSO via SAML, first add a new SAML application to your SSO provider. Specify the following values in the SAML configuration for your app in your SSO provider's settings:

* **Single Sign-On URL, Recipient URL, Destination URL:** `https://${your-sourcegraph-server-url}/.auth/saml/acs`
* **Audience URI / SP Entity ID:** `https://${your-sourcegraph-server-url}/.auth/saml/metadata`
* **Attribute statements** - These are the user attributes returned to the service provider in the SAML assertion. Some identity providers set these by default; others require them to be set explicitly. Ensure that the following attributes exist in the SAML assertion:<br>
  `Email` (required): the user's email<br>
  `Login` (optional): the user's login name (used in @mentions)<br>
  `DisplayName` (optional): the name to display in the nav bar (typically the user's first name)<br>

Now generate a private key and self-signed TLS certificate that Sourcegraph Server will use as a SAML service provider:

```bash
openssl req -x509 -newkey rsa:4096 -keyout saml-sp.key -out saml-sp.cert -days 365 -nodes -subj "/CN=myservice.example.com"
```

Now add the following lines to your `config.yaml`:

```yaml
# You can typically find this listed in your SSO Provider's admin interface after
# you've created your SAML app. It's sometimes called the "Identity Provider metadata URL".
samlIDProviderMetadataURL: https://${your-sso-saml-id-metadata-url}

samlSPCert: file!saml-sp.cert  # path to the certificate file you just created
samlSPKey: file!saml-sp.key    # path to the private key file you just created
```


### More customizations

Additional configuration points are documented in the [Sourcegraph Server Configuration README](/docs/server/readme). Note that some configuration options are only valid in some enterprise configurations of Sourcegraph Server. If you have any questions, [contact us](support@sourcegraph.com) for more information.


## Troubleshooting / FAQ

If Sourcegraph Server does not start up or shows unexpected behavior, there are a variety of ways you can determine the root cause of the failure. The most useful commands are:

* `kubectl get pods -o=wide` — lists all pods in your cluster and the corresponding health status of each.
* `kubectl log -f $POD_NAME` — tails the logs for the specified pod.
* `kubectl describe $POD_NAME` — shows detailed info about the status of a single pod.
* `kubectl get pvc` — lists all Persistent Volume Claims (PVCs) and the status of each.
* `kubectl get pv` — lists all Persistent Volumes (PVs) that have been provisioned. In a healthy cluster, there should be a one-to-one mapping between PVs and PVCs.
* `kubectl get events` — lists all events in the cluster's history.

### Common errors

* `kubectl get pods -o=wide` shows pods with status "ImagePullError" or "ImagePullBackOff". `kubectl get events` shows errors of the form "Failed to pull image ... not found".

  It's likely that the cluster has not been configured to properly authenticate to docker.sourcegraph.com. Check that you've added the `imagePullSecrets: ...` configuration via `kubectl edit serviceaccounts default`, and that you've added the right docker-registry secret via `kubectl create secret docker-registry...`

* `kubectl get pv` shows no Persistent Volumes and/or `kubectl get events` shows a 'Failed to provision volume with StorageClass "default"' error.

  Check that a storage class named "default" exists via `kubectl get storageclass`. If one does exist, run `kubectl get storageclass default -o=yaml` and verify that the zone indicated in the output matches the zone of your cluster.

* I navigate to a URL in Sourcegraph Server that should correspond to a repository, but I see a "404: Not Found" page.

  This is likely an issue with the `gitOriginMap` or `gitserverSSH` configuration settings. Double check that `gitOriginMap` maps to the correct clone URL and that `gitserverSSH` provides the necessary SSH credentials to clone from that URL. If both appear to be correct, try SSHing into the `gitserver-1` pod (`kubectl exec -it $(kubectl get pods | grep gitserver-1 | awk '{ print $1 }') -- /bin/sh`) and attempting to clone the repository from within the pod from its SSH URL. If the clone fails, that will tell you the underlying error. If the clone succeeds, then the problem is probably with `gitOriginMap` not mapping to the right clone URL.

If you have any other issues with installation, email <support@sourcegraph.com>.
