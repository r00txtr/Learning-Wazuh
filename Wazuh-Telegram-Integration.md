# Wazuh Alert Integration with Telegram Bot: A Beginner's Guide

In this guide, we’ll walk you through integrating Wazuh alerts with a Telegram bot, allowing real-time alerts to be sent to a designated chat or group. We'll go step by step, from creating the Telegram bot to integrating it with Wazuh.

### Prerequisites

- A Wazuh server already installed and running.
- Access to Telegram to create and configure a bot.
- Basic understanding of Linux commands.

### Step 1: Create a Telegram Bot and Get a Chat ID

The first step in the process is to create a Telegram bot that will handle the Wazuh alerts.

1. **Open Telegram and Find BotFather**:
   - Launch Telegram and search for **BotFather** (a verified account with a blue tick).
   - Start a chat with BotFather.

2. **Create a New Bot**:
   - In the chat with BotFather, type `/newbot` to create a new bot.
   - Follow the instructions by providing a name and username for your bot.
     - Example bot name: `alert-wazuh-telegram-bot`
   - After creating the bot, BotFather will give you an **HTTP API token**. Save this token for later use.

   ![Telegram Bot API Example](https://example.com/path-to-screenshot)

3. **Get the Chat ID**:
   - Now search for your newly created bot in Telegram (use its username) and start a chat with it.
   - Create a Telegram group, add the bot and any other team members.
   - Send a message to the group to generate a **Chat ID**.

4. **Retrieve the Chat ID**:
   - Open a browser and replace `<HTTP-API>` with your bot’s API key in the following link:
   
     ```
     https://api.telegram.org/bot<HTTP-API>/getUpdates
     ```

   - This will give you a response containing your Chat ID (e.g., `-826434xxx`). Save both the Chat ID and the HTTP API.

> **Important**: Keep your HTTP API and Chat ID private. If leaked, others could control your bot.

### Step 2: Integration with Wazuh Server

Now that we have the bot setup, let's integrate it with the Wazuh server.

1. **Check System Readiness**:
   - Ensure `requests` is installed by running:

     ```bash
     pip install requests
     ```

2. **Create Integration Files**:
   - Go to the Wazuh integrations directory:

     ```bash
     cd /var/ossec/integrations
     ```

   - Create a script named `custom-telegram`:

     ```bash
     nano /var/ossec/integrations/custom-telegram
     ```

   - Add the following code:

     ```bash
     #!/bin/sh

     WPYTHON_BIN="framework/python/bin/python3"

     SCRIPT_PATH_NAME="$0"
     DIR_NAME="$(cd $(dirname ${SCRIPT_PATH_NAME}); pwd -P)"
     SCRIPT_NAME="$(basename ${SCRIPT_PATH_NAME})"

     case ${DIR_NAME} in
         */active-response/bin | */wodles*)
             if [ -z "${WAZUH_PATH}" ]; then
                 WAZUH_PATH="$(cd ${DIR_NAME}/../..; pwd)"
             fi

             PYTHON_SCRIPT="${DIR_NAME}/${SCRIPT_NAME}.py"
         ;;
         */bin)
             if [ -z "${WAZUH_PATH}" ]; then
                 WAZUH_PATH="$(cd ${DIR_NAME}/..; pwd)"
             fi

             PYTHON_SCRIPT="${WAZUH_PATH}/framework/scripts/${SCRIPT_NAME}.py"
         ;;
          */integrations)
             if [ -z "${WAZUH_PATH}" ]; then
                 WAZUH_PATH="$(cd ${DIR_NAME}/..; pwd)"
             fi

             PYTHON_SCRIPT="${DIR_NAME}/${SCRIPT_NAME}.py"
         ;;
     esac

     ${WAZUH_PATH}/${WPYTHON_BIN} ${PYTHON_SCRIPT} "$@"
     ```

   - Save and exit the file.

3. **Create Python Script**:
   - Now create the Python script `custom-telegram.py`:

     ```bash
     nano /var/ossec/integrations/custom-telegram.py
     ```

   - Paste the following Python code:

     ```python
     #!/usr/bin/env python

     import sys
     import json
     import requests

     # Change CHAT_ID to your actual chat ID
     CHAT_ID = "-826434xxx"

     # Read alert file
     alert_file = open(sys.argv[1])
     hook_url = sys.argv[3]

     # Parse JSON alert
     alert_json = json.loads(alert_file.read())
     alert_file.close()

     # Extract important fields
     alert_level = alert_json['rule'].get('level', "N/A")
     description = alert_json['rule'].get('description', "N/A")
     agent = alert_json['agent'].get('name', "N/A")

     # Prepare message
     msg_data = {
         'chat_id': CHAT_ID,
         'text': f"Alert from {agent}:\nLevel: {alert_level}\nDescription: {description}"
     }
     headers = {'content-type': 'application/json'}

     # Send alert to Telegram
     requests.post(hook_url, headers=headers, data=json.dumps(msg_data))

     sys.exit(0)
     ```

   - Save the file and exit.

4. **Set Correct Permissions**:
   Ensure that both files have the correct permissions:

   ```bash
   chown root:wazuh /var/ossec/integrations/custom-telegram*
   chmod 750 /var/ossec/integrations/custom-telegram*
   ```

### Step 3: Configure Wazuh to Use the Telegram Integration

1. **Edit Wazuh Configuration**:
   - Open the Wazuh configuration file:

     ```bash
     nano /var/ossec/etc/ossec.conf
     ```

   - Add the following XML code under the `<integration>` section, replacing `<API_KEY>` with your bot’s API key:

     ```xml
     <integration>
         <name>custom-telegram</name>
         <level>3</level>
         <hook_url>https://api.telegram.org/bot<API_KEY>/sendMessage</hook_url>
         <alert_format>json</alert_format>
     </integration>
     ```

   - This configuration will send alerts of level 3 and above. You can adjust the `<level>` value to send higher or lower-level alerts.

2. **Restart Wazuh Manager**:
   After making changes, restart the Wazuh manager to apply the configuration:

   ```bash
   systemctl restart wazuh-manager
   ```

### Step 4: Verify Telegram Alerts

Once everything is set up, send a test alert to ensure the integration works:

- Trigger an alert in Wazuh.
- The CSIRT team or Telegram group members should receive an alert in Telegram with the alert details.

---

### Conclusion

You have successfully integrated Wazuh with a Telegram bot to receive real-time alerts. This integration helps in faster response and alert management by directly notifying your team via Telegram. Now, your Telegram CSIRT team can handle Wazuh alerts more efficiently!
