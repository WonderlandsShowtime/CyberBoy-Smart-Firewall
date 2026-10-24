# CyberBoy Smart Firewall

CyberBoy Smart Firewall is a lightweight, open-source Windows Filtering Platform
(WFP) policy manager for Windows 10 and Windows 11. It is written in C and C++,
uses a native Direct2D interface, and runs as a single self-contained x64
executable.

![CyberBoy dashboard](docs/dashboard.png)

## What it does

- Installs a high-priority WFP sublayer and applies outbound/inbound filters
  transactionally. If the process exits or crashes, the dynamic WFP session is
  torn down by Windows and filtering stops rather than leaving a broken stack.
- Offers three protection presets plus a fully independent DIY mode.
- Provides a one-stop, mouse-driven interface with a 1280x720 to 1920x1080
  adaptive window, high-DPI support, and English, Simplified Chinese, and
  Traditional Chinese UI.
- Recognizes a built-in allowlist of major Chinese and international software
  families (ByteDance, Tencent, Alibaba, Baidu, Meituan, JD, Microsoft, Apple,
  Google, Adobe, Mozilla, and others), with signed-application verification for
  Full mode.
- Supports user allow/block rules for domains, IPv4/IPv6 addresses, CIDR ranges,
  applications, and ports, plus an editable JSON policy document.
- Shows Microsoft Defender Firewall profile status read-only and co-manages
  traffic through WFP without disabling Defender Antivirus or tampering with
  Defender protection settings.

## UI technology

The interface follows the Fluent visual language with rounded cards, an acrylic-like dark palette, Segoe MDL2 icons, a custom title bar, and per-monitor DPI scaling. It is implemented directly on Win32, Direct2D, and DirectWrite. A WinUI 3 build would add the Windows App SDK runtime and a much larger packaging footprint; in this environment the native Direct2D path was selected to keep the application self-contained and lightweight, as permitted by the fallback clause in the product requirements.

## Requirements

- Windows 10 1809 (build 17763) or later, or Windows 11, x64.
- Administrator privileges for WFP operations. The app requests elevation when
  launched normally; `--safe --no-elevate` is available for UI-only testing.
- No separate runtime is required. The executable is statically linked against
  the C/C++ runtime and uses only Windows system DLLs.

## Install

Run `CyberBoySmartFirewall-Setup-1.0.0.exe` from the `dist` directory. The
installer is per-machine and requires elevation. The application is not code
signed, so Windows SmartScreen may show a warning; verify the SHA-256 published
with the release.

Silent install:

```powershell
.\CyberBoySmartFirewall-Setup-1.0.0.exe /VERYSILENT /SUPPRESSMSGBOXES /NORESTART
```

Silent uninstall removes the application and calls `--reset-quiet` to remove
dynamic WFP filters and browser extension policies.

## Protection modes

| Mode | Intent | Key behavior |
| --- | --- | --- |
| Basic | Everyday use | Blocks common C2/remote-access ports, applies threat indicators, keeps inbound unfiltered, and uses the trusted-vendor allowlist. |
| Enhanced | Recommended | Adds inbound shielding, remote-access blocks, SMB egress guard, DNS guard, Tor relay ports, and risky-service port blocks. |
| Full | Strict | Adds default-deny outbound. Only verified applications, resolved vendor addresses, approved DNS servers, loopback, and LAN are permitted. |
| DIY | Custom | Every switch is independent and a hand-written JSON policy can be applied after validation. |

Full mode is intentionally disruptive. Some applications will lose connectivity;
review the activity log and add user allow rules as needed. The app asks for
confirmation before enabling Full mode.

## How the engine works

1. The policy core parses the JSON configuration and produces a deterministic
   policy snapshot.
2. The engine resolves allowlisted domains through the system resolver, collects
   signed application paths, DNS servers, and local subnets, then builds a set of
   WFP filters.
3. Filters are installed in an explicit WFP transaction. A failed installation is
   aborted and the previous filter set is kept.
4. The WFP session is `FWPM_SESSION_FLAG_DYNAMIC`. Closing the process removes all
   CyberBoy filters automatically. A clean exit also deletes the sublayer and
   provider explicitly.
5. The activity monitor observes established TCP flows and flags flows that match
   configured block ports or block address ranges. This is a policy-match monitor,
   not a packet capture or kernel callout.

## Trusted vendor allowlist

The built-in allowlist covers categories rather than "all traffic from a vendor":

- ByteDance: bytedance.com, douyin.com, feishu.cn, larkoffice.com, capcut.com, ...
- Tencent: qq.com, wechat.com, tencent.com, tencentcloud.com, wechatpay.cn, ...
- Alibaba: alibaba.com, aliyun.com, taobao.com, tmall.com, alipay.com, ...
- Baidu: baidu.com, bdstatic.com, bcebos.com, ...
- Meituan: meituan.com, dianping.com, ...
- JD: jd.com, jingdong.com, 360buyimg.com, ...
- Microsoft, Apple, Google, Adobe, Mozilla, and common CDN/system domains.

The lists are deliberately conservative and can be extended with user rules.
Domain matching supports exact hosts and `*.example.com` suffixes. Full mode
resolves allowlisted domains to addresses at apply time and refreshes them on the
configured interval.

## DIY policy document

The editor and the Rules page accept a JSON document with this schema:

```json
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
```

Supported rule kinds: `domain`, `ipv4`, `ipv6`, `cidr4`, `cidr6`, `app`,
`port`. Invalid values are rejected locally. Unknown fields are ignored.

## Important limitations

CyberBoy is a policy manager, not an antivirus, EDR, or cloud threat-intelligence
service. It does not contain a kernel callout driver, so it cannot inspect packet
payloads, TLS contents, or DNS query names. C2/botnet handling is based on
configurable ports, imported indicators, and allowlist/default-deny behavior; it
is not a claim of detecting every C2 family.

Full mode's domain allowlist is resolved to addresses. Shared CDN addresses can
therefore be reachable by other software, and a site that changes addresses
between refreshes may temporarily fail. The default-deny rules are designed for
compatibility with Windows services, but some third-party software will require
user allow rules.

Browser extension lockdown writes Chrome, Edge, and Firefox policy registry
values. It only takes effect after a browser restart and will refuse to overwrite
an existing administrator policy. It is separate from network filtering and is
applied only after explicit confirmation.

The application does not disable Microsoft Defender Antivirus or modify Defender
Firewall profiles. It reports Defender profile state and enforces its own WFP
sublayer. This is a co-management design, not a replacement for Defender.

## Build from source

Requirements:

- LLVM-MinGW (Clang/LLD, UCRT, x86_64-w64-mingw32 target)
- Inno Setup 6 (only for the installer)
- PowerShell 5.1 or later

```powershell
.\build.ps1 -Clean -Tests -Installer
```

The build produces:

- `dist\CyberBoyFirewall.exe`
- `dist\CyberBoySmartFirewall-Setup-1.0.0.exe`
- `dist\cyberboy_tests.exe`
- `dist\default_policy.json`

See `docs/BUILD.md` for toolchain details.

## Project layout

```
src/core/     C policy core, JSON parser, logging, string/path helpers
src/engine/   C++ WFP engine, Windows integration, DNS/app collection
src/ui/       C++ Win32 + Direct2D interface and trilingual catalogue
src/tests/    Offline policy and translation tests
config/       Default policy document
installer/    Inno Setup script
assets/       Application icon
docs/         Installation, build, and security notes
```

## License

MIT. See `LICENSE`.
