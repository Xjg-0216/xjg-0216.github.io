# ubuntu搭建共享网络

### 1 检查网络管理器服务是否开启

```bash
sudo systemctl status network-manager
```

### 2 创建并配置需要共享的wifi

```bash
nm-connection-editor
```
选择`add a new connections`, 连接类型选择`WIFI`；

connection name: xujg
WIFI栏：

- SSID: xujg
- Mode: Hotspot
- Band: Automatic
- Channel: default
- Device: 
- Cloned MAC address: Random
- MTU: automatic

WIFI Security:
- Security: WEP 40/128-bit Key(Hex or ASCII)
- key: **** (5位)
- WEP index: 1(Default)
- Authetication: Open System