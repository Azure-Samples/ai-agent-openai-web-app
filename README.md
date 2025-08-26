# Fashion Assistant Web App on Azure App Service with Azure AI Agent and OpenAPI Specified Tool

This sample contains a .NET Blazor web application that uses Azure AI Agents to create an interactive fashion shopping experience. The application demonstrates how to connect a web app to an Azure AI Agent from the [Azure AI Agent Service](https://learn.microsoft.com/azure/ai-services/agents/overview). The agent is given the OpenAPI spec for the web app so that it can handle product recommendations, shopping assistance, shopping cart management and more on your behalf via a chat interface. This sample builds off of the guidance documented by the AI Agent Service in [How to use Azure AI Agent Service with OpenAPI Specified Tools](https://learn.microsoft.com/azure/ai-services/agents/how-to/tools/openapi-spec?tabs=python&pivots=overview).

## Features

- Interactive Blazor UI for fashion e-commerce
- Integration with Azure AI Agent Service for intelligent shopping assistance
- Sample usage of the OpenAPI Specified Tool with Azure App Service
- Secure authentication to Azure AI Agent Service with Azure managed identity

## Architecture

The application consists of:

- A .NET 9 Blazor web application frontend 
- Azure AI Agent integration for intelligent shopping assistance
- RESTful APIs for shopping cart management
- Azure App Service and Azure AI Foundry infrastructure using Bicep and azd template

## Prerequisites

- [.NET 9 SDK](https://dotnet.microsoft.com/download/dotnet/9.0)
- [Azure Developer CLI](https://aka.ms/azd)
- Azure subscription
- Visual Studio 2022 or Visual Studio Code

## Deploy App Service Resources

1. Clone this repository
2. Login to Azure:

   ```bash
   azd auth login
   ```

3. Initialize your environment:

   ```bash
   azd env new
   ```

4. Deploy the application:

   ```bash
   azd up
   ```

   This will:
  - Build the .NET application
  - Provision Azure resources defined in the Bicep templates
  - Deploy the application to Azure App Service

This command can take up to 10 minutes to run. At this point, once deployment is complete, you can browse to your app and see the general functionality. Feel free to check out the inventory and add some items to the cart. If you try the chat interface at this point, it will not work and it will prompt you to add the environment variables for the AI Agent.

## Create the AI Agent

You now create the AI Agent that your app uses. The Azure AI Foundry resources were created as part of the azd template. You need to create the agent and connect that agent to your App Service.

1. Sign in to the [Azure portal](https://portal.azure.com) and go to the resource group that was created by the azd template.
2. Click the **Azure AI project** resource that was created for you.
3. Click **Launch studio** to open the Azure AI Foundry studio.
4. On the left-hand side under "Build and customize", select **Agents**.
5. In the dropdown, select the auto-generated Azure OpenAI Service resource that was created for you and then click **Let's go**.
6. Click **+ New agent** to create a new agent or use the default one if one is already created for you.
7. Once your agent is created, add the following instructions on the right-hand side. These instructions will ensure your agent only answers questions and completes tasks related to the fashion store app.

  ```
  You are an agent for a fashion store that sells clothing. You have the ability to view inventory, update the customer's shopping cart, and answer questions about the clothing items that are in the inventory. You should not answer questions about topics that are unrelated to the fashion store. If a user asks an unrelated question, please respond by telling them that you can only talk about things that are related to the fashion store.
  ```

### Add the OpenAPI Specified Tool to the AI Agent

For detailed guidance with screenshots, see [Add OpenAPI spec tool in the Azure AI Foundry portal](https://learn.microsoft.com/azure/ai-services/agents/how-to/tools/openapi-spec?tabs=python&pivots=overview#add-openapi-spec-tool-in-the-azure-ai-foundry-portal). The steps will be summarized below.

1. Click **+ Add** next to **Action**.
2. Select **OpenAPI 3.0 specified tool**.
3. Give your tool a name and a description. The description will be used by the model to decide when and how to use the tool. For this sample, you can use the following description:

  ```
  This tool is used to interact with and manage an online fashion store. The tool can add or remove items from a shopping cart as well as view inventory.
  ```

4. Leave the authentication method as anonymous. There is no authentication on the web app. If the app required an API key or managed identity to access it, this is where you would specify this information.
5. Copy and paste your OpenAPI specification in the text box. The OpenAPI specification is provided in this repo and is called [swagger.json](./src/webapp/swagger.json). 
6. Before you create the tool, you need to copy and paste your app's URL into the OpenAPI specification you are providing to the tool. Replace the placeholder `<APP-SERVICE-URL>` with your app's URL. It should be in the format `https://<app-name>.azurewebsites.net`.
7. Click **Next**, review the details you provided, and then click **Create Tool**.

### Update App Service Environment Variables

After setting up the AI Agent and adding the OpenAPI Specified Tool, you need to configure your App Service with the appropriate environment variables.

1. From the Agents dashboard where you just added your tool, note the agent ID. It should be in the format `asst_<unique-id>`.
2. Click **Overview** on the left-hand side and note the project's connection string. It should be in the format `<region>.api.azureml.ms;<subscription-id>;<resource-group-name>;<project-name>`.
3. Navigate back to your App Service.
4. From the left menu, select **Environment variables**.
5. In the **App settings** tab, click **+ Add** and add the following settings:
  - **Name**: `AzureAIAgent__ConnectionString`
  - **Value**: The connection string you noted from your AI Agent Service
6. Add another app setting:
  - **Name**: `AzureAIAgent__AgentId`
  - **Value**: The Agent ID you noted when creating your agent
7. Click **Apply** at the bottom of the page and confirm when prompted.
8. The app will restart with the new settings applied.

## Use the app

Now that all of the support resources are created and updated, the app is ready for use. Ask the agent questions such as:

- What's in my cart?
- Add a small denim jacket to my cart
- Do we have any blazers in stock?

You can also ask general questions about the items and the agent should be able to provide information.

- Tell me about Red Slim Fit Checked Casual Shirt
- Is the blazer warm?

## Clean-up

When you're done with this app, run the following to delete the App Service resources:

   ```bash
   azd down
   ```

## Troubleshooting

### Common Issues

1. **Chat Not Working**
   - Verify that the environment variables (`AzureAIAgent__ConnectionString` and `AzureAIAgent__AgentId`) are correctly set in the App Service configuration.
   - Check that the AI Agent was properly created and configured with the correct OpenAPI tool.
   - Ensure the OpenAPI specification URL is accessible from the Azure AI Agent Service.
   - Ensure the App Service URL is updated in the `swagger.json` provided to the OpenAPI Specified Tool.

2. **Permission Issues**
   - If you encounter authentication errors, ensure that your App Service's managed identity has proper permissions to access the Azure AI Agent Service. The managed identity needs at least the `Microsoft.MachineLearningServices/workspaces/agents/action` permission to interact with the Agent. The provided Azure AI Developer role has this permission and should be sufficient. If you decide to change this role, be sure it has the necessary permission.

3. **API Issues**
   - If the agent is unable to perform actions on the inventory or cart, check the API routes in the OpenAPI specification.
   - Verify that the API endpoints are responding correctly by testing them directly in the Swagger UI at `/api/docs`.

### Viewing Logs

To view logs for your App Service:

1. Navigate to your App Service in the Azure portal.
2. In the left menu, select **Monitoring** > **Log stream** to view real-time logs.
3. These logs will reveal any application issues that you may need to address.

## Understanding the API Capabilities

The OpenAPI specification provides the AI agent with information about available endpoints:

### Cart API Endpoints
- **GET /api/Cart**: Retrieves the current shopping cart contents and total cost
- **DELETE /api/Cart**: Clears all items from the cart
- **POST /api/Cart/add**: Adds an item to the shopping cart
- **PUT /api/Cart/{productId}/size/{size}**: Updates the quantity of a specific item
- **DELETE /api/Cart/{productId}/size/{size}**: Removes a specific item from the cart

### Inventory API Endpoints
- **GET /api/Inventory**: Lists all available inventory items
- **GET /api/Inventory/{id}**: Gets details about a specific product
- **GET /api/Inventory/{id}/size/{size}**: Checks inventory for a specific product size
- **GET /api/Inventory/sizes**: Gets all available sizes in the inventory

## Advanced Agent Interactions

Beyond basic interactions, the AI agent can handle more complex scenarios:

- **Personalized Recommendations**: "I need a business casual outfit for a meeting"
- **Size Guidance**: "What size blazer would fit someone who's 6'2" and 180 pounds?"
- **Outfit Coordination**: "What would go well with the black denim jacket?"
- **Shopping Cart Management**: "Remove the large shirt and add a medium instead"
- **Inventory Checks**: "Do you have any red shirts in medium?"
- **Price Inquiries**: "What's the price range for blazers?"

## Security Considerations

- The application uses Azure managed identities for secure authentication to Azure AI Agent Service in production environments.
- No sensitive credentials are stored in the code.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.
