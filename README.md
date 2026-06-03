# Why are we here?
There's issues with the Unifi UI and Protect mobile app when trying to update the SSID on the G4 Doorbell Pro (Wifi). The problem? The "Set WiFi" option in the Unifi UI WiFi settings does nothing and the Protect mobile does not have the option at all. This tutorial walks you through on how to enable SSH on the UniFi Console and G4 Doorbell. As well updating the SSID for the G4 Pro. By default SSH is disabled on both devices but you cannot SSH into the G4 Pro directly without first going through the console. As of ***Protect v7.1.75*** and ***G4 Pro v5.3.90*** this issue remains unresolved. 

I'd like to thank Chris Hansen (Not that one) for providing a foundation on how to enable all this. Some instructions are directly from his "[Using SCP and SSH to Update UniFi Doorbell G4 Animations and Tones](https://chrishansen.tech/posts/unifi_g4_doorbell_pro_part4/)" with a couple of changes. Not relevant to the issue at hand but still extremely helpful to say the least.

# Enabling SSH in the console.
1. Open up your UniFi Console (https://unifi.ui.com/)
2. Click **Settings > Control Plane > Console**
3. Locate **SSH**.
4. Select either **Change Password** to reset your SSH password or check the box to enable SSH and set your own password.
5. ***Optional*** - Click **Restart** to restart the console. I've personally have never had to but YMMV.

# Remoting into your UniFi console and enabling SSH for your G4 Doorbell Pro
1.) Open Terminal or your SSH application of your choosing and type the below:
 
```json

ssh root@[console_ip_address]

```

2.) Navigate to the UniFi Protect directory:

```json
cd /etc/unifi-protect
```

3.) Open or create the configuration file. You may prompted to either hit ***enter*** or type a command just hit ***enter***.

```json

vim config.json

```

4.) Press i to enter insert mode and add the following JSON: 

```json

{ "enableSsh": true }

```
5.) Press **ESC**, then type `:wq` to save and exit VIM.

6.) Restart UniFi Protect:

```json

systemctl restart unifi-protect

```
7.) You have now enabled SSH for your G4 Doorbell Pro. We are 50% there so do not close out your ssh root@[console_ip_address] session.

# Retrieving your Doorbell SSH Password and remoting into it.
We'll be using the default **ubnt** user to SSH into our Doorbell. Firstly we'll need the recovery code as that would be the default password when remoting into the doorbell.

1. Go to UniFi Protect
2. Select your doorbell.
3. Select **Settings > Manual Recovery**.
4. Copy the **Recovery Code**.
5. From the same console you are likely still in ***root@Your-UDM-Name:/etc/unifi-protect#*** path. Run the below command using the recovery code as the password with X being your Doorbell IP address.

```json
 ssh ubnt@XX.X.X.XXX

```

# Adding a new SSID

1.) First let's see what network we have.

```json
wpa_cli list_networks

```

2.) Now we can actually start the process to add the network

```json
wpa_cli add_network

```
3.) It will return a number (probably `1`). Use that number in the next commands:

```json
wpa_cli set_network 1 ssid '"Your-SSID"'
```
```json
wpa_cli set_network 1 psk '"Your-Password"'
```
```json
wpa_cli set_network 1 key_mgmt WPA-PSK
```

```json
wpa_cli enable_network 1
```

```json
wpa_cli select_network 1
```

4.) At this point your terminal may freeze which is a great sign the commands went through. Your G4 Doorbell Pro will show back up with the new SSID.

# Summary

That's it! You have now changed the SSID on your G4 Pro. I really hope Unifi fixes this issue or at least allows us to make the change on the Protect mobile app. I captured a HAR file while pressing 'Set WiFi' and confirmed the API request is never actually sent the UI fails silently on the client side before reaching the server.
