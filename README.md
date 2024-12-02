# Integrity-app
env:
  AZURE_WEBAPP_NAME: integrity-sandbox-app  # Name of the sandbox Azure Web App
  AZURE_WEBAPP_PACKAGE_PATH: '.'  
  NODE_VERSION: '16.x'

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Install Dependencies and Build
      run: |
        npm install
        npm run build --if-present

    - name: Run Tests
      run: npm test --if-present

    - name: Upload Artifact
      uses: actions/upload-artifact@v3
      with:
        name: node-app
        path: .

  deploy:
    permissions:
      contents: none
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: 'Sandbox'
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}

    steps:
    - name: Download Artifact
      uses: actions/download-artifact@v3
      with:
        name: node-app

    - name: Deploy to Azure
      id: deploy-to-webapp
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}
