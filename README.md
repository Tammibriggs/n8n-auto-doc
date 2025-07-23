# AutoDoc
This is a submission for the [Algolia MCP Server Challenge](https://dev.to/challenges/algolia-2025-07-09)

## What I Built
AutoDoc is an AI-powered agent that analyzes recent code commits, detects which documentation pages are affected, and intelligently suggests updates—ensuring your documentation stays accurate and up-to-date with every push.

## Setup Instructions
### 1. Clone Algolia MCP Server Repo
Run the following command in your terminal to clone the [Algolia MCP Server](https://github.com/algolia/mcp-node):  
```bash
git clone https://github.com/algolia/mcp-node
```

### 2. Self-Host n8n
Self-host n8n to use the `n8n-nodes-mcp` community node, which enables integration with external MCP servers. 
As the workflow uses webhooks, you’ll need a publicly accessible URL:
- Start the n8n instance with the `--tunnel` flag
**or**
- Preferably, create a dedicated ngrok tunnel using:
```bash
ngrok http 5678
```
> Ensure `ngrok` is installed on your system.

### 3. Install MCP Client node
Install the `n8n-nodes-mcp` node to enable communication with the Algolia MCP server. 

### 4. Create a new workflow
Create a new workflow in n8n, then copy and paste the AutoDoc workflow from `auto-doc-workflow.json`. 

### 5. Configure Algolia MCP Server
Open the **List Algolia tools** node and create a new credential with: 
- **Command**: <path_to_node_bin>/node (e.g /home/<username>/.nvm/versions/node/v23.11.1/bin/node)
- **Arguments**: <path_to_project>/src/app.ts --credentials applicationId:searchApiKey
(e.g. /home/<username>/Desktop/mcp-node/src/app.ts --credentials ABC123:XYZ456)
You can get the `applicationId` and `searchApiKey` from your [Algolia Dashboard](https://dashboard.algolia.com/users/sign_in).
> Note: Your Node version must be **v23.11.1** or higher.

### 6. Add Gemini Credential
Generate an API key from [Google AI Studio](https://aistudio.google.com/app/apikey) and use it to create a Gemini credential within the workflow.

### 7. Create a Documentation Repo & Add Webhook
Fork or create a dummy documentation repository, such as [algolia-chat-me-doc](https://github.com/Tammibriggs/algolia-chat-me).
Add a webhook to the repository:
- **Payload URL**: Your AutoDoc production or Ngrok URL
- **Content type**: application/json

### 8.Index Documentation in Algolia
If you're using the [algolia-chat-me-doc](https://github.com/Tammibriggs/algolia-chat-me) repo, clone it, and run the indexing script:
```bash
npm run generate-algolia-index
```
Then, upload the `alglia-index.json` file to Algolia index called **doc**
> Note: Ensure you're using Node v23.11.1 or higher.

### 9. Add GitHub Credential
Generate a fine-grained personal access token in your [GitHub Developer Settings](https://github.com/settings/personal-access-tokens) with:
- **Repository access**: Only select repositories -> your-doc-repo
- **Repository permissions**:
  - `Contents`: Read & Write
  - `Issues`: Read & Write
In n8n:
- Create a GitHub credential using your username and the token
- Use the same token as the bearer token in the **Create GitHub branch** HTTP node

## How It Works
- Listens for push events to the documentation GitHub repo via a webhook  
- On each new push, fetches commit data where code changes occurred  
- Passes the commit data to the AI agent, which searches the Algolia index to identify and return outdated documentations.
- For each outdated doc:
  - Creates a new GitHub branch with the proposed update  
- Generates a single GitHub issue summarizing all identified outdated docs, along with the AI-suggested updates for each.

## Test the Workflow
To do this, activate the workflow, make changes to the code, commit and push.

## Future Improvements
This is just an MVP. A more robust version should be able to:
- Monitor code changes across multiple repositories, not just the documentation repo  
- Automatically update the Algolia index once a suggested update branch is merged  
- Listen to both **pull** and **push** events for more comprehensive tracking
