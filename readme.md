# Bew Prod and Dev deployments of Appsscript projects

This repository demonstrates how to set up automatic deployments of Google Apps Script projects to different environments (Development and Production) using GitHub Actions.

## Project Overview

This project uses [clasp](https://github.com/google/clasp) to manage Google Apps Script deployments with separate script IDs for development and production environments. When code is pushed to the `dev` or `main` branch, GitHub Actions automatically deploys the code to the corresponding Apps Script project.

## Setup Instructions

### Prerequisites

- [Node.js](https://nodejs.org/) (v18 or later)
- [clasp](https://github.com/google/clasp) installed globally (`npm install -g @google/clasp`)
- Google account with Apps Script API enabled
- GitHub account

### Local Development Setup

1. Clone this repository
2. Authenticate with clasp:

   ```
   clasp login
   ```
3. For development work:

   ```
   cp src/.clasp.dev.json
   ```

   This file must be manualy created with the format:

   ```
   {
       "scriptId": "DEV_APPSSCRIPT_PROJECT_ID",
       "rootDir": "./src"
   }
   ```
4. For production work:

   ```
   cp src/.clasp.main.json
   ```

   This file must be manualy created with the format:
5. ```
   {
       "scriptId": "PROD_APPSSCRIPT_PROJECT_ID",
       "rootDir": "./src"
   }
   ```

## Deployment Process

This project uses GitHub Actions for automatic deployment:

- Pushing to the `dev` branch deploys to the development Apps Script project
- Pushing to the `main` branch deploys to the production Apps Script project and creates a new versioned deployment

The deployment is configured in [`.github/workflows/deploy-apps-script.yml`](.github/workflows/deploy-apps-script.yml).

### GitHub Secrets Required

- `nano ~/.clasprc.jsonCLASPRC_JSON`: Base64 encoded content of your `.clasprc.json` file.

It is not possible to generate this secret with Clasp since Google tightend the security reculaitons. It must be obtained from the Google Clouds *APIs & Services* product.

To generate this secret:

1. Navigate to the google cloud console (https://console.cloud.google.com/apis/credentials)
2. Create a project if you dont allready have one
3. Enable the required APIs

   1. In the left sidebar, click Library
   2. Search and enable "Google Apps Script API" and "Google Drive API"
4. Configure the OAuth consent screen

   1. In the left sidebar, click **OAuth Consent Screen**
   2. Set the user type to **External** or **Internal** (based on your use case)
   3. Fill out required fields (e.g., App name, support email)
   4. You don’t need to add scopes yet – leave defaults for now
5. Create the OAuth Client ID

   1. In the left sidebar, go to **Credentials**
   2. click Create Credentials > OAuth client ID
   3. select
      * Application Type: Desktop App
      * Name: Clasp Automation (Or some other name)
   4. Click "Create" and you will get
      * Client ID
      * Client Secret
6. Install Clasp on your local machine if you have not allready done so `npm install -g @google/clasp `
7. Create your .calsprc.json file manualy `tough ~/.clasprc.json`
8. Open this (Using Nano or VS Code) and pase in this structure, replacing YOUR_CLIENT_ID and YOUR_CLIENT_SECRET withh the values obtained from your OAuth credentials

   ```
   {
     "oauth2ClientSettings": {
       "clientId": "YOUR_CLIENT_ID.apps.googleusercontent.com",
       "clientSecret": "YOUR_CLIENT_SECRET",
       "redirectUri": "urn:ietf:wg:oauth:2.0:oob"
     },
     "token": {}
   }
   ```
9. You now need to authenticate manualy: `clasp login`

   1. This will open a URL where you can approve the permissions and should add the autorisation to your .calsprc.json file
10. Check this by viewing the .clasprc.json `nano ~/.clasprc.json`

    1. You should see that the `"token": {}` is updated with the values:

       ```
       access_token
       refresh_token
       scope
       token_type
       id_token
       expiry_date
       ```
11. Base64 Encode this to store it securely in your github action `cat ~/.clasprc.json | base64`
12. Copy the full encoded output
13. Add this to your github secrets

    1. Open the github repo
    2. click Settings > Secrets > Actions > New Repository Secret
    3. Name it "CLASPRC_JSON"
14. 
15. 
16. Run `base64 ~/.clasprc.json` to encode your credentials
17. Add the output as a repository secret named `CLASPRC_JSON` in your GitHub repository settings

## Project Structure

```
.
├── .github/workflows/      # GitHub Actions workflow configuration
├── src/                    # Source code directory
│   ├── .clasp.dev.json     # clasp configuration for development environment
│   ├── .clasp.main.json    # clasp configuration for production environment
│   ├── appsscript.json     # Apps Script project manifest
│   └── hello.js            # Apps Script code
├── .clasp.json             # Local clasp configuration (gitignored)
└── README.md               # This file
```

## Development Workflow

1. Create a feature branch from `dev`
2. Make your changes in the `src` directory
3. Test locally using `clasp push` and `clasp open`
4. Submit a pull request to the `dev` branch
5. After review and testing in the development environment, merge to `main` for production deployment

## License

[MIT](LICENSE)
