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

## Step 5: 
