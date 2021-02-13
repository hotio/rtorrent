[<img src="https://hotio.dev/img/qbittorrent.png" alt="logo" height="130" width="130">](https://github.com/qbittorrent/qbittorrent)

[![GitHub Source](https://img.shields.io/badge/github-source-ffb64c?style=flat-square&logo=github&logoColor=white&labelColor=757575)](https://github.com/hotio/qbittorrent)
[![GitHub Registry](https://img.shields.io/badge/github-registry-ffb64c?style=flat-square&logo=github&logoColor=white&labelColor=757575)](https://github.com/orgs/hotio/packages/container/package/qbittorrent)
[![Docker Pulls](https://img.shields.io/docker/pulls/hotio/qbittorrent?color=ffb64c&style=flat-square&label=pulls&logo=docker&logoColor=white&labelColor=757575)](https://hub.docker.com/r/hotio/qbittorrent)
[![Discord](https://img.shields.io/discord/610068305893523457?style=flat-square&color=ffb64c&label=discord&logo=discord&logoColor=white&labelColor=757575)](https://hotio.dev/discord)
[![Upstream](https://img.shields.io/badge/upstream-project-ffb64c?style=flat-square&labelColor=757575)](https://github.com/qbittorrent/qbittorrent)
[![Website](https://img.shields.io/badge/website-hotio.dev-ffb64c?style=flat-square&labelColor=757575)](https://hotio.dev/containers/qbittorrent)

## Starting the container

CLI:

```shell
docker run --rm \
    --name qbittorrent \
    -p 8080:8080 \
    -e PUID=1000 \
    -e PGID=1000 \
    -e UMASK=002 \
    -e TZ="Etc/UTC" \
    -v /<host_folder_config>:/config \
    hotio/qbittorrent
```

Compose:

```yaml
version: "3.7"

services:
  qbittorrent:
    container_name: qbittorrent
    image: hotio/qbittorrent
    ports:
      - "8080:8080"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Etc/UTC
    volumes:
      - /<host_folder_config>:/config
```

In most cases you'll need to add additional volumes, depending on your own personal preference, to get access to your files.

## Tags

Go [here](https://hotio.dev/tags-overview/#hotioqbittorrent) to see all available tags.

## WireGuard VPN support

This is probably not going to work if your OS has no kernel with WireGuard support.

Tested Operating Systems:

* Ubuntu 18.04
* Ubuntu 20.04
* Unraid 6.8.3
* Unraid 6.9 RC1
* macOS Big Sur 11.1 Apple M1

CLI:

```shell
docker run --rm \
    --name qbittorrent \
    -p 8080:8080 \
    -e PUID=1000 \
    -e PGID=1000 \
    -e UMASK=002 \
    -e TZ="Etc/UTC" \
    -e VPN_ENABLED="true" \
    -e VPN_LAN_NETWORK="" \
    -e VPN_CONF="wg0" \
    -e VPN_ADDITIONAL_PORTS="" \
    -v /<host_folder_config>:/config \
    --cap-add=NET_ADMIN \
    --sysctl="net.ipv4.conf.all.src_valid_mark=1" \
    --sysctl="net.ipv6.conf.all.disable_ipv6=0" \
    hotio/qbittorrent
```

Compose:

```yaml
version: "3.7"

services:
  qbittorrent:
    container_name: qbittorrent
    image: hotio/qbittorrent
    ports:
      - "8080:8080"
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Etc/UTC
      - VPN_ENABLED=true
      - VPN_LAN_NETWORK
      - VPN_CONF=wg0
      - VPN_ADDITIONAL_PORTS
    volumes:
      - /<host_folder_config>:/config
    cap_add:
      - NET_ADMIN
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv6.conf.all.disable_ipv6=0
```

There needs to be a file `wg0.conf` located in `/config/wireguard` and you need to set the variable `VPN_ENABLED` to `true` for the VPN to start.

The part with `net.ipv6.conf.all.disable_ipv6=0` can be removed or set to `1` if there is no need for ipv6, no attempt will be made in that case to set ip6tables rules and can prevent an error if the module `ip6table_filter` isn't loaded on the host. The WireGuard configuration should not have any ipv6 related stuff when ipv6 is disabled, otherwise creating the interface will fail. If your vpn provider supports ipv6 and you keep it enabled, you'll have full ipv6 connectivity over the vpn connection (confirmed with Mullvad). If for any reason there's a failure trying to setup ip6tables rules, you'll probably need to do `sudo modprobe ip6table_filter` on the host, this will mostly happen on systems that have ipv6 completely disabled.

The environment variable `VPN_LAN_NETWORK`can be set to for example `192.168.1.0/24`, `192.168.1.0/24,192.168.44.0/24` or `192.168.1.33`, so you can get access to the qBittorrent webui.

If you need to expose additional ports you can use `VPN_ADDITIONAL_PORTS`, for example `VPN_ADDITIONAL_PORTS=7878/tcp,9117/tcp`. Every port in this list will be blocked on the vpn interface, so that there's no risk that they might be exposed to the world via the vpn (mostly there in case your vpn provider screws up and piece of mind). Why would you need this? Wanting to route traffic from other containers over the vpn is probably the most used scenario.

## wg0.conf example

This is an example of how your `wg0.conf` could look like.

```text
[Interface]
PrivateKey = supersecretprivatekey
Address = xx.xx.xxx.xxx/32
DNS = 1.1.1.1

[Peer]
PublicKey = publickey
AllowedIPs = 0.0.0.0/0
Endpoint = xxx.x.xxx.x:51820
```

## TorGuard instructions

While Mullvad is pretty straightforward to setup by using the `wg0.conf` example from above, TorGuard is a bit more complex.

Our `wg0.conf` should look something like this:

```text
# TorGuard WireGuard Config
[Interface]
PrivateKey = secretprivatekey
ListenPort = 51820
DNS = 1.1.1.1
Address = xx.xx.xxx.xx/24
PreUp = bash /config/wireguard/torguard.sh

[Peer]
PublicKey = publickey
AllowedIPs = 0.0.0.0/0
Endpoint = xx.xxx.xx.xxx:1443
PersistentKeepalive = 25
```

Pay attention to `PreUp = bash /config/wireguard/torguard.sh` in our config. That command will execute the below script that you should create in `/config/wireguard/torguard.sh`, this script will get executed just before starting WireGuard.

```shell
#!/usr/bin/bash
pubkey=$(grep PrivateKey "${CONFIG_DIR}/wireguard/wg0.conf" | awk '{print $3}' | wg pubkey)
wgserver=$(grep Endpoint "${CONFIG_DIR}/wireguard/wg0.conf" | awk '{print $3}')
curl -ksG -u "${TORGUARD_USER}":"${TORGUARD_PASS}" --data-urlencode "public-key=${pubkey}" "https://${wgserver}/api/v1/setup"
```

You will also have to add the additional environment variables `TORGUARD_USER` and `TORGUARD_PASS` or fill them in into the script directly (see `curl` command). These credentials can be found [here](https://torguard.net/clientarea.php?action=changepw).

My experience with getting TorGuard working wasn't the smoothest journey to say the least. I had to click around quite a bit and finally after generating my 3rd config it worked. On the `Netherlands` server for example I didn't get any internet connectivity and at first I was unable to get port forwarding working on the `Germany` server. All of a sudden after generating the 3rd config and also pasting in the ip found under `My Fixed IPs`, that seems to populate when doing a Port Forward Request, I managed to get port forwarding working. So don't give up too soon, it can all work eventually.
