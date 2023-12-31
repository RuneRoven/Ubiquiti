# Unifi controller

Since unifi needs to have port **8443** and **8080** exposed externally for the interface and the AP adoption 
it is best to run it on its own external IP to not have any port conflicts, otherwise you need to change 
ports in the unifi configuration and the service to some port you know are exclusive.

This guide is for kubernetes running on Flatcar using traefik as the network handler.
for more info: [flatcar network](https://www.flatcar.org/docs/latest/setup/customization/network-config-with-networkd/)

First, create a static ip for the the network:

`sudo vi /etc/systemd/network/static.network `

We set 2 static IPs, one for the main pods and one for the unifi controller
```
[Match]
Name=eth0
[Network]
Address=192.168.3.22/24
Gateway=192.168.3.1
Address=192.168.3.23/24
Gateway=192.168.3.1
DNS=8.8.8.8
```
to apply, restart the service:
`sudo systemctl restart systemd-networkd`

In this guide we use a local host **unifi.local**, configured in the local Gateway
To make the GUI to work we need to allow traefik to skip checking for insecure protocols by adding a **serversTransport** that handles this.
This is due to the fact that the unifi controller have its own self signed certificate and needs to be accessed using **https**. 
To make life easier we also need to redirect from **http** to **https**. This is done using a **Middleware** that intersects the requests from **http** and forward it to **https**
Change the ingress to your local host(configured in you router to the correct IP).
Note that there are 2 occurrences for the host that needs to be changed.
Ingress:
```
spec:
  routes:
    - match: Host(`unifi.local`)
```
Service:
```  
externalIPs:
    - "192.168.3.23"
```

To apply the manifests in the namespace: **unifi**(configured in the manifest files) run from the folder containing the manifests:
```
sudo kubectl create namespace unifi
sudo kubectl apply -f .
```
This will deploy all the **.yaml**-files in the folder