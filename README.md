# N8N GitHub Daily Backup Workflow

This repository contains an n8n workflow that automatically backs up all your n8n workflows to GitHub on a daily schedule.

## Features

- 🕐 **Daily Schedule**: Runs automatically every day at 2:00 AM
- 📦 **Complete Backup**: Backs up all workflows including nodes, connections, settings, and metadata
- 🔄 **Auto-commit**: Automatically commits and pushes changes to GitHub
- 📁 **Organized Storage**: Saves workflows in a `workflows/` directory with descriptive filenames
- 🔁 **Retry Logic**: Includes exponential backoff retry logic for git push operations
- 📊 **Summary Report**: Generates a summary of backed up workflows

## Workflow Components

### 1. Schedule Daily Backup (Schedule Trigger)
- Triggers the workflow daily at 2:00 AM (cron: `0 2 * * *`)
- You can modify the schedule by editing the cron expression

### 2. Get All Workflows (HTTP Request)
- Fetches all workflows from the n8n API
- Endpoint: `http://localhost:5678/api/v1/workflows`
- **Requires**: n8n API credentials

### 3. Process Workflows (Code Node)
- Processes each workflow and prepares it for saving
- Creates sanitized filenames based on workflow names
- Formats workflow data as clean JSON

### 4. Save Workflow Files (Execute Command)
- Creates the `workflows/` directory if it doesn't exist
- Saves each workflow as a separate JSON file

### 5. Aggregate All Results
- Combines all results for the commit step

### 6. Git Commit and Push (Execute Command)
- Adds all workflow files to git
- Creates a commit with timestamp
- Pushes to GitHub with retry logic (up to 5 attempts with exponential backoff)

### 7. Create Backup Summary
- Generates a summary report of the backup operation

## Setup Instructions

### 1. Configure n8n API Access

You need to create an API key in n8n and configure it as a credential:

1. In n8n, go to **Settings** → **API**
2. Create a new API key
3. In the workflow, click on the "Get All Workflows" node
4. Add a new **Header Auth** credential with:
   - **Name**: `X-N8N-API-KEY`
   - **Value**: Your n8n API key

### 2. Configure Git

Make sure git is configured in your n8n environment:

```bash
git config --global user.name "n8n-backup"
git config --global user.email "backup@n8n.local"
```

### 3. Import the Workflow

1. Copy the contents of `N8N_GitHub_Daily_Backup.json`
2. In n8n, click **Workflows** → **Import from File** (or use Ctrl+O)
3. Paste the JSON content
4. Save the workflow

### 4. Activate the Workflow

1. Click the **Inactive** toggle in the top right to activate
2. The workflow will now run daily at 2:00 AM

## Manual Execution

You can also run the workflow manually:

1. Open the workflow in n8n
2. Click **Execute Workflow** button
3. Check the execution log to verify all workflows were backed up

## Customization

### Change Backup Schedule

Edit the cron expression in the "Schedule Daily Backup" node:
- `0 2 * * *` - 2:00 AM daily (default)
- `0 */6 * * *` - Every 6 hours
- `0 0 * * 0` - Weekly on Sunday at midnight
- `0 12 * * *` - Daily at noon

### Change n8n API URL

If your n8n instance is not running on `localhost:5678`, update the URL in the "Get All Workflows" node.

### Change Git Branch

Update the branch name in the "Git Commit and Push" node if you want to push to a different branch.

## File Structure

After the workflow runs, your repository will look like this:

```
n8nworkflows/
├── workflows/
│   ├── scriptator-ftth-and-mailocr-1ngVU7VMmzTm3uRD.json
│   ├── n8n-github-daily-backup-github-daily-backup.json
│   └── ... (other workflows)
├── N8N_GitHub_Daily_Backup.json
└── README.md
```

## Troubleshooting

### Workflow fails at "Get All Workflows"
- Verify your n8n API key is correct
- Check that the API URL matches your n8n instance
- Ensure the API is enabled in n8n settings

### Workflow fails at "Git Commit and Push"
- Verify git is configured with user name and email
- Check that you have push access to the repository
- Ensure the branch name is correct
- Check network connectivity to GitHub

### No files are committed
- This is normal if no workflows have changed since the last backup
- The workflow will skip committing when there are no changes

## Security Notes

- 🔒 Keep your n8n API key secure
- 🔒 Use environment variables or n8n credentials for sensitive data
- 🔒 Consider using SSH keys for GitHub authentication instead of HTTPS
- 🔒 Ensure your repository is private if workflows contain sensitive information

## License

This workflow is provided as-is for backup purposes.
