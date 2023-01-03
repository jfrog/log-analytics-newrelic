# NEWRELIC

`We only support this integration for Artifactory versions above 7`

## Table of Contents
`Note! You must follow the order of the steps throughout NewRelic Configuration`

1. [NewRelic Setup](#newrelic-setup)
2. [Environment Configuration](#environment-configuration)
3. [Fluentd Installation](#fluentd-installation)
    * [OS / Virtual Machine](#os--virtual-machine)
    * [Docker](#docker)
    * [Kubernetes Deployment with Helm](#kubernetes-deployment-with-helm)
4. [Fluentd Configuration for NewRelic](#fluentd-configuration-for-newrelic)
    * [Configuration steps for Artifactory](#configuration-steps-for-artifactory)
    * [Configuration steps for Xray](#configuration-steps-for-xray)
    * [Configuration steps for Mission Control](#configuration-steps-for-mission-control)
    * [Configuration steps for Distribution](#configuration-steps-for-distribution)
    * [Configuration steps for Pipelines](#configuration-steps-for-pipelines)
5. [Dashboards](#dashboards)
6. [References](#references)

## NewRelic Setup

New Relic setup can be done by going through the onboarding steps below or by using license key directly if one exists. If a license key exists, use the New Relic Fluentd plugin to forward logs, violations and metrics directly to your New Relic account.

* Create an account in New Relic
* From the account dropdown, click API keys
* Copy the license key which is also referenced in the UI as ingest - license

## Environment Configuration

We rely heavily on environment variables so that the correct log files are streamed to your observability dashboards. Ensure that you set the JF_PRODUCT_DATA_INTERNAL environment variable to the correct path for your product

The environment variable JF_PRODUCT_DATA_INTERNAL must be defined to the correct location.

Helm based installs will already have this defined based upon the underlying docker images.

For non-k8s based installations below is a reference to the Docker image locations per product. Note these locations may be different based upon the installation location chosen.

| Product          | Command                                                       |
| ---------------- |---------------------------------------------------------------|
| Artifactory      | export JF_PRODUCT_DATA_INTERNAL=/var/opt/jfrog/artifactory/   |
| Xray             | export JF_PRODUCT_DATA_INTERNAL=/var/opt/jfrog/xray/          |
| Nginx            | export JF_PRODUCT_DATA_INTERNAL=/var/opt/jfrog/nginx/         |
| Mission Control  | export JF_PRODUCT_DATA_INTERNAL=/var/opt/jfrog/mc/            |
| Distribution     | export JF_PRODUCT_DATA_INTERNAL=/var/opt/jfrog/distribution/  |
| Pipelines        | export JF_PRODUCT_DATA_INTERNAL=/opt/jfrog/pipelines/var/     |


## Fluentd Installation

### OS / Virtual Machine

Ensure you have access to the Internet from VM. Recommended install is through fluentd's native OS based package installs:

| OS             | Package Manager        | Link                                                 |
|----------------|------------------------|------------------------------------------------------|
| CentOS/RHEL    | Linux - RPM (YUM)      | https://docs.fluentd.org/installation/install-by-rpm |
| Debian/Ubuntu  | Linux - APT            | https://docs.fluentd.org/installation/install-by-deb |
| MacOS/Darwin   | MacOS - DMG            | https://docs.fluentd.org/installation/install-by-dmg |
| Windows        | Windows - MSI          | https://docs.fluentd.org/installation/install-by-msi |
| Gem Install**	 | MacOS & Linux - Gem    | https://docs.fluentd.org/installation/install-by-gem | 


** For Gem based install, Ruby Interpreter has to be setup first, following is the recommended process to install Ruby

1. Install Ruby Version Manager (RVM) as described in https://rvm.io/rvm/install#installation-explained, ensure to follow all the onscreen instructions provided to complete the rvm installation
	* For installation across users a SUDO based install is recommended, the installation is as described in https://rvm.io/support/troubleshooting#sudo

2. Once rvm installation is complete, verify the RVM installation executing the command `rvm -v`

3. Now install ruby v2.7.0 or above executing the command `rvm install <ver_num>`, ex: `rvm install 2.7.5`

4. Verify the ruby installation, execute `ruby -v`, gem installation `gem -v` and `bundler -v` to ensure all the components are intact

5. Post completion of Ruby, Gems installation, the environment is ready to further install new gems, execute the following gem install commands one after other to setup the needed ecosystem

	`gem install fluentd`


After FluentD is successfully installed, the below plugins are required to be installed

```text

'gem install fluent-plugin-newrelic'
'gem install fluent-plugin-jfrog-siem'
'gem install fluent-plugin-jfrog-metrics'
'gem install fluent-plugin-jfrog-send-metrics'

```


Configure `fluent.conf.*` according to the instructions mentioned in [Fluentd Configuration for NewRelic](#fluentd-configuration-for-newrelic) section and then run the fluentd wrapper with one argument pointed to the `fluent.conf.*` file configured.

````text
./fluentd $JF_PRODUCT_DATA_INTERNAL/fluent.conf.<product_name>
````


### Docker

In order to run fluentd as a docker image to send the log, siem and metrics data to Newrelic, the following commands needs to be executed on the host that runs the docker.

1. Check the docker installation is functional, execute command 'docker version' and 'docker ps'.

2. Once the version and process are listed successfully, build the intended docker image for the observability platform using the docker file,

	* Download Dockerfile from [here](https://raw.githubusercontent.com/jfrog/log-analytics-newrelic/master/docker-build/Dockerfile) to any directory which has write permissions.

3. Download the Dockerenvfile_<observability_platform>.txt file needed to run Jfrog/FluentD Docker Images for the intended observability platform,

	* Download Dockerenvfile_newrelic.txt from [here](https://raw.githubusercontent.com/jfrog/log-analytics-newrelic/master/docker-build/Dockerenvfile_newrelic.txt) to the directory where the docker file was downloaded.

```text

For Newrelic as the observability platform, execute these commands to setup the docker container running the fluentd installation

1. Execute 'docker build --build-arg SOURCE="JFRT" --build-arg TARGET="NEWRELIC" -t <image_name> .'

Command example

'docker build --build-arg SOURCE="JFRT" --build-arg TARGET="NEWRELIC" -t jfrog/fluentd-newrelic-rt .'

The above command will build the docker image.

2. Fill the necessary information in the Dockerenvfile_newrelic.txt file, if the value for any of the field requires to have a '/' use '\/' and if '\' is required use '\\'.

3. Execute 'docker run -it --name jfrog-fluentd-newrelic-rt -v <path_to_logs>:/var/opt/jfrog/artifactory --env-file Dockerenvfile_newrelic.txt <image_name>' 

The <path_to_logs> should be an absolute path where the Jfrog Artifactory Logs folder resides, i.e for an Docker based Artifactory Installation,  ex: /var/opt/jfrog/artifactory/var/logs on the docker host.

Command example

'docker run -it --name jfrog-fluentd-newrelic-rt -v /var/opt/jfrog/artifactory/var:/var/opt/jfrog/artifactory --env-file Dockerenvfile_newrelic.txt jfrog/fluentd-newrelic-rt'


```

### Kubernetes Deployment with Helm

Recommended installation for Kubernetes is to utilize the helm chart with the associated values.yaml in this repo.

| Product        | Example Values File             |
|----------------|---------------------------------|
| Artifactory    | helm/artifactory-values.yaml    |
| Artifactory HA | helm/artifactory-ha-values.yaml |
| Xray           | helm/xray-values.yaml           |

Update the values.yaml associated to the product you want to deploy with your New Relic settings.

Then deploy the helm chart as described below:

Add JFrog Helm repository:

```text
helm repo add jfrog https://charts.jfrog.io
helm repo update
```

Replace placeholders with your `masterKey` and `joinKey`. To generate each of them, use the command
`openssl rand -hex 32`

Artifactory ⎈:

Replace the `newrelic_licensekey` in `newrelic.licensekey` at the end of the yaml file with License key copied from New Relic in [NewRelic Setup](#newrelic-setup)

Replace `jpd_url` in `jfrog.observability.metrics.jpd_url` with Artifactory JPD URL (note - if deployed on K8s use the localhost and port number combination per sidecar)

Replace `jfrog_user` in `jfrog.observability.metrics.username` with Artifactory username for authentication

Replace `jfrog_api_key` in `jfrog.observability.metrics.apikey` with [Artifactory API Key](https://www.jfrog.com/confluence/display/JFROG/User+Profile#UserProfile-APIKey)

Replace `jfrog_access_token` in `jfrog.observability.metrics.accesstoken` with [Artifactory Scoped Token](https://www.jfrog.com/confluence/display/JFROG/Access+Tokens#AccessTokens-GeneratingAdminTokens)

Replace `common_jpd_value` in `jfrog.observability.metrics.common_jpd` with true for non-kubernetes installations or installations where JPD base URL is same to access both Artifactory and Xray (ex: https://sample_base_url/artifactory or https://sample_base_url/xray). Default value is false

```text
helm upgrade --install artifactory  jfrog/artifactory \
       --set artifactory.masterKey=FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF \
       --set artifactory.joinKey=EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEE \
       -f helm/artifactory-values.yaml
```

Artifactory-HA ⎈:

For HA installation, please create a license secret on your cluster prior to installation.

```text
kubectl create secret generic artifactory-license --from-file=<path_to_license_file>artifactory.cluster.license 
```

Note: Replace placeholders with your ``masterKey`` and ``joinKey``. To generate each of them, use the command
``openssl rand -hex 32``

Replace the `newrelic_licensekey` in `newrelic.licensekey` at the end of the yaml file with License key copied from New Relic in [NewRelic Setup](#newrelic-setup)

Replace `jpd_url` in `jfrog.observability.metrics.jpd_url` with Artifactory JPD URL (note - if deployed on K8s use the localhost and port number combination per sidecar)

Replace `jfrog_user` in `jfrog.observability.metrics.username` with Artifactory username for authentication

Replace `jfrog_api_key` in `jfrog.observability.metrics.apikey` with [Artifactory API Key](https://www.jfrog.com/confluence/display/JFROG/User+Profile#UserProfile-APIKey)

Replace `jfrog_access_token` in `jfrog.observability.metrics.accesstoken` with [Artifactory Scoped Token](https://www.jfrog.com/confluence/display/JFROG/Access+Tokens#AccessTokens-GeneratingAdminTokens)

Replace `common_jpd_value` in `jfrog.observability.metrics.common_jpd` with true for non-kubernetes installations or installations where JPD base URL is same to access both Artifactory and Xray (ex: https://sample_base_url/artifactory or https://sample_base_url/xray). Default value is false

```text
helm upgrade --install artifactory-ha  jfrog/artifactory-ha \
       --set artifactory.masterKey=FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF \
       --set artifactory.joinKey=EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEE \
       -f helm/artifactory-ha-values.yaml
```

Xray ⎈:

Update the following fields in `/helm/xray-values.yaml`:

Replace the `newrelic_licensekey` in `newrelic.licensekey` at the end of the yaml file with License key copied from New Relic in [NewRelic Setup](#newrelic-setup)

Replace `jpd_url` in `jfrog.observability.jpd_url` with Artifactory JPD URL (note - if deployed on K8s use the localhost and port number combination per sidecar)

Replace `jfrog_user` in `jfrog.observability.username` with Artifactory username for authentication

Replace `jfrog_api_key` in `jfrog.observability.apikey` with [Artifactory API Key](https://www.jfrog.com/confluence/display/JFROG/User+Profile#UserProfile-APIKey)

Use the same `joinKey` as you used in Artifactory installation to allow Xray node to successfully connect to Artifactory.

```text
helm upgrade --install xray jfrog/xray --set xray.jfrogUrl=http://my-artifactory-nginx-url \
       --set xray.masterKey=FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF \
       --set xray.joinKey=EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEE \
       -f helm/xray-values.yaml
```

## Fluentd Configuration for NewRelic

Download and configure the relevant fluentd.conf files for New Relic

### Configuration steps for Artifactory

Download the artifactory fluentd configuration file to a directory the user has permissions to write, such as the $JF_PRODUCT_DATA_INTERNAL locations discussed above in the [Environment Configuration](#environment-configuration) section.

````text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://raw.githubusercontent.com/jfrog/log-analytics-newrelic/master/fluent.conf.rt
````

#### Logs data

Override the match directive(jfrog.**) of the downloaded `fluent.conf.rt`  to send logs data to New Relic

```
<match jfrog.**>
  @type newrelic
  license_key LICENSE_KEY
  logtype "jfrog_artifactory_logs"
</match>
```

_**required**_: ```LICENSE_KEY``` is the License Key from New Relic in [NewRelic Setup](#newrelic-setup)

#### OpenMetrics data

Override the source directive of the downloaded `fluent.conf.rt` in order to source metrics from Artifactory

```
<source>
  @type jfrog_metrics
  @id metrics_http_jfrt
  tag jfrog.metrics.artifactory
  interval 5s
  metric_prefix 'jfrog.artifactory'
  jpd_url JPD_URL
  username ADMIN_USERNAME
  apikey JFROG_API_KEY
  token JFROG_ACCESS_TOKEN
  target_platform "NEWRELIC"
  common_jpd COMMON_JPD
</source>
```
_**required**_: ```JPD_URL``` is the Artifactory JPD URL of the format `http://<ip_address>`

_**required**_: ```ADMIN_USERNAME``` is the Artifactory username for authentication

_**required**_: ```JFROG_API_KEY``` is the [Artifactory API Key](https://www.jfrog.com/confluence/display/JFROG/User+Profile#UserProfile-APIKey) for authentication

_**required**_: ```JFROG_ACCESS_TOKEN``` is the [Artifactory Scoped Token](https://www.jfrog.com/confluence/display/JFROG/Access+Tokens#AccessTokens-GeneratingAdminTokens)

_**required**_: ```COMMON_JPD``` is true for non-kubernetes installations or installations where JPD base URL is same to access both Artifactory and Xray (ex: https://sample_base_url/artifactory or https://sample_base_url/xray). Default value is false

Override the match directive of the downloaded `fluent.conf.rt` in order to send metrics to New Relic

```
<match jfrog.metrics.**>
  @type jfrog_send_metrics
  target_platform "NEWRELIC"
  apikey LICENSE_KEY
  url "https://metric-api.newrelic.com/metric/v1"
</match>
```

_**required**_: ```LICENSE_KEY``` is the License Key from New Relic in [NewRelic Setup](#newrelic-setup)

_**required**_: ```URL``` replace if in EU region with `https://metric-api.eu.newrelic.com/metric/v1`. Default value is `https://metric-api.newrelic.com/metric/v1`

### Configuration steps for Xray

Download the Xray fluentd configuration file to a directory the user has permissions to write, such as the $JF_PRODUCT_DATA_INTERNAL locations discussed above in the [Environment Configuration](#environment-configuration) section.

````text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://raw.githubusercontent.com/jfrog/log-analytics-newrelic/master/fluent.conf.xray
````
#### Logs and Violation data

Override the source directive of the downloaded `fluent.conf.xray` to pull Xray Violations

```
<source>
  @type jfrog_siem
  tag jfrog.xray.siem.vulnerabilities
  jpd_url JPD_URL
  username ADMIN_USERNAME
  apikey JFROG_API_KEY
  pos_file_path "#{ENV['JF_PRODUCT_DATA_INTERNAL']}/log/jfrog_siem.log.pos"
  from_date "2016-01-01"
</source>
```

_**required**_: ```JPD_URL``` is the Artifactory JPD URL of the format `http://<ip_address>` with is used to pull Xray Violations

_**required**_: ```ADMIN_USERNAME``` is the Artifactory username for authentication

_**required**_: ```JFROG_API_KEY``` is the [Artifactory API Key](https://www.jfrog.com/confluence/display/JFROG/User+Profile#UserProfile-APIKey) for authentication

_**optional**_: If not specified, value is set to current date. Setting from_date value will result in violations from the specified date

Override the match directive of the downloaded `fluent.conf.xray` to send Logs and Violations to New Relic

```
<match jfrog.**>
  @type newrelic
  license_key LICENSE_KEY
  logtype "jfrog_artifactory_logs"
</match>
```

_**required**_: ```LICENSE_KEY``` is the License Key from New Relic in [NewRelic Setup](#newrelic-setup)

#### OpneMetrics data

Override the source directive of the downloaded `fluent.conf.xray` in order to source metrics from Xray

```
<source>
  @type jfrog_metrics
  @id metrics_http_jfrt
  tag jfrog.metrics.xray
  interval 5s
  metric_prefix 'jfrog.xray'
  jpd_url JPD_URL
  username ADMIN_USERNAME
  apikey JFROG_API_KEY
  target_platform "NEWRELIC"
</source>
```

_**required**_: ```JPD_URL``` is the Artifactory JPD URL of the format `http://<ip_address>` with is used to pull Xray Violations

_**required**_: ```ADMIN_USERNAME``` is the Artifactory username for authentication

_**required**_: ```JFROG_API_KEY``` is the [Artifactory API Key](https://www.jfrog.com/confluence/display/JFROG/User+Profile#UserProfile-APIKey) for authentication

Override the match directive of the downloaded `fluent.conf.rt` in order to send metrics to New Relic

```
<match jfrog.metrics.**>
  @type jfrog_send_metrics
  target_platform "NEWRELIC"
  apikey LICENSE_KEY
  url "https://metric-api.newrelic.com/metric/v1"
</match>
```

_**required**_: ```LICENSE_KEY``` is the License Key from New Relic in [NewRelic Setup](#newrelic-setup)

_**required**_: ```URL``` replace if in EU region with `https://metric-api.eu.newrelic.com/metric/v1`. Default value is `https://metric-api.newrelic.com/metric/v1`

### Configuration steps for Nginx

Download the Nginx fluentd configuration file to a directory the user has permissions to write, such as the $JF_PRODUCT_DATA_INTERNAL locations discussed above in the [Environment Configuration](#environment-configuration) section.

````text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://raw.githubusercontent.com/jfrog/log-analytics-newrelic/master/fluent.conf.nginx
````

Override the match directive(last section) of the downloaded `fluent.conf.nginx` with the details given below

```
<match jfrog.**>
  @type newrelic
  license_key LICENSE_KEY
  logtype "jfrog_nginx_logs"
</match>
```

_**required**_: ```LICENSE_KEY``` is the License Key from New Relic in [NewRelic Setup](#newrelic-setup)

### Configuration steps for Mission Control

Download the Mission Control fluentd configuration file to a directory the user has permissions to write, such as the $JF_PRODUCT_DATA_INTERNAL locations discussed above in the [Environment Configuration](#environment-configuration) section.

````text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://raw.githubusercontent.com/jfrog/log-analytics-newrelic/master/fluent.conf.missioncontrol
````

Override the match directive(last section) of the downloaded `fluent.conf.missioncontrol` with the details given below

```
<match jfrog.**>
  @type newrelic
  license_key LICENSE_KEY
  logtype "jfrog_missioncontrol_logs"
</match>
```

_**required**_: ```LICENSE_KEY``` is the License Key from New Relic in [NewRelic Setup](#newrelic-setup)

### Configuration steps for Distribution

Download the distribution fluentd configuration file to a directory the user has permissions to write, such as the $JF_PRODUCT_DATA_INTERNAL locations discussed above in the [Environment Configuration](#environment-configuration) section.

````text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://raw.githubusercontent.com/jfrog/log-analytics-newrelic/master/fluent.conf.distribution
````

Override the match directive(last section) of the downloaded `fluent.conf.distribution` with the details given below

```
<match jfrog.**>
  @type newrelic
  license_key LICENSE_KEY
  logtype "jfrog_distribution_logs"
</match>
```

_**required**_: ```LICENSE_KEY``` is the License Key from New Relic in [NewRelic Setup](#newrelic-setup)

### Configuration steps for Pipelines

Download the pipelines fluentd configuration file to a directory the user has permissions to write, such as the $JF_PRODUCT_DATA_INTERNAL locations discussed above in the [Environment Configuration](#environment-configuration) section.

````text
cd $JF_PRODUCT_DATA_INTERNAL
wget https://raw.githubusercontent.com/jfrog/log-analytics-newrelic/master/fluent.conf.pipelines
````

Override the match directive(last section) of the downloaded `fluent.conf.pipelines` with the details given below

```
<match jfrog.**>
  @type newrelic
  license_key LICENSE_KEY
  logtype "jfrog_pipelines_logs"
</match>
```

_**required**_: ```LICENSE_KEY``` is the License Key from New Relic in [NewRelic Setup](#newrelic-setup)

## Dashboards

### Artifactory dashboard
JFrog Artifactory Dashboard is divided into three sections Application, Audit, Requests and Docker

* **Application** - This section tracks Log Volume(information about different log sources) and Artifactory Errors over time(bursts of application errors that may otherwise go undetected)
* **Audit** - This section tracks audit logs help you determine who is accessing your Artifactory instance and from where. These can help you track potentially malicious requests or processes (such as CI jobs) using expired credentials.
* **Requests** - This section tracks HTTP response codes, Top 10 IP addresses for uploads and downloads
* **Docker** - To monitor Dockerhub pull requests users should have a Dockerhub account either paid or free. Free accounts allow up to 200 pull requests per 6 hour window. Various widgets have been added in the new Docker tab under Artifactory to help monitor your Dockerhub pull requests. An alert is also available to enable if desired that will allow you to send emails or add outbound webhooks through configuration to be notified when you exceed the configurable threshold.
* **Metrics** - To gain insights into the system performance, storage consumption, and connection statistics associated with JFrog Artifactory

### Xray dashboard
JFrog Xray Dashboard is divided into two sections Logs and Violations

* **Logs** - This dashboard provides a summary of access, service and traffic log volumes associated with Xray. Additionally, customers are also able to track various HTTP response codes, HTTP 500 errors, and log errors for greater operational insight
* **Violations** - This dashboard provides an aggregated summary of all the license violations and security vulnerabilities found by Xray.  Information is segment by watch policies and rules.  Trending information is provided on the type and severity of violations over time, as well as, insights on most frequently occurring CVEs, top impacted artifacts and components.  
* **Metrics** - To gain insights into the system performance, storage consumption, connection statistics, count and type of artifacts and components scanned by JFrog Xray

## References

* [Fluentd](https://www.fluentd.org) - Fluentd Logging Aggregator/Agent
* [New Relic](https://one.newrelic.com/) - New Relic Platform
* [New Relic Fluentd plugin](https://docs.newrelic.com/docs/logs/forward-logs/fluentd-plugin-log-forwarding/) - Fluentd output plugin for sending data to New Relic
* [JFrog SIEM plugin](https://github.com/jfrog/fluent-plugin-jfrog-siem) - Fleuntd input plugin to source JFrog Xray Violations
