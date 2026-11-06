# See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

# This stage is used when running from VS in fast mode (Default for Debug configuration)
FROM mcr.microsoft.com/dotnet/aspnet:11.0@sha256:e1ba5ef29b9c0f51b4236cbd25268b84a1e15f8fb1dca21f141bd0358903a1f2 AS base
USER $APP_UID
WORKDIR /app
EXPOSE 8080
EXPOSE 8081


# This stage is used to build the service project
FROM mcr.microsoft.com/dotnet/sdk:11.0@sha256:59d092a889fbad09c1a0b0c5dd072c6315125b0294a5cddfa48ec892f63b2ab3 AS with-deps
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
