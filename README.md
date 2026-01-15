# Cloudflare Workers VLESS + D1 流量统计（精简版）

这是一个 **基于 Cloudflare Workers** 的 VLESS WebSocket 服务示例，
集成了 **Cloudflare D1 数据库** 用于全局流量统计，并提供：

- VLESS WS 中转（支持自动回源 ProxyIP）
- Base64 订阅输出（V2Ray / Clash 可用）
- 可视化主页（登录查看订阅与流量）
- D1 数据库按分钟聚合写入，节省写入次数
- 支持 ADDAPI 动态追加 CDN / IP 列表

---

## 一、项目结构

```text
.
├── _worker.js        # Cloudflare Workers 主入口文件
└── README.md         # 使用说明（本文档）
```

你只需要 **一个 `_worker.js` 文件即可运行**。

---

## 二、部署前准备

### 1️⃣ Cloudflare 账号
- 需要一个 Cloudflare 账号（免费或付费均可）
- 已开启 **Workers** 与 **D1** 功能

### 2️⃣ 本地环境（可选）
如果你想用命令行部署：
- Node.js ≥ 18
- npm / pnpm / yarn

（也可以直接在 Cloudflare Dashboard 在线编辑部署）

---

## 三、创建 D1 数据库（非常重要）

### 方式一：Dashboard（推荐）

1. 登录 Cloudflare
2. 进入 **Workers & Pages → D1**
3. 点击 **Create database**
4. 名称示例：

```
traffic_db
```

### 初始化表结构

进入该 D1 数据库 → **Console**，执行以下 SQL：

```sql
CREATE TABLE IF NOT EXISTS traffic_counter (
  id TEXT PRIMARY KEY,
  bytes INTEGER NOT NULL
);

INSERT OR IGNORE INTO traffic_counter (id, bytes)
VALUES ('global', 0);
```

⚠️ **必须先插入 `id = 'global'` 这一行，否则统计无法更新**

---

## 四、创建 Worker 并绑定 D1

### 1️⃣ 创建 Worker

- Workers & Pages → Create Worker
- 名称示例：

```
vless-d1-worker
```

- 将 `_worker.js` 内容完整粘贴进去
- 保存

### 2️⃣ 绑定 D1 数据库

进入 Worker → **Settings → Bindings → D1 database bindings**

| 项目 | 值 |
|----|----|
| Variable name | `DB` |
| D1 database | 选择你创建的 `traffic_db` |

保存设置。

---

## 五、环境变量配置

进入 Worker → **Settings → Variables → Environment Variables**

### 必填 / 常用变量

| 变量名 | 说明 | 示例 |
|----|----|----|
| UUID | VLESS UUID | `469cb497-03dd-4d8e-967d-366e0ffe9551` |
| PASSWORD | 登录主页密码 | `password` |
| SUB_PATH | 订阅路径（可选） | `link` / 留空则自动用 UUID |

### 可选变量（推荐）

#### ADDAPI（动态追加节点）

支持多个地址，用逗号分隔：

```
ADDAPI=https://example.com/ip.txt,https://example2.com/list.txt
```

返回格式示例：

```
1.1.1.1:443#节点A
example.com:8443#节点B
```

---

## 六、使用方式

### 1️⃣ 访问主页

```
https://你的域名/
```

首次需要输入 `PASSWORD` 登录。

### 2️⃣ VLESS 订阅地址

```
https://你的域名/SUB_PATH
```

或（未设置 SUB_PATH 时）：

```
https://你的域名/UUID
```

### 3️⃣ Clash 订阅

```
https://sublink.eooce.com/clash?config=https://你的域名/SUB_PATH
```

---

## 七、流量统计说明（D1）

- 所有用户流量统一累计到：

```
traffic_counter.id = 'global'
```

- Worker **不会每次断连就写数据库**
- 而是：
  - 内存累计
  - **每 60 秒 flush 一次**
- 大幅降低 D1 写入压力

⚠️ Cloudflare 可能回收 isolate，极少量流量在极端情况下可能延迟写入，但整体非常稳定。

---

## 八、ProxyIP 自动回源逻辑

- 优先直连目标
- 失败后按 `colo → 区域 → ProxyIP` 自动回源
- 支持：JP / HK / EU / AS / US

你可以在代码中修改：

```js
const proxyIpAddrs = {
  EU: 'ProxyIP.DE.xxx',
  HK: 'ProxyIP.HK.xxx',
  AS: 'ProxyIP.SG.xxx',
  JP: 'ProxyIP.JP.xxx',
  US: 'ProxyIP.US.xxx'
};
```

---

## 九、安全建议

- 修改默认 UUID 与 PASSWORD
- 不要公开管理页面密码
- 如大规模使用，建议：
  - 多 Worker
  - 多域名轮询

---

## 十、免责声明

本项目仅用于 **技术研究与学习 Cloudflare Workers / D1 架构**。
请遵守你所在地区的法律法规，**严禁滥用**。

---

## ⭐ 感谢

如果你觉得这个项目对你有帮助，欢迎自行 Fork / 修改 / 优化。
