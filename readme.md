# Companion design documentation
https://confluence.oci.oraclecorp.com/display/OCIID/Getting+Started+with+the+OCI+Reference+Application

# Layout

There are three main entry points: 
* ``scm-service-api``
* ``scm-service-worker``
* ``scm-git-api``

Both the worker and scm-service utilize ```scm-service-dal``` to talk to underlying Kiev. Both are dropwizard projects.

``scm-service-config`` contains Shepherd configs to deploy the service. 

** Note **
Make sure that you've replaced the generic information with you project specific information. (Search for `Todo-Replace` in your project )

# Build
Build the whole solution by running the following command from the repository root: 
`mvn clean install`
Always confirm this succeeds before merging any PRs

### Troubleshooting

Make sure your development environment is set up properly. Including, making sure that the maven settings.xml file has the correct content. See the following links for details:

https://confluence.oci.oraclecorp.com/display/PGI/Dev+Environment+and+Tools+Setup

https://confluence.oci.oraclecorp.com/display/IODOCS/Artifactory+-+Maven+Repositories

### Updating Coverage Numbers

`mvn clean install` automatically updates the code coverage numbers in the POM xml if the new numbers are higher than baseline. The build fails if the coverage drops below the specified level in POM.xml. (https://confluence.oci.oraclecorp.com/display/DLC/Automatic+Updates+to+pom.xml+Code+Coverage+Numbers)

For local run to skip JaCoCo/Code Coverage reports either update pom property `<jacoco.skip>false</jacoco.skip>` to true or run commands with `mvn ... -Djacoco.skip=true`


### Quicker builds for faster iteration
To build more quickly, the 'fast' option can be used to turn off findbugs,pmd, and other resource intensive tasks.
`mvn clean install -Dfast`
On my (8 core) laptop, this drops build times from 2:32 to 1:32

### Multithreaded Builds
Maven can parallelize module compilation where possible.  The '-T' option enables multithreaded builds.  This option takes 1 argument, which can be an absolute number (e.g. -T 4), or multiplier for the cores in your machine (e.g. -T 2C)
`mvn clean install -T 1C`
On my (8 core) laptop, this drops build times to 2:14 (from the default 2:32)

### Even Faster Builds
Combining threaded builds with the 'fast' option, regular builds can be even faster:
`mvn clean install -Dfast -T 1C`
Drops build times with tests to 1:12.

### EVEN MORE FASTER BUILDS
Turning off the coverage number updating to gain more speed:
`mvn clean install -Dfast -T 1C -Djacoco.skip=true`
Reduces build times to 1:03 on my machine, including tests

# Running Services 
Both the API: `scm-service-api` and the Workflow application: `scm-service-worker` need to be run individually.

See Readme.md under their respective packages for instructions on how to run them.

## Running Kiev Locally
Kiev team provides and in-memory Kiev, that runs in the process and can be used to test individual projects.

However, since both `scm-service-api` (API) and `scm-service-worker` (worker) need to use the same store, running the in-memory / in-proc Kiev instance would not be enough. In order to test end-to-end you need to run Kiev classic

* Ensure services in the desktop domain are configured to talk to the local (not in memory) kiev instance.  All services in this repository are currently configured this way by default.

* Next make sure you use the kiev-classic

```
This starts the database and creates a pdb, cleaning up an existing database if one exists
./db.sh --db-clean
```

## configuring `scm-git-api` and `scm-service-api`
If you have not already configured your local .oci directory, follow the steps in this runbook before proceeding:
https://confluence.oci.oraclecorp.com/display/DLCSCM/How+to+set+up+your+.oci+directory

## Code Style
It doesn't matter what style you use locally; Running `mvn install` will reformat all source code according to AOSP code style. Just make sure to do this before submitting any PRs.

# Creating an operations Dashboard
We provide a basic starting operational dashboard in dashboard.json. Use the following steps to setup the dashboard in Grafana.
1. Navigate to the OCI Grafana instance: https://grafana.oci.oraclecorp.com
1. Click on the `Home` icon in bar at the top.
1. Click on the `Import Dashboard` button.
1. Copy the contents of `dashboard.json` file and paste it into the input box titled `or Paste JSON`.
1. Click the `Load` button.


# Operational Readiness Checklist
Before moving your application to testing and ultimately production, address these checklist items

- [ ] Update default healthcheck (TODO: Documentation link)

## Splat Integration
OCI's Service Platform (Splat) is a fully managed API gateway and recommended for OCI control planes to integrate, please read more about splat at https://confluence.oci.oraclecorp.com/pages/viewpage.action?spaceKey=PLAT&title=Splat+-+User+Guide

## SPLAT Integration
OCI's Service Platform (Splat) is a fully managed API gateway and recommended for OCI control planes to integrate.

1. [SPLAT User Guide](https://confluence.oci.oraclecorp.com/x/goj7Aw) This page is a must read before understanding SPLAT-related features.
2. [SPLAT Swagger API Specification Extensions](https://confluence.oci.oraclecorp.com/x/UQ9jCg)
This page is a necessary prerequisite.
It gives an overview of custom annotations in the swagger API specifications and explains how to manage most of SPLAT features via those annotations.
3. [How to enable SPLAT authorization automation](https://confluence.oci.oraclecorp.com/x/-ToxCQ)
4. [How to enable SPLAT quota enforcement](https://confluence.oci.oraclecorp.com/x/yQ5jCg).
Part that explains *resource tracking* is applicable to all stateful SPLAT scenarios (including both tag store and quota enforcement).
This document includes description of semantics behind last accepted and last completed token. 
Note, that ScmService example is not using quota enforcement at the moment.
5. [How to enable SPLAT tag store](https://confluence.oci.oraclecorp.com/x/bbQZCg)

### Running Splat locally
You can run Splat locally and test your service in a similar setup to your expected production environment, please follow the instructions on splat-local/readme.md

### Running Splat in pre-production environment

In order to integrate your service with Splat in overlay, you will need a DNS record and Https (TLS) certificate.
However, currently acquiring a DNS record requires CSSAP approval (Corporate Security Solution Assurance Process) 
https://confluence.oraclecorp.com/confluence/x/7vyICw). Unfortunately trying to get approval at an early development 
stage may not be feasible for many services. Therefore Splat team developed following strategy that enables team to 
barrow DNS records and Certificates from Splat team during pre CSSAP stage. Read more at 
https://confluence.oci.oraclecorp.com/x/hfdAD

## Secret Service Integration

Secret Service is an internal secret management product offered to internal Oracle services. If you need to consume any secrets in your service like private keys, passwords ..etc. You need integrate with the Secret Service. To learn more 
https://confluence.oci.oraclecorp.com/display/OCIID/Secret+Service+v2+Overview

This project already comes with the plumbing to read Secrets from the Secret Service (during development from the local file system)
```
//first inject secretRetriever to your class
@Inject
public RepositoryManagementService(... , SecretRetriever secretRetriever) {
   // then use retrieveSecret method to read secret at ang given path 
   byte[] secretBytes = secretRetriever.retrieveSecret("mySecret/latest");
   String secretValue = new String(secretBytes, Charset.defaultCharset());
}
```
# UDX Integration
## Console
The console (Oracle Cloud UI) experience for your control plane is provided through the loom plugin. A loom-plugin and the associated javascript client is generated under ```scm-service-loom-plugin``` directory using the oci-plugin-cli tool.

Check [this link](https://confluence.oci.oraclecorp.com/display/KMS/Plugin+%3A+Onboarding+to+the+OCI+Console+and+the+Oracle+Cloud+UI+Platform+-+Loom) for more information about loom-plugin.

Check [this link](https://confluence.oci.oraclecorp.com/display/KMS/Plugin+%3A+Onboarding+to+the+OCI+Console+and+the+Oracle+Cloud+UI+Platform+-+Loom#Plugin:OnboardingtotheOCIConsoleandtheOracleCloudUIPlatform-Loom-loomquicklinks) to get more detailed instructions on dev environment setup, best practices, and more.

### Steps to run this loom plugin
Under ```scm-service-loom-plugin``` directory:

 1. Type "yarn install" to install all required npm dependencies. The initial install would take a few minutes.
 2. Type "yarn build" to build all your components including the javascript client. Make sure scm-service-spec module is already built since we would need the generated swagger spec to generate javascirpt client.
 3. Type "yarn start" to run your loom plugin locally.
 
To view or test your plugin, log into your OCI console in your browser, then copy the URI in previous step's output. Note: you would need to whitelist your plugin's path in your teanncy first, check [how to run your plugin](https://confluence.oci.oraclecorp.com/display/KMS/Plugin+%3A+How+to+run+your+Loom+plugin) for more detailed instructions. 

### Update or re-generate loom plugin:
For significant changes in swagger spec it is best to use the Loom plugin CLI to regenerate the loom plugin.
To regenerate your loom plugin: remove all components under ```scm-service-loom-plugin```(including hidden .* files) and run the following command:
```shell
oci-plugin-cli generate \
  -t pegasus \
  -l local \
  --compile \
  -n scm-svc \
  --resource-name Repository \
  --service-name Repository \
  --api-client-name scm-service \
  -s ../scm-service-spec/target/classes/release/api.yaml
```
### Known bugs
1. Build may fail with oci-plugin-compliance due to a bug in react version checking. The bug is tracked in [this jira](https://jira.oci.oraclecorp.com/browse/CSDK-72).
2. The relative path ```--spec ../../../scm-service-spec/target/classes/release/api.yaml```in ```scm-service-loom-plugin/packages/scm-service-api-client/package.json``` is resolved wrong. To unblock your build, simply change the spec path to ```--spec ../../../scm-service-spec/target/classes/release/api.yaml``` if loom team haven't fixed the bug yet.

### What's next
Check the [Console UDX Federation Bar](https://confluence.oci.oraclecorp.com/display/UIPLAT/Console+Federation+Bar) for deployment, monitoring, and operational requirements.

## RQS Integration 
Providing a search on OCI resources is a necessary feature so that Customers can access them easily and quickly. 
As part of this, every OCI service has to follow steps to integrate with RQS. RQS integration can now be automated through SPLAT. 

### Related links 
[How to enable RQS automation through SPLAT](https://confluence.oci.oraclecorp.com/display/PLAT/How+to+enable+RQS+automation#HowtoenableRQSautomation-EnableRQSautomationforaresource)
This page is a good read to understand RQS integration through splat.

[Task template for on-boarding to RQS Automation](https://confluence.oci.oraclecorp.com/display/PLAT/Task+template+for+onboarding+to+RQS+automation)
Teams can create a copy of this task template to track the changes for on-boarding to RQS.

[Authoring RQS schema](https://confluence.oci.oraclecorp.com/display/QP/Author+RQS+schema)
This page provides details of every field defined in the RQS schema. Please go through this before making any changes to the RQS schema.

[Integrating with RQS ](https://confluence.oci.oraclecorp.com/pages/viewpage.action?pageId=17313907)
This page contains information for service owners on how to integrate with RQS.

### Steps needed to enable RQS automation
1. Update the RQS schema defined in the ```resource-query-service``` module, which is present in the ```infrastructure``` directory of shepherds flock config.
    a) Update the ```scope``` of the schema, and the ```scope``` defined in ```rqs_integration_through_splat``` policy.
    b) For a new service, the ```target_lifecycle_state``` should start from ```CONFIGURED``` state, and should transition in ```CONFIGURED``` -> ```INDEXABLE``` → ```VISIBLE``` order.
    c) If you need to define additional fields, uncomment the line where variable ```additional_fields_json``` is declared, and define the additional fields in the ```repository_additional_fields.json``` file.

2. Update the API Spec in ```definitions.cond.yaml``` file
   a) Update the ```scope``` in the ```searchMetadata``` section to match the ```scope``` defined in the RQS schema.

3. If your service is past LA then the ```mode``` defined under ```search``` extension in ```paths.cond.yaml``` should be ```backfilling```. Before moving to the ```VISIBLE``` lifecycle state, co-ordinate with Splat-store dev to validate the backfill process.
   If service is new and no customers are using, ```mode``` can be ```automated```, in which case, we are not doing any backfilling. 
   More details can be found in - https://confluence.oci.oraclecorp.com/display/PLAT/How+to+enable+RQS+automation#HowtoenableRQSautomation-EnableRQSautomationforaresource


### Script to launch or teardown service(s):
service-launcher.sh is a script to launch or teardown services locally, 
which includes kiev, workflow as a service, project-api, scm-service-api, scm-service-worker.

These components can be launched or destroyed in the right sequence by one command, 
or they can be managed individually, assuming the correct order is followed. 

Kiev service need extra time to get fully up and functional, so have added 30 sec wait time after kiev starts.

Docker images and versions in use:
1. WFAAS: odo-docker-local.artifactory.oci.oraclecorp.com/wfaas-in-memory:487-master
2. KIEV:  odo-docker-local.artifactory.oci.oraclecorp.com/iod-opdb121-base:12.1.0.2.190115_1
3. proj-cp-api (Project Service Control Plane API): odo-docker-signed-local.artifactory.oci.oraclecorp.com/dlc-project-service-api:0.1.90
   (We need to keep the proj-cp-api version up to date in case project service get some code changes)

In case any error occurs or if you want to check the logs you can search log generated 
at parent directory of project source code with name scm-api.log and scm-worker.log

#####Assumptions
* source code should already been build successfully.

#####Prerequisits
* jq is installed. On macOS, run "brew install jq"
* Docker is installed
* Source code needs to be built before running this script

#####Usage

```shell
    *./service-launcher.sh [command] [option] [optional: --debug]

    * [command] can be either "launch" or "teardown"

    * [option] is one of components: kiev | wfaas | scm-api | scm-worker | scm-git-api | all (default)

    * [optional: --debug] is optional mode you could utilized to debug the service
```

#####Example

  * launch all component (all services):

    ./service-launcher.sh launch all
    
  * launch all component in debug mode (all services):
    
    ./service-launcher.sh launch all --debug

  * teardown all component (all services):

    ./service-launcher.sh teardown all

  * launch scm-service-api only (assuming all dependent components are ready):

    ./service-launcher.sh launch scm-api
    
  * launch scm-service-worker in debug mode (assuming all dependent components are ready):
    
    ./service-launcher.sh launch scm-worker --debug

  * teardown scm-git-api only:

    ./service-launcher.sh teardown scm-git-api
    
#####Need more details
[Running and Debugging](https://confluence.oci.oraclecorp.com/display/DLCSCM/Pegasus)