# Creating a Build/CI pipeline for java app

## Introduction

The objective of this workshop is to create a Build pipeline for web app that uses Java and Maven. We'll take a sample application from Github. The pipeline will be created in Azure DevOps using the YAML syntax.

---

### Prerequisites:
Azure DevOps.

---

### 1. Connect to the repo.
Either to connect to a Github repo or just import the Code files to the Repos in Azure DevOps project.

---

### 2. Create the YAML pipeline for Build 

Go to pipelines and select "New pipeline"
Specify where the code resides.
Select the starter pipeline, the started pipeline should look like this :
```yaml
# Starter pipeline
# Start with a minimal pipeline that you can customize to build and deploy your code.
# Add steps that build, run tests, deploy, and more:
# https://aka.ms/yaml
trigger:
- main
pool:
  vmImage: ubuntu-latest
steps:
- script: echo Hello, world!
  displayName: 'Run a one-line script'
- script: |
    echo Add other tasks to build, test, and deploy your project.
    echo See https://aka.ms/yaml
  displayName: 'Run a multi-line script'
```

It's a sample pipeline that will be triggered each time we have a new 'Git Commit' on the main branch.
It will rune on a hosted agent using an Ubuntu machine. The steps section describes what will run in the pipeline. Two steps here: both will run scripts for showing a message on the console.

---


### 3. change the YAML pipeline to build the Java app with Maven.

Edit the azure-pipelines.yaml to build the Java app using Maven. Simply replace the content with this new one:
```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- task: JavaToolInstaller@0
  inputs:
    versionSpec: '8'
    jdkArchitectureOption: 'x64'
    jdkSourceOption: 'PreInstalled'
- task: Maven@4
  displayName: 'Maven Build and Test'
  inputs:
    mavenPomFile: 'pom.xml'
    options: '-DskipITs --settings ./maven/settings.xml'
    publishJUnitResults: true
    testResultsFiles: '**/surefire-reports/TEST-*.xml'
    testRunTitle: 'Test Run'

- task: CopyFiles@2
  displayName: 'Copy Files'
  inputs:
    SourceFolder: '$(build.sourcesdirectory)'
    Contents: |
      **/target/*.war
      *.sql
    TargetFolder: '$(build.artifactstagingdirectory)'
    
- task: PublishBuildArtifacts@1
  displayName: 'Publish Artifact'
  inputs:
    PathtoPublish: '$(build.artifactstagingdirectory)'
```

### 4. Tasks:
there are 4 tasks here.

1. JavaToolInstaller@0: This task ensures the right Java Development Kit (JDK) version is set up on the build agent.
Version Specification: Here, you’ve set Java 8 (versionSpec: '8'), which is essential if your application or dependencies are Java 8-compatible.
Consideration: Verify that your Java application is compatible with Java 8; otherwise, update the version spec accordingly.

2. Maven@4: This task handles Maven commands, including build, test, and any other specified options.
Maven POM File: The pipeline uses pom.xml, the project’s configuration file, to resolve dependencies and execute tests.
Options: The -DskipITs flag skips integration tests, which could be useful if you're focusing on unit tests only or aiming to shorten the pipeline duration. Be sure that skipping integration tests aligns with your CI/CD testing requirements.
JUnit Results: Setting publishJUnitResults to true ensures test results are available in Azure DevOps. testResultsFiles: '**/surefire-reports/TEST-*.xml' specifies the location of the test reports, allowing the pipeline to publish test results automatically.
Consideration: Check if integration tests need to run in this stage; otherwise, consider adding a separate stage or pipeline for them.

3. CopyFiles@2: This task copies output files (e.g., .war files and SQL files) to the specified target directory.
Source and Target Folders: $(build.sourcesdirectory) is the default directory for source code on the agent, and $(build.artifactstagingdirectory) is used to store artifacts before publishing.
Consideration: Ensure the Contents parameter includes all necessary files for deployment, particularly any configuration or environment files needed in production.


4. PublishBuildArtifacts@1: This task publishes artifacts to Azure DevOps, making them available for further stages, such as deployment.
Artifact Staging: The PathtoPublish parameter points to $(build.artifactstagingdirectory), where files have been staged in the previous step.
Consideration: Validate that your staging directory contains all the essential files for a seamless deployment process. You may also want to configure retention policies on artifacts.