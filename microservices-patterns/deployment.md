## Chapter 12: Deploying Microservices
Chapter 12 addresses the operational side of microservices. Because you are now managing dozens or hundreds of service instances, manual deployment is impossible. You need a highly automated deployment pipeline and a robust runtime environment.
### 12.1 The Patterns of Deployment
Richardson outlines several ways to package and run your services:
 * **Language-specific packaging:** (e.g., JAR files or NuGet packages). Fast to build but lacks isolation and makes it hard to limit resource usage (CPU/RAM) between services on the same host.
 * **Service as a Container:** (The industry standard). Using **Docker** to package the service with its dependencies. This ensures that "it works on my machine" translates to production.
 * **Service as a Virtual Machine:** Deploying each service as its own VM (e.g., AWS EC2). High isolation but slow to start and heavy on resources.
 * **Serverless Deployment:** (e.g., AWS Lambda, Azure Functions). You only upload the code, and the cloud provider handles scaling and execution.
### 12.2 Orchestration with Kubernetes
When you have hundreds of containers, you need an orchestrator. **Kubernetes (K8s)** is the primary tool discussed for:
 * **Resource Management:** Deciding which physical server has room for a new container.
 * **Self-healing:** Restarting containers that fail their health checks.
 * **Service Discovery:** Providing a stable DNS name for a shifting group of containers.
## Technical Examples: Infrastructure as Code
### C# Example: Dockerizing a .NET Service
To deploy a C# service, you create a Dockerfile. This ensures the environment (SDK, runtime) is identical everywhere.
```dockerfile
# Build Stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["OrderService.csproj", "./"]
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app

# Run Stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "OrderService.dll"]

```
### Python Example: Serverless Function (Azure Functions)
Python is a favorite for Serverless/FaaS (Function as a Service). Here is a simple entry point for an Order processing function.
```python
import azure.functions as func
import logging
import json

def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    try:
        req_body = req.get_json()
        order_id = req_body.get('order_id')
        
        # Logic to process order...
        
        return func.HttpResponse(f"Order {order_id} processed successfully", status_code=200)
    except ValueError:
        return func.HttpResponse("Invalid request body", status_code=400)

```
## Key Takeaways
 1. **Immutability:** Once a container image is built, it should never be changed. If you need to update code, build a *new* image.
 2. **Service Mesh:** For very large systems, consider a Service Mesh (like Istio or Linkerd) to handle service-to-service communication, security, and metrics automatically.
 3. **Deployment Pipelines:** Use CI/CD tools (GitHub Actions, Jenkins, Azure DevOps) to ensure that code is automatically tested, containerized, and deployed to Kubernetes.
