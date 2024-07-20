
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
name: Node.js CI/CD

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: self-hosted

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14'

    - name: Install dependencies
      run: npm install

    - name: Run tests
      run: npm test

    - name: Deploy to EC2
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        AWS_REGION: 'us-east-1'  # Change to your AWS region
        EC2_USER: 'ec2-user'    # Change to your EC2 username
        EC2_HOST: ${{ secrets.EC2_HOST }}
        EC2_KEY_PATH: '/path/to/your/key.pem'  # Path to the key on the runner
      run: |
        chmod 400 $EC2_KEY_PATH
        scp -o StrictHostKeyChecking=no -i $EC2_KEY_PATH -r ./* $EC2_USER@$EC2_HOST:~/app
        ssh -o StrictHostKeyChecking=no -i $EC2_KEY_PATH $EC2_USER@$EC2_HOST << 'EOF'
          cd ~/app
          npm install
          pm2 restart all || pm2 start index.js
        EOF
      shell: bash
```

### 3. Storing Secrets in GitHub

Go to your repository on GitHub, navigate to **Settings > Secrets and variables > Actions** and add the following secrets:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `EC2_HOST`

### 4. Configuring the Self-Hosted Runner

Ensure that the self-hosted runner machine has access to the SSH key for the EC2 instance. The path to the key should match the `EC2_KEY_PATH` specified in the workflow file. This key should be kept secure and accessible only by the runner.

### Additional Tips

- **Runner Configuration**: Make sure the self-hosted runner has Node.js, npm, and PM2 installed.
- **IAM Permissions**: Ensure the IAM user associated with the AWS credentials has the necessary permissions to access EC2 and other required services.
- **Security Groups**: Ensure the EC2 instance’s security group allows SSH access and HTTP/HTTPS traffic.

By following these steps, you can set up a CI/CD pipeline using a self-hosted runner and GitHub Actions to automate the deployment of your Node.js application on an AWS EC2 instance.
