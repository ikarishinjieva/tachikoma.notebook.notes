---
title: 20260203 - 搭建openclaw服务器
confluence_page_id: 4620377
created_at: 2026-02-03T02:52:48+00:00
updated_at: 2026-02-03T05:26:06+00:00
---

服务器: ssh [root@10.186.61.17](<mailto:root@10.186.61.17>)

环境变量: 

```
export http_proxy=http://10.186.16.135:7890
export https_proxy=http://10.186.16.135:7890
export no_proxy=127.0.0.1,localhost,ucloud.cn
``` 

一键安装: 

```
curl -fsSL https://openclaw.ai/install.sh | bash
``` 

报错: 

```
You might want to run 'apt --fix-broken install' to correct these.
The following packages have unmet dependencies:
 gnupg : Depends: dirmngr (>= 2.4.4-2ubuntu17.4) but 2.4.4-2ubuntu17.3 is to be installed
         Depends: gnupg-utils (>= 2.4.4-2ubuntu17.4) but 2.4.4-2ubuntu17.3 is to be installed
         Depends: gpg (>= 2.4.4-2ubuntu17.4) but 2.4.4-2ubuntu17.3 is to be installed
         Depends: gpg-agent (>= 2.4.4-2ubuntu17.4) but 2.4.4-2ubuntu17.3 is to be installed
         Depends: gpgsm (>= 2.4.4-2ubuntu17.4) but 2.4.4-2ubuntu17.3 is to be installed
         Depends: gpgv (>= 2.4.4-2ubuntu17.4) but 2.4.4-2ubuntu17.3 is to be installed
         Depends: keyboxd (>= 2.4.4-2ubuntu17.4) but 2.4.4-2ubuntu17.3 is to be installed
         Breaks: dirmngr (< 2.4.4-2ubuntu17.4) but 2.4.4-2ubuntu17.3 is to be installed
         Recommends: gnupg-l10n (= 2.4.4-2ubuntu17.4) but 2.4.4-2ubuntu17.3 is to be installed
         Recommends: gpg-wks-client (>= 2.4.4-2ubuntu17.4) but 2.4.4-2ubuntu17.3 is to be installed
 ubuntu-server-minimal : Depends: cloud-init but it is not going to be installed
E: Unmet dependencies. Try 'apt --fix-broken install' with no packages (or specify a solution).
2026-02-03 02:25:13 - Error: Failed to install packages (Exit Code: 0)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
You might want to run 'apt --fix-broken install' to correct these.
The following packages have unmet dependencies:
 nodejs : Depends: libnode109 (= 18.19.1+dfsg-6ubuntu5) but it is not going to be installed
          Recommends: nodejs-doc but it is not going to be installed
 ubuntu-server-minimal : Depends: cloud-init but it is not going to be installed
E: Unmet dependencies. Try 'apt --fix-broken install' with no packages (or specify a solution).
``` 

执行修复: 

```
sudo apt update
sudo apt --fix-broken install
``` 

重新安装, 安装完成的信息: 

```
◇  How do you want to hatch your bot?
│  Open the Web UI
│
◇  Dashboard ready ────────────────────────────────────────────────────────────────╮
│                                                                                  │
│  Dashboard link (with token):                                                    │
│  http://127.0.0.1:18789/?token=a0b110a467c7f45e86d09e421a1d6c65415625c8fde868c3  │
│  Copy/paste this URL in a browser on this machine to control OpenClaw.           │
│  No GUI detected. Open from your computer:                                       │
│  ssh -N -L 18789:127.0.0.1:18789 root@10.186.61.17                               │
│  Then open:                                                                      │
│  http://localhost:18789/                                                         │
│  http://localhost:18789/?token=a0b110a467c7f45e86d09e421a1d6c65415625c8fde868c3  │
│  Docs:                                                                           │
│  https://docs.openclaw.ai/gateway/remote                                         │
│  https://docs.openclaw.ai/web/control-ui                                         │
│                                                                                  │
├──────────────────────────────────────────────────────────────────────────────────╯
│
◇  Workspace backup ────────────────────────────────────────╮
│                                                           │
│  Back up your agent workspace.                            │
│  Docs: https://docs.openclaw.ai/concepts/agent-workspace  │
│                                                           │
├───────────────────────────────────────────────────────────╯
│
◇  Security ──────────────────────────────────────────────────────╮
│                                                                 │
│  Running agents on your computer is risky — harden your setup:  │
│  https://docs.openclaw.ai/security                              │
│                                                                 │
├─────────────────────────────────────────────────────────────────╯
│
◇  Web search (optional) ─────────────────────────────────────────────────────────────────╮
│                                                                                         │
│  If you want your agent to be able to search the web, you’ll need an API key.           │
│                                                                                         │
│  OpenClaw uses Brave Search for the `web_search` tool. Without a Brave Search API key,  │
│  web search won’t work.                                                                 │
│                                                                                         │
│  Set it up interactively:                                                               │
│  - Run: openclaw configure --section web                                                │
│  - Enable web_search and paste your Brave Search API key                                │
│                                                                                         │
│  Alternative: set BRAVE_API_KEY in the Gateway environment (no config changes).         │
│  Docs: https://docs.openclaw.ai/tools/web                                               │
│                                                                                         │
├─────────────────────────────────────────────────────────────────────────────────────────╯
│
◇  What now ─────────────────────────────────────────────────────────────╮
│                                                                        │
│  What now: https://openclaw.ai/showcase ("What People Are Building").  │
│                                                                        │
├────────────────────────────────────────────────────────────────────────╯
│
└  Onboarding complete. Use the tokenized dashboard link above to control OpenClaw.

│
◇  Install shell completion script?
│  Yes
Completion installed. Restart your shell or run: source /root/.bashrc
``` 

重连ssh: 

```
ssh -N -L 18789:127.0.0.1:18789 root@10.186.61.17 
``` 

打开浏览器: 

```
http://localhost:18789/?token=a0b110a467c7f45e86d09e421a1d6c65415625c8fde868c3
``` 

搭建后对话失败, 通过以下命令诊断: 

```
root@24-04:~# openclaw agent --local --message "Hello, testing connection" --verbose full --agent main

OpenClaw 2026.2.1 (ed4529e) — I'm not saying your workflow is chaotic... I'm just bringing a linter and a helmet.

exception TypeError: fetch failed sending request
``` 

# 暂停自搭建工作

通过火山引擎上的openclaw应用搭建, 已经理解了openclaw的本质, 就不再进行自搭建
