# devops




### Web App Azure Pipeline


```yaml

trigger:
- master

pool:
  vmImage: ubuntu-latest

variables:
  solutionPath: './mywebapp'
  webappPath: './mywebapp/mywebapp/mywebapp.csproj'
  appName : 'devopsteknologisk'

stages:
- stage: dev
  jobs:
  - job: Build_Test_Publish
    steps:
      - script: dotnet build $(solutionPath)
      - script: dotnet test $(solutionPath)
      - task: DotNetCoreCLI@2
        inputs:
          projects: '$(webappPath)'
          command: 'publish'
          publishWebProjects: true
      - task: AzureWebApp@1
        displayName: 'Publish to web site'
        inputs:
          azureSubscription: $(mysubscription)
          appName: $(appName)
          package: '$(System.DefaultWorkingDirectory)/**/*.zip'


```

### Exercises

#### Deploy Azure Web App

1. Create new Resource Group in Azure
2. Create an App Service Plan in the Resource Group (west europe)
3. Create a new Web App (App Service) place in Plan

4. Create a web project in c#
   - dotnet new sln -o [name]
   - in the solution folder
   - dotnet new gitignore
   - dotnet new webapp -o [name]
   - dotnet build

5. In Root pipeline (azure-pipelines.yaml) (copy code, change variables where needed)
6. Add, Commit, Push check Pipeline starter (error)

7. Project Settings -> Service Connection  (ARM Azure Resource Manager, Recommended (automatic) (Subscription, allow all pipelines, name!)
8. Pipeline -> EDIT -> Variables (mysubscription -> name!)

9. Run Pipeline Again
