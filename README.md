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


#### Change github credentials to Service Principal

1. Create an *App Registration* in your **Azure AD**
2. Under Cert & Secrets create a new *Client Secret* (Copy the secret **VALUE** into notepad)
3. Get the ClientId from the *App Registration*
4. Get your TenantId (AAD Id) and your Azure SubscriptionId
5. Create *JSON* in notepad etc.

```json

{
    "clientId": "...",
    "clientSecret": "...",
    "subscriptionId": "...",
    "tenantId": "..."
}

```

6. Grant your new *App Reg* the *Contributor* role on the **Resource Group** holding your Web App
7. Change the Action to (variable changed might be needed)

```yaml

name: newactions

on:
  push:
    branches:
      - main

env:
  SOLUTION_PATH: ./githubgraph
  PROJECT_PATH: ./githubgraph/githubgraph
  BUILD_CONFIG: Release
  PUBLISH_PATH: ./apppath
  APPSERVICE_NAME: githubgraph

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: restore
        run: dotnet restore ${{ env.SOLUTION_PATH }}
      - name: build
        run: dotnet build ${{ env.SOLUTION_PATH }} --configuration ${{ env.BUILD_CONFIG }} --no-restore
      - name: publish
        run: dotnet publish ${{ env.SOLUTION_PATH }} --configuration ${{ env.BUILD_CONFIG }} --output ${{ env.PUBLISH_PATH }}
      - name: upload
        uses: actions/upload-artifact@v2
        with:
          name: artifact
          path: ${{ env.PUBLISH_PATH }}
  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: download
        uses: actions/download-artifact@v2
        with:
          name: artifact
          path: ${{ env.PUBLISH_PATH }}
      - uses: azure/login@v1
        with:
          creds: ${{ secrets.PRINCIPAL }} 
      - name: deploy web app
        uses: azure/webapps-deploy@v2
        with: 
          app-name: ${{ env.APPSERVICE_NAME }} 
          package: ${{ env.PUBLISH_PATH }}

```

8. Create a new Github *secret* called **PRINCIPAL** and paste the *JSON* into the value. Also delete the old secret holding the *Publish Profile*

9. Make a change to your web app and push the changes, follow the action execution and verify that the changes were deployed



### Use Key Vault in Pipeline

1. Create Key Vault in Azure
2. Create a Secret with a value, verify that you can see that value afterwards
3. Change Access Control to RBAC
4. Verify that you no longer can access the secret
5. Give yourself Key Vault Admin Role on the Key Vault
6. Verify that you now can see the secret again

7. Clone the repo
8. create an *azure-pipelines.yaml*

```yaml

trigger:
  - master

pool:
  vmImage: ubuntu-latest

variables:
  keyVaultName: devopsvaultmlc

steps:
  - task: AzureKeyVault@2
    inputs:
      azureSubscription: $(thesubscription)
      KeyVaultName: $(keyVaultName)
      SecretsFilter: '*'
      RunAsPreJob: false
  - task: CmdLine@2
    inputs:
      script: 'echo $(ftppassword) > secret.txt'
    
  - task: CopyFiles@2
    inputs:
      Contents: secret.txt
      targetFolder: '$(Build.ArtifactStagingDirectory)'
    
  - task: PublishBuildArtifacts@1
    inputs:
      PathtoPublish: '$(Build.ArtifactStagingDirectory)'
      ArtifactName: 'drop'
      publishLocation: 'Container'

     

```


10. Push
11. Create new Pipeline point to .yaml file
12. create a variable in the Pipeline (thesubscription -> [name of your Service Connection]) 
13. Run Pipeline (error!!!)

