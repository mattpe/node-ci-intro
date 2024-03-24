# Simple Node CI/CD demo including integration testing

Tests & code based on <https://github.com/ilkkamtk/integration-testing-ready>

## Testing Scenarios

Testing successful API responses and error handling. The test cases for both scenarios are provided in the test folder.

## Prerequisites

1. Clone this repository
1. Install dependencies with `npm install`
1. Create a database and a user for the application (example script: [db/create-db.sql](./db/create-db.sql))
1. Create `.env` file in the root of the project (see `.env.example` for reference)
1. Run tests with `npm test` locally and make sure they pass

## Setting up CI/CD

1. Create a new repository in GitHub
1. Push this repository to the new repository (change the remote `origin` URL first with `git remote set-url origin <new-repo-url>`)
1. Open the repository in GitHub in a browser and choose `Actions` from the top menu
   - Browse the available CI/CD templates and choose `Node.js` as the template
   - Commit the suggested changes to the repository
   - Pull the changes to your local repository
1. View & edit the `.github/workflows/node.js.yml` [yaml](https://yaml.org/) file and update the content according your needs
1. Test action by committing the changes and pushing them to the remote repository, check the status of the action in GitHub

## Example of setting up a CD pipeline for a Server (e.g. Virtual Machine in Azure)

1. First, make sure you have a server running and you can pull the repository from GitHub (without a need type password) and run the application
    - Create the database and user for the application
    - Create and edit `.env` file
    - Test run app & run tests manually
    - Use `pm2` to keep the application running, remember to add a name for the application `pm2 start app.js --name node-ci-example`
1. Create a new SSH key pair on the server for authentication using GitHub action
    - Generate a new key pair with `ssh-keygen -t rsa -b 4096 -m PEM -C "github-actions-node"`
    - Copy the public key `~/.ssh/id_rsa-github-node.pub` to the server's `~/.ssh/authorized_keys` file
    - Copy the private key `~/.ssh/id_rsa-github-node` to the repository secrets (use name `PRIVATE_KEY`) in GitHub (_Settings -> Secrets -> Actions -> New repository secret_)
    - Add other necessary properties to the repository secrets
      - `HOST`: IP address or domain name of the server
      - `USERNAME`: your username on the server
1. Create a new action in the `.github/workflows` folder, e.g. `deploy.yml`:

    ```yaml
    name: Node.js CD

    # Controls when the action will run. 
    on:
      # Triggers the workflow on push or pull request events but only for the main branch
      push:
        branches: [ main ]

    # A workflow run is made up of one or more jobs that can run sequentially or in parallel
    jobs:
      # This workflow contains a single job called "deploy"
      deploy:
        # The type of runner that the job will run on
        runs-on: ubuntu-latest
        # Steps represent a sequence of tasks that will be executed as part of the job
        steps:
        - name: Deploy using ssh
          uses: appleboy/ssh-action@master
          with:
            host: ${{ secrets.HOST }}
            username: ${{ secrets.USERNAME }}
            key: ${{ secrets.PRIVATE_KEY }}
            port: 22
            script: |
              cd ~/test-ci-node
              git pull origin main
              git status
              npm install
              npm run test
              pm2 restart node-ci-example
    ```

1. Test by committing the changes and pushing them to the remote repository, check the status of the action in GitHub
