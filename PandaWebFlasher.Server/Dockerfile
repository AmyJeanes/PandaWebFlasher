# See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

# This stage is used when running from VS in fast mode (Default for Debug configuration)
FROM mcr.microsoft.com/dotnet/aspnet:10.0@sha256:1fe86375600b62e6566b465da9553eef0621f13c67f40fe764cd8dbb1dee1497 AS base
USER $APP_UID
WORKDIR /app
EXPOSE 8080
EXPOSE 8081


# This stage is used to build the service project
FROM mcr.microsoft.com/dotnet/sdk:10.0@sha256:4ea6fe75dd36706bb6d8c3c293d4c4315840f5d76ea28ac97def77e3ec487fa5 AS with-deps
RUN apt-get update
RUN apt-get install curl -y
RUN curl -sL https://deb.nodesource.com/setup_20.x | bash
RUN apt-get -y install nodejs

FROM with-deps AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["PandaWebFlasher.Server/PandaWebFlasher.Server.csproj", "PandaWebFlasher.Server/"]
COPY ["PandaWebFlasher.Client/PandaWebFlasher.Client.esproj", "PandaWebFlasher.Client/"]
RUN dotnet restore "./PandaWebFlasher.Server/PandaWebFlasher.Server.csproj"
COPY . .
WORKDIR "/src/PandaWebFlasher.Server"
RUN dotnet build "./PandaWebFlasher.Server.csproj" -c $BUILD_CONFIGURATION -o /app/build

# This stage is used to publish the service project to be copied to the final stage
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./PandaWebFlasher.Server.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# This stage is used in production or when running from VS in regular mode (Default when not using the Debug configuration)
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "PandaWebFlasher.Server.dll"]
