# **Comprehensive Wazuh Deployment on WSL2 and Windows Host**

This guide provides a detailed walkthrough for setting up the Wazuh central components (Manager, Indexer, Dashboard) on a WSL2 Linux distribution and configuring the Windows host to report to it via the Wazuh Agent.

The focus is on establishing **reliable network communication** to overcome the dynamic IP assignment within WSL2.

## **Phase 1: WSL2 Preparation and Reliable Addressing**

### **Step 1: Verify WSL2 Distribution and Install Prerequisites**

Ensure you are running a WSL2 instance (e.g., Ubuntu). WSL2 is required for full system call compatibility.

1. **Check WSL Version (in PowerShell):**  
   wsl \-l \-v

   *(The version column for your distribution should show '2'.)*  
2. **Update Linux Packages (in WSL2 terminal):**  
   sudo apt update && sudo apt upgrade \-y  
   sudo apt install curl vim wget gnupg apt-transport-https \-y

3. **Install systemd:** If your distribution doesn't support systemd natively in WSL2 (which is needed to easily manage Wazuh services), ensure you have a compatible setup. For recent Windows/WSL2 versions, native systemd should be available.

### **Step 2: Establish Reliable IP Address Access (Addressing Dynamic IP)**

Since the IP address of your WSL2 instance changes upon reboot, we will use a reliable hostname mapping on the Windows side.

1. Get the WSL2 IP (in WSL2 terminal):  
   This is the address your Windows Agent will use to talk to the Manager.  
   hostname \-I | awk '{print $1}'

   *Note this IP (e.g., 172.20.10.5).*  
2. Edit the Windows Hosts File (in Windows as Administrator):  
   Open Notepad or VS Code as an administrator and edit the hosts file:  
   C:\\Windows\\System32\\drivers\\etc\\hosts

3. **Map a Hostname:** Add the IP you noted above and map it to a friendly name, which we will use for the Agent installation.  
   \# Replace \<WSL2\_IP\> with the actual IP from Step 2.1  
   \<WSL2\_IP\>    wazuh-manager.local

   *If the WSL2 IP changes, you will need to update this entry, but using a hostname makes the Agent configuration more consistent.*

## **Phase 2: Installing Wazuh Central Components (in WSL2)**

We will use the Wazuh installation assistant script, which simplifies the setup of the Manager, Indexer (OpenSearch), and Dashboard (OpenSearch Dashboards).

### **Step 3: Download and Run the Installation Assistant**

1. **Download the script:**  
   curl \-sO \[https://packages.wazuh.com/4.8/wazuh-install.sh\](https://packages.wazuh.com/4.8/wazuh-install.sh)

2. **Run the script to install the all-in-one stack:**  
   sudo bash wazuh-install.sh \-a

   This process is fully automated and will take several minutes.

### **Step 4: Retrieve Credentials and Verify Wazuh Manager IP Bind**

Once the script finishes, it will display the credentials for the Wazuh Dashboard. **Save these immediately.**

1. **Retrieve Dashboard Credentials (from the script output):**  
   * **User:** admin  
   * **Password:** \<GENERATED\_PASSWORD\>  
2. **Verify Manager Binding:** By default, the Manager often binds to 0.0.0.0, but you can verify it's listening correctly on the Agent port (1514/tcp):  
   ss \-tlpn | grep 1514

3. Access the Dashboard (in Windows Browser):  
   Use the IP address you found in Step 2.1 (e.g., https://172.20.10.5). Your browser will show a certificate warning; you can safely click through it for a local lab environment.

## **Phase 3: Deploying and Configuring the Windows Agent**

### **Step 5: Download and Install the Windows Agent**

1. **Download:** On your Windows host, download the latest Wazuh Agent installer (MSI) from the official Wazuh documentation.  
2. **Run Installer:** Execute the MSI installer.  
3. **Configuration:** When prompted for the Wazuh Manager address, use the reliable hostname you set up in the Windows hosts file in Step 2.3 (e.g., wazuh-manager.local). If you skipped the hosts file step, use the WSL2 IP address directly (e.g., 172.20.10.5).  
4. **Complete Installation:** Finish the installation wizard.

### **Step 6: Start and Verify the Agent**

1. Start the Agent (in Windows Search/Service Manager):  
   Search for "Wazuh" and click "Wazuh Agent Manager." Click Start and ensure the status is "Running."  
2. Verify Communication (in WSL2 terminal):  
   A few moments after starting the agent, you should see the new agent register with the Manager.  
   sudo /var/ossec/bin/agent\_control \-l

   *This command lists all connected agents. Your Windows host should appear in the list with a green 'Active' status.*

### **Step 7: Final Verification in the Wazuh Dashboard**

1. **Log in:** Go to your Wazuh Dashboard URL (e.g., https://\<WSL2\_IP\>) and log in using the admin credentials from Step 4\.  
2. **Check Agents:** Navigate to **Wazuh** \-\> **Agents**. You should see the newly enrolled Windows host listed and active (green status).  
3. **Review Data:** Click on the Windows Agent and check the Security Events tab. You should immediately start seeing security events, logs, and configuration information flowing from your Windows system into the Dashboard.