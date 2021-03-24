# AZ304-Day2Worksop - Serverless architecture

## Case Study Location
https://github.com/microsoft/MCW-Serverless-architecture/blob/master/Whiteboard%20design%20session/WDS%20student%20guide%20-%20Serverless%20architecture.md

## Team Solution

![Alt text here](https://github.com/narenvivek/AZ304-Day2Worksop/blob/d4c77d11b56477fe55ad2da7b1ffb8c6fd665268/AZ-304%20Day%202%20Workshop.png)

1. Azure API Management exposes a POST method to intake images of Number Plates produced by the camera

2. (using a small two-step App Logic) the image file is stored on a Blob storage (Create Blob uses today's date to store image files into)

3. Azure Functions / Azure App Logic picks up the image an submits the image to Azure OCR to Text (Azure Cognitive Services)

![image](https://user-images.githubusercontent.com/29542480/112323000-c1831900-8c87-11eb-8dfd-395cb6da0dfa.png)

![image](https://user-images.githubusercontent.com/29542480/112323193-ef685d80-8c87-11eb-9601-606235e19359.png)

4. Irrespective of success of converting OCR to text or failure, events would be sent to Event Grid topic to indicate presence of new images on the Blob that is being serviced (doing this before OCR to text is also a design option). This will ensure extensibility of design for other applications to subscribe to the topic so that autonomous, parallel processing is possible and is loosly coupled.

5. The next step in the Logic App (or Azure Functions) is to ensure that the semi-structured data is pushed into Cosmos DB.

6. Images that cannot be processed using OCR to text is pushed to failed processing blob (there could be more optimal ways for this) which could be picked-up by a human operator to manually process the images

## Application Indights
Due to the objection raised on possible latency of processing, Application Insights (backed by ALA workspace) and Azure Monitor alerts enable operations team to monitor performance and tune the application accordingly.

## Azure DevOps (ADO)
ADO provides code repository (Repo) for the entire solution along with CI/CD pipelines for incremental deployments. Repos could be seperated for:

1. ARM templates (for Infrastructure-as-code) - setup with working service connection to the subscription
2. Azure Functions (discrete Visual Studio solutions)
3. Azure Blueprints (for policy enforcement and RBAC role assignments)
4. API versioning and management
5. WebApp code

etc.

Due to loosely coupled architecture, it is possible to deploy these components seperately either using Azure Pipelines or Release Pipelines (that faclilate staged deployments into Dev-Test, Pre-prod and Prod)

ADO could also become the centerpiece for work-item management and Agile Boards.

## Security
1. API management security could include: rate limiting (number of invokations of API) to mitigate against DDoS type of attacks, response policies to delete backend application URLs and various other HTTP header manipulations to keep the front end secure. If cost allows, Azure Front Door could frontend the APIs and/or web services to ensure increased threat protection.
2. Azure Key Vault: All secrets (such as connection strings) and keys (API keys) could be stored and fetched programmatically from AKV for all the applications
3. AAD Machine Identities: Managed Identities and/or Service Principals to use least privilege.
