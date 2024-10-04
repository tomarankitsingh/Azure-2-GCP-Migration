# MsAzure-2-GCP-Migration
This repository provides a comprehensive, step-by-step guide for migrating virtual machines from Microsoft Azure to Google Cloud. It covers the entire process, from configuring the environment in Azure to executing the migration and ensuring a seamless transition to Google Cloud. The guide is designed to help users manage the migration efficiently while maintaining system integrity and minimizing downtime.

## Prerequisites
Before starting the migration process, ensure the following tools and accounts are set up:

1. **Microsoft Azure Account:** You must have access to the Azure Portal with administrator privileges to export the VM.
2. **Google Cloud Account:** A Google Cloud account with sufficient IAM permissions to create and manage virtual machines.
3. **Google Cloud SDK:** Install Google Cloud SDK for managing GCP services.
     - link to download the CLI - https://cloud.google.com/sdk/docs/install
     - After opening the link, click on Download the **Google Cloud CLI Installer** after choosing your `Operating System`.
     - Once downloaded, Perform the steps to install the CLI.
     - Open the CMD and run command `gcloud components update` to update the components.
     - Then run the command `gcloud auth login` to login to the environment.
     - Enable the APIs, type:
          ```bash
          gcloud services enable vmmigration.googleapis.com servicemanagement.googleapis.com servicecontrol.googleapis.com iam.googleapis.com cloudresourcemanager.googleapis.com compute.googleapis.com
          ```

## What is Cloud Migration?

Cloud migration refers to the process of moving data, applications, or workloads from one cloud environment to another. like, Azure to Google Cloud migration involves transferring virtual machines, applications, or other resources from Microsoft Azure to Google Cloud Platform (GCP). This process includes exporting VM images, converting formats if needed, and reconfiguring services to ensure they work efficiently in GCP, while maintaining data integrity and minimizing disruptions.

### Phases of Migration

1. **Assessment & Planning:** Identify workloads to migrate, evaluate the compatibility between source (Azure) and target (Google Cloud), and create a migration strategy.
2. **Preparation:** Set up both Azure and Google Cloud environments, configure networking, security, and resource management. Back up data to ensure safety during migration.
3. **Migration:** Transfer VMs, applications, or services. For Azure to GCP migration, this involves exporting Azure VMs and importing them into GCP.
4. **Testing & Validation:** Validate that the migrated workloads function correctly in Google Cloud, ensuring performance, security, and availability match requirements.
5. **Optimization:** Fine-tune the migrated environment in GCP for performance improvements, cost savings, and scalability.
6. **Post-Migration Monitoring:** Continuously monitor the workloads for performance issues, security vulnerabilities, and ensure proper integration within GCP.

Let's begin the Migration Process!

## In the Azure Portal

## Step 1: Create a Resource Group
![Screenshot 2024-09-29 112902](https://github.com/user-attachments/assets/d1634405-9747-4b94-8cdd-78aea7fe7e2e)

1. Open the Azure portal.
2. Use the search bar to find "Resource Groups" and click on `Create`.
3. Enter `Migrate2GCP-RG` as the **Resource Group Name**.
4. Choose `West US 2` (or your preferred region) for the **Region**.
5. Click `Review + Create`, then select `Create`.

## Step 2: Create a Virtual Network with a Subnet
![Screenshot 2024-09-29 113819](https://github.com/user-attachments/assets/daa9f095-9af8-43ac-ba56-5722adafb18b)

1. In the search bar, look for `Virtual Networks` and click on `Create`.
2. Enter `MigrateVNet` as the **Name**.
3. Select `West US 2` for the **Region**.
4. Set the **Address Space** to `10.0.0.0/24`.
5. Add a Subnet:
   - **Web-Subnet:** Address range `10.0.0.0/27`.
6. Click `Review + Create`, then `Create`.

## Step 3: Create a Virtual Machine
![image](https://github.com/user-attachments/assets/3e8a7c68-f3c0-46a3-a74c-e1b369428e14)

1. In the search bar, look for `Virtual Machines` and select `Create`.
2. Enter `USW-WEB-VM` as the **Name**.
3. Choose `WestUS 2` for the **Region**.
4. Pick your preferred OS image (e.g., Ubuntu Server or Windows).
5. Under the Networking section, select `MigrateVNet` and `Web-Subnet`.
6. Click `Review + Create`, and then hit `Create`.

## Step 4: Host `Index.html` on Web VM (IIS Windows Server)

### 1. Promote WebVM to IIS Windows Server
![Screenshot (1)](https://github.com/user-attachments/assets/1eea8fa4-1dfd-4d0d-88f8-49eb1421b405)

- After your Web VM is deployed, connect to it using Remote Desktop (RDP).
- Launch `Server Manager` on the Windows VM.
- In `Server Manager`, choose `Add roles and features`.
- Proceed through the wizard, and in the `Roles` section, select `Web Server (IIS)`.
- Finish the wizard and wait for the IIS role to install.

### 2. Navigate to the wwwroot Directory and Upload your HTML File
![Screenshot (2)](https://github.com/user-attachments/assets/385ae002-eb17-4612-bb51-8a7dd3545840)

- Once IIS is installed, go to the following path on your VM: `C:\inetpub\wwwroot`.
- In the `wwwroot` folder, delete the default `index.html` file (if it exists).
- Upload your frontend HTML file into the `wwwroot` folder.
- Rename your HTML file to `index.html`.

### 3. Verify the Website
![Screenshot 2024-10-03 132048](https://github.com/user-attachments/assets/64319883-24c6-4396-8f51-f1199dc1d171)

- In a browser, navigate to your Web VM’s public IP address: `http://<your-web-vm-public-ip>`.
- You should now see your frontend site, accessible via the Web VM's public IP.

For example, if your Web VM's public IP is 52.183.21.18, enter http://52.183.21.18 in your browser to view the site.

## Step 5: Create an App Registration
![image](https://github.com/user-attachments/assets/875d4b1f-6b9e-4447-a7d5-72cdced19b6b)

1. In the search bar, search for `App Registration` and hit `New Registration`.
2. Enter the **Name** and hit `Register`.
3. Go to manage, inside the app registered and click on `Certificates & Secrets`.
4. Click on `New client secret`. Enter the description and click on `Add`.
5. Copy the **Value** in Notepad.

## Step 6: Add Custom Role
![image](https://github.com/user-attachments/assets/c75e3cab-be56-42aa-8b70-426e17077955)

1. Go to Subscriptions, inside the subscription click on `Access control IAM`.
2. Click `Add` then `Add Custom Role` and then click on `Start from json`.
3. paste the below json file after entering your **SUBSCRIPTION_ID** and hit 'Review + Create`.
```json
       {
  "properties": {
        "roleName": "Minimum M2VM permissions role",
        "description": "This role contains the bare minimum of Azure IAM permissions to support M2VM flow",
        "assignableScopes": [
              "/subscriptions/SUBSCRIPTION_ID"
        ],
  "permissions": [
              {
              "actions": [
                    "Microsoft.Resources/subscriptions/resourceGroups/write",
                    "Microsoft.Resources/subscriptions/resourceGroups/read",
                    "Microsoft.Resources/subscriptions/resourceGroups/delete",
                    "Microsoft.Compute/virtualMachines/read",
                    "Microsoft.Compute/virtualMachines/write",
                    "Microsoft.Compute/virtualMachines/deallocate/action",
                    "Microsoft.Compute/disks/read",
                    "Microsoft.Compute/snapshots/delete",
                    "Microsoft.Compute/snapshots/write",
                    "Microsoft.Compute/snapshots/beginGetAccess/action",
                    "Microsoft.Compute/snapshots/read",
                    "Microsoft.Compute/snapshots/endGetAccess/action"
              ],
              "notActions": [],
              "dataActions": [],
              "notDataActions": []
              }
        ]
  }
  }
  ```

## Step 7: Add Role Assignment
![image](https://github.com/user-attachments/assets/1d29221c-45f9-4b82-af30-1eb83320db89)

1. Go to Subscriptions, inside the subscription click on `Access control IAM`.
2. Click `Add` then `Add role assignment`.
3. Inside the `Role`. Search for your custom role like for me it is **Minimum M2VM permissions role**.
4. Inside the `Members`. click on `select members` and search for the `App Registration Name` for me it is **TestingGCPMigration**.
5. Click `Review + Assign`.

## In the Google Cloud

## Step 1: Click on Migrate to Virtual Machines
![Screenshot 2024-10-04 100810](https://github.com/user-attachments/assets/4a38b709-6e31-4acc-9dcd-77f91b89c194)

1. In the GCP, search for `Migrate to Virtual Machines`.
2. Click on `sources` and then `Add Source`, then hit the `Add Azure Source`.
3. Enter the **Name**, **GCP Region**, and **Azure Location** where the VM exists. Then, enter the following details from **Azure**:
   - **Subscription ID** of Azure.
   - **Client ID** from `Azure`, available inside `App Registration`.
   - **Tenant ID** from `Azure`, available inside `App Registration`.
   - **Client Secret Value** that was copied in `Step 5` of Azure.
4. Click on `Create`.

## Step 2: ADD the VM Migration
![image](https://github.com/user-attachments/assets/f745f590-7802-454d-a363-4af737cbbd68)

1. Wait till the `source` status becomes **Active**.
2. Select the VMs, click on `Add Migrations` and then click on `VM Migration`.
3. Go to `VM Migration`. Select the VMs click on `Migration` and then hit `Start Replication`.

## Step 3: Edit Target Details
![image](https://github.com/user-attachments/assets/151ba4f6-4452-4274-b6d1-730110024e27)

1. Click on `Edit target details` inside the `VM Migrations` after selecting the VMs.
2. Enter the **Instance Name**, **Project** and **Zone** inside the `General`.
3. Inside the `Machine Configuration`
   - Choose Machine type series - **e2**
   - Choose Machine type - **e2-highcpu-2**
   - On host maintenance, choose **Migrate VM Instance**
   - Set the Automatic Start to **OFF**
4. Inside the `Networking`, choose everything **Default**.
5. Inside the `Additional Configuration`
   - Choose service account as - **Compute Engine default service account**
   - disk type - **Balanced**
6. Click on `Save`.
   
   
