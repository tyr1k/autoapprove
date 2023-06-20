# Auto-Approve Pull Requests

This is a Bash script for automatically approving pull requests in Bitbucket based on certain conditions. It is designed to work in the Bamboo CI/CD environment and uses the Bitbucket API.

## Script Explanation

The script first extracts the project key and repository slug from the Bamboo predefined variable `${bamboo.planRepository.1.repositoryUrl}`. It then gets the pull request ID from `${bamboo.repository.pr.key}`.

Next, the script fetches pull request data from Bitbucket, checks if the pull request contains any changes in `.sh` files, and if not, automatically approves the pull request.

## Usage

1. Store the script in a location accessible by Bamboo.
2. In your Bamboo plan, add a task to execute this script. This could be done as a final task in your build plan, or as part of a deployment project.
3. Ensure that the user credentials used in the script (currently set to `username:password`) have the necessary permissions to approve pull requests in Bitbucket.

## Script

```bash
#!/bin/bash

repositoryUrl=${bamboo.planRepository.1.repositoryUrl} # source URL
projectkey=$(echo $repositoryUrl | sed -n 's/.*:7999\/\([^\/]*\)\/.*/\1/p') 
repoSlug=$(echo $repositoryUrl | sed -n 's/.*\/\([^\/]*\)\.git/\1/p')
pullRequestId=${bamboo.repository.pr.key}
pullRequestData=$(curl -u username:password "http://bitbucket/rest/api/1.0/projects/${projectkey}/repos/${repoSlug}/pull-requests/${pullRequestId}")

echo "================================================"
echo "repositoryUrl: ${repositoryUrl}"
echo "${pullRequestId}"
echo "${projectkey}"
echo "${repoSlug}"
echo "================================================"

# Check if there is a pull request without .sh file changes
if [[ $pullRequestData != *"\"size\":0"* && $pullRequestData != *".sh"* ]]; then
  # Perform auto-approval of the pull request
  curl -X POST -u username:password -H "Content-Type: application/json" -d '{"status": "APPROVED"}' "http://bitbucket/rest/api/1.0/projects/${projectkey}/repos/${repoSlug}/pull-requests/${pullRequestId}/approve"
  curl -X POST -u username:password -H "Content-Type: application/json" -d '{"text": "Auto-approve"}' "http://bitbucket/rest/api/1.0/projects/${projectkey}/repos/${repoSlug}/pull-requests/${pullRequestId}/comments"
else 
  curl -X POST -u username:password -H "Content-Type: application/json" -d '{"text": "Cannot auto-approve - .sh file was changed"}' "http://bitbucket/rest/api/1.0/projects/${projectkey}/repos/${repoSlug}/pull-requests/${pullRequestId}/comments"
fi
