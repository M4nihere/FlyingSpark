# Creating Azure Pipeline from Bitbucket

### Prerequisites for the project

<ul>
<li>Bitcbuket Account
<li>Azure DevOps Account
</ul>

### Setup Bitbucket Repository 

<ul>
<li>First Push your code repository on bitbucket.
<li>Then create a azure-pipelines.yml file.
</ul>

#### Note : File name can be anything suitable to you but it should be in "yml" format.

### Azure pipeline that I used in my project :
#### azure-pipeline.yml
```yaml
trigger:
- main

pr:
- main

stages:
- stage: Build
  jobs:
  - job: BuildJob
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.x'
        addToPath: true
    - script: |
        pip install azure-devops
        python -m pip install --upgrade pip
      displayName: 'Install Python and Azure DevOps Python module'
    - checkout: self
    - script: echo $(ls)
    - script: echo $(pwd)
    - script: mkdir ~/.m2 && cp -rf settings.xml ~/.m2/
      displayName: "copying settings"
    - script: echo Building Mule project
      displayName: 'Building Mule project'
    - script: mvn clean package -DmavenTestSkip=true
      displayName: 'Maven Build'
    - task: CopyFiles@2
      inputs:
        sourceFolder: '$(System.DefaultWorkingDirectory)'
        contents: '**/*.jar'
        targetFolder: '$(Build.ArtifactStagingDirectory)'
    - script: ls $(Build.ArtifactStagingDirectory)
      displayName: 'List Files in Artifact Staging Directory'

    - script: mvn -B verify --file pom.xml
      displayName: 'Maven Verify'

    - script: mvn clean install
      displayName: 'Maven Install'

    - script: mvn clean package
      displayName: 'Maven Package'

    - script: mvn test
      displayName: 'Maven Test'
    - script: mvn deploy 
      displayName: 'Maven Deploy'

    - task: CopyFiles@2
      inputs:
        sourceFolder: '$(System.DefaultWorkingDirectory)'
        # contents: '**/*.jar'
        targetFolder: '$(Build.ArtifactStagingDirectory)'
    - upload: $(Build.ArtifactStagingDirectory)
      artifact: azure-devops
