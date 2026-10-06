<p align="center">
  <img src=".github/assets/product-mark.svg" width="96" height="96" alt="RST AI Copilot for Zabbix">
</p>

<h1 align="center">RST AI Copilot for Zabbix</h1>

<p align="center">
  <b>问网络，找根因。</b><br>
  私有化部署的 Zabbix AI 运维助手：把告警风暴收敛成一个根因，用自然语言回答网络问题，并给出处置建议。<br>
  默认只读、变更需人工确认、可离线运行。
</p>

<p align="center">
  <a href="https://github.com/reallysec/RST-Zabbix-AI-Copilot/releases"><img src="https://img.shields.io/github/v/release/reallysec/RST-Zabbix-AI-Copilot?label=release&color=D40000" alt="最新版本"></a>
  <img src="https://img.shields.io/badge/免费-社区版-1BA9F5" alt="免费社区版">
  <img src="https://img.shields.io/badge/Zabbix-6.0%E2%80%937.4-D40000" alt="Zabbix 6.0 至 7.4">
  <img src="https://img.shields.io/badge/部署-Docker-2496ED?logo=docker&logoColor=white" alt="Docker 部署">
  <a href="https://reallysec.com/docs/zabbix-ai-copilot"><img src="https://img.shields.io/badge/文档-reallysec.com-343741" alt="文档"></a>
</p>

<p align="center">
  <a href="README.md">English</a> · <b>简体中文</b> · <a href="https://reallysec.com/docs/zabbix-ai-copilot">文档</a> · <a href="https://github.com/reallysec/RST-Zabbix-AI-Copilot/releases">下载</a> · <a href="https://github.com/reallysec/RST-Zabbix-AI-Copilot/issues">反馈问题</a>
</p>

## 为什么选 RST AI Copilot for Zabbix

- **直接用你现有的 Zabbix。** 在现有 Zabbix（6.0 至 7.4）旁边起一个 Docker 网关即可。不新增数据存储、不装 agent，通过你掌控的 API token 调用 Zabbix JSON-RPC。
- **只读优先。** 查询只走只读 API 白名单。会改动 Zabbix 的操作（确认问题、创建触发器）都由运维人员显式发起，生成的触发器以禁用状态创建、审核后再启用。主机组白名单限定模型能看到的范围。
- **数据留在你的网络里。** 发给模型之前先做字段脱敏。模型可以用火山方舟、任意 OpenAI 兼容端点，也可以用本地 vLLM / Ollama 完全离线运行。
- **每一步都可追溯。** 每次登录、查询、模型调用和设置变更都是可检索的审计事件。

## 快速开始

需要：一台装有 Docker Engine 24+ 和 Compose v2 的 Linux 主机；能访问 Zabbix API（`…/api_jsonrpc.php`）并有 API token；一个 OpenAI 兼容的大模型端点；一个供运维人员访问的域名或 IP。

一条命令安装：

```bash
curl -fsSL https://github.com/reallysec/RST-Zabbix-AI-Copilot/releases/latest/download/install.sh | sudo bash
```

脚本下载最新安装包，用签名的发版清单校验，解压到 `/opt/rst-zabbix-ai-copilot` 并运行 `deploy.sh`。所有版本是同一个安装包：不导入许可时即为免费的社区版，在**许可证**页面激活许可后原地解锁专业版或企业版。离线主机可以在另一台机器上加 `--download-only` 下载，再把安装包拷过去。

也可以自己从 [Releases](https://github.com/reallysec/RST-Zabbix-AI-Copilot/releases) 下载：

```bash
sha256sum -c RST-Zabbix-AI-Copilot-<版本>.tar.gz.sha256
tar xzf RST-Zabbix-AI-Copilot-<版本>.tar.gz
cd RST-Zabbix-AI-Copilot-<版本> && ./deploy.sh
```

`deploy.sh` 会加载镜像、生成密钥和主机指纹、询问大模型和 Zabbix 端点，然后在 Caddy TLS 后面启动服务。网关不直接对外：主机上只有 Caddy 监听端口。服务就绪后打开 `https://<域名或IP>/v2/`。

默认证书由 Caddy 内置 CA 签发，浏览器会提示不受信任，信任其根证书即可。要换成自己的证书，改 `Caddyfile` 里的 `tls` 一行（见安装包内的部署手册 `docs/INSTALL.md`）。

**激活许可**（社区版跳过）：以管理员身份打开**许可证**页面。在线：粘贴许可密钥并激活，自动绑定本机（需要出站访问 `license.reallysec.com:443`）。离线（企业版）：复制主机指纹，到 [console.reallysec.com](https://console.reallysec.com) 或联系销售获取许可文件后导入。

**升级**：网关会在设置页检查本仓库 Releases 上有没有更新的签名版本。下载后版本进入暂存，在主机上运行 `sudo ./deploy/rst-update.sh` 安装，带健康检查，失败自动回滚。

## 功能

以下功能社区版全部免费（1 个用户、1 个节点、模型调用不限次数，模型自备）。

- **智能查询**：自然语言转 Zabbix API 调用（问题、事件、监控项、历史数据、触发器），结果表和趋势图，支持多轮追问。
- **实时告警**：轮询 Zabbix 问题或接入 Zabbix webhook 媒介，每条问题自动摘要，可直接确认。
- **态势**：值班看板，按严重度汇总问题、最吵的触发器和最近事件。
- **模型上下文**：监控项字典、运维知识库（RAG）、主机资产和拓扑。
- **定期巡检**：阈值、问题汇总、无数据三类检查，带环比变化。
- **隐私控制**：字段脱敏（云端 / 私有 / 离线三档）和按任务设置推理强度。
- **通知**：飞书、钉钉、企业微信、Teams、Slack 和邮件。

## 专业版与企业版

专业版解锁 AI 引擎：**告警批量研判**（告警风暴收敛到根因）、**告警调查**（自动取证与故障排查）、**触发器助手**（自然语言生成 Zabbix 触发器表达式，以禁用状态创建供审核）和**平台运维助手**（Zabbix Server 自身体检：队列、内部监控项、HA 节点、proxy），另有定时**运维报告**和更多用户数。企业版增加面向组织规模的集成能力。升级只需在同一套部署上导入许可：不用重装，数据和主机指纹都保留。14 天试用包含企业版全部功能，限 1 台主机。

<details>
<summary><b>版本对比</b></summary>

| | 社区版（免费） | 专业版 | 企业版 |
|---|:---:|:---:|:---:|
| [功能](#功能) 中的全部内容 | ✅ | ✅ | ✅ |
| 用户数 | 1 | 按席位 | 按报价 |
| **告警批量研判**：告警风暴收敛与根因定位 | — | ✅ | ✅ |
| **告警调查**：自动取证、时间线、受影响主机、处置建议 | — | ✅ | ✅ |
| **触发器助手**：阈值 / 聚合 / 无数据三类触发器表达式 | — | ✅ | ✅ |
| **平台运维助手**：AI 解读 Zabbix Server 体检结果 | — | ✅ | ✅ |
| **运维报告**：日报 / 周报 / 月报，定时推送 | — | ✅ | ✅ |
| 审计转发到外部 SIEM（syslog / webhook） | — | — | ✅ |
| 多模型供应商故障切换 | — | — | ✅ |
| OIDC 单点登录 / 企业身份集成 | — | — | ✅ |
| 离线许可激活 | — | — | ✅ |
| 节点数 | 1 | 1 | 不限 |
| 模型调用 | 不限 | 不限 | 不限 |

详情与价格：[版本说明](https://reallysec.com/docs/zabbix-ai-copilot/editions)。试用与许可：[console.reallysec.com](https://console.reallysec.com)。

</details>

## 架构与数据边界

- 入站：只有运维人员浏览器经 Caddy 访问 443。出站：大模型端点、你的 Zabbix API，以及 `license.reallysec.com`（企业版使用离线许可时不需要）。
- 对 Zabbix 只走只读 API 白名单；主机组白名单限定模型能查询的范围。
- 发给模型之前先做字段脱敏；离线模式下数据不出网。
- 每次登录、查询、模型调用和设置变更都是审计事件，企业版可转发到外部 SIEM。

## 支持的版本

| 组件 | 支持情况 |
|---|---|
| Zabbix | 6.0 LTS、6.4、7.0 LTS、7.2 和 7.4（JSON-RPC API，使用 API token） |
| 大模型端点 | 火山方舟、任意 OpenAI 兼容 API、自建 vLLM / Ollama |
| 主机 | Linux，Docker Engine 24+ 和 Docker Compose v2 |

## 支持

- **问题和 Bug**：提交 [issue](https://github.com/reallysec/RST-Zabbix-AI-Copilot/issues)。
- **安全漏洞**：请勿公开提交 issue，按[安全策略](SECURITY.md)报告。

## 许可

RST AI Copilot for Zabbix 是专有软件，以编译后的容器镜像形式按[《最终用户许可协议》](LICENSE)分发，每个安装包内也附有 `docs/EULA.md`。第三方组件及其许可见安装包内的 `THIRD-PARTY-NOTICES.md`。社区版无需许可即可免费使用；专业版和企业版通过在线激活或从 [console.reallysec.com](https://console.reallysec.com) 获取的离线许可文件启用。“RST”“Reallysec”“斯普朗克”及产品标识为商标。Zabbix 是 Zabbix LLC 的注册商标。RST AI Copilot for Zabbix 与 Zabbix LLC 无关联，也未获其认可或背书。

© 安徽斯普朗克信息技术有限公司
