# marxistgeek.org

`marxistgeek.org` 的静态站点骨架，托管在 **GitHub Pages** 上。

## 为什么搬到 GitHub Pages

原来的域名解析到阿里云 ECS `47.112.146.115`，但因为域名**未做 ICP 备案**，
阿里云的拦截服务（响应头 `Server: Beaver`）会对 80 端口返回
`Non-compliance ICP Filing` 403 页面，同时 443 端口未监听、HTTPS 完全不可用。
结果是境内访问该域名永远只能看到一个备案拦截页。

放在 GitHub Pages 上时流量不再落到境内服务器，因此**不需要备案**，
并且 GitHub 会为该域名自动签发 Let's Encrypt 证书，HTTPS 一并解决。

> 注意：GitHub Pages 在境内的可达性本身并非稳定有保障
> （`*.github.io` / `185.199.x.x` 时常缓慢或间歇不通）。
> 它解决的是"备案拦截"，不等于"境内访问体验一定好"。

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

## DNS 配置（阿里云域名控制台）

把 `marxistgeek.org` 的解析从 ECS 改到 GitHub Pages：

```
类型   主机记录   记录值
A      @          185.199.108.153
A      @          185.199.109.153
A      @          185.199.110.153
A      @          185.199.111.153
CNAME  www        wangsutan.github.io
```

配置完成后在仓库 **Settings → Pages** 里勾选 **Enforce HTTPS**。

## 后续

本仓库刻意保持为纯静态骨架。若要发布真正的动态应用（例如 Flask 实现的
「热搜助手」，含爬虫与 SQLite），需要先想清楚数据从哪来——
GitHub Pages 只能托管静态文件，无法运行后端进程。
