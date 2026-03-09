# 🔬 LLM 安全研究

> **作者**: nullinject
> 本仓库记录针对 LLM 应用与 AI 工具生态的安全研究与实战案例分析。
> 所有测试均在授权环境下进行，仅供安全研究与教育目的。

---

## 📋 研究案例索引

| # | 标题 | 目标 | 漏洞类型 | 严重程度 |
|---|------|------|----------|----------|
| 01 | [DeepSeek 伪造 API 上下文 Prompt Injection 攻击](./DeepSeek-Prompt-Injection.md) | DeepSeek Chat | Prompt Injection / Jailbreak | 🟠 高危 |
| 02 | [OpenClaw 恶意 Skill 供应链攻击：无沙箱 Shell 执行与配置窃取](./OpenClaw-Skill-Supply-Chain-Attack.md) | OpenClaw Desktop | Supply Chain / Arbitrary Shell Execution | 🔴 严重 |

---

## 🗂️ 漏洞类型覆盖

| OWASP LLM Top Ten | 对应案例 |
|-------------------|----------|
| LLM01:2025 Prompt Injection | [DeepSeek 案例](./DeepSeek-Prompt-Injection.md) |
| LLM03:2025 Supply Chain Vulnerabilities | [OpenClaw 案例](./OpenClaw-Skill-Supply-Chain-Attack.md) |
| LLM06:2025 Excessive Agency | OpenClaw 案例（次要） |

---

## ⚖️ 免责声明

本仓库所有内容仅供安全研究与教育目的。所有测试均在本人设备上的授权环境中进行，未对任何第三方用户造成影响。发现的漏洞均已按负责任披露原则提交给相关厂商。

---

## 📬 联系

如有问题或漏洞线索交流，欢迎通过 GitHub Issues 联系。
