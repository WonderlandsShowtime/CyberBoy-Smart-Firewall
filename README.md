CyberBoy智能防火墙
CyberBoy 智能防火墙是一款轻量级、开源的 Windows 过滤平台 （WFP）Windows 10 和 Windows 11 的策略管理器。它用C和C++编写， 使用原生 Direct2D 接口，并作为一个自包含的 x64 运行 可执行。

CyberBoy dashboard

它的作用
安装高优先级的WFP子层，并应用出站/入站过滤 交易性质。如果进程退出或崩溃，动态 WFP 会话为 被Windows和过滤停止拆除，而不是留下破碎的堆栈。
提供三种保护预设，并有一个完全独立的DIY模式。
提供一站鼠标驱动界面，分辨率为1280x720至1920x1080 自适应窗口，高DPI支持，以及英文、简体中文，以及 繁体中文界面。
认可内置的中国及国际主要软件允许列表 家族（字节跳动、腾讯、阿里巴巴、百度、美团、京东、Microsoft、苹果， Google、Adobe、Mozilla等），并支持签名应用验证 全模式。
支持用户允许/阻止域名、IPv4/IPv6地址、CIDR范围的规则， 应用程序和端口，以及可编辑的JSON策略文档。
显示 Microsoft Defender 防火墙配置文件状态，只读并共同管理 通过WFP进行流量，未禁用Defender杀毒软件或篡改 防御者保护设置。
用户界面技术
界面采用Fluent视觉语言，采用圆润卡片、类似丙烯的深色调色板、Segoe MDL2图标、自定义标题栏以及按显示器进行DPI比例调整。它直接在 Win32、Direct2D 和 DirectWrite 上实现。WinUI 3 构建将增加 Windows App SDK 运行时和更大的封装空间;在此环境中，选择了原生 Direct2D 路径，以保持应用程序自包含且轻量级，符合产品需求中的备用条款。

要求
Windows 10 1809（版本 17763）或更高版本，或 Windows 11、x64。
WFP运营的管理员权限。应用请求抬高 正常发射; 可仅供用户界面测试。--safe --no-elevate
不需要单独的运行时。可执行文件被静态链接到 C/C++运行时，仅使用Windows系统DLLs。
安装
从目录里跑。该 安装是按机器计算的，需要高度。这个应用不是代码 签名，因此 Windows SmartScreen 可能会显示警告;请核实发布的SHA-256 随着发布。CyberBoySmartFirewall-Setup-1.0.0.exedist

静音安装：

.\CyberBoySmartFirewall-Setup-1.0.0.exe /VERYSILENT /SUPPRESSMSGBOXES /NORESTART
静默卸载会移除应用程序并调用移除 动态 WFP 过滤器和浏览器扩展策略。--reset-quiet

保护模式
模式	意图	密钥行为
基础	日常使用	屏蔽常见的C2/远程访问端口，应用威胁指示，保持入站不过滤，并使用可信供应商允许列表。
增强版	推荐阅读	新增入站屏蔽、远程访问块、SMB出口保护、DNS保护、Tor中继端口和风险服务端口块。
全套	严格	新增默认拒绝外站功能。仅允许验证过的应用、已解析的厂商地址、批准的DNS服务器、环回和局域网。
自己动手	习俗	每个交换机都是独立的，验证后可以应用手写的JSON策略。
全模式是故意制造干扰。部分应用会失去连接; 查看活动日志，并根据需要添加用户允许规则。应用要求 在启用全模式前确认一下。

发动机的工作原理
策略核心解析JSON配置并生成确定性 政策快照。
引擎通过系统解析器解析允许列出的域，收集 签名应用路径、DNS服务器和本地子网，然后构建一组 WFP过滤器。
过滤器安装在显式的WFP事务中。安装失败是 中止，之前的滤波器集保留。
WFP会议为 。关闭进程会移除所有 CyberBoy自动过滤。干净的退出还会删除子层， 明确地说，提供者。FWPM_SESSION_FLAG_DYNAMIC
活动监控器观察已建立的TCP流，并标记匹配的流程 配置的块端口或块地址范围。这是一个政策匹配监控器， 不是包捕获或内核调用。
可信供应商允许列表
内置的允许列表涵盖类别，而非“供应商所有流量”：

字节跳动：bytedance.com、douyin.com、feishu.cn、larkoffice.com、capcut.com......
腾讯：qq.com，wechat.com，tencent.com，tencentcloud.com，wechatpay.cn，......
阿里巴巴：alibaba.com，aliyun.com，taobao.com，tmall.com，alipay.com，......
百度：baidu.com，bdstatic.com，bcebos.com，......
美团：meituan.com，dianping.com，......
JD：jd.com，jingdong.com，360buyimg.com，......
Microsoft、Apple、Google、Adobe、Mozilla 以及常见的 CDN/系统域。
这些列表刻意保守，可以通过用户规则进行扩展。 域匹配支持精确的主机和后缀。全模式 在应用时将允许列出的域名解析为地址，并在 配置区间。*.example.com

自行操作政策文件
编辑器和规则页面接受包含以下模式的JSON文档：

{
  "schema": 1,
  "mode": "diy",
  "toggles": {
    "inbound": true,
    "c2": true,
    "remoteAccess": true,
    "smbGuard": true,
    "riskyPorts": true,
    "defaultDeny": false,
    "browserLock": false,
    "dnsGuard": true,
    "knownVendors": true,
    "blockTor": true,
    "allowLan": true,
    "notify": true
  },
  "allow": [
    {"kind": "domain", "value": "example.com", "note": "work"},
    {"kind": "cidr4", "value": "192.0.2.0/24"}
  ],
  "block": [
    {"kind": "port", "value": "4444"},
    {"kind": "app", "value": "C:\\Program Files\\Bad\\bad.exe"}
  ]
}
支持的规则类型：， ， ， ， ， 。无效值会被本地拒绝。未知场被忽略。domainipv4ipv6cidr4cidr6appport

重要限制
CyberBoy 是一个策略管理器，而不是杀毒软件、EDR 或云威胁情报 服务。它不包含内核调用驱动程序，因此无法检查数据包 有效载荷、TLS内容或DNS查询名称。C2/僵尸网络处理基于 可配置端口、导入指示器以及允许列表/默认拒绝行为;它 并非声称检测到所有C2族。

完整模式的域名允许列表被解析为地址。共享CDN地址可以 因此，其他软件和更改地址的网站都能访问 刷新之间可能会暂时失效。默认拒绝规则的设计对象是 与Windows服务兼容，但部分第三方软件需要 用户允许规则。

浏览器扩展锁令 Chrome、Edge 和 Firefox 编写策略注册表 价值观。它只在浏览器重启后生效，并且拒绝覆盖 现有的管理员策略。它与网络过滤是分开的，是 只有在明确确认后才申请。

该应用程序不会禁用 Microsoft Defender 杀毒软件，也不会修改 Defender 防火墙配置文件。它报告Defender的个人资料状态并执行自己的WFP 次层。这是一个共管理设计，不是Defender的替代品。

源代码构建
要求：

LLVM-MinGW（Clang/LLD，UCRT，x86_64-w64-mingw32 目标）
Inno Setup 6（仅限安装人员使用）
PowerShell 5.1 或更高版本
.\build.ps1 -Clean -Tests -Installer
该构建生成：

dist\CyberBoyFirewall.exe
dist\CyberBoySmartFirewall-Setup-1.0.0.exe
dist\cyberboy_tests.exe
dist\default_policy.json
详见工具链详情。docs/BUILD.md

项目布局
src/core/     C policy core, JSON parser, logging, string/path helpers
src/engine/   C++ WFP engine, Windows integration, DNS/app collection
src/ui/       C++ Win32 + Direct2D interface and trilingual catalogue
src/tests/    Offline policy and translation tests
config/       Default policy document
installer/    Inno Setup script
assets/       Application icon
docs/         Installation, build, and security notes
许可
麻省理工。参见。LICENSE
