### Docker部署
```
services:
  softether-vpn:
    image: siomiz/softethervpn
    container_name: softethervpn
    restart: always
    ports:
      - "443:443"       # SSTP使用的端口
      - "1194:1194/udp" # OpenVPN使用的端口
      - "500:500/udp"   # IPsec/IKE使用的端口
      - "4500:4500/udp" # IPsec NAT-T使用的端口
      - "1701:1701/udp" # L2TP使用的端口
    environment:
      - PSK=yM5XdQXECfR6Xbg7      # 预共享密钥，用于IPsec连接
      - USERNAME=admin            # VPN用户名
      - PASSWORD=admin7890        # VPN密码
      - OPENVPN_ENABLE=1          # 启用OpenVPN
      - SSTP_ENABLE=1             # 启用SSTP
    cap_add:
      - NET_ADMIN
    volumes:
      - /lib/modules:/lib/modules
```

> 或者可以直接使用host网络模式：`network_mode: host`
> 部分协议不需要可以去掉

---
### Docker部署`IKEv2和L2TP`
```
services:
  ipsec-vpn-server:
    image: hwdsl2/ipsec-vpn-server
    container_name: vpn
    restart: always
    ports:
      - "500:500/udp"
      - "4500:4500/udp"
      - "1701:1701/udp"
    privileged: true
    volumes:
      - ./data:/etc/ipsec.d
      - /lib/modules:/lib/modules:ro
```
查看连接信息
```
docker logs vpn
```
更多配置信息在`data`目录

[连接教程](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/ikev2-howto-zh.md#android)

[故障排查](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients-zh.md#ikev1-%E6%95%85%E9%9A%9C%E6%8E%92%E9%99%A4)

---

### WireGuard VPN



```
services:
  wg-easy:
    image: ghcr.io/wg-easy/wg-easy
    container_name: wg-easy
    ports:
      - "51820:51820/udp"  # 映射 WireGuard 的 UDP 端口
      - "51821:51821/tcp"  # 映射 Web 界面的 TCP 端口
    restart: always
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
    environment:
      - LANG=en
      - WG_HOST=主机IP或者域名
      - PASSWORD_HASH=面板密码的哈希值
    volumes:
      - ./wireguard:/etc/wireguard
```

[在线生成密码哈希](https://uutool.cn/php-password/)

需要将哈希值的`$`替换为`$$`才是正确的


[官方文档](https://github.com/wg-easy/wg-easy)



---


