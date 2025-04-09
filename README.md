# Windows-Network-Connection-Unavailable
Windows出现网络连接不可用时，一些常用的解决办法

## **Windows 物理机**

如果你的 Windows  连接不了网络，可以通过以下方法重置网络连接：

---

### **方法 1：使用网络重置功能**
1. **打开“设置”**（Win + I）。
2. 选择 **“网络和 Internet”** 选项。
3. 在左侧选择 **“状态”**，然后点击 **“网络重置”**（位于页面底部）。
4. 点击 **“立即重置”**，然后确认。
5. 电脑会自动重启，并重置所有网络设置。

---

### **方法 2：通过命令提示符重置网络**
如果方法 1 无效，可以使用命令行工具：
1. 按 **Win + S**，搜索 **cmd**，右键点击“命令提示符”，选择 **“以管理员身份运行”**。
2. 依次输入以下命令，每输入一行后按回车：
   ```
   netsh winsock reset
   netsh int ip reset
   ipconfig /release
   ipconfig /renew
   ipconfig /flushdns
   ```
3. 重启计算机，检查网络是否恢复。

---

### **方法 3：检查网络适配器**
1. **打开“设备管理器”**（Win + X，选择“设备管理器”）。
2. 找到 **“网络适配器”**，展开后右键你的网络设备，选择 **“卸载设备”**。
3. 卸载完成后，重启电脑，Windows 会自动重新安装驱动。
4. 如果问题仍然存在，尝试**更新驱动**（右键设备 -> “更新驱动程序”）。

---

### **方法 4：检查防火墙和代理设置**
1. **关闭防火墙**：
   - 进入 **控制面板 -> Windows Defender 防火墙**，暂时**关闭**所有网络的防火墙。
2. **检查代理设置**：
   - 进入 **设置 -> 网络和 Internet -> 代理**，确保“使用代理服务器”处于**关闭**状态。

如果仍然无法连接网络，可能是 **路由器问题** 或 **运营商问题**，建议尝试：
- **重启路由器**。
- **使用手机热点测试**。
- **联系网络服务提供商**。
---
## **虚拟机中的 Windows 系统（比如 VirtualBox、VMware、Hyper-V 等）**

在**虚拟机中的 Windows 系统（比如 VirtualBox、VMware、Hyper-V 等）**网络出问题时，
可能的原因比实体机更复杂一些，
因为除了操作系统自身设置，
还有**虚拟机软件的网络配置**也会影响。
以下是常见问题和解决办法：

---

### 🧩 一、常见问题分类

| 问题类型 | 原因可能在 | 举例 |
|----------|-------------|------|
| 无网络 | 虚拟机网络适配器未启用或未正确连接 | 网络图标打叉 |
| 无法获取 IP（提示“无效 IP 配置”） | DHCP 服务失效、桥接/NAT配置错误 | `ipconfig` 显示 169 开头的地址 |
| 网络连接但无法上网 | DNS 配置问题、主机网络限制 | ping 百度失败 |
| 网络类型错误（公用/专用） | 防火墙或适配器识别错误 | 无法与局域网其他设备通信 |

---

### 🛠️ 二、解决步骤（通用）

#### ✅ 1. 检查虚拟机软件的网络适配器设置

以 VMware 和 VirtualBox 为例：

##### 👉 VMware：
- 关机虚拟机 → 设置 → 网络适配器：
  - **桥接模式**：让虚拟机与主机共享同一网络（适合访问内网其他设备）。
  - **NAT 模式**：虚拟机通过主机共享上网（适合仅需上网）。
  - **仅主机模式**：虚拟机只能与主机通信，无法联网。
- 选项：确保勾选 **“已连接”** 和 **“开机连接”**。

##### 👉 VirtualBox：
- 设置 → 网络 → 适配器 1：
  - 勾选 **“启用网络适配器”**。
  - 连接方式：建议选择 **NAT** 或 **桥接适配器**。
  - 桥接时需选择实际的网卡（如 Intel/Wi-Fi）。

---

#### ✅ 2. 在虚拟机中重置 IP 和 Winsock

在虚拟机 Windows 中打开 **命令提示符（以管理员身份）**，输入以下命令：

```cmd
netsh winsock reset
netsh int ip reset
ipconfig /release
ipconfig /renew
ipconfig /flushdns
```

然后重启虚拟机。

---

#### ✅ 3. 检查网络适配器是否启用

- 打开“控制面板” → “网络和共享中心” → “更改适配器设置”。
- 右键虚拟网卡 → 确保是“启用”状态。
- 如果看到虚拟网卡图标为灰色或禁用，右键 → 启用。

---

#### ✅ 4. 重装或重启虚拟网卡驱动

1. 在设备管理器中（`devmgmt.msc`）展开“网络适配器”。
2. 找到对应的虚拟网卡（可能名称中有 VMware、VirtualBox、Hyper-V）。
3. 右键卸载 → 然后重启系统，系统会自动重新安装驱动。

---

#### ✅ 5. 检查主机网络服务

主机系统（__物理机__）中必须运行对应的虚拟网络服务（尤其是使用 NAT 模式）：

- 对于 VMware，确保如下服务运行：
  - `VMware NAT Service`
  - `VMware DHCP Service`

- 对于 VirtualBox，可以尝试重新安装增强功能或重启网络服务：
  ```
  VBoxManage natnetwork stop --netname "NatNetwork"
  VBoxManage natnetwork start --netname "NatNetwork"
  ```

---

#### ✅ 6. 设置静态 IP（如果 DHCP 失败）

如果虚拟机无法自动获取 IP（例如 `ipconfig` 显示 169.254.x.x），可以尝试手动设置：

- 打开“网络适配器属性” → 选择 `IPv4` → 手动输入：
  - IP 地址（如：192.168.1.123）
  - 子网掩码：255.255.255.0
  - 默认网关：路由器地址（如：192.168.1.1）
  - DNS：8.8.8.8 或学校/公司提供的

> 注意：设置时要确保 IP 在路由器范围内且不冲突。

---

### 🧪 三、常见虚拟机网络模式简要对比

| 模式 | 能联网 | 能与主机通信 | 能与局域网通信 | 说明 |
|------|--------|----------------|------------------|------|
| NAT | ✅ | ✅ | ❌ | 主机作为中转 |
| 桥接 | ✅ | ✅ | ✅ | 与主机同一网络 |
| 仅主机 | ❌ | ✅ | ❌ | 局域环境调试使用 |
| 内部网络 | ❌ | ❌ | ✅（虚机间） | 虚拟机间通信 |

---

### 📌 总结快速排查清单

| 检查点 | 是否通过 |
|--------|----------|
| 虚拟机网络适配器启用且已连接 | ✅/❌ |
| 网络模式是否合适（NAT / 桥接） | ✅/❌ |
| IP 是否正确（不是 169 开头） | ✅/❌ |
| 虚拟网卡驱动是否正常工作 | ✅/❌ |
| 主机虚拟网络服务是否运行 | ✅/❌ |
| 防火墙 / 安全软件是否阻止 | ✅/❌ |

---
## **教室用的多媒体电脑或企业内部电脑**

>如果**教室用的多媒体电脑或企业内部电脑**提示 **“没有有效的 IP 配置”**，可能是由于固定网络设置、校园网/局域网限制、网络适配器异常或 DHCP 分配问题导致的。你可以尝试以下方法：

>>⚠⚠⚠
>>>特别提醒：
>>>>**教室用的多媒体电脑或企业内部电脑** **静态ip**使用较多，尤其注意方法三。

---

### **方法 1：检查网线连接**
1. **检查网线**是否插紧，可以尝试拔出后重新插入。
2. **更换网线**（如果可能的话，尝试用另一根网线连接）。
3. **换个网络端口**（如果网线连接的是墙上的网口，可以换一个试试）。

---

### **方法 2：使用命令提示符重置 IP**
1. **按 Win + R**，输入 `cmd`，然后**右键以管理员身份运行**。
2. 依次输入以下命令，每输入一行后按 **Enter**：
   ```
   netsh winsock reset
   netsh int ip reset
   ipconfig /release
   ipconfig /renew
   ipconfig /flushdns
   ```
3. 执行完毕后，**重启电脑**，检查网络是否恢复。

---

### **方法 3：手动设置 IP**
很多校园网/教室电脑/企业内网的网络可能是**静态 IP**，可以尝试手动设置：
1. **按 Win + R**，输入 `ncpa.cpl`，按 Enter，打开“网络连接”。
2. 找到你的 **“以太网”**（如果是有线网络），右键选择 **“属性”**。
3. 选中 **“Internet 协议版本 4（TCP/IPv4）”**，然后点击 **“属性”**。
4. >选择 **“使用以下 IP 地址”**，输入学校提供的 IP 信息（如果没有，可以联系校园网管理员）。
   >>如果不知道 **静态ip** 信息，可以尝试用其他手机或电脑连接网络，再打开网络配置找到分配的 **IP 子网掩码 网关 DNS**等信息，抄过来就行啦。
   >>>注意
7. 如果不确定 IP 地址，可以尝试 **“自动获取 IP 地址”** 和 **“自动获取 DNS 服务器地址”**。
8. 点击 **确定**，然后测试网络连接。

---

### **方法 4：检查 DHCP 是否启用**
1. **按 Win + R**，输入 `services.msc`，按 Enter，打开“服务”窗口。
2. 找到 **“DHCP Client”（DHCP 客户端）**。
3. 双击打开，确保 **“启动类型”** 设为 **“自动”**，如果未启动，点击 **“启动”** 按钮。
4. 确认后，点击 **“应用”** 并 **重启电脑**。

---

### **方法 5：重置网络**
1. **打开“设置”**（Win + I）。
2. 进入 **“网络和 Internet”** -> **“状态”**。
3. 向下滚动，点击 **“网络重置”**。
4. 点击 **“立即重置”**，然后电脑会**自动重启**，重置所有网络配置。

---

### **方法 6：检查校园网认证**
有些学校的网络需要 **认证登录**：
1. 打开浏览器，看看是否跳转到**校园网登录页面**。
2. 如果有，请输入**账号和密码**进行认证。
3. 如果不确定登录方式，可以联系学校的网络管理部门。

---

### **方法 7：联系学校网络管理员**
如果上面的方法都无效，可能是：
- **电脑 MAC 地址未绑定**（部分校园网会限制特定设备连接）。
- **IP 地址冲突**（可能其他设备占用了相同的 IP）。
- **校园网 DHCP 服务器故障**（无法分配 IP）。

你可以联系学校的 IT 部门或网管，提供 **电脑的 MAC 地址**（在 cmd 里输入 `ipconfig /all`，找到 **“物理地址”**），让他们检查是否有网络限制。

---

### **总结**
1. **先检查网线和端口**，确认物理连接正常。
2. **使用命令提示符** 重置 IP 配置。
3. **尝试自动或手动配置 IP**，根据学校网络需求设置。
4. **检查 DHCP 是否开启**，保证自动获取 IP 正常。
5. **尝试网络重置**，恢复网络适配器默认状态。
6. **如果是校园网，看看是否需要认证登录**。
7. **联系学校 IT 部门**，排查是否有网络限制。

可以按顺序尝试，看哪种方法能解决问题！



