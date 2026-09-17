# marxistgeek.org

`marxistgeek.org` 的静态站点骨架，托管在 **GitHub Pages** 上。

## 为什么搬到 GitHub Pages

原来的域名解析到阿里云 ECS `47.112.146.115`，但因为域名**未做 ICP 备案**，
阿里云的拦截服务（响应头 `Server: Beaver`）会对 80 端口返回
`Non-compliance ICP Filing` 403 页面，同时 443 端口未监听、HTTPS 完全不可用。
结果是境内访问该域名永远只能看到一个备案拦截页。

放在 GitHub Pages 上时流量不再落到境内服务器，因此**不需要备案**，
并且 GitHub 会为该域名自动签发 Let's Encrypt 证书，HTTPS 一并解决。

### 境内可达性实测

对 Pages 的四个边缘 IP 做过采样（境内网络，IPv4）：

| 测试项 | 结果 |
|---|---|
| 四个 IP 逐个直连 | 全部可达，0.31–0.66s |
| IPv4 连续 5 次采样 | 0.30 / 0.34 / 0.30 / 0.39 / 0.42s |
| 落点分布 | 108 / 109 / 110 / 111 均出现，四个 IP 都在健康轮询 |
| 强制 IPv6 | 4–7ms 即失败，无 IPv6 出口（本机环境） |

结论：走 IPv4 时到 Pages CDN 是快且稳定的。

> 但要注意这是**单机单点采样**，不代表所有境内网络都如此。曾有单次冷启动采样
> 出现 10s 量级的抖动，说明该链路存在偶发拥塞。域名切换后仍应以实网访问结果为准。
> 另外建议**不要**为自定义域名添加 IPv6（AAAA）记录：境内 IPv6 到该 CDN
> 路径质量参差，只用 IPv4 更稳。

## 这个仓库是什么

当前只有一个占位页，不含任何后端：

| 文件 | 作用 |
|---|---|
| `index.html` | 首页占位页（纯内联样式，不依赖任何外部 CDN，境内加载不受影响） |
| `404.html` | 自定义 404 页面 |
| `CNAME` | 绑定自定义域名 `marxistgeek.org` |
| `.nojekyll` | 关闭 Jekyll 处理，原样发布静态文件 |

## 部署方式

采用 **Pages「从分支部署」**：直接发布 `main` 分支根目录，无需构建步骤。
推送到 `main` 即自动更新线上站点。

## DNS 配置（GoDaddy）

域名的注册商与 DNS 托管**都是 GoDaddy**（权威 NS 为 `ns31.domaincontrol.com` /
`ns32.domaincontrol.com`），与阿里云无关——阿里云只是原先托管 ECS 的那一方。
因此解析要在 GoDaddy 的 DNS 管理页修改，不是阿里云域名控制台。

改前的记录只有两条，且没有 MX / TXT 需要保留：

```
@     A   47.112.146.115   TTL 600
www   A   47.112.146.115   TTL 3600   ← 是 A 记录，不是 CNAME
```

改成指向 GitHub Pages：

```
类型   主机记录   记录值
A      @          185.199.108.153
A      @          185.199.109.153
A      @          185.199.110.153
A      @          185.199.111.153
CNAME  www        wangsutan.github.io   ← 删除原有的 www A 记录后新建
```

**Enforce HTTPS 已在 Settings → Pages 开启。**

## 当前状态（已验证）

| 入口 | 结果 |
|---|---|
| `https://marxistgeek.org/` | 200，HTTP/2，证书有效 |
| `http://marxistgeek.org/` | 301 → https（各边缘节点灰度下发中） |
| `https://www.marxistgeek.org/` | 301 → `https://marxistgeek.org/`（证书 SAN 含 www） |
| 自定义 404 | 生效 |
| 阿里云 ECS `47.112.146.115:80` | 已关闭（nginx 已 stop + disable） |

证书由 Let's Encrypt 自动签发并续期，SAN 覆盖 apex 与 `www` 两个域名。
`www` 在 GoDaddy 侧是 CNAME 指向 `wangsutan.github.io`。

## 后续

本仓库刻意保持为纯静态骨架。若要发布真正的动态应用（例如 Flask 实现的
「热搜助手」，含爬虫与 SQLite），需要先想清楚数据从哪来——
GitHub Pages 只能托管静态文件，无法运行后端进程。
