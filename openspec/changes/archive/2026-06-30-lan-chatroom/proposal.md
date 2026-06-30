## Why

内网中小团队（3-8人）经常需要临时沟通，但公司内网环境往往不能搭建专用聊天服务器，使用微信/钉钉等外网工具又不安全或受限。需要一个零部署、零依赖、双击即用的纯前端局域网聊天方案。

## What Changes

- 创建一个单 HTML 文件 `chat.html`，包含完整聊天室功能
- 基于 WebRTC DataChannel 实现 P2P 直连，Mesh 拓扑，每对 peer 直接通信
- 信令通过手动复制粘贴 SDP 文本交换（base64 编码），无需任何信令服务器
- 支持创建房间（生成邀请码）和加入房间（粘贴邀请码）
- 实时文本消息收发，显示在线成员列表
- 可选 QRCode CDN 加载，支持手机扫码传递邀请码

## Capabilities

### New Capabilities

- `p2p-connection`: WebRTC PeerConnection 建立、DataChannel 管理、Mesh 拓扑维护、断开重连
- `signaling`: SDP Offer/Answer 生成、base64 编解码、剪贴板交互、解析验证
- `chat-room`: 房间创建/加入、在线成员管理、昵称设定
- `messaging`: 文本消息收发、消息列表渲染、消息持久化（localStorage）

### Modified Capabilities

<!-- No existing specs are modified -->

## Impact

- 在项目根目录新增 `chat.html`，无其他文件变更
- 无新增依赖（WebRTC API 为浏览器原生，QRCode 为可选 CDN 加载）
- 无构建步骤，无服务器端代码
