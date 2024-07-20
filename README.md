
# Node.js CI/CD with GitHub Actions and Self-Hosted Runner

This guide explains how to set up a CI/CD pipeline using GitHub Actions with a self-hosted runner to automate the deployment of a Node.js application on an AWS EC2 instance.

## Prerequisites
1. Node.js application hosted on GitHub.
2. AWS EC2 instance with proper security groups allowing SSH access and HTTP/HTTPS traffic.
3. AWS CLI installed and configured on your local machine.
4. EC2 key pair for SSH access to the instance.
5. AWS credentials (Access Key ID and Secret Access Key).

## Steps

### 1. Setting Up the Self-Hosted Runner

1. **Add a self-hosted runner**:
   - Go to your repository on GitHub.
   - Navigate to **Settings > Actions > Runners > New self-hosted runner**.
   - Follow the instructions to set up the runner on your EC2 instance or another machine.

2. **Install the runner** on your machine:
   - Run the commands provided by GitHub to install and configure the runner.
   - Start the runner.

### 2. GitHub Actions Workflow Configuration

Create a `.github/workflows/deploy.yml` file in your repository:

```yaml
# This workflow will do a clean installation of node dependencies, cache/restore them, build the source code and run tests across different versions of Node.js
# For more information see: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-nodejs

name: Node.js CI/CD

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: self-hosted
    strategy:
      matrix:
        node-version: [ 18.x ]
        # See supported Node.js release schedule at https://nodejs.org/en/about/releases/
    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    - run: npm ci
    - run: |
        touch .env
        echo "${{ secrets.PROD_ENV_FILE }}" > .env
    - run: pm2 restart BackendAPI
```

### 3. Storing Secrets in GitHub

Go to your repository on GitHub, navigate to **Settings > Secrets and variables > Actions** and add the following secrets:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `EC2_HOST`
- `PROD_ENV_FILE` (Your environment variables content)

### 4. Configuring the Self-Hosted Runner

Ensure that the self-hosted runner machine has access to the SSH key for the EC2 instance. The path to the key should match the `EC2_KEY_PATH` specified in the workflow file. This key should be kept secure and accessible only by the runner.

### Additional Tips

- **Runner Configuration**: Make sure the self-hosted runner has Node.js, npm, and PM2 installed.
- **IAM Permissions**: Ensure the IAM user associated with the AWS credentials has the necessary permissions to access EC2 and other required services.
- **Security Groups**: Ensure the EC2 instance’s security group allows SSH access and HTTP/HTTPS traffic.

By following these steps, you can set up a CI/CD pipeline using a self-hosted runner and GitHub Actions to automate the deployment of your Node.js application on an AWS EC2 instance.
