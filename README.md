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


### Github action

```yaml

name: thefirstworkflow

on:
  push:
    branches: [ "main" ]


env:
  WEBPROJECT_PATH: ./thegithubwebapp
  DOTNET_VERSION: 6.0.x
  PUBLISH_FOLDER: './publishapp'
  APP_NAME: devopsteknologisk
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Hello world
        run: echo 'Hello World!!'
      - name: Setup dotnet
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}
      - name: Install dependency
        run: dotnet restore ${{ env.WEBPROJECT_PATH }}
      - name: Build
        run: dotnet build ${{ env.WEBPROJECT_PATH }} --configuration Release --no-restore
      - name: Publish
        run: dotnet publish ${{ env.WEBPROJECT_PATH }} --configuration Release -o ${{env.PUBLISH_FOLDER}}
      - name: Deploy
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${{ env.APP_NAME }}
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          package: ${{ env.PUBLISH_FOLDER }}

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


#### Deploy github action

1. New github repo if needed
2. Create web project (or solution and project)
3. Create an action (.github/workflows/pipelinename.yaml) -> Paste sample in (correct env when needed)
4. Push (error!!!)


5. Goto Azure Web App (Overview)
6. Download **Publish Profile**
7. Create new *github* secret (settings->secrets)
8. Name: AZURE_WEBAPP_PUBLISH_PROFILE value: the content of the downloaded **Publish Profile**
9. Re-run pipeline