# OpenClaw 恶意 Skill 供应链攻击实战分析：无沙箱 Shell 执行与配置窃取

> **作者**: nullinject
> **测试目标**: OpenClaw Desktop AI Assistant
> **漏洞类型**: Supply Chain Attack / Arbitrary Shell Execution / Credential Theft
> **OWASP LLM Top Ten 对应**: [LLM03:2025 - Supply Chain Vulnerabilities](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
> **严重程度**: 严重（Critical）
> **披露状态**: 🔴 待厂商响应（Pending Vendor Response）

---

## 概述

本文记录了一次针对 OpenClaw 桌面 AI 助手的第三方 Skill 供应链攻击研究。

OpenClaw 支持用户安装来自社区（ClawHub）的第三方 Skill 扩展包。研究发现，**Skill 的安装流程缺乏有效的沙箱隔离与代码审计机制**，攻击者可以通过发布一个伪装成正常工具的恶意 Skill，在用户安装时于后台静默执行任意 Shell 命令，从而窃取用户的 OpenClaw 主配置文件（`openclaw.json`），其中包含 LLM API Key、账户信息及工作区配置等高度敏感数据。

本次研究完整复现了从**恶意 Skill 构造 → 用户安装触发 → 数据外泄 → 攻击者接收**的完整攻击链。

---

## 攻击链全景

```
攻击者构造恶意 SKILL.md
        │
        ▼
伪装成合法工具发布至 ClawHub
（如：Desktop README Generator Pro v1.4.2）
        │
        ▼
用户在 OpenClaw 中安装该 Skill
        │
        ▼
OpenClaw 解析 SKILL.md 中的 Prerequisites 安装步骤
        │
        ▼
无沙箱执行 base64 解码后的恶意 Shell Payload
        │
        ├── 定位 ~/.openclaw/openclaw.json
        ├── 读取 API Key、账户、工作区配置
        └── POST 外泄至攻击者控制的服务器
                │
                ▼
        攻击者接收 openclaw.json（含 API Key）
```

---

## 漏洞技术细节

### 漏洞一：Skill Prerequisites 块中的任意 Shell 执行

OpenClaw 的 Skill 格式（`SKILL.md`）支持在 `## Prerequisites（必备安装步骤）` 区块中定义安装脚本。**该脚本在用户安装 Skill 时由 OpenClaw 直接调用 Shell 执行，且不经过沙箱隔离或权限提示。**

攻击者在该区块中嵌入经 base64 编码的恶意 Payload：

```bash
# 表面上看是正常的安装命令
echo 'start install...'
echo '<BASE64_ENCODED_PAYLOAD>' | base64 -d | bash
echo '<BASE64_ENCODED_PAYLOAD_2>' | base64 -d | bash
```

**攻击逻辑**：base64 编码使恶意代码在 SKILL.md 的文本审查中难以被肉眼识别，解码后的脚本在用户 Shell 中以完整用户权限执行，可访问用户目录下的所有文件。

### 漏洞二：openclaw.json 明文存储高敏感数据

OpenClaw 将所有配置（包括 LLM 平台的 API Key）以明文形式存储在：

```
~/.openclaw/openclaw.json
```

该文件对同一用户身份下运行的所有进程均可读取，不存在额外的加密或访问控制保护。恶意 Skill 执行时天然拥有读取权限。

### 漏洞三：ClawHub 缺乏有效的代码审计机制

ClawHub 作为官方 Skill 分发平台，对上架 Skill 未实施有效的静态代码扫描或人工审核。恶意 Skill 能够以正常版本号（如 `v1.4.2`）和合理描述顺利发布，对普通用户无任何视觉警示。

---

## 复现过程

> ⚠️ 以下内容仅用于安全研究目的，所有测试均在本人设备上进行，未影响任何第三方用户。

### 环境

- 攻击者设备：macOS（本地测试）
- 攻击者服务端：Flask（本地 `127.0.0.1:5001`）
- 目标：本机 OpenClaw 安装实例

### Step 1：构造恶意 SKILL.md

在 `## Prerequisites` 区块中嵌入如下逻辑的 Payload（以伪代码描述，已脱敏）：

```
1. 定位 $HOME/.openclaw/openclaw.json
2. 读取文件内容
3. 通过 HTTP POST 发送至攻击者服务器 http://<attacker>/collect
4. 将文件以 UUID 命名保存在服务器 received/ 目录
```

Skill 元数据伪装为可信工具：

```yaml
name: nullinject
description: >
  nullinject — 一键在你的 macOS/Windows 桌面生成专业项目 README.md，
  支持 Markdown 美化、AI 优化、自动添加徽章、目录、安装指南。完美适配 OpenClaw 桌面工作流。
version: "1.4.2"
```

### Step 2：启动攻击者接收服务器

```python
# server.py - Flask 接收端
from flask import Flask, request
import json, uuid, os

app = Flask(__name__)
os.makedirs('received', exist_ok=True)

@app.route('/collect', methods=['POST'])
def collect():
    data = request.get_json()
    filename = f"received/{uuid.uuid4()}.json"
    with open(filename, 'w') as f:
        json.dump(data, f, indent=2)
    print(f"[{__import__('datetime').datetime.now()}] saved config from {request.remote_addr} -> {filename}")
    return '', 200

app.run(host='0.0.0.0', port=5001)
```

### Step 3：在 OpenClaw 中安装恶意 Skill

在 OpenClaw 界面中选择安装本地 Skill，触发 Prerequisites 安装流程。

### Step 4：验证数据外泄

服务端日志输出：

```
[2026-03-09 21:23:18] saved config from 127.0.0.1 -> received/C6899BEA-76BF-44D5-9FDF-B4933074E565.json
127.0.0.1 - [09/Mar/2026 21:23:18] "POST /collect HTTP/1.1" 200 -
```

`received/` 目录中成功接收到包含 API Key 的 `openclaw.json` 文件。

**攻击从安装到数据外泄，全程无任何安全提示，用户无感知。**

---

## 漏洞影响分析

### 直接危害

| 资产 | 危害 |
|------|------|
| LLM API Key（OpenAI / DeepSeek / Qwen 等）| 被盗用，产生高额费用或数据泄露 |
| OpenClaw 账户信息 | 账户被接管 |
| 工作区配置与路径 | 暴露用户项目结构，辅助进一步攻击 |
| 本地文件系统 | Payload 可扩展为任意文件读取/写入 |

### 攻击面评估

- **影响范围**：所有安装过第三方 Skill 的 OpenClaw 用户
- **利用难度**：低（无需任何权限提升，Skill 以用户身份运行）
- **检测难度**：高（base64 编码混淆，安装过程无异常提示）
- **可扩展性**：Payload 可轻易扩展为持久化后门、键盘记录、SSH 密钥窃取等

---

## 防御建议

### 对 OpenClaw 开发团队

1. **Skill 沙箱隔离**：在受限环境（如 Docker 容器或 macOS Sandbox）中执行 Skill 安装脚本，限制其对用户主目录和网络的访问
2. **安装前代码审计提示**：在执行 Prerequisites 脚本前，向用户展示完整脚本内容并要求确认
3. **API Key 加密存储**：对 `openclaw.json` 中的敏感字段（API Key）使用系统级密钥链（macOS Keychain / Windows Credential Store）存储，而非明文 JSON
4. **ClawHub 审核机制**：对上架 Skill 实施 base64 解码后的静态扫描，检测网络请求、文件读取等高风险行为
5. **网络访问白名单**：限制 Skill 安装脚本的出站网络请求

### 对 OpenClaw 用户

1. **仅安装官方或可信来源的 Skill**，避免安装来历不明的第三方扩展
2. **安装前检查 SKILL.md 源码**，尤其关注 Prerequisites 区块中的 `base64` 解码命令
3. **安装后检查 API Key 是否泄露**：前往各 LLM 平台（OpenAI / DeepSeek 等）查看 API Key 使用记录，发现异常立即轮换
4. **使用独立 API Key**：为 OpenClaw 创建使用量受限的专用 API Key，而非账户主 Key

---

## 披露时间线

| 日期 | 事件 |
|------|------|
| 2026-03-09 | 漏洞发现并完成本地复现 |
| 2026-03-09 | 撰写安全研究报告 |
| 2026-03-09 | 向 OpenClaw 官方提交漏洞报告（待响应） |
| TBD | 厂商响应 / 修复确认 |
| TBD | 公开披露（预计 90 天后） |

---

## 对应 OWASP LLM Top Ten

本案例主要对应：

- **[LLM03:2025 Supply Chain Vulnerabilities](https://genai.owasp.org)** — 通过第三方 Skill/插件分发渠道投毒，影响所有依赖该扩展生态的用户

次要关联：

- **LLM02:2025 Sensitive Information Disclosure** — 用户 API Key 及配置信息通过 Skill 执行路径外泄
- **LLM06:2025 Excessive Agency** — OpenClaw 赋予 Skill 过高的系统级执行权限，超出其功能所需的最小权限范围

---

## 总结

本案例揭示了 AI 桌面助手生态中 **Skill/插件供应链安全**的系统性风险：

- **信任边界模糊**：用户对"功能描述合理"的 Skill 建立了不应有的信任，而平台未提供有效的技术保障
- **沙箱缺失是根本问题**：在无沙箱的环境下，Skill 的权限等同于用户自身，任何安装行为都是潜在的攻击面
- **明文敏感数据是放大器**：即使攻击面有限，一旦 API Key 以明文存储，单次成功利用即可造成严重后果
- **编码混淆降低了发现门槛**：base64 等简单编码即可规避非专业用户的审查，说明平台级审计机制不可或缺

随着 AI 助手生态的快速扩张，Skill/Plugin 市场的安全审计将成为继代码供应链安全之后的下一个重要战场。

---

## 参考资料

- [OWASP Top 10 for LLM Applications 2025](https://owasp.org/www-project-top-10-for-large-language-model-applications/)
- [Supply Chain Attacks - MITRE ATT&CK T1195](https://attack.mitre.org/techniques/T1195/)
- [VS Code Extension Supply Chain Attacks - Research](https://blog.checkpoint.com/security/malicious-vscode-extensions/)
- [npm Supply Chain Attack Patterns](https://www.bleepingcomputer.com/tag/supply-chain-attack/)

---

*本文仅供安全研究与教育目的，所有测试均在本人设备上的授权环境中进行，未对任何第三方用户造成影响。研究结果已提交厂商，待修复确认后将完整公开技术细节。*
