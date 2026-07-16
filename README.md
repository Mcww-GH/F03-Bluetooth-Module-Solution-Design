# F03 BLE 卫星消息 Android App V1.1.0

这是用于连接 F03 船舶卫星定位通信一体机的 Android BLE 客户端源码。界面提供明确的连接结果、文字发送弹窗和本地发送历史；技术通道信息收在“查看连接信息”内，不要求使用者手动选择 UUID。

## 当前范围

- 面向 Android 12 及以上，最低支持 Android 8（API 26）。
- 手机作为 BLE Central，F03 作为 BLE Peripheral / GATT Server。
- 只发送文字；输入上限为 100 个 Unicode 字符（100 个中文字符符合该限制）。
- 自动识别已验证的 BLE-UART 服务：`FFE0` 服务中的 `FFE1` 特征。`FFE1` 同时作为手机写入和设备通知通道；`FFE2` 不参与正常消息发送。
- 本地保存最近 500 条发送记录：设备名、地址、内容、时间、手机蓝牙确认、设备数据、超时或错误原因。
- 写入成功后，记录会等待 10 秒的设备数据；收到的数据会原样保留为文本或十六进制。没有 F03 回执协议时，应用不会把它误报为卫星上行成功。

## 重要：F03 协议尚待实机确认

现有资料没有提供 F03 的文字消息帧格式、校验规则或卫星发送回执定义。已提供的大夏龙雀测试 APP 与一次实机 GATT 截图可以确认 BLE 传输层采用 `FFE0 / FFE1`，但不能证明 F03 的业务级“终端已接收”或“卫星上行成功”回执含义。因此，`F03Protocol` 目前只提供**诊断用的原始 UTF-8 写入**，不能视为最终卫星报文协议。

首次连接 F03 时：

1. 打开 F03，打开 App 并授予“附近设备”权限。
2. 扫描后直接点击“连接”；BLE 一般不需要先在系统设置中配对。
3. 等待“F03 已连接”弹窗，点击“开始发消息”。
4. 发送一条已知安全的测试文字，并记录“收到设备数据”中的原始返回内容或“未收到设备回执”。
5. 将结果填入 `docs/F03_Protocol_Capture_Template.md`，再把明确的协议写入 `F03Protocol.kt`，然后构建 release 版本。

不要盲发未知十六进制控制帧；它们可能修改设备配置或触发卫星上行。

## 在 Android Studio 中构建

1. 使用 Android Studio 打开本目录。
2. 选择 JDK 17，安装 Android SDK Platform 35 与对应 Build Tools。
3. 让 Android Studio 同步 Gradle 依赖后，选择 **Build → Generate App Bundles or APKs → Generate APKs**。生成的 `app-debug.apk` 已签名，可直接传到手机安装；或执行 `assembleRelease`，得到同样可直接安装的 `app-release.apk`。

如需在当前环境之外构建，请使用 Android Studio；选择 JDK 17，安装 Android SDK Platform 35 与对应 Build Tools。

## 代码结构

- `ble/BleClient.kt`：扫描、连接、GATT 发现、通知订阅、串行分包写入。
- `ble/F03Protocol.kt`：文字限制、UTF-8 诊断编码和未来 F03 帧协议的唯一接入点。
- `data/MessageRepository.kt`：本地历史记录。
- `ui/MainViewModel.kt`：界面状态、发送阶段、设备数据和超时处理。
- `ui/F03App.kt`：连接成功直达发送、简明连接信息和发送记录界面。
