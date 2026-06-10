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

### Notes

- SonarQube analysis is performed during the pipeline.
- No local Sonar installation is required in this repo.