# VPN Container Tabs
Force the traffic of a Firefox container tab to pass through a VPN

On newer version of Firefox, it is possible to use [container tabs](https://support.mozilla.org/en-US/kb/containers), which isolate the cookies from the normal tabs.

It is also possible to assign different proxies to be used in different containers using [Container proxy](https://addons.mozilla.org/en-US/firefox/addon/container-proxy/).

Then, we can create a docker container with a VPN client and a proxy server running. By doing this, we can assign our docker proxy to a container tab so that all the traffic of that tab goes through the VPN that is running only inside the docker container.

## Screenshots

![image](https://user-images.githubusercontent.com/3837916/136898149-86ac1a47-a30e-47ee-8719-7146c3718826.png)
![image](https://user-images.githubusercontent.com/3837916/136897205-98309893-bd73-4f03-8ff7-4e308f2b3c8e.png)


## Usage

### 1. Install docker
[See instructions here](https://docs.docker.com/engine/install/)

### 2. Clone the repository
```
git clone https://github.com/Nickguitar/VPNTabs
```
**Important note:** if your user doesn't have permission to run docker containers you will need to run the script with `sudo`

### 3. Place your VPN files in the diretory `ovpn_files`
```
cp /path/to/vpn/files/* ovpn_files/
```

### 4. Build the docker image
```
./VPNTabs --build
```
### 5. Run the container with the specified ovpn file
```
./VPNTabs --run <OVPN FILE> [--port <PORT>] [--name <CONTAINER NAME>]
e.g.:
./VPNTabs --run mullvad_us_all.ovpn --port 3131 --name Mullvad_US
```

*[Alternative] Instead using* `VPNTabs` *You can run your custom script or use docker-compose. Here is an example:*
```
docker run -d \
--cap-add=NET_ADMIN \
--device /dev/net/tun \
--sysctl net.ipv6.conf.all.disable_ipv6=0 \
-p 3128:3128 \
-e OVPN_FILE=<YOUR_VPN_FILE> \
-v <PATH_OF_VPN_FILES_DIRECTORY>:/ovpn \
squid_openvpn:1.0
```
*The envoriment variable* `OVPN_FILE` *is used to know which file OpenVPN should use*


### If everything is ok, you should see port 3128 (or whatever you have chosen) listening on your machine.
```
$ netstat -tapeno | grep 3128
tcp     0    0 0.0.0.0:3128       0.0.0.0:*    LISTEN    0   5767579  -  off (0.00/0/0)
```

### 6. Add Multi Account Containers and Container Proxy to Firefox

- [Multi Account Containers](https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/)
- [Container Proxy](https://addons.mozilla.org/en-US/firefox/addon/container-proxy/) 

### 7. Set the proxy on Container Proxy
Open Container Proxy, click on "Proxy", set protocol to HTTP, server to 127.0.0.1 and use the port you chose in step 2.3 (the default port is 3128). Also, uncheck the checkbox "Do not proxy local addresses". Then, click "save".

![image](https://user-images.githubusercontent.com/3837916/136625420-925f7d61-41c1-4b41-aa41-abea137475b7.png)

Now, click on "Assign" and change the proxy of the container to the one you've just created.
![image](https://user-images.githubusercontent.com/3837916/136626051-4b05ea82-bae4-427e-875b-4b959308d6e9.png)


To use the container tab with VPN, right click the new tab button and choose the container for which you configured the proxy

![image](https://user-images.githubusercontent.com/3837916/136625934-b389fba1-db40-43a2-9066-92e1bd657555.png)


### You're done

Now every website you access using those container tabs will pass through your local proxy, which points to a docker container whose traffic pass through your VPN. =)

### Comments

- You can generate as many containers as you want, each one running a different VPN config file. In this way, it is possible to have multiple container tabs, each with a different VPN.
- To generate another container with another ovpn config file, just place the config file inside `ovpn_files` and follow step 5.
- Note that **this doesn't have a kill switch** yet. If your VPN goes down and you access some website within the container tab, your IP will be exposed. It's at your own risk.
- Since the VPN client is running inside a docker container, all your other network traffic isn't being tunneled through the VPN. The only connections going through the VPN are those pointing to the local proxy you've created.
