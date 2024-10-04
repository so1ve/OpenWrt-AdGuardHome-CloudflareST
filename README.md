# OpenWrt-AdGuardHome-CloudflareST

- 基于 [OpenWrt-CloudflareSTHosts](https://github.com/zjjxwhh/OpenWrt-CloudflareSTHosts) 项目中脚本修改
- 需要配合 [openwrt-cdnspeedtest](https://github.com/immortalwrt-collections/openwrt-cdnspeedtest) 项目使用

## 项目简介

供 OpenWrt 路由器使用的 Cloudflare IP 优选脚本

可以测试 Cloudflare CDN 延迟和速度，获取最快 IP (IPv4 及 IPv6)，自动替换 AdGuardHome 中域名对应的重写 IP 地址，并重启 AdGuardHome

## 使用方法

### 1. 软件包安装

- 预编译软件包

1. 下载 [openwrt-cdnspeedtest](https://github.com/immortalwrt-collections/openwrt-cdnspeedtest/releases) 预编译 .ipk 软件包，上传至 OpenWrt 路由器 `/tmp` 目录
2. ssh 连接至路由器并安装软件包

    ```bash
    cd /tmp
    opkg install cdnspeedtest_2.2.4-1_x86_64.ipk
    ```

- 自行编译

    ```bash
    # 进入 OpenWrt 编译根目录
    cd ${OpenWrt-Build-Dir}

    # 添加 cdnspeedtest 源码
    echo 'src-git cdnspeedtest https://github.com/immortalwrt-collections/openwrt-cdnspeedtest.git' >> feeds.conf.default
    ./scripts/feeds update -a
    ./scripts/feeds install -a

    # 进入 menuconfig 配置界面
    make menuconfig

    # 选中 Network -> cdnspeedtest
    # 继续进行后续编译操作
    ```

### 2. 脚本配置

1. 在 AdGuardHome `DNS重写` 中添加需要进行 Cloudflare IP 优选的域名，首次运行时需要将所有域名对应的 IP 配置为相同值，IPv4 与 IPv6 需要分别配置，如分别添加 `cloudflare.com` 重写至 `1.1.1.1` (IPv4)，`cloudflare.com` 重写至 `2.2.2.2` (IPv6)
2. 下载优选脚本及 Cloudflare IP 列表，上传至 OpenWrt 路由器 `/etc/AdGuardHome-CloudflareST` 目录
3. ssh 连接至路由器并手动运行

    ```bash
    cd /etc/AdGuardHome-CloudflareST
    bash cfst_adg.sh
    # 分别输入 IPv4 地址 (1.1.1.1) 及 IPv6 地址 (2.2.2.2)
    ```

5. 配置任务计划

    ```cron
    # 每天 2:00 运行
    0 2 * * * cd /etc/AdGuardHome-CloudflareST/ && bash cfst_adg.sh
    ```

## 注意事项

1. 可在 `/etc/sysupgrade.conf` 文件中增加条目，避免升级时相关脚本及配置被清除

   ```text
   # AdGuardHome-CloudflareST
   /etc/AdGuardHome-CloudflareST
   ```
