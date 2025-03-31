# Containerize a .NET 8 Web API Application with Docker and Upload/Download the Image to/from Docker Hub

## 0. Prerequisites
Before getting started, ensure that you have the following installed:
- **Docker Desktop**: Install it from [Docker's official site](https://www.docker.com/products/docker-desktop/)
- **Docker Hub Account**: Create an account at [Docker Hub](https://hub.docker.com/)
- **Visual Studio 2022 Community Edition**: Download it from [Microsoft's official site](https://visualstudio.microsoft.com/)

## 1. Create a .NET 8 Web API in Visual Studio 2022 Community
**Important:** Install and run Docker Desktop before creating the application in Visual Studio 2022.

![image](https://github.com/user-attachments/assets/e335b5ff-05c4-4fcc-b91b-ec2049146c2a)

1. Open **Visual Studio 2022** and select **Create a new project**.

![image](https://github.com/user-attachments/assets/6e0cdd02-92a4-4355-b3f2-3603ba07dfb4)

2. Choose **ASP.NET Core Web API** and click **Next**.

![image](https://github.com/user-attachments/assets/110d6241-b30b-4d1a-871c-0af4ee1a5275)

3. Configure your project:
   - Name: `GcpDeployment`
   - Location: Choose your desired folder
   - Click **Next**

![image](https://github.com/user-attachments/assets/3b48e3bc-dcd8-4d86-a178-4e138c37096c)

4. Select **.NET 8.0 (Long-term support)** and click **Create**.

![image](https://github.com/user-attachments/assets/fb624c02-ecc5-4b0e-8626-b4783eb3e644)

## 2. Add Docker Support to the .NET 8 Web API Project
1. Right-click on the project in **Solution Explorer**.
2. Select **Add > Docker Support**.

![image](https://github.com/user-attachments/assets/91da0486-2e04-4b90-9582-680168456bff)
   
3. Choose **Linux** as the target OS.

![image](https://github.com/user-attachments/assets/9c3a4fbc-8506-4cbf-b896-69af0e440a81)

4. Visual Studio will generate a `Dockerfile` in the project.

   ![image](https://github.com/user-attachments/assets/91c33b34-6eac-418a-8c96-bf3b6a380013)


```sh
# See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

# This stage is used when running from VS in fast mode (Default for Debug configuration)
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
USER $APP_UID
WORKDIR /app
EXPOSE 8080
EXPOSE 8081


# This stage is used to build the service project
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["GcpDeployment.csproj", "."]
RUN dotnet restore "./././GcpDeployment.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "./GcpDeployment.csproj" -c $BUILD_CONFIGURATION -o /app/build

# This stage is used to publish the service project to be copied to the final stage
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./GcpDeployment.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# This stage is used in production or when running from VS in regular mode (Default when not using the Debug configuration)
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "GcpDeployment.dll"]
```

## 3. Build and Run the Docker Container Locally
1. Open a terminal or command prompt and navigate to the project directory.
2. Build the Docker image:
   ```sh
   docker build -t my-dotnet8-api .
   ```
3. Run the container:
   ```sh
   docker run -p 8080:80 my-dotnet8-api
   ```
4. Open `http://localhost:8080/swagger` in a browser to test the API.

## 4. Push the Docker Image to Docker Hub
1. Log in to Docker Hub:
   ```sh
   docker login
   ```
2. Tag the image with your Docker Hub username:
   ```sh
   docker tag my-dotnet8-api <your-docker-hub-username>/my-dotnet8-api
   ```
3. Push the image to Docker Hub:
   ```sh
   docker push <your-docker-hub-username>/my-dotnet8-api
   ```

## 5. Pull and Run the Image from Docker Hub
1. Pull the image:
   ```sh
   docker pull <your-docker-hub-username>/my-dotnet8-api
   ```
2. Run the container:
   ```sh
   docker run -p 8080:80 <your-docker-hub-username>/my-dotnet8-api
   ```
3. Verify the API by opening `http://localhost:8080/swagger` in a browser.

## 6. Cleanup (Optional)
To remove the container and image:
```sh
docker rm -f $(docker ps -aq)
docker rmi $(docker images -q)
```

## Conclusion
You have successfully containerized a .NET 8 Web API application, uploaded it to Docker Hub, and pulled it back for deployment. Happy coding!

