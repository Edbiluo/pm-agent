---
【标签】Figma, MCP, REST API, Figma Token, Desktop Bridge, design-to-code, 设计稿读取
【场景】Claude Code 通过 Figma MCP 读取设计稿数据，不依赖 Figma Desktop Bridge（WebSocket）
【问题】Desktop Bridge 需要本地插件在线且 WebSocket 端口易被多 session 占满，无法稳定读取设计稿
【根因】Desktop Bridge 使用 WebSocket（端口 9223-9232），多个 Claude Desktop session 并存时端口冲突；官方 Figma MCP（plugin:figma）通过 REST API + FIGMA_ACCESS_TOKEN 独立工作，不受端口限制
【标准解法】
使用官方 Figma MCP（plugin:figma）的 REST API 工具读取设计数据：
- 读取设计：mcp__plugin_figma_figma__get_design_context（最常用，返回代码参考 + 截图）
- 截图：mcp__plugin_figma_figma__get_screenshot
- 结构：mcp__plugin_figma_figma__get_metadata
- 搜索组件：mcp__plugin_figma_figma__search_design_system
- 变量定义：mcp__plugin_figma_figma__get_variable_defs

前提条件：FIGMA_ACCESS_TOKEN 环境变量已配置
URL 解析：从 figma.com/design/:fileKey/:fileName?node-id=:nodeId 提取 fileKey 和 nodeId（"-"转":"）

写入操作（创建/修改节点）仍需 Desktop Bridge：figma_execute / use_figma
【避坑要点】
- Desktop Bridge 状态检测：figma_get_status 的 probe:true 可主动验证连接是否存活
- 端口占满处理：关闭多余 Claude Desktop session 或终端，重启当前 session
- figma_capture_screenshot 属于 Desktop Bridge 工具；get_screenshot 属于 REST API 工具，不要混淆
- get_design_context 返回的代码是 React+Tailwind 参考实现，需按项目技术栈（uni-app/Vue3）适配
【复用提示】当需要读取 Figma 设计稿内容、获取组件规范、截图时，优先使用 plugin:figma 的 REST API 工具（get_design_context / get_screenshot），无需检查 Desktop Bridge 状态。仅在需要写入 Figma 时才启动 Desktop Bridge。
---
