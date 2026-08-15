# Agent Note: 禁止持久缓存按请求生成的 Web index

Status: implemented

[English](2026-08-15-dynamic-index-no-store.md) | 中文

## Problem

Web 回退会为每次请求重新读取并转换 `index.html`，但响应没有任何缓存指令。所得文档不是静态资产：index 转换器会注入当前客户端插件图和插件启动前的主题脚本。长期使用的浏览器配置目录因此可能保留一份启动数据与下次启动时 Host 不一致的入口文档；受损的已存储响应还可能在应用挂载前暴露内联启动内容或样式文本，而使用另一套配置目录的浏览器仍能正常加载同一个 Host。

## Decision

`dsh-host-frontend-static` 为所有 index 响应发送 `Cache-Control: no-store`，包括根路径、显式 `index.html`、SPA 回退路径及其 HEAD 形式。插件仍会在每次响应前读取 dist index 并应用当前 index 转换器。

静态资产响应保持现有行为。带内容哈希的 Vite 文件名和带修订号的客户端插件 URL 仍是各自的缓存身份；只有包含按请求组合数据的入口文档被排除在持久缓存之外。

## Alternatives considered

**使用 `no-cache`。** 未采用，因为它仍允许存储并依赖重新验证，而服务器没有为 index 提供验证器。入口文档很小，也没有离线使用约定，保留它收益有限。

**给 index URL 加版本。** 未采用，因为浏览器和桌面壳都从稳定根 URL 进入，不应在加载前先知道当前 Host 图。带版本的子资产已经拥有各自的身份。

**启动时清除每个客户端的完整缓存。** 未采用，因为响应语义由服务器拥有；清空整个配置目录的缓存会丢弃无关的不可变资产，也无法保护其他客户端。

## Consequences

每次页面导航都会获取当前入口文档，并收到当前启动图和主题启动脚本。重新加载无法复用持久存储的 index，而脚本、样式、插件 bundle、图标和 manifest 继续保持原有缓存行为。

真实 Loader 组合测试固定了根路径、显式 index、SPA 回退和 HEAD 响应上的 `no-store`，并固定 JavaScript 资产不会获得新的缓存指令。
