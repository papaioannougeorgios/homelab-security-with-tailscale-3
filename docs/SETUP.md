Tailscale:

First thing I did was install Tailscale in my Ubuntu server with ```curl -fsSL https://tailscale.com/install.sh | sh```, after that was finished I ran ```sudo tailscale up``` and logged in after being prompted to. To access your dashboards after Tailscale has been properly configured, you're going to need the application on a computer, for this I used my own PC. You can get the Tailscale desktop application at ```https://tailscale.com/download```.

Updating the AdGuard configuration:

I restricted access to the AdGuard dashboard by making it only accessible from my own server through ```127.0.0.1``` by changing:

```
http:
  address: 0.0.0.0:80
```

In my AdGuardHome.yaml (this http header is near the top of the file), to:

```
http:
  address: 127.0.0.1:80
```

Afterwards, I configured my docker container for localhost binding by replacing the ```network_mode: "host"``` to:

```
ports:
      - "127.0.0.1:80:80/tcp"
      - "127.0.0.1:3000:3000/tcp"
      - "100.X.X.X:53:53/tcp"          # GET YOUR OWN TAILSCALE IP BY USING tailscale ip -4
      - "100.X.X.X:53:53/udp"          # GET YOUR OWN TAILSCALE IP BY USING tailscale ip -4
```

You can view my entire docker-compose in the ```/docker/adguard/``` file.

Updating the Nextcloud docker:

I changed ```ports: 8080:80``` to:

```
ports:
      - "127.0.0.1:8080:80"
```

Nextcloud database and internal routing:

I fixed my database host binding by running ```sudo docker exec -u www-data YOUR_OWN_CONTAINER_NAME php occ config:system:set dbhost --value="YOUR_OWN_DATABASE_CONTAINER_NAME"``` and then configured the reverse proxy settings with:

```
sudo docker exec -u www-data YOUR_OWN_CONTAINER_NAME php occ config:system:set overwriteprotocol --value="https"
sudo docker exec -u www-data YOUR_OWN_CONTAINER_NAME php occ config:system:set trusted_domains 2 --value="YOUR_OWN_TAILSCALE_DOMAIN"
sudo docker exec -u www-data YOUR_OWN_CONTAINER_NAME php occ config:system:set trusted_proxies 0 --value="127.0.0.1"
```

Tailscale serve:

Using Tailscale serve, we can generate valid Let's Encrypt certificates. I used the following command to clean up any existing rules ```sudo tailscale serve reset```, then since my AdGuard is running under port 80, I simply used ```sudo tailscale serve --bg 80``` for it. My Nextcloud used port 8080, therefore I used a secure HTTPS port for it by running ```sudo tailscale serve --bg --https 8443 http://127.0.0.1:8080```.

Nextcloud desktop client:

To make sure that my desktop client isn't still making requests over HTTP, I went in the settings of it and simply logged in using the Tailscale domain instead of my server's IP.

Miscellaneous:

With everything done, visiting the server's IP to access the dashboards is no longer possible from any device other than the server itself, to access the dashboards you now have to visit your own Tailscale domain under whatever ports you used for the services. Meaning that now the dashboards are running through HTTPS.
