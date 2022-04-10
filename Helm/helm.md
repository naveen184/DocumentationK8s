# Helm
## What is Helm?
Helm is a Kuberentes deploymnet tool for automating creation, packaging, configuration and deployment of applications and services to Kubernetes cluster.

Helm is the best way to find, share, and use software built for kubernetes.

Helm came from - ***Deis Workflow***

Helm is written in the **GO** Programming Language.

Helm v2 is obsoleat from Nov 2021 nad last version of v2 is - ***v2.17.0***

Right now the latest version is ***v3.8.1 - on 10-Mar-2022***

Helm has a strong support system.

Helm is graduates within Cloud Native Computing Foundation CNCF in 2020.
- Must be ready for mainstream majority
- Prove stability, security, healthy governance, and strong community.
- Scored 198% on the certification test.
- CNCF TOC voted by supermajority to make helm a top-level project.

### Supports:
MacOs, Linux, Windows

### Helm is more K8s Native
- Inheriting security controls from kubeconfig
- Using K8s RBAC to limit access & resources.
- Replacing custom APIs for charts and deployments with secrets.

## What is Package Management?
Tooling that enables someone who has knowledge of an application and a platform to package up an application so that someone else who has neither extensive knowledge of the application or the way it neds to be run on the platform can use it.

***Wiki-Defination***
Its a collection of software tools that automats the process of installing, upgrading, configuing and removing computer programs for a computer in a consistent manner.


Helm would be the package manager for kubernetes.
## Features of Helm:
- Package Manager for Kubernetes
- Templating Engine
- Same applications across different environments
- Release Management

==============================
- Manage complexity
- Easy updates
- Simple sharing
- Rollbacks

## Helm Charts
- Bundle of YAML Files
- Create your own helm Charts with Helm
- Push them to Helm Repository 
- Download and use existing ones

Charts are comprised of a collection of files (Mostly YAML) at well-known locations.

### Commonly used Helm Charts are:
- Database Apps such as 
    - ElasticSearch
    - MongoDB
    - Mysql
- Montoring Apps
  - Prometheus

## Syntax
helm install <chart_name>

helm search <Keywords>

Public Registries:

Private Registies:


### Template Engine 
1. Define a common blueprint
2. Dynamic values are replaced by placeholders

Here we create template files for all object those are needed. As we need to change some lines we will use place holders or sort of vraibles on these places.
We define these placeholder or variables vaule in ***values.yaml*** file.
Syntax will be 

In template yaml file
```
{{.Values.container.name}}
{{.Values.container.image}}
{{.Values.container.port}}
```
In values yaml file
```
name: my-app
container:
    name: my-app-container
    image: my-app-image
    port: 9001
```
These placeholders or varibles can be passed
- using values.yaml file or
- using --set flag in command-line

Value.yaml file will create a .Value object

Just create a template for multiple deployments and use values.yaml for the varaibles/placeholders for multiple deployments

## Same applications across different environments
To deploy a same application on different environment which have different clusters we can create our own helm chart and deploy them on dfferent different enviornments as per the needs.

## Helm Chart Directory Structure

```
mychart/
    Chart.yaml
    values.yaml
    charts/
    temlates/
        test/
        _helpers.tpl
        NOTES.txt
    ....
```
Top level mychart folder --> name of chart

Chart.yaml --> meta info about chart + containes dependencies informations.

values.yaml --> values for the template files

charts folder --> chart dependencies

_helpers.tpl --> 

test --> any file under this folder treated as testing file

NOTES.txt --> This a readme type file which helps end user to unserstand and provide details related to the chart.

Template files will be filled with the values from values.yaml

Optionally other files like Readme or licence file


# Templates + Values = vaild Kubernetes YAML

### Values injection into template files
values.yaml containes the default values which must be used by the charts.

To overrides these values we have couple of ways such as:
1. Provide another values.yaml while runing hel install command. This file will container only the values which we need to change in the template other than default values.yaml
```
    helm install --values=my-values.yaml <chartname>
```
2. Use flags in the commands or directly inject value for the vairables using set flag.
syntax:
```
    helm install --set version=2.0.0
```

## Release Management
This feature is available in version 2 of Helm as it has different architecture.

Helm version 2 has client and server relationship. In which heml CLI act as CLient and Tiller as Server which is deployed inside the Kubernetes cluster.

So when we upgrade the chart it will instead of creating new it will update the exixting deployment.

It also help in handling the rollbacks.

### Downside of Helm
- Tiller has too much power inside of K8s cluster.
- Security issues. {Requires some admin level permissions for update and deploy.}

Solves the Security Concern, but makes it more difficult to use.

***In Helm 3 Tiller got removed.***


## Helm Commands
- helm create 
- helm dependency
   - list
   - build
   - update
- helm env
- helm get
   - all
   - hooks
   - manifest
   - notes
   - values
- helm history
- helm install
- helm lint
- helm package
- helm plugin
   - install
   - list
   - uninstall
   - update
- helm push
- helm pull
- helm registry
   - login
   - logout
- helm repo
   - add
   - index
   - list
   - remove
   - update
- helm rollback
- helm search
   - hub
   - repo
- helm show
   - all
   - chart
   - crds
   - readme
   - values
- helm status
- helm template
- helm test
- helm uninstall
- helm upgarde
- helm verify
- helm Version

### Helm create command
This commands scarfolds the helm chart. 

Syntax:
```
helm create my-helm-chart
```

### Helm Dependency
Manage the dependencies of a chart.

Sub-Commands:
  - build
  - list 
  - update

**Helm Dependency build:**

This will build out the charts/ directory from the Chart.lock file.

Build is used to reconstruct a chart's dependencies to the state specified in the lock file. This will not re-negotiate dependencies, as 'helm dependency update' does.

If no lock file is found, helm dependency build' will mirror the behaviour of 'helm dependency update'.

Syntax
```
helm dependency build CHART [flags]
```

**Helm Dependency list:**

List all the dependencies declared in a chart.

Syntax:
```
helm dependency list CHART [flags]
```

**Helm Dependency update**

Update the on-disk dependencies to mirror Chart.yaml.

This command verifies that the required charts, as expressed in 'Chart.yaml', are present in 'charts/' and are at an acceptable version. It will pull down the latest charts that satisfy the dependencies and clean up old dependencies.

On successful update, it will generate a lock file that can be used to rebuild the dependencies to an exact version.

Dependencies  are not required to be represented in 'Chart.yaml'. For that reason, an update command will not remove charts unless they are (a) present in the Chart.yaml file, but at the wrong version.

Synatx:
```
helm dependency update CHART [flags]
```

### Helm env
helm client environment information.

It prints out all the environment information in use by Helm.

Syntax:
```
helm env [flags]
```

### Helm get 
Helm get command download extended information of a named release.

This command consists of multiple subcommands which can be used to get extended information  about the release, including:

- The values used to generate the release.
- The generated manifest file
- The notes provided by the chart of the release.
- The hooks associated with the release.

**Sub-commands for helm get are:**

**Helm get all**

This command prints the human readable collection of information about the notes, supplied values, and generated manifest file of the given release.

Syntax:
```
helm get all <release_name> [flag if needed]
```

**Helm get hook**

This command downloads hooks for a given release.

Hooks are formatted in YAML and seperated by the YAML '--\n'seperator.

Syntax:
```
helm get hooks <release_name> [flag if needed]
```

**Helm get manifest**

This command fetch the geenrated manifest for a given release.

A manifest is a YAML-encoded representation of the kubernetes resources that were generated from this release charts. if a chart is dependent on other charts, those resources will also be included in the manifest.

Syntax:
```
helm get manifest <relase_name> [flag if needed]
```

**Helm get notes**

This command shows notes provided by the chart of a named relase.

Syntax:
```
helm get notes <release_name> [flag if needed]
```

**Helm get values**

This commanddownloads a value file for a given release.

Syntax:
```
helm get values <release_name> [flag if needed]
```

### Helm History

Histroy  prints historical versions for a given release.

A default maximum of 256 revisions will be returned. Setting '--max' configure the maximum length of the revision list returned.

Syntax:
```
helm history <release_name> [flag if needed]
```

### Helm install
This command download the repo of provided charts and install that on kubernetes.

Syntax:
```
helm install <ChartName> 
```
Example for chat at local directory:
```
helm install myrelase ./myapp
```
Example for chat at remote directory:
```
helm install myrelease myrepo/myapp
```
Using a values file:
```
helm install myrealse ./myapp -f custom.yaml
```
Using individual key-value pair:
```
helm install myreleae ./myapp --set image.tag=master
```
Advance usage:
```
helm install myrelease ./myapp \
    -f staging.yaml \
    -f us-east-1.yaml \
    --set tracing.enable=true
```
### Helm lint
THis command takes a path to a chart and runs a series of tests to varify that the chart is well-formated.

If the linter encounters things that will cause the chart to fail installation, it will emit [ERROR] messages. If it encounters issues that break with convention or recommendation, it will emit [WARNING] messages.

Syntax:
```
helm lint PATH [flag if needed]
```
### Helm list
This command list all of the releases for a specified namespace (uses current namespace context if namespace not specified).

BY default , it lists only releases that are deployed or failed. Flags like '--uninstalled' and '--all' will alter this behavior. Such flags can be combined :

'--uninstalled --failed'

sorted alphabetically.

Use the '-d' flag to sort by release date.

If the --filterflag is provided, it will be treaded as a filter. Filters are regular expressions (Perl compatible) that are applied to the list of releases. Only items that match the filter will be returned.
```
helm list --filter 'ara[a-z]+'
```

By Default:

256 items may be return

'--max' flag for increasing or decreasing the result

'--max' = 0 will show all the results

'--max' with '--offset' will allows you to page through result.

Syntax:
```
helm list [flag if needed]
```

### Helm status
This command provides the status of installation of your chart was successful or not.

Syntax:
```
helm status myrelease
```

### Helm list
Helm has the ability to track all applications that have been installed in the cluster using Helm.

Syntax:
```
helm list
```

### Helm Upgrade
Create a new version of your release by updating either the template sources of configuration values.

Syntax:
```
helm upgrade myrelaase ./myapp --set image.tag=1.16.1-alpha
```

### Helm Rollback
Helm tracks every revison made on relaseses. When something goes wrong, revert back to a working version.

Syntax:
```
helm rollback myrelease 1
```

### Helm Delete 
Remove all Kuberentes resources from the cluster that were created as part of a release.

Syntax:
```
helm delete myrelease
```

## Helm Hub
Central location for the Charts

[Helm Hub](https://artifacthub.io/)

We can host our own helm hub with the github repo called [monocular](https://github.com/helm/monocular) But its **Obsolete**

Another way of hosting helm chart privately is [CHARTMUSEUM](https://chartmuseum.com/)

### Chartmuseum:

Chartmuseum is an open-source, easy to deploy, Helm Chart Repository sever.

### Testing a Helm chart 
To test a helm chart we can use a repo called [chart-testing](https://github.com/helm/chart-testing)

It can perform lint test and other functionality tests on provided chart.

## Migration from v2 to v3
For this Helm create a project called [helm-2to3](https://github.com/helm/helm-2to3). Its a plugin which help in migrating from helm version 2 to version 3.


## Chart Releaser
A project help in releaseing charts and called [Chart Releaser](https://github.com/helm/chart-releaser)

cr is a tool designed to help GitHub repos self-host their own chart repos by adding Helm chart artifacts to GitHub Releases named for the chart version and then creating an index.yaml file for those releases that can be hosted on GitHub Pages (or elsewhere!).

### Helm v3 feature:
- OCI Registry As Storage.

Its a feature newly invented but its not stable yet but in future version it will be.
Or might be now it will be available.




















### Important Links
[An Introduction to Helm - Matt Farina, Samsung SDS & Josh Dolitsky, Blood Orange](https://www.youtube.com/watch?v=Zzwq9FmZdsU)







# Importnt Queries
1. How to create Helm chart.
2. How to manage multi-environment in helm chart for deployment on different clusters of kubernetes.
3. How to deloy helm chart on kbernetes using below technologies:
   - Jenkins
   - ArgoCD
   - Spinnaker
   - CloudBuild
   - CloudDeploy
4. Name some private repository hubs which we can use?
5. Can Google Artifact Registry support Helm charts?
