# JDownloader through NordVPN in Docker (Synology)

A simple step by step guide on how to configure JDownloader 2 to run inside a Docker container (on a Synology NAS) with only its traffic routed through NordVPN.

- Synology NAS running DSM 6.2
- Default Synology Docker application
- JDownloader running on a container
- NordVPN running on a separate container

Some of the tips in the Troubleshooting section might also apply to other Docker hosts or VPN clients.

## What makes this method special?
### TL;DR
Only traffic from the JDownloader Docker container is routed through the VPN.

### The slightly longer version
The Synology NAS is a great tool even if only for its basic functionality. However, many will use it for many purposes, just like me. For example, this is a small insight into my setup:

- Shared folders
- Volume dedicated to Time Machine
- Plex Server
- Docker
    - Pi-Hole
    - Ombi server for Plex
    - Telegram Bot for Ombi
    - Homebridge
    - On-demand Minecraft servers

For a while I used a regular installation of JDownloader 2 for DSM.

My next move is to have all traffic for JDownloader routed through the VPN. For exactly that: I present to you this guide.

##### 🤔 Wait a second, can't NordVPN already be used on Synology simply by installing an OpenVPN profile?
Indeed, it can. If what you are looking for is to route all the traffic through the VPN, go ahead and follow the simple [official setup guide](https://support.nordvpn.com/Connectivity/NAS/1047411072/How-to-configure-Synology-6-1-NAS.htm). It works fine and does exactly what it's supposed to.

But take a second look at my setup above. As soon as all traffic goes through the VPN, everything starts to slow down, fall apart and I lose local and/or remote access to the rest of containers and applications; far from ideal.


## Getting started

- install docker
- ssh access

## Downloading the images
- nordvpn
- jdownloader-2
## Creating the Docker containers

> Running `sudo` commands will prompt you to enter a password from time to time. Use the password for the Synology user currently logged into the SSH-session.

could create containers from DSM Docker GUI, however, it's limited.

```
sudo docker run -ti \
--cap-add=NET_ADMIN \
--cap-add=SYS_MODULE \
--device /dev/net/tun \
--name nordvpn \
-e USER=<NORDVPN_EMAIL> \
-e PASS='<NORDVPN_PASSWORD>' \
-e CONNECT=de \
-e TECHNOLOGY=NordLynx \
-e NETWORK=<ROUTER_LOCAL_IP>/24 \
-e TZ=Europe/Berlin \
-p 5800:5800 \
-p 3129:3129 \
-d bubuntux/nordvpn
```
> Just in case you were wondering what those `\` at the end of each line are doing: They tell your Terminal not to execute the command but instead await a new line. This is often done for easier readability when using lots of arguments.

You will need to replace the following information:

|Tag|Content|
|:--|:--|
|`<NORDVPN_EMAIL>`|Your NordVPN email.|
|`<NORDVPN_PASSWORD>`|Your NordVPN password.|
|`<ROUTER_LOCAL_IP>`|Your router's local IP. This will probably something like `192.168.0.1`. <br>The `/24` after the IP represents the subnet mask. Use: <br>`/24` for `255.255.255.0` <br>or <br>`/16` for `255.255.0.0`|

- country code
- technology NordLynx vs. TCP
- TZ
- port explanation


```
sudo docker run -ti \
--network=container:nordvpn \
--name jdownloader \
-v "/volume1/docker/jdownloader:/config:rw" \
-v "/volume1/Downloads:/output:rw" \
-d jlesage/jdownloader-2
```

> In case you ever need to delete the container, as long as you map the /config directory from the container to the same path on your host, your configuration will still be available

## Where to go from here?

- refresh vpn for a new IP to bypass download limits
    - restart nordvpn
        - jdownload restart required, too, because it loses the network reference
        - not guaranteed to connect to a different vpn server
    - manually connect to 
    

## Troubleshooting

### Missing `/dev/net/tun` device
`Error response from daemon: linux runtime spec devices: error gathering device information while adding custom device "/dev/net/tun": no such file or directory.`

Indeed, if we take a look at the Docker log, we will find an Error entry with the following Event:

`Start container nordvpn failed: {"message":"linux runtime spec devices: error gathering device information while adding custom device \"/dev/net/tun\": no such file or directory"}.`

https://github.com/binhex/arch-delugevpn/issues/67#issuecomment-399380209


### invalid download directory
- set UID and GUID

### restart jdownload requires join network
restart from command line
```
sudo docker container restart jdownloader
```