# Deploying your app with GitHub Actions

In this tutorial we will show how to set up deployment of an app with
[GitHub actions](https://github.com/features/actions), so that every push to
your GitHub repository will automatically deploy the new version of your app to
Numerous

In the repository
[numerous-com/deploy-app-with-github-actions-tutorial](https://github.com/numerous-com/deploy-app-with-github-actions-tutorial)
we have set up the example that we will go through in this tutorial. You can use
it as a reference for your own repository.

## Clone the repository to your machine

This tutorial assumes that you have a repository set up on GitHub. First we
either clone it, or if it is already on the computer, navigate to it in our
terminal.

## Create a simple app

Inside the folder of our repository, we create a very simple app. For this
tutorial we create a Streamlit app.

```bash
numerous init --app-library streamlit
```

After following the wizard, we open `app.py` in our code editor, and
add the following content to it.

```python copy
import streamlit as st

st.text("This app has been deployed with GitHub actions")
```

Now the app should be ready to be deployed, we just need to set up the
deployment.

## Define the GitHub actions workflow

In your repository folder create the directory `.github/workflows` and inside
that add the file `deploy.yml`, and add the following workflow definition.

```yaml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      # First we check out the code
      - uses: actions/checkout@v4

      # Then we set up python
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: "pip"

      # We install the Numerous SDK, which includes the CLI
      - run: pip install numerous

      # And finally we deploy our app, setting the NUMEROUS_ACCESS_TOKEN
      # environment  variable with the GitHub actions secret that contains the
      # actual value.
      # We also insert the app slug and organization slug from secrets.
      - env:
          NUMEROUS_ACCESS_TOKEN: ${{secrets.NUMEROUS_ACCESS_TOKEN}}
        run: numerous deploy --app ${{secrets.APP_SLUG}} --organization ${{secrets.ORGANIZATION_SLUG}}
```

## Create a personal access token

We need a personal access token to give the CLI access to run in the GitHub
actions environment without logging in. Create one with the below command.

```bash copy
numerous token create --name github-actions
```

Copy the token that is printed, so you can add it to a GitHub secret later.

## Finding your organization slug

You can find your organization slug either in the browser or with the CLI.

Using the CLI we can run `numerous organization list`, and a list of all your
organizations will be displayed, which includes. You can also create an organization with
`numerous organization create` if you do not have one.

In the browser you can find the organization slug in the address bar, like in the
screenshot below.

![](/images/numerous_github_actions_browser_organization_slug.png)

## Add secret values to GitHub actions

In our workflow file we used the secrets `NUMEROUS_ACCESS_TOKEN`, `APP_SLUG` and
`ORGANIZATION_SLUG`. We need to define the values for these.

1. Go to the repository **Settings** page.
   ![](/images/numerous_github_actions_repo_settings.png)
2. Open the **Secrets and variables** for **Actions** page.
   ![](/images/numerous_github_actions_repo_settings_actions.png)
3. Click the button to create a new repository secret.
   ![](/images/numerous_github_actions_repo_settings_actions_repository_secret.png)
4. First we add the secret named `NUMEROUS_ACCESS_TOKEN` that we use in the
   workflow. Add the personal access token that you created earlier as the
   **Secret**.

   The value in the screenshot is just an example of how it will look like.
   ![](/images/numerous_github_actions_repo_settings_add_secret_numerous_access_token.png)

5. Now we add the secret named `ORGANIZATION_SLUG` that we use in the
   workflow. Add the organization slug you found before as the **Secret**.
   ![](/images/numerous_github_actions_repo_settings_add_secret_organization_slug.png)
6. Finally, The app slug is user defined. You can select a different value, but it has
   to be composed of only lowercase alphanumeric characters with dashes as the
   separator. It will become part of the URL for the app.
   ![](/images/numerous_github_actions_repo_settings_add_secret_app_slug.png)

## Push your repository

Now commit and push the simple app and the workflow file, and GitHub actions
should start a workflow for your repository!

![](/images/numerous_github_actions_workflow_running.png)

And when the workflow has completed, the app should be deployed to the specified
organization and app slug.

![](/images/numerous_github_actions_app_deployed.png)
