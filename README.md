# Containerize a .NET 8 Web API Application with Docker

## 0. Prerequisites
Before getting started, ensure that you have the following installed:
- **Docker Desktop**: Install it from [Docker's official site](https://www.docker.com/products/docker-desktop/)
- **Visual Studio 2022 Community Edition**: Download it from [Microsoft's official site](https://visualstudio.microsoft.com/)

## 1. Create a .NET 8 Web API in Visual Studio 2022 Community
**Important:** Install and run Docker Desktop before creating the application in Visual Studio 2022.

![image](https://github.com/user-attachments/assets/15cbb403-084b-4884-8a7d-1ccca714edb4)

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
   
5. Configure your Program.cs file as below.

   ```sh
   
   using Microsoft.AspNetCore.Builder;
   using Microsoft.Extensions.Hosting;

   var builder = WebApplication.CreateBuilder(args);

   // Add services to the container.
   builder.Services.AddControllers();
   // Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
   builder.Services.AddEndpointsApiExplorer();
   builder.Services.AddSwaggerGen();

   var app = builder.Build();

   // Enable Swagger in all environments
   app.UseSwagger();
   app.UseSwaggerUI(c =>
   {
       c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1");
       c.RoutePrefix = "swagger"; // Make Swagger UI available at "/"
   });

   app.UseHttpsRedirection();

   app.UseAuthorization();
   app.MapControllers();

   app.Run();

   ```


## 2. Add Docker Support to the .NET 8 Web API Project
1. Right-click on the project in **Solution Explorer**.
2. Select **Add > Docker Support**.

![image](https://github.com/user-attachments/assets/4b33af54-e8be-4f9d-ace6-5057413e3416)
   
3. Choose **Linux** as the target OS.

![image](https://github.com/user-attachments/assets/3ef0517a-6b7a-424c-80d7-ceb9fdeadf0c)

4. Visual Studio will generate a `Dockerfile` in the project.

![image](https://github.com/user-attachments/assets/d230e998-e961-4d5b-b70c-b159ca580cd2)

This is the DockerFile source code.

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
   docker build -t gcp-deployment .
   ```
3. Run the container:
   ```sh
   docker run -p 8080:80 --name gcp-deployment-container gcp-deployment
   ```
4. Open `http://localhost:8080/swagger` in a browser to test the API.

5. Rebuild & Restart Docker Container (If not working)

```sh
docker build --no-cache -t gcp-deployment:latest .
```

🔹 docker build: Builds a new Docker image.
🔹 Image Name: gcp-deployment:latest
🔹 .: Current directory, where the Dockerfile is located.

```sh
docker stop gcp-deployment-container
docker rm gcp-deployment-container
```

🔹 docker stop gcp-deployment-container: Stops the running container named gcp-deployment-container.
🔹 docker rm gcp-deployment-container: Removes (deletes) the stopped container.
🔹 Container Name: gcp-deployment-container

```sh
docker run -d -p 8080:8080 --name gcp-deployment-container gcp-deployment:latest
```

🔹 docker run: Starts a new container.
🔹 -d: Runs the container in detached mode (in the background).
🔹 -p 8080:8080: Maps port 8080 on the host machine to port 8080 inside the container.
🔹 --name gcp-deployment-container: Assigns the name gcp-deployment-container to the running container.
🔹 gcpdeployment:latest: Specifies the image to use.



6. Try accessing Swagger again:

`http://localhost:8080/swagger` or `http://localhost:8080/WeatherForecast`

![image](https://github.com/user-attachments/assets/b8d039ed-8501-41ad-b6e8-0e00a41e2b93)

## Conclusion
I have successfully containerized a .NET 8 Web API application.

