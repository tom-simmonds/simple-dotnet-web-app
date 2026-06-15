# Simple .NET Web App

For learning Jenkins.

Project structure:

* Web Api Project - [SimleWebApi](./SimpleWebApi/)
* Test Project - [SimpleWebApi.Test](./SimpleWebApi.Test/)
* Solution - [Jenkisfile](./jenkins/Jenkinsfile).

The solution provided only compares the starter and finished projects.

## License

MIT

## CI/CD Setup

This project is built using Jenkins running in a Docker container.

### Build Environment

The Jenkins instance uses a custom Docker image that includes:

- .NET SDK 8.0
- SonarScanner for .NET (`dotnet-sonarscanner`)

The Dockerfile for this image is located in:

../jenkins-dotnet/dockerfile

- Jenkins container (jenkins-dotnet-sonar) runs on port 8080
- SonarQube (sonarqube) Community Edition container runs on port 9000
- Both containers run on a custom Docker bridge network (devops-net), allowing container-to-container communication by container name rather than IP or localhost.
- jenkins_home is the Docker volume which preserves Jenkins configuration, jobs, plugins, and credentials across container rebuilds

### Notes

- SonarQube analysis is performed during the pipeline.
- No local Sonar installation is required in this repo.

### Custom Jenkins image

A new jenkins image (jenkins-dotnet-sonar) was built on top of jenkins/jenkins:lts, adding the following:
- .NET SDK 8 installed with official Microsoft script
- libicu-dev required by .NET
- dotnet-sonarscanner installed as a global .NET tool, run as the jenkins user
- PATH/DOTNET_ROOT updated so dotnet and sonarscanner global tool is available to Jenkins

I have chosen to bake tools into the image rather than install at pipeline runtime. This ensures immutable infrastructure.

### SonarQube Configuration

- Project created in SonarQube with project key simple-dotnet-web-app
- User token generated for analysis authentication
- Webhook configured. Required so SonarQube can notify Jenkins when analysis is complete - without this, the Quality Gate stage hangs indefinitely

### Jenkins Configuration 

- SonarQube plugin installed
- Sonarqube server registered under Manage Jenkins -> System:
  - Name: SonarQube
  - Server URL: http://sonarqube:9000
  - Authentication token: stored as a Secret Text credential (sonar-token)

Using withSonarQubeEnv('SonarQube') in the Jenkinsfile means the server URL and token are injected automatically from this configuration. No need to input token in the pipeline code itself.


For my own benefit. How SonarQube works, is it wraps the build. This is done in three phases:

<img width="768" height="348" alt="image" src="https://github.com/user-attachments/assets/0ab9fb84-be79-4c48-98ff-d438afd63a97" />

Begin - hooks into the compiler, starts collecting code metrics and information
Build - compilation
End - packages everything collected and uploads it to the SonarQube server for analysis

/k: is the project key - telling SonarQube which project these results belong to. 
Quality gate is a set of pass/fail conditions definited in SonarQube.
waitForQualityGate abortPipeline:true pauses the pipeline, waits for SonarQube's webhook callback with the result and fails the build if the gate isn't passed.

### Result

Pipeline runs end-to-end successfully:

<img width="1846" height="591" alt="image" src="https://github.com/user-attachments/assets/b0b2c304-0288-42b6-a17a-8fead7b71e86" />

SonarQube analysis output:

<img width="1894" height="805" alt="image" src="https://github.com/user-attachments/assets/4a362c61-6b33-47cd-914e-cb25ab176670" />

Key takeaways: --no-cache is essential when a Dockerfile change doesn't appear to take effect. 
Services on the same custom Docker network communicate via container name.
SonarQube Jenkins integration requires a webhook
Dockerfile config. ROOT and USER complexities.
