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

### 2. Configuration (Set Node)
- **IMPORTANT**: Configure the repository path here
- Sets the following variables:
  - `REPO_PATH`: Path to your git repository (e.g., `/home/user/n8nworkflows`)
  - `GIT_BRANCH`: Branch to push to
  - `GIT_USER_NAME`: Git username for commits
  - `GIT_USER_EMAIL`: Git email for commits

### 3. Get All Workflows (HTTP Request)
- Fetches all workflows from the n8n API
- Endpoint: `http://localhost:5678/api/v1/workflows`
- **Requires**: n8n API credentials

### 4. Process Workflows (Code Node)
- Processes each workflow and prepares it for saving
- Creates sanitized filenames based on workflow names
- Formats workflow data as clean JSON

### 5. Save Workflow Files (Execute Command)
- Navigates to the repository path from Configuration
- Creates the `workflows/` directory if it doesn't exist
- Saves each workflow as a separate JSON file

### 6. Aggregate All Results
- Combines all results for the commit step

### 7. Git Commit and Push (Execute Command)
- Navigates to the repository path from Configuration
- Configures git user name and email
- Adds all workflow files to git
- Creates a commit with timestamp
- Pushes to GitHub with retry logic (up to 5 attempts with exponential backoff)
- Includes error checking for repository path

### 8. Create Backup Summary
- Generates a summary report of the backup operation

## Setup Instructions

### 1. Import the Workflow

1. Copy the contents of `N8N_GitHub_Daily_Backup.json`
2. In n8n, click **Workflows** → **Import from File** (or use Ctrl+O)
3. Paste the JSON content
4. Save the workflow

### 2. Configure Repository Path

**⚠️ IMPORTANT**: You MUST configure the repository path before running the workflow!

1. In the workflow, click on the **"Configuration"** node (the second node)
2. Update the `REPO_PATH` value to match your git repository location:
   - For Docker: Usually something like `/data/git/n8nworkflows` or where you mounted your volume
   - For local installation: The full path to your git repository (e.g., `/home/user/n8nworkflows`)
3. Optionally update other configuration values:
   - `GIT_BRANCH`: The branch to push to (default: `claude/n8n-github-daily-backup-016XHDhyEKzieL9eAJE2gguW`)
   - `GIT_USER_NAME`: Git username for commits (default: `n8n-backup`)
   - `GIT_USER_EMAIL`: Git email for commits (default: `backup@n8n.local`)

### 3. Configure n8n API Access

You need to create an API key in n8n and configure it as a credential:

1. In n8n, go to **Settings** → **API**
2. Create a new API key
3. In the workflow, click on the "Get All Workflows" node
4. Add a new **Header Auth** credential with:
   - **Name**: `X-N8N-API-KEY`
   - **Value**: Your n8n API key

### 4. Configure Git (if needed)

Make sure git is configured in your n8n environment. If you're using Docker, you may need to configure git inside the container. The workflow will attempt to configure git automatically using the values from the Configuration node.

### 5. Activate the Workflow

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

### Workflow fails with "can't cd to /home/user/n8nworkflows"
This is the most common error! It means the repository path in the Configuration node doesn't exist in your n8n environment.

**Solution**:
1. Click on the **Configuration** node in the workflow
2. Update the `REPO_PATH` value to the correct path where your git repository is located
3. To find the correct path, add a temporary Execute Command node and run: `pwd` to see the current directory, or `echo $HOME` to see your home directory
4. For Docker installations, the path might be `/data` or wherever you mounted your volume

### Workflow fails at "Get All Workflows"
- Verify your n8n API key is correct
- Check that the API URL matches your n8n instance (default: `http://localhost:5678`)
- Ensure the API is enabled in n8n settings

### Workflow fails at "Save Workflow Files"
- Verify the `REPO_PATH` in the Configuration node is correct
- Ensure n8n has write permissions to that directory
- Check that the git repository is properly initialized

### Workflow fails at "Git Commit and Push"
- Verify the `REPO_PATH` in the Configuration node is correct
- Verify git is configured with user name and email (or use the Configuration node values)
- Check that you have push access to the repository
- Ensure the branch name in the Configuration node is correct
- Check network connectivity to GitHub
- Make sure the repository is initialized with `git init` and has a remote configured

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
