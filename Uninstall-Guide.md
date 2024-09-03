If you need to remove Wazuh completely from your system, including the Wazuh Server, Indexer, and Dashboard, you can follow these steps to ensure a clean removal:

### Step 1: Stop All Wazuh Services

Before you start removing Wazuh components, you should stop all running Wazuh services:

1. **Stop the Wazuh Services**:

   ```bash
   sudo systemctl stop wazuh-manager
   sudo systemctl stop wazuh-indexer
   sudo systemctl stop wazuh-dashboard
   ```

2. **Disable the Services**:

   To prevent the services from starting on boot, disable them:

   ```bash
   sudo systemctl disable wazuh-manager
   sudo systemctl disable wazuh-indexer
   sudo systemctl disable wazuh-dashboard
   ```

### Step 2: Remove Wazuh Packages

Now that the services are stopped, you can proceed with removing the Wazuh packages from your system.

1. **Remove the Wazuh Packages**:

   ```bash
   sudo apt-get remove --purge wazuh-manager wazuh-indexer wazuh-dashboard -y
   ```

   The `--purge` option ensures that all configuration files are removed along with the packages.

2. **Remove Wazuh Dependencies**:

   You can also remove any unused dependencies that were installed with Wazuh:

   ```bash
   sudo apt-get autoremove -y
   ```

### Step 3: Delete Wazuh Configuration Files and Directories

Even after removing the packages, some configuration files and directories might still exist. You should manually delete these to clean up completely.

1. **Remove Configuration Files**:

   ```bash
   sudo rm -rf /etc/wazuh
   ```

2. **Remove Wazuh Data Directories**:

   ```bash
   sudo rm -rf /var/ossec
   sudo rm -rf /var/lib/wazuh-indexer
   sudo rm -rf /var/lib/wazuh-dashboard
   ```

3. **Remove Log Files**:

   ```bash
   sudo rm -rf /var/log/wazuh*
   sudo rm -rf /var/log/elasticsearch
   ```

### Step 4: Clean Up Any Remaining Files

Finally, it's a good idea to search for any remaining Wazuh files and directories that might have been overlooked.

1. **Search for Leftover Files**:

   You can use the `find` command to look for any files or directories related to Wazuh:

   ```bash
   sudo find / -name "*wazuh*" -exec rm -rf {} \;
   ```

   **Note**: Use the above command with caution as it will delete any file or directory with "wazuh" in its name.

### Step 5: Verify Removal

After completing the above steps, it's important to verify that all components have been successfully removed:

1. **Check Services**:

   Ensure that no Wazuh services are running:

   ```bash
   sudo systemctl status wazuh-manager
   sudo systemctl status wazuh-indexer
   sudo systemctl status wazuh-dashboard
   ```

   These commands should return that the services are not found.

2. **Check for Remaining Files**:

   You can also manually browse the `/etc`, `/var`, and `/usr` directories to ensure no Wazuh-related files or folders remain.

### Conclusion

By following these steps, you should have completely removed Wazuh and all its components from your system. This will free up resources and ensure that no residual files are left behind. If you plan to reinstall Wazuh or a similar tool, your system will now be in a clean state, ready for a fresh setup.
