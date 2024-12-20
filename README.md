# Boilerplate of Playwright - Specflow - Csharp

## Usage 

- Install playwright

...
npx playwright install 
...

-  Restore .NET depends

...
dotnet restore
...


## BDD

...
Feature: Login

  Scenario: Successful login with valid credentials
    Given I am on the login page
    When I login with valid credentials
    Then I should see the products page
...

## Using 

> BaseUrl as 'https://www.saucedemo.com/v1/index.html'

> Set headless in Hooks.cs

...
_browser = await playwright.Chromium.LaunchAsync(new BrowserTypeLaunchOptions
{
    Headless = true,
    Channel = "chrome"

});
...


## .yml pipeline

...
trigger:
- none # change it to main/ branch name if needed

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '6.x'
    installationPath: $(Agent.ToolsDirectory)/dotnet

- script: |
    npm install -g npx
    npx playwright install
  displayName: 'Install Playwright'

- script: |
    dotnet restore
  displayName: 'Restore .NET dependencies'

- script: |
    dotnet build --configuration Release --no-restore
  displayName: 'Build solution'

- script: |
    dotnet test --configuration Release --no-build --logger "trx;LogFileName=test_results.trx"
  displayName: 'Run tests'

- task: PublishTestResults@2
  inputs:
    testRunner: 'VSTest'
    testResultsFiles: '**/*.trx'
    searchFolder: '$(System.DefaultWorkingDirectory)'
  displayName: 'Publish test results'
  ...

  ## Set pipeline for GitHUb Actions

  Create '.github/workflows/ci.yml'

  and

  ...
  name: CI

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '6.x'  # Replace with your desired .NET Core version

    - name: Install Node.js and Playwright
      run: |
        curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
        sudo apt-get install -y nodejs
        npm install -g npx
        npx playwright install

    - name: Restore dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --configuration Release --no-restore

    - name: Test
      run: dotnet test --configuration Release --no-build --verbosity normal --logger "trx;LogFileName=test_results.trx"

    - name: Publish Test Results
      uses: actions/upload-artifact@v2
      with:
        name: TestResults
        path: '**/TestResults/*.trx'
...

