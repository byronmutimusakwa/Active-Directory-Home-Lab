# Lab Setup

Using VMWare Workstation 17 Player, set up the following virtual machines:

- 1 x Windows Server 2022 *(Domain controller)*
- 1 x Windows 10 Enterprise — *User-machine 1*
- 1 x Windows 10 Enterprise — *User-machine 2*
- 1 x Kali Linux — *Attacker*

Each virtual machine should have approximately 2GB of RAM, totaling 8GB RAM for all machines.

Download the required ISO from the official Microsoft Evaluation Center website: https://www.microsoft.com/en-us/evalcenter

# Virtual Machine Setup

Remove floppy drives and set the network to NAT. Perform the following steps for each virtual machine:

**Domain Controller** (OS Installation)

1. Create a new virtual machine.
2. Select the ISO file for the installer disc image.
3. Choose “Windows Server 2022” as the version to install.
4. Power on the virtual machine after creation.
5. In VM Settings, remove the floppy disk and set the network to NAT.
6. Start the virtual machine and press any key to enter setup mode.
7. Select “Windows Server 2022 Standard Eval (Desktop Experience)”.
8. Choose “Custom install” and select the unallocated space for installation.
9. Follow the on-screen instructions to complete the installation.

**Setting up Domain Controller**

1. In the virtual machine settings, go to “View Your PC Name” and rename the PC to something like `yourDomainController`. Restart the virtual machine.
2. Set a password for the PC, such as `CrowdStrike123`.
3. In VMWare Player, go to “Manage” and install VMWare Tools.
4. Open Server Manager and navigate to “Dashboard” > “Manage” > “Add Roles and Features”.
5. Choose “Role-based or feature-based installation” and select the Active Directory Domain Services role. Continue with the installation.
6. After the installation, click the flag icon in Server Manager and select “Promote this server to a domain controller”.
7. Choose “Add a new forest” and enter a root domain name like `ADENVIRONMENT.local`. Proceed with the installation, setting a password for DSRM (Directory Services Restore Mode) and accepting the default options.

**User Machine** (OS Installation)

Perform the following steps for each user machine:

1. Create a new virtual machine.
2. Select the ISO file for the installer disc image.
3. Choose “Windows 10 Enterprise” as the version to install.
4. Power on the virtual machine after creation.
5. In VM Settings, remove the flPoppy disk and set the network to NAT.
6. Start the virtual machine and press any key to enter setup mode.
7. Select “Custom install” and select the unallocated space for installation.
8. Choose “Domain join instead” and enter an account name (e.g., `lebron james`) and password (`Password23`).
9. Proceed with the installation, selecting “No” for “Do more across devices…” and declining “Get help from digital assistant”. Choose your preferred privacy settings.

**Setting up User Machine**

1. In VMWare Player, go to “Manage” and install VMWare Tools.
2. In the virtual machine settings, go to “View Your PC Name” and rename the PC (e.g., `LeBron-PC`). Restart the virtual machine.
3. Repeat these steps for the second user machine, using the account name `michael jordan`, password `Password23`, and PC name `Michael-PC`.

# **Active Directory (AD) Setup, Groups, and Policies**

Log in to the Windows Server

(Domain Controller) and perform the following steps:

1. Open Server Manager and go to “Tools” > “Active Directory Users and Computers”.
2. Right-click on the root domain (e.g., `ADENVIRONMENT.local`) and select "New" > "Organizational Unit" > "Groups".
3. Move all users (except Administrator and Guest) from the “Users” directory to the “Groups” directory.
4. Right-click under the “Users” directory and create the following users:
- Uncheck “User must change password at next logon” and check “Password never expires”.
- Copy the “administrator” domain admin account with the login `adam` and password `2024Password@!`.
- Copy the “administrator” domain admin account with the logon `SQLService`, password `MYpassword123#`, and description "Password is MYpassword123#".
- Create a new user domain account with the logon `lebron` and password `Password23`.
- Create a new user domain account with the logon `michael` and password `Password23`.

Note: It is generally not recommended to assign domain-admin rights to service accounts like SQL Service. In real-life situations, granting domain-admin rights to such services should be avoided. Similarly, storing passwords in the description field is not a secure practice.

# **Configuring File Server (Opening SMB)**

Enable SMB file sharing and open ports 445 and 139 for later lab activities.

1. Open Server Manager and go to “Dashboard” > “File and Storage Services” > “Shares”.
2. Click “New Share” > “SMB Share — Quick” > “Next”.
3. Set the share name as “hackme” and follow the prompts to create the share.

# **Creating Service Principal Name (SPN)**

Set up for a Kerberoasting attack.

1. Open the Command Prompt as an administrator.
2. Use the following command to set the SPN: `setspn -a yourDomainController/SQLService.ADENVIRONMENT.local:60111 ADENVIRONMENT\SQLService`.
3. Check if the SPN is set using the command: `setspn -T ADENVIRONMENT.local -Q */*`.

# **Group Policy Configuration**

Disable Windows Defender for lab purposes (note: topics like AV bypass and evasion are not covered in this lab).

1. Open Group Policy Management as an administrator.
2. Expand the Forest > Domains > `ADENVIRONMENT.local`.
3. Right-click and create a new Group Policy Object (GPO) named “Disable Windows Defender” in this domain.
4. Edit the newly created GPO.
5. Navigate to “Computer Configuration” > “Policies” > “Administrative Templates” > “Windows Components” > “Windows Defender Antivirus”.
6. Double-click on “Turn off Windows Defender Antivirus” and enable the policy.

# **Connecting all Machines**

Perform the following steps on both machines:

1. Create a folder and set up a network share.
2. On the first machine, give the first domain user local administrator rights.
3. On the second machine, give both domain users local administrator rights.
4. Create a network share folder.
5. Create a folder named “share” in the C: drive.
6. Right-click on the folder, go to “Properties” > “Sharing” > “Share” > “Share” > “Yes, turn on network discovery…”.
7. Configure network settings.
8. Open network & internet settings, change adapter options, and go to IPv4 > Preferred DNS Server.
9. Enter the IP address of the Domain Controller.
10. Join the domain.
11. Go to “Access work or school” > “Connect” > “Join this device to local Active Directory domain”.
12. Enter the domain name as `ADENVIRONMENT.local`.
13. Join the domain as the Administrator with the password `P@$$w0rd`.
14. When prompted to add an account, skip and restart the machine.
15. Upon restart, select the other user and log in as the domain user (e.g., `lebron` with the password `Password23`).
16. Give local administrator rights:
- Go to “Computer Management” > “Local Users and Groups” > “Groups”.
- Double-click on “Administrators”.
- Add the user (e.g., `lebron`), check the name, apply, and click OK.

17. Turn on Network Discovery:

- Go to “Computer” > “Network”.
- Click OK when prompted and click “Turn on network discovery…” in the top menu bar.

18. Verify that both computers have joined the domain:

- On the Domain Controller, open “Active Directory Users and Computers” > `ADENVIRONMENT.local` > "Computers".
- Check if both computers are added.

19. Set up for mitm6 attack lab example:

- Login to the Domain Controller.
- Open Server Manager > Dashboard > “Add roles and features” > Next.
- Select “Active Directory Certificate Services” > Add features > Next.
- Choose to restart the destination server automatically and click Install.
- Click the flag notification icon and configure Active Directory services on the destination server.
- Provide the credentials as `ADENVIRONMENT\\\\Administrator`.
- Proceed with the configuration, selecting the Certification Authority and setting the validity to 99 years. Click Next and configure the settings.

# Conclusion

Building an Active Directory Environment is an excellent opportunity to expand knowledge and develop skills in this critical technology. Understanding Active Directory and its features is crucial for IT professionals in network administration, security, and system management roles. It is beneficial to invest in ones professional growth by acquiring skills in Active Directory to open doors to exciting opportunities in the IT field and advance your career