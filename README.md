# raspberry-pi-setup

## Contains different setup like automatic start vnc in background in a raspberry pi also starting ssh by default (at startup )

---

# Starting a vnc server when the raspberry pi is connected to internet 

## Creating .service file to use systemmd
```
sudo nano /etc/systemd/system/vncserver@1.service
```

## Code for the .service file

```
[Unit]
Description=Start VNC server when network is online
After=network-online.target
Wants=network-online.target

[Service]
Type=forking
User=pi
Group=pi
WorkingDirectory=/home/pi
ExecStart=/usr/bin/vncserver :1
ExecStop=/usr/bin/vncserver -kill :1

[Install]
WantedBy=multi-user.target
```

## Save

## Open raspi-config tool
```
sudo raspi-config
```

### System Options --> Boot/AutoLogin --> B4 Desktop/AutoLogin

## To enable the service

```
sudo systemctl enable vncserver@1
```

## To Start the service

```
sudo systemctl start vncserver@1
```
> ### In case you made any changes you should run this commands
> ```
> sudo systemctl daemon-reload
> ```
> ### Then enable the service
> ```
> sudo systemctl enable vncserver@1
> ```

## Now reboot

```
sudo reboot
```
---

# To start ssh with system boot

> ## create ssh file with no extension in the sd card directly in the sd card not inside any folder like overlays doing this will start a ssh server by boot
> ## The password is the device password
> ## ssh pi@ip_address

---

# Setting a Automated script that automatically writes the local ip to a text which can be accessed from anywhere by opening a website in the default browser for ssh or vnc server

## Automatic ip grabber of raspberry pi(it opens a website with ipaddress as the parameter which is stored in a website )

## PHP code for the ip grabbing and writting it to a file
```
<?php
$filePath = 'ipr.txt';
error_reporting(E_ALL);
ini_set('display_errors', 1);

if (isset($_GET['ip'])) {
    // Get the IP address from the query parameter
    $ipAddress = $_GET['ip'];

    // Write the IP address to the file
    file_put_contents($filePath, $ipAddress . PHP_EOL, FILE_APPEND);

    // Output a success message
    echo "IP Address logged: $ipAddress\n";
} else {
    echo "No IP address provided.\n";
}
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Remove Chromium Autostart</title>
</head>
<body>
    <h3>Instructions to Remove Browser from Autostart</h3>
    <p>To remove this browser opening automatically, run these commands:</p>

    <p><strong>Step 1:</strong> Stop the Chromium service.</p>
    <pre>sudo systemctl stop chromium.service</pre>

    <p><strong>Step 2:</strong> Disable the Chromium service to prevent it from starting on boot.</p>
    <pre>sudo systemctl disable chromium.service</pre>

    <p><strong>Step 3:</strong> Remove the Chromium service file.</p>
    <pre>sudo rm /etc/systemd/system/chromium-ip.service</pre>

    <p><strong>Step 4:</strong> Reload the systemd daemon to apply changes.</p>
    <pre>sudo systemctl daemon-reload</pre>

    <p><strong>Step 5:</strong> Verify that the Chromium service is no longer active.</p>
    <pre>sudo systemctl status chromium.service</pre>
</body>
</html>
```

## Create a .service file 

```
sudo nano /etc/systemd/system/chromium.service
```
## Code
```
[Unit]
Description=Start Chromium Browser with dynamic IP at boot
After=graphical.target

[Service]
Environment=DISPLAY=:0
ExecStart=/bin/bash -c '/usr/bin/chromium-browser --no-sandbox --start-fullscreen "https://yourwebsite.com/ipgrabber.php?ip=$(hostname -I | awk \'{print $1}\')"'
Restart=on-failure
User=pi # here pi is the username of the raspberry pi

[Install]
WantedBy=default.target
```
## To enable the service
```
sudo systemctl enable chromium.service
```

## To start the service
```
sudo systemctl start chromium.service
```
## To see if the service is running or not
```
sudo systemctl status chromium.service
```
## Reboot the system so that the script is loaded and applied correctly to machine
```
sudo reboot
```

