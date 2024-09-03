# Installing and Configuring Wazuh: A Beginner's Guide

Wazuh is an open-source security platform that provides comprehensive visibility into your security posture. It offers intrusion detection, log analysis, file integrity monitoring, and more. In this guide, we will walk you through the process of installing and configuring Wazuh on your system, covering the Wazuh Server components: Indexer, Server, and Dashboard.

### Prerequisites

Before we begin, ensure you have:
- A clean installation of a supported Linux distribution (e.g., Ubuntu 20.04 LTS).
- Root or sudo access on your system.
- Basic familiarity with the Linux command line.

### Step 1: Download the Wazuh Installation Script and Configuration File

To get started, we need to download the Wazuh installation script and the default configuration file. Open a terminal and run the following commands:

1. **Download the Installation Script**:

   ```bash
   curl -sO https://packages.wazuh.com/4.8/wazuh-install.sh
   ```

2. **Download the Configuration File**:

   ```bash
   curl -sO https://packages.wazuh.com/4.8/config.yml
   ```

### Step 2: Edit the Configuration File

The `config.yml` file contains settings for the Wazuh installation. You may need to customize it based on your environment.

1. **Open the Configuration File**:

   ```bash
   nano config.yml
   ```

2. **Edit as Needed**:
   - In most cases, the default settings will work fine, but you might want to adjust the configuration to match your network setup or desired installation options.
   - Save the file by pressing `CTRL + O`, and then exit the editor with `CTRL + X`.

### Step 3: Generate Configuration Files

Before we proceed with the installation, we need to generate the configuration files:

1. **Generate Configuration Files**:

   ```bash
   chmod +x ./wazuh-install.sh && sudo ./wazuh-install.sh --generate-config-files
   ```

   This command makes the installation script executable and then runs it to generate the necessary configuration files.

### Step 4: Install the Wazuh Indexer

The Wazuh Indexer is responsible for storing and searching data. Let's install it now:

1. **Install the Wazuh Indexer**:

   ```bash
   sudo ./wazuh-install.sh --wazuh-indexer node-1
   ```

   The `node-1` specifies the name of the indexer node. If you’re setting up a cluster, you can specify additional nodes later.

### Step 5: Start the Wazuh Cluster

Once the indexer is installed, you need to start the Wazuh cluster:

1. **Start the Cluster**:

   ```bash
   sudo ./wazuh-install.sh --start-cluster
   ```

   This command initializes the Wazuh cluster, ensuring that all components can communicate with each other.

### Step 6: Retrieve and Secure Admin Credentials

After setting up the cluster, it's crucial to retrieve the admin credentials for the Wazuh Dashboard:

1. **Extract Admin Credentials**:

   ```bash
   sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt | grep -P "\'admin'" -A 1
   ```

   This command extracts the admin credentials from the installation archive. You should see something like this:

   ```
   'admin'
   NwUG64TwWjWrrYhsCdXaKRyiM?QzLKIq
   ```

2. **Save the Admin Credentials**:

   Store the credentials securely by creating a text file:

   ```bash
   nano admin.txt
   ```

   Paste the credentials into this file, then save and close it.

### Step 7: Verify the Wazuh Indexer

To ensure the Wazuh Indexer is working correctly, you can run the following commands:

1. **Check the Indexer**:

   ```bash
   curl -k -u admin:NwUG64TwWjWrrYhsCdXaKRyiM?QzLKIq https://192.168.1.8:9200
   ```

2. **Check the Nodes**:

   ```bash
   curl -k -u admin:NwUG64TwWjWrrYhsCdXaKRyiM?QzLKIq https://192.168.1.8:9200/_cat/nodes?v
   ```

   Replace `192.168.1.8` with the IP address of your Wazuh server. If everything is set up correctly, these commands will return information about the Wazuh Indexer and its nodes.

### Step 8: Install the Wazuh Server and Dashboard

Now it’s time to install the Wazuh Server, which handles analysis and alerting, and the Dashboard, which provides a web interface for managing Wazuh.

1. **Install the Wazuh Server**:

   ```bash
   sudo ./wazuh-install.sh --wazuh-server wazuh-1
   ```

   This command installs the Wazuh Server with the node name `wazuh-1`.

2. **Install the Wazuh Dashboard**:

   ```bash
   sudo ./wazuh-install.sh --wazuh-dashboard dashboard
   ```

   This installs the Wazuh Dashboard, allowing you to access the Wazuh interface via a web browser.

### Step 9: Access the Wazuh Dashboard

With everything installed, you can now access the Wazuh Dashboard:

1. **Open a Web Browser**:
   - Go to `https://<your-wazuh-server-ip>:5601`.
   - Log in using the admin credentials you saved earlier (`admin` and the password from `admin.txt`).

   ![Wazuh Dashboard Login](https://documentation.wazuh.com/current/_images/vulnerability-detection1.png)

2. **Explore the Dashboard**:
   - Once logged in, you can explore various features like real-time monitoring, alert management, and more.

### Conclusion

Congratulations! You've successfully installed and configured Wazuh on your system. Wazuh provides a comprehensive security monitoring solution, and with the Wazuh Dashboard, you have a powerful tool at your fingertips to manage and analyze security events across your network.

If you found this guide helpful, share it with others who are interested in enhancing their network security, and stay tuned for more tutorials!
