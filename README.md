https://docs.servicenow.com/bundle/madrid-application-development/page/build/applications/task/t_AccessStudio.html

https://www.terraform.io/docs/cloud/integrations/service-now/index.html

# What is this?

<p align="center">
    <img align="center" src="doc/Part11.gif" alt="example"/>
</p>

HashiCorp created an extension for ServiceNow that allows users to interact with the Terraform API via ServiceNow. The goal of this example is to showcase how we can build on those concepts to integrate automated Sentinel policies that will inject manual approvals into ServiceNow when necessary. The GIF above showcases the end result in which we utilize a customized version of the Terraform ServiceNow Integration to accomplish the following items.

1. Create a Workspace
2. Stream Status Updates back a ServiceNow Ticket
3. Have a Sentinel Cost Policy fail and require manual approval
4. Generate a ServiceNow Approval Request for Spa Ghetti
5. Manually approve the request in ServiceNow 
6. Trigger a Policy Override with the Terraform API with a custom ServiceNow Workflow

# Installation

## Setting Up Terraform Cloud

<p align="center">
    <img align="center" src="doc/Part1.gif" alt="example"/>
</p>

For simplicity, we are going to use Terraform Cloud with an Enterprise Trial. However, if you're using a Terraform Enterprise installation you would follow the same steps.

1. Once you've logged in, you'll want to create a new Terraform organization. I called it service-now-test but feel free to name it anything.
2. Navigate to the Organization Settings tab which is located at the top of the screen.
3. Enable the 30 day Enterprise Trial for the organization that you created.
4. Create a Service Now team within this organization and enable Manage Policies and Manage Workspaces.
5. Navigate to the API Tokens page, generate an organization token and save it somewhere locally.

## Setting up GitHub VCS Connection to Terraform Cloud

<p align="center">
    <img align="center" src="doc/Part2.gif" alt="example"/>
</p>

1. Navigate to the VCS Providers page and Add a GitHub OAuth Provider.
2. Navigate to your GitHub Settings Page
3.  Click on Developer Settings
4. Click on OAuth Apps
5. Create a New OAuth App
6. Name it Terraform Cloud
7. Set Homepage URL to https://app.terraform.io or your Terraform Enterprise endpoint
8. Set Authorization Callback URL to https://app.terraform.io
9. Copy the OAuth Client ID and Client Secret into Terraform Cloud's respective inputs
10. Click on the Connect organization service-now-example wbutton
11. Copy the Callback URL from the newly added Terraform Cloud VCS Provider
12. Enter the copied callback URL into your GitHub App's Authorization callback URL and hit save.
13. Copy and save the OAuth Token ID from Terraform Cloud


## Installing the Terraform ServiceNow Application

<p align="center">
    <img align="center" src="doc/Part3.gif" alt="example"/>
</p>

You'll need to request the Terraform Service Now Integration repo from HashiCorp which you can then include in a personal or company VCS repository.
1. Create an account at developer.servicenow.com
2. Request a Madrid Instance from the Developer Portal
3. Log into the ServiceNow Account
4. Filter for "Studio" using the left hand search bar
5. Select "Import from Source Control"
6. Enter the GitHub URL
7. Enter your username
7. Generate a Personal Access Token from GitHub
8. Enter the Personal Access Token as your Password

# Customization

## Add Spa Ghetti

<p align="center">
    <img align="center" src="doc/Part4.gif" alt="example"/>
</p>

1. Use the Left Hand Search Bar to filter for Users
2. Select the Users result under Organization
3. Click New at the top of the Content Pane
4. Set User ID: `spa.ghetti`
5. Set First Name: `Spa`
6. Set Last Name: `Ghetti`
7. Click Submit

## Add Policy Override REST Message

<p align="center">
    <img align="center" src="doc/Part5.gif" alt="example"/>
</p>

1. Navigate to the Studio
2. Select the Terraform Integration
3. Select Create Application File from the Top Left
4. Search for and select `REST Message`
5. Name it `TF Policy Checks`
6. Enter `${hostname}/api/v2/policy-checks` as the Endpoint
7. Select the `HTTP Request Tab`
8. Add an `Authorization` Header with the value of `Bearer ${api_team_token}`
9. Add a `Content-Type` Header with the value of `application/vnd.api+json`
10. Click Submit
11. Delete the `Default GET` HTTP Method
12. Create a new HTTP Method
13. Name it `Override Policy`
14. Enter `${hostname}/api/v2/policy-checks/${policy_check_id}/actions/override` as the Endpoint
15. Select `POST` as the HTTP Method
16. Click Submit
17. Add a Variable Substitution called `api_team_token` and select Submit
18. Add a Variable Substitution called `policy_check_id` and select Submit

## Add TF_Policy Script

<p align="center">
    <img align="center" src="doc/Part6.gif" alt="example"/>
</p>

1. Navigate to the Studio
2. Select the Terraform Integration
3. Select Create Application File from the Top Left
4. Search for and select `Script Include`
5. Name it `tf_policy`
6. Paste the contents of `service_now/script_tf_policy.js` into the Script section
7. Click Submit

## Update Poll Run State

<p align="center">
    <img align="center" src="doc/Part7.gif" alt="example"/>
</p>

1. Navigate to the Studio
2. Select the Terraform Integration
3. Scroll to the Bottom and select the `Poll Run State` Workflow
4. Select the Left Hand Hamburger Menu
5. Click Checkout
6. Paste the contents of `service_now/wf_poll_run_state.js` into the Script section
7. Select the Left Hand Hamburger Menu
8. Click Publish
9. Select `Worker Poll Run State` under Workflow Schedule
10. Select `Periodically` from the Run dropdown
11. Click Update

# Configuration

## Add a Terraform API Config

<p align="center">
    <img align="center" src="doc/Part8.gif" alt="example"/>
</p>

1. Utilize the left hand search bar to search for Terraform
2. Select Configs
3. Enter the API Organization Token that we saved earlier
4. Enter the name of your Terraform Organization, mine was `service-now-example`

## Add a Terraform VCS Repository

<p align="center">
    <img align="center" src="doc/Part9.gif" alt="example"/>
</p>

1. Utilize the left hand search bar to search for Terraform
2. Select VS Repositories
3. Name it Terraform Example
4. Enter `peytoncasper/terraform-servicenow-example` into the Identifier
5. Enter the GitHub OAuth token from earlier

## Add the Terraform Service Catalog

<p align="center">
    <img align="center" src="doc/Part10.gif" alt="example"/>
</p>

1. Utilize the left hand search bar to search for `Service Catalog`
2. Select `Catalogs` under `Service Catalog`
2. Click the `+` in the upper right 
3. Select `Terraform Catalog`
4. Click Add Here

# Action!

1. Utilize the left hand search bar to search for `Service Catalog`
2. Select `Catalogs` under `Service Catalog`
3. Select the `Terraform Catalog`
4. Select `Create Workspace`
5. Select `Order Now`
6. Navigate to Terraform Cloud
7. Select the Workspace
8. Add a `AWS_ACCESS_KEY_ID` Environment Variable
9. Add a `AWS_SECRET KEY` Environment Variable
10. Add an `instance_type` Terraform Variable with a value of `m5.large`
11. Queue a Terraform Apply
12. Wait for the `Soft Policy Check`
13. Refresh the ServiceNow Request and Navigate to the Approval that was added
14. Approve it
15. Sit back and watch Terraform process the Override Request!

