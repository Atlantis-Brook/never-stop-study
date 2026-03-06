#debug
核心原理：**Java 虚拟机 (JVM) 通过特定的通信协议（JDWP）监听一个端口，本地 IDE 通过网络连接该端口。**
> ⚠️：线上不可开启，会直接阻塞服务
```bash
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 -jar your-app.jar
```

## 1. 服务端配置（启动参数）

根据你的 Java 版本，在启动命令（`java -jar`）中加入以下参数。

### Java 8 及以上版本（推荐方式）

Bash
```
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 -jar your-app.jar
```

### 参数详解：
- **`transport=dt_socket`**：使用套接字传输。
- **`server=y`**：开启调试服务器模式，等待客户端（IDE）连接。
- **`suspend=n`**：启动时是否挂起。如果设为 `y`，程序会卡在启动处，直到调试器连接才继续运行（适用于调试启动阶段的 Bug）。
- **`address=5005`**：指定监听的端口号
> **注意：** 如果你的服务器有防火墙，**必须开启 5005 端口** 的入站访问权限。

---

## 2. 本地 IDE 配置 (IntelliJ IDEA)
在服务器启动后，你需要在本地配置：
1. 点击顶部工具栏的 **Add Configuration...**（或 Edit Configurations）。
2. 点击左上角 **+** 号，选择 **Remote JVM Debug**。
3. 进行如下设置：
    - **Name**: 随意起名（如 `Server-Debug`）。  
    - **Host**: 服务器的 IP 地址。 
    - **Port**: 刚才设置的端口号（如 `5005`）。
    - **Use module classpath**: 选择你当前项目的模块。
4. 点击 **OK**。

---

## 3. 开始调试
1. 在代码中打上断点（Breakpoint）。
2. 在 IDEA 中点击那个**小虫子图标（Debug）**。
3. 如果控制台显示 `Connected to the target VM...`，说明连接成功。
4. 触发服务器上的业务逻辑，代码就会在你本地的断点处停下。

---

## 4. 常见坑点（避坑指南）
1. **代码一致性**：本地代码必须与服务器上运行的代码**完全一致**。如果代码行号对不上，断点会跳到奇怪的地方或根本不生效。
2. **网络隔离**：
    - 如果服务器在云端（如阿里云、AWS），除了系统防火墙，还要检查**安全组策略**。
    - 如果是 Docker 容器，记得在 `docker run` 时做端口映射：`-p 5005:5005`。
3. **性能损耗**：虽然 Debug 模式损耗极小，但在高并发场景下停住断点会导致请求堆积，**千万不要在生产环境流量高峰期进行断点调试**。

---

### 总结
开启远程 Debug 只需要两步：**服务器启动加参数** + **本地 IDE 配 IP 端口**。