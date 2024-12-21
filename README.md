# On-premises Active Directory Deployed in Azure Cloud
![68747470733a2f2f692e696d6775722e636f6d2f6432324648496d2e706e67](https://github.com/user-attachments/assets/79aec912-7d74-4d95-8149-4de96019b268)

## Environments and Applications Used:
* Azure Portal (Virtual Machines/Compute)
* Remote Desktop
* Active Directory Domain Services 
* Powershell

## Operating Systems Used
* Windows 2022 Server (DC-1)
* Windows 10 (Client-1)

## Deployment and Configuration Steps
### Step 1: Set up resources in Azure
![create 2 VMS (1)](https://github.com/user-attachments/assets/f09a4886-aec9-4fee-888e-eb7897f254c1)
![set domain to static ip address (2)](https://github.com/user-attachments/assets/10b01b85-508a-4445-8d49-3be3bb890a5b)

First, create two virtual machines under the same v-net in the Azure portal. The first VM will be the domain controller named “DC-1” and the second VM will be named “Client-1” (make sure both VMs are under the same v-net). We will ensure that the DC-1 NIC IP address will be changed to Static due to it offering Active Directory Domain services. We need to ensure that Client-1 can connect to the same domain as DC-1.

* DC-1 username: lexuser8
* Client-1 username: labuser9

### Step 2: Ensure Connectivity Between the Client and Domain Controller
Remotely connect to Client-1 and ping DC-1’s Private IP  Address

>RUN COMMAND: ping 10.0.0.4

![-ping (3)](https://github.com/user-attachments/assets/79850a1e-62a6-4c90-8c50-cf8eaddfd122)

Notice how we cannot connect to DC-1, we will run another command to continuously ping DC-1.

>RUN COMMAND: ping 10.0.04 -t

We will then remotely connect to DC-1 to make sure that the echos start connecting to Client-1. To do this we will enable the ICMPv4 on DC-1’s local firewall. 

![firewall (4)](https://github.com/user-attachments/assets/51704bc6-8d56-419f-9bbd-c888f84d342d)

>type “run” in Windows search bar > type in “wf.msc” > click on Inbound Rule > click on Protocol column > search for ICMPv4 > right click to enable the networking diagnostics echo requests.

![echo reached (5)](https://github.com/user-attachments/assets/bbc84bd8-c3a0-46e3-86ef-24330d0e8278)

Go back to Client-1’s desktop to see if the ping is successful.

### Step 3: Install Active Directory

Log back to DC-1 and install Active Directory Domain Services on the Server Manager dashboard.

![install A (6)](https://github.com/user-attachments/assets/c79e3aff-3dea-4a13-bf58-9bac6fd724c1)

>Add roles and features > Active Directory Domain Services > add features > install

![add domain admin (7)](https://github.com/user-attachments/assets/b39842e9-3d3d-4dd8-8a66-ade382839132)

We will promote DC-1’s VM as a Domain Controller (DC) by setting up a new forest as “mydomain.com”.

DC-1 should restart the computer and log back into as "mydomain.com\lexuser8".

### Step 4: Create an Admin and Employee Accounts in Active Directory

In the Service Manager dashboard at the top right click tools and select Active Directory Users and Computers. We will then begin to create Organizational Units (OU) in our domain. We will create two folders named: _EMPLOYEE and _ADMINS. 

![add admin (8)](https://github.com/user-attachments/assets/3f6728b8-85c1-4885-be63-d2c2ca1896f0)

>Create a new employee named “Alexis Ali” under _ADMIN > create password > password never expires > add user to member of “Domain Admins” Security Group > apply > ok

Log back in as mydomain.com\alexis_admin with the same password in DC-1.

### Step 5: Join Client-1 to your domain

![add client-dns (9)](https://github.com/user-attachments/assets/d75df052-d34b-4b77-abbd-28939f1bcafc)

From the Azure portal, set Client-1’s DNS server to DC-1 Private IP Address (10.0.0.4).

Restart Client-1 VM within the Azure portal. 

Log back in to Client-1 as the original local admin.

![join dm client-1 (10)](https://github.com/user-attachments/assets/a5878fb8-5c7b-4866-b0c1-d65ad55e243a)

> Open System Settings > rename this PC (advanced) > add domain (mydomain.com) > fill in with alexis_admin username and password to join domain.

The Computer should restart.

![computer joined (11)](https://github.com/user-attachments/assets/5db1d547-ac87-416b-8d7d-1d823bf0353d)

Log back into the DC-1 remotely to verify that Client-1 is shown in Active Directory Users and Computers inside the “Computers” folder (container) on the domain.

Create a new OU named “_Clients” and then drag and drop Client-1 into the folder. 

### Step 6: Setup Remote Desktop for non-admin users on Client-1

We need to allow users access to be able to use Remote Desktop on Client-1.

Log back in to Client-1 on Remote Desktop as "mydomain.com\alexis_admin".

![step 6 (12)](https://github.com/user-attachments/assets/bfc8d826-5d51-4148-b25a-e6a7f3bdc5a6)

>open system properties > click Remote Desktop > under user accounts click select that can remotely access this PC” > enter “Domain Users” in empty box > click check names (should now be underlined) > ok


You can now log into Client-1 as normal. 

### Step 7: Create a bulk of additional users and log into Client-1 with a random user

![create user accounts (13)](https://github.com/user-attachments/assets/2b58df1d-d713-4a38-be7f-cb8f56f0a639)

>Login to DC-1 as alexis_admin > open Powershell ISE as an administrator > create a new file and paste the script 

Here is a link of the script where I got the user list from: [AD_PS/Generate-Names-Create-Users.ps1 at master · joshmadakor1/AD_PS](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1)

You can change the password and add as many users you want when you review the script. 

*sidenote when I created the OU folder for “_EMPLOYEES” I entered “_EMPLOYEE” so make sure that the script matches the folder and easy passwords you want to use.

>Run the script > save > check to see if users have been added to the "_EMPLOYEE" folder.

These are just regular users, not admins. 

![pick user (14)](https://github.com/user-attachments/assets/4f6d16aa-0815-423a-91cc-164037afb976)

Now let’s try to log into one of the new random users we created on Client-1.

![new user login !!!! (15)](https://github.com/user-attachments/assets/aa4c0e61-68b5-4370-a557-c4908e921067)

We can now log into Client-1 :)

![WHO AM I !!!! (16)](https://github.com/user-attachments/assets/a7a679be-e08f-4c2b-b598-4bdabe883d99)

>RUN THE COMMAND: whoami

Username should match the random user we created in the script. 






