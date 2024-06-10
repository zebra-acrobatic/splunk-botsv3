# Splunk and BOTSv3 installation
Simplified instructions for installing Splunk and BOTSv3 data.

## Source ##
Original instructions for BOTSv3 can be found [here](https://github.com/splunk/botsv3):

## Step 1: Prepare a VM ##
You will require a Linux machine to follow the below instructions.

1. Download (or create) a Rocky 9 minimal virtual machine:
    * Download the x86_64 minimal ISO from the [official page](https://rockylinux.org/download/).
    * Create a new virtual machine in your Hypervisor of choice, it requires:
      * 8GB of RAM.
      * 100 GB of storage.
      * 4 vCPU.
      * 1 Network card bridged to the external network (Custom > TDM if using the lab machines).
    * Install Rocky Linux on the VM:
       * Set a root password you will not forget.
       * Create a user account and password you will not forget.
       * Ensure the software selection is set to minimal.
       * Under the network settings:
          * Ensure the machine gets a valid DHCP address.
          * Configure a unique host name.
        *  Reboot the VM when the installation is completed.
2. When the system is back up, log in with the username and password you created during installation.
4. Ensure the system has functioning Internet connectivity:
   ```bash
   ping google.com
   ```
5. Update the system:
   ```bash
   sudo dnf update
   ```
   ```bash
   reboot
   ```
6. Log in again when the system restarts.
  
7. Install [Cockpit](https://cockpit-project.org/).
   ```bash
   sudo dnf install cockpit -y
   ```
8. Configure Cockpit to run at boot time:
   ```bash
   sudo systemctl enable cockpit.socket
   ```
9. Reboot the system once more.

10. The system should start up and display how to access the web console using the IP and port (example https://192.168.1.145:9090). Record this information for the next steps. If it does not display the message, check the trouble shooting steps at the end of this section.

11. Open a browser on your host machine (or any other machine on the network) and connect to the Cockpit management page using the details you discovered in the last step.

12. Login with the user you created earlier.

**You should no longer need to log in using the local terminal, all future administration tasks should be done using Cockpit or SSH.**

### Trouble shooting Cockpit ###
Issue | Possible fix | Description
--- | --- | ---
No IP displayed on boot (just https://localhost:9090) | Log in via the terminal and check your IP address manually using `nmcli` | Sometimes the exact ordering of events required to correctly display your IP and port for Cockpit does not occur correctly. If this happens, you need to obtain your IP manually (the port should always use port 9090).
Can't connect to Cockpit | Check IP in browser| Do not use the DNS name, it will display a name and IP method of connecting. Do not use the name.
Still can't connect to Cockpit | Check if it running using `systemctl status cockpit`. If it is not running, re-issue the `sudo systemctl enable cockpit.socket` command from the steps and `reboot` | Perhaps you did not issue the command correctly.
Really still can't connect to Cockpit | use `sudo firewall-cmd --list-all` and check if Cockpit is allowed through the firewall, if not use `sudo firewall-cmd --add-service=cockpit --perm` followed by `sudo firewall-cmd --reload` | There is a possibility the firewall rules required for Cockpit to function were not implemented during installation.
I have done everything right and I still can't connect. | No you have not, check everything again. | Unless there is a serious bug in your system, everything should be working.
## Step 2: Install Splunk ##
You will need to aquire Splunk.

<ins>_Optional step_</ins>: Shut down your VM and take a snap shot, name the snapshot "Clean install" or something similar.

1. Go to the [Splunk website](https://splunk.com) and sign up for a free account.
2. Log in to your account.
3. Click **Products** > **Free trials and downloads**.
4. Click **Splunk Enterprise** > **Get My Free Trial**.
5. Click **Linux** > **.rpm**.
6. On the next page, the download will probably begin. You need to download it on your virtual machine, not your desktop machine. To do this, att the top right; click **Download via Command Line (wget)**.
7. A **wget** command with a URL will appear in a small window. Copy the _entire command_.
8. In Cockpit, click **Terminal**.
9. Install [wget](https://www.gnu.org/software/wget/):
    ```bash
    sudo dnf install wget -y
    ```
11. Paste in the wget command from the download page to begin downloading the Splunk installation rpm.
12. Install Splunk (replacing x.y.z with the version number of Splunk (consider typing "s" then pressing tab)):
    ```bash
    sudo dnf localinstall splunk.x.y.z
    ```
    If you see the error `useradd: cannot create directory /opt/splunk`, reinstall Splunk:
    ```bash
    sudo dnf reinstall splunk.x.y.z
    ```
13. Start Splunk and follow the inststructions (_do not forget the details_):
    ```bash
    sudo /opt/splunk/bin/splunk start --accept-license
    ```
14. Open the required firewall port:
    ```bash
    sudo firewall-cmd --add-port=8000/tcp --perm
    ```
15. Reload the firewall rules:
    ```bash
    sudo firewall-cmd --reload
    ```
16. Using a new browser tab, try to connect to your installation of Splunk on port 8000 (example http://192.168.1.145:8000)
17. If the page loads, you have installed Splunk correctly. Your next step is to make sure Splunk starts at system boot. To do this, you must first stop Splunk:
    ```bash
    sudo /opt/splunk/bin/splunk stop
    ```
18. Change the file persmissions for the Splunk user:
    ```bash
    sudo chown -R splunk: /opt/splunk/
    ```
19. Tell Splunk to start at boot time.
    ```bash
    sudo /opt/splunk/bin/splunk enable boot-start -systemd-managed 1
    ```
21. Splunk should restart, confirm it works by refreshing your Splunk page on your browser.
22. Reboot your system and ensure everything starts as expected at boot time (Cockpit and Splunk on ports 9090 and 8000 respectively).
## Step 3: Install plugins ##
The data you will soon import requires a set of Splunk plugins. You now need to install the list of plugins from the [here](https://github.com/splunk/botsv3).

<ins>_Optional step_</ins>: Shut down your VM and take a snap shot, name the snapshot "Splunk installed" or something similar.

1. Open the original instructions and download all of the addons.
   NoteL AWS Guard Duty is not available and is replaced with [Splunk Add-on for Amazon Web Services (AWS)](https://splunkbase.splunk.com/app/1876).
  Note: Splunk Add-on for Tenable is not available and is replaced with [Tenable Add-On for Splunk](https://splunkbase.splunk.com/app/4060)
2. From the Splunk Web home screen, click the gear icon next to Apps.
3. Click Install app from file.
4. Locate the downloaded file and click Upload. 
5. Repeat the process for all of the plugins (Restart Splunk when the last plugin is installed).
6. When all plugins are installed, log into **Cockpit** and click **Terminal**.
7. Download the sample data set with **wget**:
   ```bash
   wget https://botsdataset.s3.amazonaws.com/botsv3/botsv3_data_set.tgz
   ```
9. Install tar to extract the data set:
   ```bash
   sudo dnf install tar -y
   ```
11. Install the data:
   ```bash
   sudo tar zxvf botsv3_data_set.tgz -C /opt/splunk/etc/apps/
   ```
12. Change the permissions of the installed data:
   ```bash
   sudo chown -R splunk /opt/splunk
   ```
13. Restart Splunk:
    ```bash
    sudo systemctl restart Splunkd.service
    ```
15. Log back into Splunk.
16. Click "**Search & Reporting**".
17. Perform the following search to confirm the data is working as expected: `index="botsv3" earliest=0`. It will take some time to process the data.
