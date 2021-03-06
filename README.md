![MICROFOCUS LOGO](https://upload.wikimedia.org/wikipedia/commons/4/4e/MicroFocus_logo_blue.png)

Project status: [![Build status](https://ci.appveyor.com/api/projects/status/umvce8v9113dcxnt?svg=true)](https://ci.appveyor.com/project/mstekel/octane-gitlab-service/branch/master)

Master release status: [![Build status](https://ci.appveyor.com/api/projects/status/umvce8v9113dcxnt/branch/master?svg=true)](https://ci.appveyor.com/project/mstekel/octane-gitlab-service/branch/master)

## Relevant links
-	**Download the most recent LTS version of the service** at [Github releases](https://github.com/MicroFocus/octane-gitlab-service/releases)
-	**Check the open issues (and add new issues)** at [Github issues](https://github.com/MicroFocus/octane-gitlab-service/issues)


# ALM Octane GitLab CI service
This service integrates ALM Octane with GitLab, enabling ALM Octane to display GitLab build pipelines and track build and test run results.

## Installation and configuration instructions
1. Create a new directory to serve as the installation target. This can be on any machine that can access GitLab. (If GitLab is behind a firewall, this machine must also be behind it.)
2. Download the octane-gitlab-service-\<version\>.jar file from the release assets and copy it to the target directory.
3. In ALM Octane, create an API access Client ID and Client secret with the CI/CD Integration role.
4. In GitLab, create a personal access token for a particular user.
5. In the target directory create a new application.properties file. Populate it with the following properties, and modify the placeholders with the appropriate values.
```application.properties
# Jetty oriented properties
# =========================
server.port=9091


# ALM Octane GitLab Service oriented properties
# =========================================
server.baseUrl=<mandatory: the base URL of this service, should be accessible by GitLab>
ciserver.identity=<optional: CI server identity, by default a hash value generated from the service URL is used>
octane.location=<mandatory: the URL of the ALM Octane server with the /ui path and the sharedspace parameters. Example: https://myserver:8080/ui?p=1005>
octane.apiClientID=<mandatory: ALM Octane API client ID with CI/CD integration role over the target workspace(s)>
octane.apiClientSecret=<sensitive,mandatory: ALM Octane API client secret>
gitlab.location=<mandatory: the base URL of the GitLab server>
gitlab.personalAccessToken=<sensitive,mandatory: GitLab personal access token>
gitlab.testResultsFilePattern=<optional: 'glob:pattern' or 'regex:pattern' pattern for finding the test result files inside GitLab job artifact ZIP>.
For complete documentation of the applicable patterns see https://docs.oracle.com/javase/8/docs/api/java/nio/file/FileSystem.html#getPathMatcher-java.lang.String-


# HTTP(S) proxy oriented properties
# =================================
#http.proxyUrl=<optional: http proxy URL>
#http.proxyUser=<optional: http proxy username>
#http.proxyPassword=<sensitive,optional: http proxy password>
#http.nonProxyHosts=<optional: comma separated list of hosts to access without the http proxy>
#https.proxyUrl=<optional: https proxy URL>
#https.proxyUser=<optional: https proxy username>
#https.proxyPassword=<sensitive,optional: https proxy password>
#https.nonProxyHosts=<optional: comma separated list of hosts to access without the https proxy>
```
6. From a command line prompt open on the target directory, run the command line below:
```bash
java –jar octane-gitlab-service-<version>.jar
```

### Properties information:

#### server.port
The TCP port for this service to listen on.

#### server.baseUrl
The base URL of this service, should be accessible by GitLab.

Example:

    http://myServiceServer.myCompany.com:9091

#### ciserver.identity
The identity of the GitLab server in the ALM Octane server. By default a hash value generated from the service URL is used.
Overriding this property is useful for preserving the CI server identity while migrating GitLab to another location.

#### octane.location
The URL of the ALM Octane server, using its fully qualified domain name (FQDN).

Use the following format (port number is optional):

    http://<ALM Octane hostname / IP address> {:<port number>}/ui/?p=<shared space ID>

Example:  
In this URL, the shared space ID is 1002:

    http://myALMOctaneServer.myCompany.com:8081/ui/?p=1002

**Tip: You can copy the URL from the address bar of the browser in which you opened ALM Octane.**

##### octane.apiClientID
The API access Client ID that the plugin should use to connect to ALM Octane.
##### octane.apiClientSecret
The API Client secret that the plugin should use to connect to ALM Octane. This is a sensitive token. See the 'Password Encryption' section below.
##### gitlab.location
The base URL of the GitLab server.
Example:

    http://myGitLabServer.myCompany.com:30080
##### gitlab.personalAccessToken
A personal access token for a particular user in GitLab. This is a sensitive token. See the 'Password Encryption' section below.
##### gitlab.testResultsFilePattern
A 'glob:pattern' or 'regex:pattern' pattern for finding the test result files inside GitLab job artifact ZIP.
For complete documentation about the applicable patterns see [java.nio.file.FileSystem::getPathMatcher](https://docs.oracle.com/javase/8/docs/api/java/nio/file/FileSystem.html#getPathMatcher-java.lang.String-)
Note: In order to report test results to ALM Octane, a GitLab job must add the test results file into its artifacts at a path that matches this pattern.
Example (any XML file recursively):

     glob:**.xml

## Usage instructions
1. Create a new GitLab CI server entity in ALM Octane.
2. Go to the Pipelines module.
3. Create a new pipeline from the new GitLab CI server.
4. Note that the list only shows pipelines that are owned by the user whose API access was used.
5. Run the pipeline and wait for the run to finish.
6. Look at the result – statuses, topology, build entries, test entries.
7. If you modify anything in the code, a new build is automatically triggered.
8. Look at the results again, and pay attention to the “Commits” area.

## Password encryption

Values of the properties marked as 'sensitive' in the above list can be encrypted before writing them down in the *'application.properties'* file. To encrypt a sensitive token, run the following command line:

```bash
java –jar octane-gitlab-service-<version>.jar encrypt <sensitive_token>
``` 

The result should look like this:

```bash
Encrypted token: AES:oLPF209dEPuL69RUxfG6Wg==:HOsPY6YTUj2OG5aVNtp/xQ==
```

The encrypted token that starts with 'AES:' can be directly copied (including the 'AES:' prefix) to the *'application.properties'* file.
However, password encryption is optional. You can enter the password plain values directly.

## Troubleshooting

### GitLab Runner HTTP(S) proxy
GitLab uses registered runners for running pipelines. Runners are registered as described in this article: https://docs.gitlab.com/runner/register/.
If a runner communicates with this integration service over a network that requires HTTP(S) proxy settings, an easy way to set the proxy is to add it to the runner’s registration entry.

For example, after registering a runner on a Linux machine, its registration entry resides in the /etc/gitlab-runner/config.toml file. (Tip: If you can’t find this file, run the ‘gitlab-runner list’ shell command – it displays the registration file location as ConfigFile=…). This entry may look as below:

```
[[runners]]
  name = "my-runner"
  url = "http://my-runner.cloudlab.net:30080/"
  token = "4329809ufewjfewkfjnoeihtrjfoif"
  executor = "shell"
  builds_dir = "/workspace"
  [runners.cache]
```

In order to set the HTTP(S) proxy, an “environment” row should be added to this entry (replace the URL placeholders with the correct values):

```
[[runners]]
  name = "my-runner"
  url = "http://gitlab-placeholder.net:30080/"
  token = "4329809ufewjfewkfjnoeihtrjfoif"
  executor = "shell"
  builds_dir = "/workspace"
  environment = ["HTTPS_PROXY=http://proxy-placeholder.net:8080", "HTTP_PROXY=http://proxy-placeholder.net:8080"]
  [runners.cache]
```

After adding the proxy settings, restart the gitlab-runner service.

### Logging (Log4J 2 Configuration)
The project supports Log4J 2 configuration. In application.properties use the property below for specifying the Log4J 2 configuration file.
```
logging.config
```

For example:
```
logging.config=./log4j2.xml
``` 

Below is an example of a Log4J 2 XML configuration:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="warn">
    <Properties>
        <Property name="basePath">./logs</Property>
    </Properties>
 
    <Appenders>
        <RollingFile name="fileLogger" fileName="${basePath}/app-info.log" filePattern="${basePath}/archive/app-info-%d{yyyy-MM-dd}.log">
            <PatternLayout>
                <pattern>[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %c{1} - %msg%n</pattern>
            </PatternLayout>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true" />
            </Policies>
        </RollingFile> 
        <Console name="console" target="SYSTEM_OUT">
            <PatternLayout pattern="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %c{1} - %msg%n" />
        </Console>
    </Appenders>
    <Loggers>
        <Root level="info">
            <appender-ref ref="console" />
            <appender-ref ref="fileLogger" />
        </Root>
    </Loggers>
</Configuration>
``` 

### Allowing requests to the local network
If the GitLab server and the octane-gitlab-service app both run on the same network, you need to enable "Allow requests to the local network from hooks and services" as follows:
- Open the [your_gitlab_server]/admin/application_settings page.
- In the “Outbound requests” section, check the "Allow requests to the local network from hooks and services" checkbox.

