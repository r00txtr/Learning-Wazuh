# Installing & Configuring ELK & Wazuh Integration: A Beginner's Guide

In this guide, you'll learn how to install and configure Elasticsearch, Logstash, and Kibana (ELK) alongside Wazuh for powerful threat detection, monitoring, and logging. The process includes setting up Elasticsearch for data storage, configuring Wazuh as the security monitoring manager, and integrating Kibana for visualizing security events.

We will walk through the following steps:
1. Installing Elasticsearch
2. Setting up Wazuh Manager
3. Configuring Filebeat for log shipping
4. Installing and integrating Kibana for dashboards

Let's get started!

### Step 1: Installing Elasticsearch

**Elasticsearch** is the core component for storing and searching security events. Follow these steps to install and configure it:

1. **Install Required Packages**:

   Open the terminal and run the following commands to install necessary packages for your environment:

   ```bash
   apt-get install apt-transport-https zip unzip lsb-release curl gnupg
   ```

2. **Add Elasticsearch GPG Key**:

   Import the Elasticsearch GPG key to ensure the authenticity of the package:

   ```bash
   curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/elasticsearch.gpg --import && chmod 644 /usr/share/keyrings/elasticsearch.gpg
   ```

3. **Add Elasticsearch Repository**:

   Add the Elasticsearch repository to your sources list:

   ```bash
   echo "deb [signed-by=/usr/share/keyrings/elasticsearch.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
   ```

4. **Install Elasticsearch**:

   Update the package list and install Elasticsearch:

   ```bash
   apt-get update
   apt-get install elasticsearch=7.17.13
   ```

5. **Configure Elasticsearch**:

   Download the preconfigured Elasticsearch configuration file provided by Wazuh:

   ```bash
   curl -so /etc/elasticsearch/elasticsearch.yml https://packages.wazuh.com/4.5/tpl/elastic-basic/elasticsearch_all_in_one.yml
   ```

   This file includes optimizations for using Elasticsearch with Wazuh.

6. **Generate Certificates for Secure Communication**:

   Generate the necessary certificates for Elasticsearch:

   ```bash
   curl -so /usr/share/elasticsearch/instances.yml https://packages.wazuh.com/4.5/tpl/elastic-basic/instances_aio.yml
   /usr/share/elasticsearch/bin/elasticsearch-certutil cert ca --pem --in instances.yml --keep-ca-key --out ~/certs.zip
   ```

7. **Unzip and Move Certificates**:

   Unzip the generated certificates:

   ```bash
   unzip ~/certs.zip -d ~/certs
   ```

   Move them to the proper directory:

   ```bash
   mkdir /etc/elasticsearch/certs/ca -p
   cp -R ~/certs/ca/ ~/certs/elasticsearch/* /etc/elasticsearch/certs/
   ```

8. **Set Permissions**:

   Set the necessary permissions for the certificates:

   ```bash
   chown -R elasticsearch: /etc/elasticsearch/certs
   chmod -R 500 /etc/elasticsearch/certs
   chmod 400 /etc/elasticsearch/certs/ca/ca.* /etc/elasticsearch/certs/elasticsearch.*
   rm -rf ~/certs/ ~/certs.zip
   ```

9. **Start Elasticsearch**:

   Reload system daemons and start Elasticsearch:

   ```bash
   systemctl daemon-reload
   systemctl enable elasticsearch
   systemctl start elasticsearch
   ```

10. **Set Elasticsearch Passwords**:

    Generate secure passwords for Elasticsearch users:

    ```bash
    /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
    ```

    Note the generated passwords for various users (e.g., `elastic`, `kibana_system`, etc.).

    Example output:

    ```
    PASSWORD elastic = AN4UeQGA7HGl5iHpMla7
    ```

11. **Verify Elasticsearch is Running**:

    Test Elasticsearch by sending a request with the `elastic` user:

    ```bash
    curl -XGET https://localhost:9200 -u elastic:AN4UeQGA7HGl5iHpMla7 -k
    ```

    You should see a JSON response with Elasticsearch version and details.

---

### Step 2: Installing Wazuh Manager

**Wazuh Manager** is responsible for collecting and analyzing security events from various agents. Follow these steps to install it:

1. **Add Wazuh Repository**:

   Import the Wazuh GPG key and add the Wazuh repository:

   ```bash
   curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
   echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
   ```

2. **Install Wazuh Manager**:

   Update the package list and install the Wazuh manager:

   ```bash
   apt-get update
   apt-get install wazuh-manager=4.5.4-1
   ```

3. **Start Wazuh Manager**:

   Enable and start Wazuh Manager:

   ```bash
   systemctl daemon-reload
   systemctl enable wazuh-manager
   systemctl start wazuh-manager
   ```

4. **Check Wazuh Manager Status**:

   Confirm that Wazuh Manager is running properly:

   ```bash
   systemctl status wazuh-manager
   ```

---

### Step 3: Installing and Configuring Filebeat

**Filebeat** collects and ships logs to Elasticsearch. Let’s configure it to send logs from Wazuh to Elasticsearch.

1. **Install Filebeat**:

   Install Filebeat, which is compatible with Elasticsearch 7.x:

   ```bash
   apt-get install filebeat=7.17.13
   ```

2. **Configure Filebeat**:

   Download the Wazuh Filebeat configuration template:

   ```bash
   curl -so /etc/filebeat/filebeat.yml https://packages.wazuh.com/4.5/tpl/elastic-basic/filebeat_all_in_one.yml
   ```

   Add the Elasticsearch password to the Filebeat configuration file:

   ```bash
   nano /etc/filebeat/filebeat.yml
   ```

   In the `output.elasticsearch` section, set the `password` to the password generated for the `elastic` user.

3. **Set Up Certificates**:

   Copy Elasticsearch certificates to Filebeat:

   ```bash
   cp -r /etc/elasticsearch/certs/ca/ /etc/filebeat/certs/
   cp /etc/elasticsearch/certs/elasticsearch.crt /etc/filebeat/certs/filebeat.crt
   cp /etc/elasticsearch/certs/elasticsearch.key /etc/filebeat/certs/filebeat.key
   ```

4. **Start Filebeat**:

   Enable and start Filebeat:

   ```bash
   systemctl daemon-reload
   systemctl enable filebeat
   systemctl start filebeat
   ```

5. **Test Filebeat Output**:

   Run a quick test to ensure Filebeat is correctly outputting to Elasticsearch:

   ```bash
   filebeat test output
   ```

---

### Step 4: Installing and Configuring Kibana

**Kibana** provides the visualization layer for Elasticsearch, enabling you to analyze security data from Wazuh.

1. **Install Kibana**:

   Install Kibana for Elasticsearch 7.x:

   ```bash
   apt-get install kibana=7.17.13
   ```

2. **Set Up Certificates**:

   Copy Elasticsearch certificates to Kibana:

   ```bash
   mkdir /etc/kibana/certs/ca -p
   cp -R /etc/elasticsearch/certs/ca/ /etc/kibana/certs/
   cp /etc/elasticsearch/certs/elasticsearch.key /etc/kibana/certs/kibana.key
   cp /etc/elasticsearch/certs/elasticsearch.crt /etc/kibana/certs/kibana.crt
   ```

3. **Set Kibana Configuration**:

   Download and edit the Kibana configuration file:

   ```bash
   curl -so /etc/kibana/kibana.yml https://packages.wazuh.com/4.5/tpl/elastic-basic/kibana_all_in_one.yml
   nano /etc/kibana/kibana.yml
   ```

   In the configuration file, set the Elasticsearch password for Kibana.

4. **Install Wazuh Kibana Plugin**:

   Install the Wazuh plugin for Kibana:

   ```bash
   sudo -u kibana /usr/share/kibana/bin/kibana-plugin install https://packages.wazuh.com/4.x/ui/kibana/wazuh_kibana-4.5.4_7.17.13-1.zip
   ```

5. **Start Kibana**:

   Enable and start Kibana:

   ```bash
   systemctl daemon-reload
   systemctl enable kibana
   systemctl start kibana
   ```

6

. **Access Kibana**:

   Open your browser and navigate to:

   ```
   https://<wazuh_server_ip>
   ```

   Use the `elastic` username and the password you generated earlier to log in.

---

### Conclusion

You have successfully installed and configured ELK and Wazuh. Now, you can use Kibana to visualize logs and security events collected by Wazuh, ensuring comprehensive threat detection across your infrastructure. 

Feel free to explore the dashboards, set up alerts, and monitor security events in real-time!
