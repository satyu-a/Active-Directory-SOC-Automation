# Active-Directory SOC Automation

## Objective

This lab aims to design and automate a Security Operations Center (SOC) lab that integrates Active Directory as the core identity infrastructure, simulating realistic enterprise environments. The project focuses on detecting, analyzing, and responding to adversary attack chains by automating log collection, correlation, and response workflows. This includes mapping simulated threats to MITRE ATT&CK techniques within Active Directory, automating detection pipelines, and orchestrating incident response to improve speed, accuracy, and resilience of security operations.

### Skills Learned

### Tools Used

### Procedure
1. Login to the Virtual Machine via RDP/SSH
2. VPC Configuration
3. Active Directory Setup
4. Splunk Enterprise Configuration
5. Tools Configuration
6. Malware Analysis (Mimikatz)
7. Automation and enrichment

## Logical Diagram of the Setup

> [!NOTE]
> - At the time of drafting this readme, Windows standard 2022 OS is not available in India location, so Atlanta, USA server is used.
> - Server Setup is not included in this repo but the configuartions are.
> - Refer to [SERVER SETUP](https://github.com/satyu-a/SOC-automation?tab=readme-ov-file#step-2-ubuntu-server-installation-on-vultr) for guide. Everything except the OS selection is same.
> - OS is **Windows standard 2022x64**.
> - After Clicking on a deployed server, the server credentials can be accessed from the overview page. These will be used for RDP and SSH logins
> <img width="863" height="945" alt="image" src="https://github.com/user-attachments/assets/fdf25679-94e0-418f-9c6a-2ef57bee739d" />
> - If you have a dynamic public IP for your home network, you might have to keep updating the VULTR firewall rules as the IP keeps changing.

## Machine Configurations:
1. Cloud Hosting: VULTR
2. Domain controller:
    - Hostname: DEFYER-ADDC-01
    - Specs: Atlnta USA location, Shared CPU, 2vCPU, 4GB RAM, 80GB SSD, Windows Standard 2022x64
    - VULTR firewall rule 1: Protocol - SSH, Port - 22, Action - Accept, Source - Custom, [Your_IP_Address] 
    - VULTR firewall rule 2: Protocol - MS RDP, Port - 3389, Action - Accept, Source - Custom, [Your_IP_Address]
2. Test Machine:
    - Hostname: DEFYER-ADTM-01
    - Specs: Atlnta USA location, Shared CPU, 1vCPU, 2GB RAM, 55GB SSD, Windows Standard 2022x64
    - VULTR firewall rule 1: Protocol - SSH, Port - 22, Action - Accept, Source - Custom, [Your_IP_Address] 
    - VULTR firewall rule 2: Protocol - MS RDP, Port - 3389, Action - Accept, Source - Custom, [Your_IP_Address]
3. Splunk Enterprise:
    - Hostname: DEFYER-ADSP-01
    - Specs: Atlnta USA location, Shared CPU, 4vCPU, 8GB RAM, 160GB SSD, Ubuntu 22.04
    - VULTR firewall rule 1: Protocol - SSH, Port - 22, Action - Accept, Source - Custom, [Your_IP_Address] 
    - VULTR firewall rule 2: Protocol - MS RDP, Port - 3389, Action - Accept, Source - Custom, [Your_IP_Address]
4. VPC configuration so that all VM can communicate with each other:
    - To put all the VM in the same private network, go the VM > Settings > VPC Networks > Enable VPC <br>
        <img width="881" height="899" alt="image" src="https://github.com/user-attachments/assets/73560235-843b-4814-b9c0-a57eecca07d5" />
    - Now a private IP has been assigned to the VM. Keep a note of it:<br>
       <img width="695" height="775" alt="image" src="https://github.com/user-attachments/assets/ce8d9b9d-e795-4bea-b685-854b2f71170b" />





## Procedure

### Login to the server using RDP(For windows VM) and SSH(for Ubuntu VM)

- Using RDP:
  1. Open RDP on the host machine by searching in thr start menu "RDP":
     
     <img width="787" height="675" alt="image" src="https://github.com/user-attachments/assets/7f16e995-07ba-4fed-88dc-ac8aeda4998c" />     
  2. Enter the Server IP and click on "Show Options"
  
     <img width="416" height="271" alt="image" src="https://github.com/user-attachments/assets/77e08a17-9fed-4a37-8e57-70cad8161379" />
  3. Enter the Username of the server and click "Connect":
  
     <img width="408" height="493" alt="image" src="https://github.com/user-attachments/assets/8dab2391-8d77-47a1-8855-6735e49e233f" />
  4. Enter the password of the server:
  
     <img width="460" height="375" alt="image" src="https://github.com/user-attachments/assets/02daeca6-5126-42b5-94e4-19b73614ae38" />
  5. "Yes" if prompted
  
     <img width="410" height="490" alt="image" src="https://github.com/user-attachments/assets/747bfe30-5629-4a72-a6a1-b3faf1f4fb67" />
  6. And it should show the server Desktop:
  
     <img width="798" height="634" alt="image" src="https://github.com/user-attachments/assets/10341f91-96b4-40b2-aef3-b3d993b0601e" />

- Using SSH:
  1. Open Powershell on the host machine and enter the command replace the username and server_IP with your own, enter "yes" if prompted:

          ssh <Username>@<SERVER_IP>
     <img width="845" height="445" alt="image" src="https://github.com/user-attachments/assets/307155db-8e3e-4abc-98f8-7b1208b7d287" />
  2. Enter the password and click "Enter", you should be able to see the home directory of the server:

     <img width="856" height="726" alt="image" src="https://github.com/user-attachments/assets/ff51d1f0-9cea-41b6-a7dd-ccdfab6aa132" />

### VPC Connection configuration
1. Connect to the server using RDP (Windows) and open Powershell and run command. If the private IP of the VM is not the same assigned by the VPC, the connection between VM will not establish. In this case, its not same so I wil be manually configuring it:

        ipconfig
    <img width="1035" height="567" alt="image" src="https://github.com/user-attachments/assets/1a961691-effe-4559-9dc3-46c2dc71c400" />
2. Open control pannel and go to "**Network nad Internet > Network and Sharing Center > Change Adapter Settings**"

    <img width="1025" height="619" alt="image" src="https://github.com/user-attachments/assets/7b3345f6-af50-4d9c-aa84-95a414cc7627" />

3. Select Ethernet Instance 0 2 as this is the one we need to edit according to the powershell results and open "Properties":

   <img width="1068" height="652" alt="image" src="https://github.com/user-attachments/assets/902f7649-bf5c-44f9-8021-c9d9e4e78ab8" />

4. Select TCP/IPv4 and select **"Use the following IP address"**. Enter the IP address form the VPC pannel:
   
   <img width="1621" height="690" alt="image" src="https://github.com/user-attachments/assets/b4f52d00-4b3a-4a8a-a36d-bd3559e496be" /><br>
   <img width="403" height="453" alt="image" src="https://github.com/user-attachments/assets/1dc401ad-2744-4305-a75c-e46f35d8352c" />

5. Click "OK" and close everything. On the powershell rerun the command and check if IP has been updated:
   
   <img width="1049" height="575" alt="image" src="https://github.com/user-attachments/assets/b7e0ed40-6d96-4613-8a1e-727b436e4af2" />
   
6. Do the above steps for the other Windows VM too.

7. In ubuntu, connect using SSH and run the command. The IP address should be the same as VPC:

           ip a
   <img width="849" height="731" alt="image" src="https://github.com/user-attachments/assets/b58a13c4-33b3-4f6a-9e3c-0e4e0dc59aeb" />

8. Use ping to check if all the VMs are in the same VPC and able to talk to each other:

           ping <VPC_Private_IP_Address>
   <img width="842" height="172" alt="image" src="https://github.com/user-attachments/assets/6c57ce61-5982-48de-babf-9ad26c1cf5e2" />

### Active Directory setup
- Connect to DEFYER-ADDC-01 sever via RDP and go to "Server Manager". We will be installing Active directory and promote this server to the     domain controller:
  
    <img width="1073" height="644" alt="image" src="https://github.com/user-attachments/assets/5f33a6a9-1800-4bab-bf5b-7f1f5eea158f" />
- Go to **Manage > Add roles and Features**:

    <img width="1065" height="641" alt="image" src="https://github.com/user-attachments/assets/1b31ff56-5ec2-4862-8d96-ae5718ff3377" />
- Click **Next**
  
    <img width="789" height="570" alt="image" src="https://github.com/user-attachments/assets/9365c2e3-f913-4627-864e-80675d7112af" />

- Click **Next**

    <img width="784" height="569" alt="image" src="https://github.com/user-attachments/assets/d59e6209-61a2-40fb-98ad-48ed4ccfc180" />

- Click **Next**

    <img width="785" height="571" alt="image" src="https://github.com/user-attachments/assets/2e27ef93-48a7-49a1-b56d-1bc0dcda83e9" />

- Select **Active Directory Domain Services**:

    <img width="784" height="561" alt="image" src="https://github.com/user-attachments/assets/0b96e405-cc99-4030-a0b7-cdc2b9d0c2b0" />

- Click **Add Features**:

    <img width="417" height="432" alt="image" src="https://github.com/user-attachments/assets/91f53315-5daa-446c-b7a5-d738265bd3ab" />

- Click **Next**:
  
    <img width="784" height="562" alt="image" src="https://github.com/user-attachments/assets/73b5dac9-5c59-4e46-8c2a-a5cb19931e05" />

- Click **Next**:

    <img width="783" height="556" alt="image" src="https://github.com/user-attachments/assets/94738fdb-1c40-499a-802a-10d1d748ebfc" />

- Click **Next**:
  
    <img width="783" height="562" alt="image" src="https://github.com/user-attachments/assets/ce9d2d52-a702-4c6b-9820-2cd5044711e4" />

- Click **Install**:

    <img width="783" height="559" alt="image" src="https://github.com/user-attachments/assets/710db312-fc0f-4d73-815f-56d6f182a6dd" />

- Click **Close** once the installation is complete:

    <img width="788" height="562" alt="image" src="https://github.com/user-attachments/assets/710bb7db-afeb-4cac-8990-d1e2dcdc0df6" />

- Click on the Flag icon on the Server manager:

    <img width="1074" height="646" alt="image" src="https://github.com/user-attachments/assets/b414d4de-dcd4-46ca-9116-ad8fdaaaa0be" />

- Promote the server to Domain Controller:

    <img width="1070" height="645" alt="image" src="https://github.com/user-attachments/assets/55b5d6f9-d2df-4c43-aae3-9c35c6c971dc" />

- Add a New Forest and enter a domain name then click **Next**:
  
    <img width="764" height="563" alt="image" src="https://github.com/user-attachments/assets/0156f235-2fcd-491d-8dab-242ac0303086" />

- Enter a secure password and click **Next**:

    <img width="758" height="562" alt="image" src="https://github.com/user-attachments/assets/9768b4fd-d89f-4f19-b38b-e9ef01fc262b" />

- Click **Next**:

    <img width="761" height="563" alt="image" src="https://github.com/user-attachments/assets/60dfbfc0-b1b2-4cf7-8ff9-ca65b50c8867" />

- Click **next**:

    <img width="757" height="563" alt="image" src="https://github.com/user-attachments/assets/eb52d0d2-3d7d-4f6c-a11b-8c944e27e5bf" />

- Click **Next**:

    <img width="760" height="564" alt="image" src="https://github.com/user-attachments/assets/8c03d3d9-eb63-4a82-9a56-9ed381bb3dd0" />

- Click **Next**:

    <img width="762" height="562" alt="image" src="https://github.com/user-attachments/assets/9f7c9671-b169-462a-b640-590594ad5b4e" />

- Click **Install**:

    <img width="764" height="562" alt="image" src="https://github.com/user-attachments/assets/1ac9ef3d-6985-492d-b886-8cb455f45fe5" />

- Click **Close** if prompted and the session will terminate. Reconnect via RDP and wait for the configurations to finish. Once the server    is rebooted, search for Active Directory in the start menu and if you can see the Admin center, the installation was successful:

    <img width="1040" height="858" alt="image" src="https://github.com/user-attachments/assets/d938d208-7245-4f29-83f0-ccc910ddb8dd" />

- Lets create a few user accounts for the AD. Search for Users and open the AD users and Computers:

    <img width="794" height="685" alt="image" src="https://github.com/user-attachments/assets/4c0546f5-1d2a-4dc5-9e9e-322ab8da8dc2" />

- Under the active directory name, **Users > New > User** to create a new user;

    <img width="1071" height="593" alt="image" src="https://github.com/user-attachments/assets/6fb88d75-044e-4aa9-8630-6c0c521305b1" />

- Add user info and click **Next**:

    <img width="439" height="378" alt="image" src="https://github.com/user-attachments/assets/8870aa4a-f9f6-48cd-b266-ec87719d42fa" />

- Enter a Password and keep the first box unchecked as we are in a lab environment:

    <img width="437" height="374" alt="image" src="https://github.com/user-attachments/assets/7a72d28c-8f4c-4bc4-9e9c-5e1bcdb7c234" />

- Click **Finish**:

    <img width="434" height="378" alt="image" src="https://github.com/user-attachments/assets/c90dc1ee-74a4-4fc6-a797-d9949baa9e83" />

- Now connect to the target machine DEFYER-ADTM-01 via RDP and go to **About** in **Settings** and click on **Rename this PC**:

    <img width="1197" height="616" alt="image" src="https://github.com/user-attachments/assets/25f3db76-7982-4e63-923e-4c02585a9fdb" />

- Before adding this server to the domain, we need to add the Domain DNS to the IPv4 configuration or the server won't be able to 'see' the   domain. Add the Domain Controller's ,i.e, DEFYER-ADDC-01's internal IP address in the DNS:
  
    <img width="1263" height="698" alt="image" src="https://github.com/user-attachments/assets/21b7890f-4fb8-4640-9766-1c474b189ff4" />

- Click **Change** if you don't see the Domain options.

    <img width="405" height="471" alt="image" src="https://github.com/user-attachments/assets/e83b371e-cff8-4e22-a633-4bc8f78c03d5" />

- Enter the domain name and clock **OK**:

    <img width="409" height="464" alt="image" src="https://github.com/user-attachments/assets/c962822e-1146-4777-a423-4ef66e36c007" />

- Enter the VULTR DEFYER-ADDC-01 (Domain Controller) credentials as it is the admin account:

   <img width="458" height="300" alt="image" src="https://github.com/user-attachments/assets/618210a5-7d35-4781-b0eb-0941ccaad8aa" />

- Successful connection looks like this:

    <img width="1250" height="691" alt="image" src="https://github.com/user-attachments/assets/cf4ae955-1c58-48ab-9b7a-a24af59ee169" />

- Click **OK** if prompts to restart:

    <img width="1258" height="698" alt="image" src="https://github.com/user-attachments/assets/31434814-7c3f-48e5-a228-2dd6afdb9beb" />

- Close all windows and restart when prompted:

    <img width="1268" height="727" alt="image" src="https://github.com/user-attachments/assets/e4c8a3ff-85c0-4ea0-8ed6-e97e44604c2f" />

- Now to Reconnect via RDP and to use the user account we created to login, we nned to enable RDP login for the account using the console     provided by VULTR. On the VULTR dashboard of the target machine, open the console:

    <img width="1712" height="919" alt="image" src="https://github.com/user-attachments/assets/75192b9f-e3d3-4d01-87eb-80076e8f4102" />

- Expand the controls and send the unlock command:

    <img width="1016" height="756" alt="image" src="https://github.com/user-attachments/assets/67553090-e4c1-4b2e-8dd0-8663b309cd5b" /><br>
    <img width="974" height="720" alt="image" src="https://github.com/user-attachments/assets/15944484-2967-46dd-b9e5-614e2867ceb6" /><br>
    <img width="974" height="724" alt="image" src="https://github.com/user-attachments/assets/b29f5878-71be-47b3-9278-3254863b16cd" />

- Select **Other user** and enter the credentials of the user we created in the AD:

    <img width="1910" height="1042" alt="image" src="https://github.com/user-attachments/assets/328ef01d-cd86-4cef-9de7-47f7fd2a274b" />

- In the start menu search for "Allow remote connections" and select it:

    <img width="1625" height="1009" alt="image" src="https://github.com/user-attachments/assets/961eb35e-2c05-46cb-9531-fc700f0b2d94" />

- Click **Show Settings**:

    <img width="958" height="747" alt="image" src="https://github.com/user-attachments/assets/1e5b25bd-efe2-4246-9dda-a836c00bc89f" />

- Enter VULTR ADDC-01 (Domain Controller) credentials:

    <img width="366" height="381" alt="image" src="https://github.com/user-attachments/assets/92d0345c-f0a7-4345-a69a-c193287a95c8" />

- Click **Select Users**

    <img width="323" height="378" alt="image" src="https://github.com/user-attachments/assets/10a396e6-3215-421c-b6b6-5f027f31ecf8" />

- Click **Add**

    <img width="323" height="319" alt="image" src="https://github.com/user-attachments/assets/eb1ace62-7ee9-402c-8a7b-ca1463a037b7" />

- Enter the Username of the USer created in the AD and click **Check Names** and then **OK** and close everything.

    <img width="399" height="286" alt="image" src="https://github.com/user-attachments/assets/08891134-3630-4e7a-a4ff-039cdfc6a39a" />

- Now open RDP on the local system and click **More options** then enter the IP address of the ADTM-01(target Machine) and check the box "Always Ask for credentials" to edit the username. To logine as user, the username will be YOUR_DOMAIN_NAME\USERNAME and enter the password for the user:

    <img width="405" height="499" alt="image" src="https://github.com/user-attachments/assets/4fd42c54-e9c7-48eb-8504-0c78120e61c5" /><br>
    <img width="460" height="371" alt="image" src="https://github.com/user-attachments/assets/e312a68c-e73a-4cdb-8cd7-af2cdb5fb296" />

- Active Directory has been successfully configured with a user added to it.


### Splunk Enterprise Configuration
- Login to the ADSP-01(Splunk Server) via SSH:

    <img width="847" height="646" alt="image" src="https://github.com/user-attachments/assets/478666fb-acaa-4505-b5f8-f4cd5a90b30c" />

- Run update and upgrade command to make sure the Ubuntu packages are up to date

        apt-get update && apt-get upgrade -y

- On the local machine, navigate to splunk's website using this link [Splunk Signup](https://www.splunk.com/en_us/form/sign-up.html?redirecturl=https://www.splunk.com/).

    <img width="1637" height="901" alt="image" src="https://github.com/user-attachments/assets/499ed969-92b0-433c-bb8f-90dc30ce9a50" />

- Create a splunk enterprise account and login, then Click on **"Trials and Downloads"** on the top right corner. Then click on **"Get my free Trial"** button below the **"Splunk Enterprise"** section.

    <img width="1561" height="925" alt="image" src="https://github.com/user-attachments/assets/60fe043f-d1e9-4d31-8d5b-a77299f9285b" />

- It will take you to a downloads page with several splunk versions. Click on **"Previous Releases"** to see all the options. Make sure you select the **"Linux"** tab and click **"Copy wget link"** of the **".deb"** file.

    <img width="1899" height="776" alt="image" src="https://github.com/user-attachments/assets/53f366c4-b5b6-4cb8-8a8d-ad65ea557147" />

- Paste the command in the SSH session Powershell and click "Enter":

    <img width="956" height="270" alt="image" src="https://github.com/user-attachments/assets/9573aa19-eadc-403d-b96c-b4f780a1f6aa" />

- Check if the file is downloaded:

        ls
    <img width="956" height="107" alt="image" src="https://github.com/user-attachments/assets/53659cc7-f811-4154-b6b1-5ee90e284d53" />

- Install the file using **dpkg**

          dpkg -i splunk-10.0.0-e8eb0c4654f8-linux-amd64.deb
    <img width="955" height="85" alt="image" src="https://github.com/user-attachments/assets/9a5cd657-255e-40bb-8626-0b60906e6052" />

- After the installation is complete, head over to sthe Splunk directory and check the contents:

        cd /opt/splunk
        ls
    <img width="955" height="146" alt="image" src="https://github.com/user-attachments/assets/a9178ec5-de90-4dc1-8f83-002d4bfec833" />

- To start the splunk setup, head in the bin directory and run the Splunk binary:

        cd bin
    <img width="959" height="432" alt="image" src="https://github.com/user-attachments/assets/bd2d4d6a-f682-487a-b33d-b3941db8d2e1" />

        ./splunk start
    <img width="937" height="739" alt="image" src="https://github.com/user-attachments/assets/9f4a5c18-3f88-4925-90f5-80cc3f2e11bc" />

- Keep holding the Space bar till you reach at the end of licensing terms and entry "y" to agree

    <img width="850" height="97" alt="image" src="https://github.com/user-attachments/assets/b2868ae3-6d15-4584-897b-ed67c41f4ccb" />

- Enter a username and password to login into the splunk instance. These credentials have nothing to do with the Splunk.com credentials and can be different.

    <img width="880" height="180" alt="image" src="https://github.com/user-attachments/assets/75f7208b-674a-43a8-9dfa-3cc0fad61f7b" />

- A URL to access the Splunk web console is generated and can be accessed from the web browser of the local machine. However instead of the servername, use the public IP of the ADSP-01 server and port 8000.

    <img width="959" height="113" alt="image" src="https://github.com/user-attachments/assets/1b55c592-12d6-43cf-a259-5294e72822a4" />

- Before trying to connect, edit the VULTR firewall rule for ADSP-01 to allow TCP connection on port 8000 for the local machine:

    <img width="1807" height="605" alt="image" src="https://github.com/user-attachments/assets/9b946745-7ee9-4739-839c-1e0544587833" />

- Also check if internal UFW firewall is enabled on the Ububtu OS. If it is, either allow the port or disable the filrewall:

        ufw status
    <img width="957" height="134" alt="image" src="https://github.com/user-attachments/assets/f278e5be-e680-4db6-80e7-27898211e830" />

        ufw allow 8000
    <img width="957" height="199" alt="image" src="https://github.com/user-attachments/assets/d187055a-127d-4ceb-a2ea-4de6af84dd38" />

- Now the Web console should be accessible. Enter the credentials that were created with splunk installation:

    <img width="1901" height="920" alt="image" src="https://github.com/user-attachments/assets/db6c697a-51c3-44be-a931-8de15e50b987" />

- On the dashboard Click on **Administrator > Preferences**:

    <img width="1898" height="986" alt="image" src="https://github.com/user-attachments/assets/0f7f55b2-8442-41fe-8d95-ffcb0a2b8cfd" />

- Select "GMT timezone" and click "Apply":

    <img width="1881" height="938" alt="image" src="https://github.com/user-attachments/assets/0d3ce7a4-017a-4947-bb76-ef579ccf561f" /><br>
    <img width="882" height="691" alt="image" src="https://github.com/user-attachments/assets/183901ab-28e2-4925-9f42-4ee0ecf7ff17" />

- To add an add-on, click **Apps > Find more apps**:

     <img width="1898" height="709" alt="image" src="https://github.com/user-attachments/assets/b543bad7-a9fe-44c5-ba84-c2a2a5a9b8d8" />

- Search for windows add on and install it:

    <img width="1891" height="874" alt="image" src="https://github.com/user-attachments/assets/6aedd875-2fc4-4b27-bdd0-563a205d18e9" />

- Enter your Splunk.com username and password:

    <img width="864" height="772" alt="image" src="https://github.com/user-attachments/assets/2bbdd494-f470-4982-b4fa-bf6417763063" />

- After installation is complete go to **"Settings > Indexes"**:

    <img width="1878" height="811" alt="image" src="https://github.com/user-attachments/assets/82663cea-f3bf-425d-992e-ba563ffbde69" />

- Click on **New Index**:

    <img width="1891" height="819" alt="image" src="https://github.com/user-attachments/assets/cdfa8c98-1332-48ef-b634-530e3138f4ea" />

- This is where all the logs releated to AD will be stored. Name the index and click "Save":

    <img width="1001" height="790" alt="image" src="https://github.com/user-attachments/assets/18ee108a-e9a2-4b81-a844-e080a24cfb66" />

- Navigate to **Settings > Forwarding and Receiving**:

    <img width="1874" height="861" alt="image" src="https://github.com/user-attachments/assets/e70381ee-fedc-44a0-907e-923ded53e85b" />

- Click **Configure Receiving** and enter the port 9997 as this is where the logs will be forwarded to:

    <img width="1876" height="733" alt="image" src="https://github.com/user-attachments/assets/af82f422-e6f4-4da5-8944-842d3f44d760" /><br>
    <img width="1907" height="518" alt="image" src="https://github.com/user-attachments/assets/1cb796e0-0936-43c0-bb31-a188882105cf" /><br>
    <img width="1807" height="547" alt="image" src="https://github.com/user-attachments/assets/f3459cd3-dc8e-4e0a-a347-99a304695739" />


### Splunk universal forwarder configuration
- Go to Splunk.com and login. under the "trials and downloads", select the Splunk **"Universal Forwarder"**:

    <img width="1840" height="781" alt="image" src="https://github.com/user-attachments/assets/9b8a8028-98d7-43d1-b9e1-4e4dae4cbfcd" /><br>

- We are using Windows server 2022 so select that file and download it. Agree to the licensing terms:

    <img width="1879" height="805" alt="image" src="https://github.com/user-attachments/assets/86981d43-8d8b-4bc8-82d7-07ba5775e181" />

- Login to the Target machine (ADTM-01) using Administrator Credentials and copy the Universal Forwarder file over there using copy-paste:

    <img width="405" height="486" alt="image" src="https://github.com/user-attachments/assets/89796c7e-3a4a-457a-a1a5-fbb0e1bae272" /><br>
    <img width="1883" height="1009" alt="image" src="https://github.com/user-attachments/assets/e8efce5f-79f8-4230-a6cf-7fe1ea93c4ef" /><br>
    <img width="1238" height="918" alt="image" src="https://github.com/user-attachments/assets/61dcf8cf-65da-45b0-83f3-e914b103a000" />

- Now run the installer:

    <img width="502" height="389" alt="image" src="https://github.com/user-attachments/assets/a7ea76ab-6d33-4d44-98df-f56e1e5dc503" />
    
- Enter a username:

    <img width="494" height="393" alt="image" src="https://github.com/user-attachments/assets/85e4048c-ce63-4bae-9df5-f7f20d368fb5" /><br>
    <img width="494" height="391" alt="image" src="https://github.com/user-attachments/assets/9377ccdc-91da-49e0-a5c5-0e14e1050111" />

- Enter the internal IP of the ADSP-01 server:

    <img width="495" height="391" alt="image" src="https://github.com/user-attachments/assets/77026d7d-afa5-4b32-ac6a-5319b47eeabc" /><br>
    <img width="495" height="397" alt="image" src="https://github.com/user-attachments/assets/abf0ad80-a38e-4fde-b8e6-31b309582541" /><br>
    <img width="497" height="393" alt="image" src="https://github.com/user-attachments/assets/141a1d6e-1595-4dce-a38c-e3533aaa6b2d" />

- Repeat the above steps for the ADDC-01 server.

- Open powershell and connect to ADSP-01 server via SSH and add a firewall rule to allow traffic on port 9997:

        ufw allow 9997
    <img width="848" height="196" alt="image" src="https://github.com/user-attachments/assets/ae4f23ff-520b-4f3c-9659-d5a8cda0f61e" />
    
- Once Splunk Forwarder is setup on both machines, head over to the splunk Web Console and navigate to **Apps > Searching and Reporting**. Enter the query in the search field **"index=Your_index_name"**:

    <img width="1885" height="942" alt="image" src="https://github.com/user-attachments/assets/fa170ac2-ef97-4b78-9e39-fa4131a12dc3" /><br>
    <img width="1913" height="844" alt="image" src="https://github.com/user-attachments/assets/83b87edc-c3c3-4852-92c2-d65ae68251d5" />

- Once the logs are fetched, click on **host** and if the Splunk Forwaarder was successfully installed, both servers should be visible there:

    <img width="1892" height="934" alt="image" src="https://github.com/user-attachments/assets/17a08257-df74-4f28-95d9-2e29c993fc87" />

- 

    



    
























      













    

    
   

