# Cloudflare Workers VLESS + D1 流量统计（Lite）

<p align="center">
  <b>轻量 · 单文件 · 可直接部署</b><br/>
  基于 Cloudflare Workers 的 VLESS over WebSocket 实现，集成 D1 数据库进行全局流量统计
</p>

<p align="center">
  Cloudflare Workers · Cloudflare D1 · VLESS · WebSocket
</p>

---

## ✨ 项目简介（Hero）

本项目是一个 **基于 Cloudflare Workers 的轻量化 VLESS WebSocket 服务**，  
集成 **Cloudflare D1 数据库** 用于全局流量聚合统计，支持订阅生成与 ProxyIP 自动回源。

项目以 **单文件部署、低写入压力、可读性优先** 为设计目标，  
适合用于学习与实践 Cloudflare Workers 网络能力与 D1 数据库的实际使用。

> ⚠️ **流量统计说明**  
> 由于 D1 写入次数限制，采用 **内存累计 + 每 60 秒批量写入** 的方式。  
> 在 isolate 被回收等极端情况下，统计结果可能与实际流量存在轻微差异。

---

## 🚀 功能特性

- VLESS over WebSocket（TLS / WS）
- 单 `_worker.js` 文件即可部署
- D1 全局流量统计（分钟级聚合）
- Base64 订阅输出（V2Ray / Clash）
- ProxyIP 自动回源（按 colo 区域）
- ADDAPI 动态追加节点
- 内置管理页面（密码访问）

---

## 📂 项目结构

```text
.
├── _worker.js
└── README.md
```

---

## 🛠 部署前准备

- Cloudflare 账号（已启用 Workers 与 D1）
- 可选：Node.js ≥ 18

---

## 🗄 创建 D1 数据库

```sql
CREATE TABLE IF NOT EXISTS traffic_counter (
  id TEXT PRIMARY KEY,
  bytes INTEGER NOT NULL
);

INSERT OR IGNORE INTO traffic_counter (id, bytes)
VALUES ('global', 0);
```

---

## ⚙️ Worker 与 D1 绑定

- Variable name：`DB`
- Database：`traffic_db`

---

## 🔐 环境变量

| 名称 | 说明 |
|----|----|
| UUID | VLESS UUID |
| PASSWORD | 管理密码 |
| SUB_PATH | 订阅路径 |
| ADDAPI | 额外节点 API |

---

## 🔗 使用方式

- 管理页面：`https://你的域名/`
- VLESS 订阅：`https://你的域名/SUB_PATH`
- Clash 订阅：`https://sublink.eooce.com/clash?config=https://你的域名/SUB_PATH`

---

## ⚠️ 免责声明

仅供学习与研究使用，请遵守当地法律法规。

---

⭐ 欢迎 Star / Fork / 学习
