---
title: API Reference

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - install
  - api

search: true
---

Sourcegraph Server README
==============================

Generate helm chart for new version
-----------------------------------

On OS X:

```
wget https://storage.googleapis.com/sourcegraph-assets/sourcegraph-server-gen/darwin_amd64/sourcegraph-server-gen
chmod +x ./sourcegraph-server-gen
./sourcegraph-server-gen config.json ./helm-chart
```

On Linux:

```bash
wget https://storage.googleapis.com/sourcegraph-assets/sourcegraph-server-gen/linux_amd64/sourcegraph-server-gen
chmod +x ./sourcegraph-server-gen
./sourcegraph-server-gen config.json ./helm-chart
```

View differences of new version
-------------------------------
```
[helm plugin install https://github.com/databus23/helm-diff]
helm diff sourcegraph ./helm-chart
```

Pre- and post-upgrade check
---------------------------
Check that all pods are eventually running:
```
watch kubectl get pods -o wide
```

Upgrade
-------
```
[helm init --upgrade]
helm upgrade sourcegraph ./helm-chart
```

Rollback
--------
```
helm history sourcegraph
helm rollback sourcegraph [REVISION]
```

Troubleshooting
---------------
Check if pods are running and if failing pods are all on the same node:
```
kubectl get pods -o wide
```

Inspect a specific pod:
```
kubectl describe pod [POD]
kubectl logs [POD]
```

Delete a failing pod so it gets recreated, possibly on a different node:
```
kubectl delete pod [POD]
```

Remove all pods from a node and mark it as unschedulable to prevent new pods from arriving:
```
kubectl drain --force --ignore-daemonsets --delete-local-data [NODE]
```

Configuration
-------------

A JSON file is passed as the first argument to the `sourcegraph-server-gen` tool to specify the configuration that will be baked into the helm chart it generates. The available configuration options are listed below. If a value has the prefix `file!`, then the file with the name given after the prefix is being read and its contents are used as the value instead.

**appID:** [string] Application ID to attribute front end user logs to. Providing this value will send usage data back to Sourcegraph (no private code is sent and URLs are sanitized to prevent leakage of private data).

**appURL:** [string] Publicly accessible URL to web app (e.g., what you type into your browser).

**tlsCert:** [string] TLS certificate for the web app.

**tlsKey:** [string] TLS key for the web app.

**corsOrigin:** [string] Value for the Access-Control-Allow-Origin header returned with all requests.

**autoRepoAdd:** [bool] Automatically add external public repositories on demand when visited.

**disablePublicRepoRedirects:** [bool] Disable redirects to sourcegraph.com when visiting public repositories that can't exist on this server.

**github:** [[]config.GitHubConfig] JSON array of configuration for GitHub hosts. See GitHub Configuration section for more information.

**githubClientID:** [string] Client ID for GitHub.

**githubClientSecret:** [string] Client secret for GitHub.

**githubPersonalAccessToken:** [string] (Deprecated: Use GitHub) Personal access token for GitHub.

**githubEnterpriseURL:** [string] (Deprecated: Use GitHub) URL of GitHub Enterprise instance from which to sync repositories.

**githubEnterpriseCert:** [string] (Deprecated: Use GitHub) TLS certificate of GitHub Enterprise instance, if from a CA that's not part of the standard certificate chain.

**githubEnterpriseAccessToken:** [string] (Deprecated: Use GitHub) Access token to authenticate to GitHub Enterprise API.

**gitoliteHosts:** [string] Space separated list of mappings from repo name prefix to gitolite hosts.

**gitOriginMap:** [string] Space separated list of mappings from repo name prefix to origin url, for example "github.com/!https://github.com/%.git".

**inactiveRepos:** [string] Comma-separated list of repos to consider 'inactive' (e.g. while searching).

**lightstepAccessToken:** [string] Access token for sending traces to LightStep.

**lightstepProject:** [string] The project id on LightStep, only used for creating links to traces.

**noGoGetDomains:** [string] List of domains to NOT perform go get on. Separated by ','.

**repoListUpdateInterval:** [int] Interval (in minutes) for checking code hosts (e.g. gitolite) for new repositories.

**ssoUserHeader:** [string] Header injected by an SSO proxy to indicate the logged in user.

**storageClass:** [string] Storage class name to use for Persistent Volume claims. If you set this, you need to ensure a storage class with the same name exists in your cluster.

**executeGradleOriginalRootPaths:** [string] Java: A comma-delimited list of patterns that selects repository revisions for which to execute Gradle scripts, rather than extracting Gradle metadata statically. **Security note:** these should be restricted to repositories within your own organization. A percent sign ('%') can be used to prefix-match. For example, <tt>git://my.internal.host/org1/%,git://my.internal.host/org2/repoA?%</tt> would select all revisions of all repositories in org1 and all revisions of repoA in org2.

**oidcProvider:** [string] The URL of the OpenID Connect Provider

**oidcClientID:** [string] OIDC Client ID

**oidcClientSecret:** [string] OIDC Client Secret

**oidcEmailDomain:** [string] Whitelisted email domain for logins, e.g. 'mycompany.com'

**samlIDProviderMetadataURL:** [string] SAML Identity Provider metadata URL (for dyanmic configuration of SAML Service Provider)

**samlSPCert:** [string] SAML Service Provider certificate

**samlSPKey:** [string] SAML Service Provider private key

**searchScopes:** [string] JSON array of custom search scopes (e.g., [{"name":"Text Files","value":"file:\.txt$"}])

**disableSearch2:** [bool] disable new query UX by default ('localStorage.search2=true;window.reload()' to enable)

**privateArtifactRepoID:** [string] Java: Private artifact repository ID in your build files. If you do not explicitly include the private artifact repository, then set this to some unique string (e.g,. "my-repository").

**privateArtifactRepoURL:** [string] Java: The URL that corresponds to privateArtifactRepoID (e.g., http://my.artifactory.local/artifactory/root).

**alertmanagerURL:** [string] Publicly accessible URL to alertmanager, used in alert messages.

**alertmanagerConfig:** [string] Contents of the alertmanager configuration file, see below.

**gitserverSSH:** [map[string]string] Contents of the ~/.ssh directory used by gitserver.

**htmlHeadTop:** [string] HTML to inject at the top of the <head> element on each page, for analytics scripts

**htmlHeadBottom:** [string] HTML to inject at the bottom of the <head> element on each page, for analytics scripts

**htmlBodyTop:** [string] HTML to inject at the top of the <body> element on each page, for analytics scripts

**htmlBodyBottom:** [string] HTML to inject at the bottom of the <body> element on each page, for analytics scripts

**httpNodePort:** [int] Port exposed on all nodes directing HTTP traffic to sourcegraph-frontend pod.

**httpsNodePort:** [int] Port exposed on all nodes directing HTTPS traffic to sourcegraph-frontend pod.

**langGo:** [bool] Use Go language server.

**langJava:** [bool] Use Java language server.

**langJavaScript:** [bool] Use JavaScript language server.

**langTypeScript:** [bool] Use TypeScript language server.

**langPython:** [bool] Use Python language server.

**langPHP:** [bool] Use PHP language server.

**langSwift:** [bool] Use Swift language server.

**licenseKey:** [string] License key. You must purchase a license to obtain this.

**useRBAC:** [bool] Enable this if RBAC is enabled on your Kubernetes cluster.

**authProxyIP:** [string] External IP address to proxy HTTP traffic to sourcegraph-frontend with basic authentication

**authProxyPassword:** [string] Password used by auth-proxy for basic authentication.

**deploymentOverrides:** [map[string]config.DeploymentOverrides] Kubernetes configuration overrides

**adminUsernames:** [string] Space-separated list of usernames that indicates which users will be treated as instance admins

GitHub Configuration
----------

For integratino with Github.com or GitHub Enterprise, multiple configurations be provided to the `github` field.

Here is an example:

```
{
  "github": [
	{
	  "url": "https://github-enterprise.mycompany.com",
	  "token": "..."
	},
	{
	  "url": "https://github.com",
	  "token": "..."
	  "repos": ["facebook/react","golang/go"]
	}
  ]
}
```

**url:** [string] URL of a GitHub instance e.g. https://github.com.

**token:** [string] A GitHub personal access token with repo and org scope.

**certificate,omitempty:** [string] TLS certificate of a GitHub Enterprise instance, if from a CA that's not part of the standard certificate chain.

**repos,omitempty:** [[]string] Optional whitelist of additional public repos to clone.

Components
----------

**bi-logger:** Logger for business intelligence events.

**github-proxy:** Rate-limiting proxy for the GitHub API.

**gitserver-1:** Stores clones of repositories to perform Git operations.

**indexer:** Asynchronous indexing for global references.

**lsp-proxy:** Multiplexer between frontend and LSP servers.

**npm-proxy:** Cache for NPM.

**pgsql:** Postgres database for various data.

**redis-cache:** Redis for storing short-lived caches.

**redis-store:** Redis for storing semi-persistent data like user sessions.

**repo-list-updater:** Periodically triggers updates of the repository list.

**searcher:** Backend for text search operations.

**sourcegraph-frontend:** Serves the frontend of Sourcegraph via HTTP(S).

**syntect-server:** Backend for syntax highlighting operations.

**xlang-go:** LSP server for Go (used for live requests).

**xlang-go-bg:** LSP server for Go (used for background indexing jobs).

**xlang-java:** LSP server for Java (used for live requests).

**xlang-java-bg:** LSP server for Java (used for background indexing jobs).

**xlang-php:** LSP server for PHP.

**xlang-python:** LSP server for Python (used for live requests).

**xlang-python-bg:** LSP server for Python (used for background indexing jobs).

**xlang-swift:** LSP server for Swift.

**xlang-typescript:** LSP server for JavaScript and TypeScript (used for live requests).

**xlang-typescript-bg:** LSP server for JavaScript and TypeScript (used for background indexing jobs).

Terminology
-----------

**Container:** Typically a Docker container.

**Image:** A filesystem template to spawn a container.

**Kubernetes:** Manages containers running on a cluster of machines.

**Resource:** An entity in the state of the Kubernetes cluster. Resources are managed via the `kubectl` command.

**Node:** A Kubernetes resource that represents a machine that can run containers, for example a VM instance.

**Pod:** A Kubernetes resource that represents a group of containers running on a node. A pod can automatically restart on error. A pod never gets rescheduled to a different node.

**ReplicaSet:** A Kubernetes resource that monitors pods and spawns new pods if less pods are available than desired. Only cares about pods with a specific configuration (e.g. image version, environment variables, etc.). A change to that configuration requires a new ReplicaSet.

**Deployment:** A Kubernetes resource that manages ReplicaSets. When a change to the configuration of a Deployment is applied, it creates a new ReplicaSet for that configuration. It then orchestrates a rolling update by gradually scaling up the new ReplicaSet and scaling down the ReplicaSet that belongs to the previous configuration.

**Node Port**: A port with a fixed number that can be used from the outside of the Kubernetes cluster to talk to a service inside of the cluster. The port is available on each node of the cluster and is automatically redirected internally by Kubernetes.

**Helm:** A "package manager" for Kubernetes.

**Chart:** A bundle of files describing Kubernetes resources. It can be uploaded via Helm to create or update all resources at once.

**Release:** The bundle of actual Kubernetes resources that has been created by uploading a Helm Chart.

Alerts
------

A [Prometheus](https://prometheus.io/) pod is included in the Sourcegraph helm chart. It has the following alerts included:

**PodsMissing:** Alerts when pods are missing.

**NoPodsRunning:** Alerts when no pods are running for a service.

**ProdPageLoadLatency:** Alerts when the page load latency is too high.

**GoroutineLeak:** Alerts when a service has excessive running goroutines.

**FSINodesRemainingLow:** Alerts when a node's remaining FS inodes are low.

**PrometheusMetricsBloat:** Alerts when a service is probably leaking metrics (unbounded attribute).

**DiskSpaceLow:** Alerts when a node has less than 10% available disk space.

**DiskSpaceLowCritical:** Alerts when a node has less than 5% available disk space.

**SearcherErrorRatioTooHigh:** Alerts when the search service has more than 10% of requests failing.

**XLangAbsent:** Alerts when the Go language server is not running.



Alert Notification
------------------

To be notified of alerts an [alertmanager](https://prometheus.io/docs/alerting/alertmanager/) configuration is required.

An example configuration for sending emails:

```
global:
  smtp_smarthost: 'localhost:25'
  smtp_from: 'alertmanager@example.org'

route:
  receiver: 'team-ops'

receivers:
- name: 'team-ops'
  email_configs:
  - to: 'ops@example.org'
```

To find our more read the [alertmanager configuration documentation](https://prometheus.io/docs/alerting/configuration/).


Sourcegraph Server can be installed on AWS, Google Cloud, Azure, OpenStack, and most major infrastructure providers that support Kubernetes. It can also be installed on any Mac, Linux, or Windows machine via Minikube. Sourcegraph Server integrates with the following code hosts:

* GitHub Enterprise
* Bitbucket Server
* GitLab
* GitHub.com
* Bitbucket.org
* Gitolite
* Phabricator
* AWS CodeCommit
* Microsoft Visual Studio Team Services
* Microsoft Team Foundation Server
* Google Cloud Source Repositories
* Any standalone git server

## Prerequisites

* <a href="https://kubernetes.io/docs/tasks/tools/install-kubectl/" target="_blank">kubectl</a>, v1.7.5 or later
* <a href="https://docs.helm.sh/using_helm/#installing-helm" target="_blank">Helm</a>, v2.2.1 or later
* An access token and license key that allow you to download and run Sourcegraph Server. If you have not yet received these, email <server@sourcegraph.com> and you should receive them within 24 hours.
* Access to an infrastructure provider where you can create a Kubernetes cluster OR use Minikube on any machine.

For production settings, we recommend the following resource allocation based on your seat count:

| Seats      | vCPUs | Memory | Attached Storage | Root Storage |
|------------|-------|--------|------------------|--------------|
| 10-500     | 10    | 24 GB  | 500 GB           | 50 GB        |
| 500-2000   | 16    | 48 GB  | 500 GB           | 50 GB        |
| 2000-4000  | 32    | 72 GB  | 900 GB           | 50 GB        |
| 4000-10000 | 48    | 96 GB  | 900 GB           | 50 GB        |
| 10000+     | 64    | 200 GB | 900 GB           | 50 GB        |