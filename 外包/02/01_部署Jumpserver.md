1、部署SoftEther VPN

```yaml
version: '3'
services:
  softether:
    image: softethervpn/vpnserver:stable
    cap_add:
      - NET_ADMIN
    restart: always
    ports:
      - 53:53
      - 443:443
      - 992:992
      - 1194:1194/udp
      - 5555:5555
      - 500:500/udp
      - 4500:4500/udp
      - 1701:1701/udp
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
      - "./softether_data:/mnt"
      - "./softether_log:/root/server_log"
      - "./softether_packetlog:/root/packet_log"
      - "./softether_securitylog:/root/security_log"
```

