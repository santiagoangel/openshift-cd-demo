*For other versions of OpenShift, follow the instructions in the corresponding branch e.g. ocp-3.10, ocp-3.9, etc

# Multi user CI/CD Demo - OpenShift Container Platform 3.10

This repository includes the infrastructure and pipeline definition for continuous delivery using Jenkins, Nexus, SonarQube and Eclipse Che on OpenShift. 

* [Introduction](#introduction)
* [Prerequisites](#prerequisites)
* [Deploy on RHPDS](#deploy-on-rhpds)
* [Automated Deploy on OpenShift](#automatic-deploy-on-openshift)
* [Manual Deploy on OpenShift](#manual-deploy-on-openshift)
* [Troubleshooting](#troubleshooting)
* [Demo Guide](#demo-guide)
* [Using Eclipse Che for Editing Code](#using-eclipse-che-for-editing-code)


## Introduction

On every pipeline execution, the code goes through the following steps:

1. Code is cloned from Gogs, built, tested and analyzed for bugs and bad patterns
2. The WAR artifact is pushed to Nexus Repository manager
3. A container image (_tasks:latest_) is built based on the _Tasks_ application WAR artifact deployed on JBoss EAP 6
4. The _Tasks_ container image is deployed in a fresh new container in DEV project
5. If tests successful, the DEV image is tagged with the application version (_tasks:7.x_) in the STAGE project
6. The staged image is deployed in a fresh new container in the STAGE project

The following diagram shows the steps included in the deployment pipeline:

![](images/pipeline.png?raw=true)

The application used in this pipeline is a JAX-RS application which is available on GitHub and is imported into Gogs during the setup process:
[https://github.com/OpenShiftDemos/openshift-tasks](https://github.com/OpenShiftDemos/openshift-tasks/tree/eap-7)

## Prerequisites
* 10+ GB memory
* JBoss EAP 7 imagestreams imported to OpenShift (see Troubleshooting section for details)


  ```
  
## userX Deploy on OpenShift
Using your current OpenShift cluster, create the following projects for CI/CD components, Dev and Stage environments:

  ```shell
  # Create Projects
  oc new-project dev-userX --display-name="Tasks - Dev"
  oc new-project stage-userX --display-name="Tasks - Stage"
  oc new-project cicd-userX --display-name="CI/CD"

  # Grant Jenkins Access to Projects
  oc policy add-role-to-user edit system:serviceaccount:cicd-userX:jenkins -n dev-userX
  oc policy add-role-to-user edit system:serviceaccount:cicd-userX:jenkins -n stage-userX
  ```  

And then deploy the demo:

  ```
  # Deploy Demo
  
  ```



  ```shell
  oc new-app -n cicd-userX -f https://raw.githubusercontent.com/santiagoangel/openshift-cd-demo/ocp-3.10/cicd-template.yaml --param DEV_PROJECT=dev-userX --param STAGE_PROJECT=stage-userX
  ```


## Troubleshooting

* If pipeline execution fails with ```error: no match for "jboss-eap70-openshift"```, import the jboss imagestreams in OpenShift.
  ```
  oc create -f https://raw.githubusercontent.com/openshift/library/master/official/eap/imagestreams/jboss-eap70-openshift-rhel7.json -n openshift
  ```
* If Maven fails with `/opt/rh/rh-maven33/root/usr/bin/mvn: line 9:   298 Killed` (e.g. during static analysis), you are running out of memory and need more memory for OpenShift.

## Demo Guide


* Take note of these credentials and then follow the demo guide below:

  * Gogs: `gogs/gogs`
  * Nexus: `admin/admin123`
  * SonarQube: `admin/admin`

* A Jenkins pipeline is pre-configured which clones Tasks application source code from Gogs (running on OpenShift), builds, deploys and promotes the result through the deployment pipeline. In the CI/CD project, click on _Builds_ and then _Pipelines_ to see the list of defined pipelines.

    Click on _tasks-pipeline_ and _Configuration_ and explore the pipeline definition.

    You can also explore the pipeline job in Jenkins by clicking on the Jenkins route url, logging in with the OpenShift credentials and clicking on _tasks-pipeline_ and _Configure_.

* Run an instance of the pipeline by starting the _tasks-pipeline_ in OpenShift or Jenkins.

* During pipeline execution, verify a new Jenkins slave pod is created within _CI/CD_ project to execute the pipeline.

* Pipelines pauses at _Deploy STAGE_ for approval in order to promote the build to the STAGE environment. Click on this step on the pipeline and then _Promote_.

* After pipeline completion, demonstrate the following:
  * Explore the _snapshots_ repository in Nexus and verify _openshift-tasks_ is pushed to the repository
  * Explore SonarQube and show the metrics, stats, code coverage, etc
  * Explore _Tasks - Dev_ project in OpenShift console and verify the application is deployed in the DEV environment
  * Explore _Tasks - Stage_ project in OpenShift console and verify the application is deployed in the STAGE environment  

![](images/sonarqube-analysis.png?raw=true)

* Clone and checkout the _eap-7_ branch of the _openshift-tasks_ git repository and using an IDE (e.g. JBoss Developer Studio), remove the ```@Ignore``` annotation from ```src/test/java/org/jboss/as/quickstarts/tasksrs/service/UserResourceTest.java``` test methods to enable the unit tests. Commit and push to the git repo.

* Check out Jenkins, a pipeline instance is created and is being executed. The pipeline will fail during unit tests due to the enabled unit test.

* Check out the failed unit and test ```src/test/java/org/jboss/as/quickstarts/tasksrs/service/UserResourceTest.java``` and run it in the IDE.

* Fix the test by modifying ```src/main/java/org/jboss/as/quickstarts/tasksrs/service/UserResource.java``` and uncommenting the sort function in _getUsers_ method.

* Run the unit test in the IDE. The unit test runs green. 

* Commit and push the fix to the git repository and verify a pipeline instance is created in Jenkins and executes successfully.

![](images/openshift-pipeline.png?raw=true)

## Using Eclipse Che for Editing Code

An [Eclipse Che](https://www.eclipse.org/che/) instances will be deployed within the CI/CD project which allows you to use the Eclipse Che web-based IDE for editing code in this demo.

In your workspace, click on **Import Project...** in order to import the `openshift-tasks` Gogs repository into your workspace.

![](images/che-import-project.png?raw=true)

Enter the Gogs repository HTTPS url for `openshift-tasks` as the Git repository url with Git username and password in the 
url: <br/>
`http://gogs:gogs@[gogs-hostname]/gogs/openshift-tasks.git`

 You can find the repository url in Gogs web console. Make sure the check the **Branch** field and enter `eap-7` in order to clone the `eap-7` branch which is used in this demo. Click on **Import**

![](images/che-import-git.png?raw=true)

Change the project configuration to  **Maven** and then click **Save**

![](images/che-import-maven.png?raw=true)

Configure you name and email to be stamped on your Git commity by going to **Profile > Preferences > Git > Committer**.

![](images/che-configure-git-name.png?raw=true)

Follow the steps 6-10 in the above guide to edit the code in your workspace. 

![](images/che-edit-file.png?raw=true)

In order to run the unit tests within Eclipse Che, wait till all dependencies resolve first. To make sure they are resolved, run a Maven build using the commands palette icon or by clicking on **Run > Commands Palette > build**. 

Make sure you run the build again, after fixing the bug in the service class.

Run the unit tests in the IDE after you have corrected the issue by right clicking on the unit test class and then **Run Test > Run JUnit Test**

![](images/che-run-tests.png?raw=true)

![](images/che-junit-success.png?raw=true)


Click on **Git > Commit** to commit the changes to the `openshift-tasks` git repository. Make sure **Push committed changes to ...** is checked. Click on **Commit** button.

![](images/che-commit.png?raw=true)

As soon the changes are committed to the git repository, a new instances of pipeline gets triggers to test and deploy the 
code changes.
