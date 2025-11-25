# X 账号归属地标注工具

[简体中文](README.md) | [English](docs/README.en.md) | [日本語](docs/README.ja.md)

## 目录

- [项目简介](#项目简介)
- [安装与使用](#安装与使用)
- [技术实现](#技术实现)
- [隐私与安全](#隐私与安全)

## 项目简介

X 在"关于此账号"页面中显示账号位置和 App Store 区域信息，但这些数据在时间线上不可见。本脚本通过在每条推文时间戳旁添加交互按钮，让您可以按需查询位置信息。

**主要功能：**
- **按需查询**：点击查询，无自动批量请求，避免速率限制
- **VPN 检测**：颜色编码结果（灰色表示已验证位置，红色表示可能使用 VPN）
- **智能缓存**：已查询的账号自动显示标签，无需额外 API 调用

脚本使用 X 的内部 GraphQL API 获取位置数据，并根据位置准确度显示相应的视觉标识。

## 安装与使用

1. 安装 [Tampermonkey](https://www.tampermonkey.net/) 或 [Violentmonkey](https://violentmonkey.github.io/)
2. （推荐）直接从 [Greasy Fork](https://greasyfork.org/zh-CN/scripts/556852-x-account-location-tagger) 安装，或手动创建新的用户脚本
3. 如果选择手动安装，复制 `X.js` 的内容
4. 保存并启用脚本
5. 访问 X 并开始点击位置按钮

访问任意 X 时间线，您会看到推文时间戳旁有地图图标：

- 点击图标查询位置
- 按钮在获取数据时显示加载状态
- 结果显示为：`[国家/地区 / App Store 区域]`
- 灰色文字表示已验证的位置
- 红色文字表示可能使用了 VPN

## 技术实现

脚本使用 X 的内部 GraphQL API 端点 `AboutAccountQuery`，这是支持"关于此账号"功能的同一端点。当您点击位置按钮时，它会：

1. 从推文的时间戳链接中提取用户名
2. 使用您的会话凭证发送 GraphQL 查询
3. 解析响应中的 `account_based_in` 和 `source` 字段
4. 检查 `location_accurate` 以判断 VPN 可能性
5. 使用适当的颜色编码显示结果

所有请求都包含来自您活跃 X 会话的正确 CSRF 令牌和身份验证标头。

脚本解析以下格式的响应：

```json
{
  "data": {
    "user_result_by_screen_name": {
      "result": {
        "about_profile": {
          "account_based_in": "Japan",
          "location_accurate": true,
          "source": "Japan App Store"
        }
      }
    }
  }
}
```

关键字段：
- `account_based_in`：地理位置
- `source`：App Store 区域或客户端应用
- `location_accurate`：VPN 检测标志

脚本只需最少配置，所有设置均已硬编码：

```javascript
const ABOUT_ENDPOINT = 'https://x.com/i/api/graphql/zs_jFPFT78rBpXv9Z3U2YQ/AboutAccountQuery';
const AUTH_BEARER = 'Bearer [官方 Web 客户端令牌]';
const DEBUG = true;
```

调试模式会将所有请求和响应记录到控制台。设置为 `false` 可禁用。

按需设计自然避免了速率限制问题。您只在点击时发送请求，缓存机制防止重复查询。如果遇到 429 错误，请等待几分钟后继续。

## 隐私与安全

本脚本：
- 仅使用公开的 X API 端点
- 显示的信息在界面中已经可见
- 不向第三方服务器发送任何数据
- 所有处理都在浏览器本地完成
- 需要您的活跃 X 会话（必须已登录）

您看到的数据是 X 已经知道并在账号详情页显示的内容。本脚本只是让这些信息更易访问。

## 许可证

MIT 许可证 - 自由使用、按需修改、不提供保证。

## 致谢

通过逆向工程 X 的内部 GraphQL API 构建。感谢让这一切成为可能的浏览器开发者工具。

---

**注意**：本脚本依赖 X 的内部 API，该 API 可能随时更改而不另行通知。如果 X 修改其 GraphQL 架构或端点结构，可能需要更新脚本。
