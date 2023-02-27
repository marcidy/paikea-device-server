# Paikea Device Server
The device server intermediates messaging between the front end and the devices. It works as a bi-directional one-to-many router.  A device connects to the device port and is put in to the device set by the device IAM string.  Any number of pages may connect to the page port and the connections are aggregated in the pages dictionary keyed by the device IAM.

messages from the pages are sent to the device, and messages from the device are sent to all the subscribed pages.

## Requirements
The device server is intended to run behind a reverse proxy for the page connections.  Page connections are accepted only on localhost, so an e.g. nginx reverse proxy from the external port to the internal port is required.

The device server is exposed to the world.  The devices and front-end communicate via the javascipt served up by the backend.

## Logging
The device server creates a log in it's execution directory for all the messages.


## Example systemd service file

```bash
[Unit]
Description=Paikea backend
After=network.target

[Service]
Type=simple
User=paikea_user
WorkingDirectory=/srv/paikea/device_server
ExecStart=/srv/paikea/venv/bin/python /srv/paikea/device_server/device_server.py
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

## Example nginx config
```bash
server {
    listen 7778 ssl;
    server_name device.hostname.com;
    ssl_certificate /etc/letsencrypt/live/device.hostname.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/device.hostname.com/privkey.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://127.0.0.1:7770;
        proxy_set_header host $host;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";


    }


    access_log  /var/log/nginx/paikea_page_wss_access.log  main;

}
```

Note port 7778 is exposed over with TLS enabled for incoming requests from the front-end client-side webpage websockets.  These are proxied for the internal port 7770, which is the page port on the device server.
