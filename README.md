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

![image](https://github.com/user-attachments/assets/1f8b45fc-c682-4984-aeca-5ce99d6ed07b)

2. Choose **ASP.NET Core Web API** and click **Next**.




3. Configure your project:
   - Name: `DotNet8WebAPI`
   - Location: Choose your desired folder
   - Click **Next**
4. Select **.NET 8.0 (Long-term support)** and click **Create**.

## 2. Add Docker Support to the .NET 8 Web API Project
1. Right-click on the project in **Solution Explorer**.
2. Select **Add > Docker Support**.
3. Choose **Linux** as the target OS.
4. Visual Studio will generate a `Dockerfile` in the project.

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

