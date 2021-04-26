# Running Cobalstrike Teamserver as a Service

These scripts can be used as a template to set up teamserver as a service and autostart listeners.

These scripts have been tested on Ubuntu server, and will need to be adjusted based on your use case.

## Steps

- Update the service files to match your environment.
  - teamserver.service
  - listener.service
  - listener_service.cna
- Copy the service files to your teamserver
  - /etc/systemd/system/teamserver.service
  - /etc/systemd/system/listener.service
  - /etc/cobaltstrike/listener_service.cna
- Register the new services
  - `systemclt daemon-reload`
- Start the services
  - `systemctl start teamserver.service`
  - `systemctl start listener.service`
- Test

## Teamserver Service

Update the settings to match your environment.

- WorkingDirectory: set to the cobaltstrike directory
- ExecStart: Set with your values

```

# teamserver.service
[Unit]
Description=Cobalt Strike Teamserver Service
After=network.target
Wants=network.target

[Service]
Type=Simple
WorkingDirectory=/opt/cobaltstrike
ExecStart=/opt/cobaltstrike/teamserver <TEAMSERVER IP> <PASSWORD> <PATH TO C2 PROFILE>

# Example
# ExecStart=/opt/cobaltstrike/teamserver `hostname -I` thisismypassword /opt/cobaltstrike/c2.profile

[Install]
WantedBy=multi-user.target

```

## Listener Service

Update the settings to match your environment.

- WorkingDirectory: set to the cobaltstrike directory
- ExecStart: Set with your values

```
# listener.service
[Unit]
Description=Cobalt Strike aggressor service
After=teamserver.service network.target
Wants=teamserver.service
StartLimitIntervalSec=33

[Service]
Restart=on-failure
RestartSec=10
WorkingDirectory=/opt/cobaltstrike
ExecStartPre=/bin/sleep 60
ExecStart=/bin/bash /opt/cobaltstrike/agscript 127.0.0.1 50050 <USER to LOGON TO COBALTSTRIKE> <TEAMSERVER PASSWORD> <PATH TO listener_service.cna>

# Example
# ExecStart=/bin/bash /opt/cobaltstrike/agscript 127.0.0.1 50050 listener_service thisismypassword /opt/cobaltstrike/listener_service.cna


[Install]
WantedBy=multi-user.target
```

## Headless aggressor script

This must be updated to reflect your environment. 

This example aggressor script is used to start an HTTP, HTTPS, and SMB listener with all the needed parameters

What to change?

- HTTP Listener
  - Listnername
  - host
  - althost
- HTTPS Listener
  - Listnername
  - host
  - althost
- SMB Listener
  - 


```
println("
###################################################################
 CobaltStrike Aggressor Script          
 Author:      Joe Vest
 Description: Headless script to create listeners
###################################################################");

println('Loading listener_service.cna...');

on ready{
    println('listener_service.cna: Creating HTTP Listener...');
	listener_create_ext("HTTP", "windows/beacon_http/reverse_http", %(host => "iheartredteams.com", port => 80, beacons => "iheartredteams.com", althost => "iheartredteams.com", bindto => 80));

    println('listener_service.cna: Creating HTTPS Listener...');
	listener_create_ext("HTTPS", "windows/beacon_https/reverse_https", %(host => "iheartredteams.com", port => 443, beacons => "iheartredteams.com", althost => "iheartredteams.com", bindto => 443));

    println('listener_service.cna: Creating SMB Listener...');
	listener_create_ext("SMB", "windows/beacon_bind_pipe", %(port => "mojo.5887.8051.34782273429370473##"));
	sleep(10000);
}
```