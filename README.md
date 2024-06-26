# Dotnet Isolated Azure Function Dapr Extension docker image

This project addresses an issue with building a .NET isolated Azure Function Docker image with Dapr, where it looks for `Microsoft.Azure.WebJobs.Extensions.Dapr` with version `>= 99.99.99`.

## Solution

To fix this issue, you need to create a folder called `local-packages` in the root directory of the project and create a `nuget.config` file to let the dotnet build use the local NuGet package.

### nuget.config

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
    <add key="nuget.local" value="local-packages" />
  </packageSources>
</configuration>
```

## Dockerfile

```Dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS installer-env

COPY . /src/dotnet-function-app
RUN cd /src/dotnet-function-app && \
mkdir -p /home/site/wwwroot && \
dotnet publish *.csproj --configfile nuget.config --output /home/site/wwwroot

# To enable ssh & remote debugging on app service change the base image to the one below
# FROM mcr.microsoft.com/azure-functions/dotnet-isolated:4.0-dotnet-isolated8.0-appservice
FROM mcr.microsoft.com/azure-functions/dotnet-isolated:4.0-dotnet-isolated8.0
ENV AzureWebJobsScriptRoot=/home/site/wwwroot \
    AzureFunctionsJobHost__Logging__Console__IsEnabled=true

COPY --from=installer-env ["/home/site/wwwroot", "/home/site/wwwroot"]
```

## Docker Build and Push
To build the Docker image:
```sh
docker build -t mdashique/dotnet-isolated-func-dapr:0.1 .
```

To push the Docker image:
```sh
docker push docker.io/mdashique/dotnet-isolated-func-dapr:0.1
```

[This](https://hub.docker.com/repository/docker/mdashique/dotnet-isolated-func-dapr/general) is the sample docker image for Dotnet Isolated Azure Function Dapr Extension.

## Conclusion
By following these steps, you should be able to build and push a .NET isolated Azure Function Docker image that correctly references the Microsoft.Azure.WebJobs.Extensions.Dapr package.

