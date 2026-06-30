## Context

局域网聊天室是一个单 HTML 文件的纯前端应用。内网团队无法/不愿搭建聊天服务器时，双击 HTML 即可开始 P2P 聊天。所有数据通过 WebRTC DataChannel 直连传输，不经任何服务器中转。

## Goals / Non-Goals

**Goals:**
- 单个 HTML 文件，打开即用，零构建步骤
- P2P 直连通信，不依赖任何后端服务器
- 支持 3-8 人小团队实时文本聊天
- 复制粘贴 SDP 作为信令交换方式
- 浏览器原生 API 实现，零 npm 依赖

**Non-Goals:**
- 文件传输（预留扩展空间，首版不做）
- 消息已读回执
- 音视频通话
- 与外部网络互通（纯内网）
- 跨 NAT 支持（依赖 TURN 服务器，本设计不包含）

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     chat.html                                │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              State Management (全局对象)              │    │
│  │  peers: Map<peerId, {nickname, pc, channel, state}> │    │
│  │  messages: Array<{type, from, text, id, timestamp}>  │    │
│  │  myNickname: string                                  │    │
│  └─────────────────────────────────────────────────────┘    │
│              │                        │                      │
│     ┌────────▼────────┐     ┌─────────▼────────┐           │
│     │  Signaling      │     │  WebRTC Layer    │           │
│     │  (手动信令交换)   │     │  PeerConnections  │           │
│     │  - 创建 Offer    │     │  DataChannels    │           │
│     │  - 创建 Answer   │     │  ICE 协商        │           │
│     │  - 解析加入      │     │  连接状态管理     │           │
│     └─────────────────┘     └─────────▲────────┘           │
│                                       │                      │
│                              ┌────────▼────────┐           │
│                              │  Messaging       │           │
│                              │  消息发送接收    │           │
│                              │  JSON 序列化     │           │
│                              │  localStorage    │           │
│                              └─────────────────┘           │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              UI Layer                                │    │
│  │  连接面板 | 消息列表 | 输入框 | 成员列表            │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Data Structures

### Peer State
```
peer = {
  id: string,           // 随机 UUID
  nickname: string,
  pc: RTCPeerConnection,
  channel: RTCDataChannel | null,
  state: 'connecting' | 'connected' | 'disconnected' | 'failed'
}
```

### Message Format
```
message = {
  type: 'text' | 'system',
  from: string,         // 发送者昵称
  text: string,         // 消息内容
  id: string,           // UUID
  timestamp: number     // unix ms
}
```

### Invite Code Format
```
// 创建者 -> 加入者
inviteCode = base64Encode({
  version: 1,
  type: 'offer',
  roomName: string,     // 房间名（可选，仅标识用途）
  sdp: string,          // SDP offer
  peerId: string,       // 创建者 peer ID
  nickname: string      // 创建者昵称
})

// 加入者 -> 创建者
replyCode = base64Encode({
  version: 1,
  type: 'answer',
  sdp: string,          // SDP answer
  peerId: string,       // 加入者 peer ID
  nickname: string      // 加入者昵称
})
```

## Decisions

### Decision 1: 手动复制粘贴信令 vs 局域网广播发现
- **选择**: 手动复制粘贴
- **原因**: 浏览器没有可靠的局域网 peer 发现 API。mDNS 和 SSDP 不可从网页中使用。复制粘贴是最通用、最可靠的方案。
- **替代方案**: 
  - mDNS/mDNS-SD: 浏览器不支持
  - WebTorrent/IPFS: 太重，引入不可控的外部依赖

### Decision 2: Mesh 拓扑 vs 星型拓扑
- **选择**: Mesh（全互联）
- **原因**: 3-8 人场景没问题。每增加一人新增 N-1 个连接，8 人时每人 7 个连接，远低于浏览器上限（通常 30+）。无需中心节点，没有单点故障。
- **替代方案**: SFU/MCU 需要转发服务器，违背"纯前端"目标

### Decision 3: 单个 HTML 文件架构
- **选择**: 所有 JS/CSS/HTML 合并为一个文件
- **原因**: 零部署，双击即用。WebRTC 核心逻辑 300-400 行，UI 渲染 200-300 行，CSS ~200 行，单文件完全可维护。使用原生 DOM API (document.createElement etc.)，不引入框架。

### Decision 4: ICE 服务器策略
- **选择**: 使用 Google 公共 STUN 服务器
- **原因**: 纯前端无法自建 STUN，公共 STUN 对内网场景已足够（同内网通常不需要 TURN）。如果严格内网不允许出站，可注释掉 STUN 配置，走 host 候选即可建立直连。
- **风险**: 某些企业内网可能屏蔽 STUN 端口（3478）。此时可降级为 host-only。

### Decision 5: 可选 QRCode 支持
- **选择**: 通过 CDN 动态加载 qrcode.min.js（运行时按需加载）
- **原因**: 需要时才加载，不增加核心功能复杂度。手机扫码传递邀请码比复制粘贴更方便。不影响内网使用（CDN 不可用时扫码不可用，复制粘贴不受影响）。

## UI Layout

```
┌────────────────────────────────────────────────────────────┐
│  💬 LAN Chat  [在线: 3]                                    │
├────────────────┬───────────────────────────────────────────┤
│                │                                           │
│  [成员列表]    │  [消息列表]                                │
│                │  ┌─────────────────────────────────────┐  │
│  ● Alice       │  │ Alice: 大家今天上线了吗?    10:30  │  │
│  ● Bob         │  │ Bob: 来了                     10:31  │  │
│  ● Carol       │  │ Carol: 我在调试那个bug        10:31  │  │
│  (我: Alice)   │  │                                     │  │
│                │  │                                     │  │
│                │  │                                     │  │
│                │  └─────────────────────────────────────┘  │
│                │  ┌────────────────────────────────────┐   │
│                │  │ [输入消息...]            [发送] 🔍 │   │
│                │  └────────────────────────────────────┘   │
├────────────────┴───────────────────────────────────────────┤
│  📡 连接面板（可折叠）                                      │
│  ┌────────────────────────────────────────────────────┐   │
│  │  昵称: [Alice                ]                     │   │
│  │                                                     │   │
│  │  [🚀 创建房间]    或    粘贴邀请码加入:             │   │
│  │  [________________________________] [🔗 加入]     │   │
│  │                                                     │   │
│  │  邀请码: (创建房间后显示)                           │   │
│  │  ┌────────────────────────────────────────────┐    │   │
│  │  │ eyJ2ZXJzaW9uIjoxLCJ0eXBlIjoib2ZmZXIi...  │    │   │
│  │  └────────────────────────────────────────────┘    │   │
│  │  [📋 复制邀请码]  [📱 显示二维码]                  │   │
│  └────────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────────┘
```

UI 状态机：

```
┌────────────┐    创建房间      ┌──────────────┐    连接建立    ┌─────────┐
│            │────────────────→│              │───────────────→│         │
│  连接面板   │                 │  等待加入者   │               │ 聊天中  │
│  (初始状态) │←────────────────│ (展示邀请码)  │               │         │
│            │    重新连接      │              │               │         │
└────────────┘                 └──────────────┘               └─────────┘
       │                                                          │
       │   粘贴邀请码                                              │  断开连接
       │   +  加入                                                │
       └──────────────────────────────────────────────────────────┘
```

## Connection Flow (Sequence)

```
Creator                            Joiner
  │                                  │
  │ 1. Create PeerConnection         │
  │ 2. Generate SDP Offer            │
  │ 3. Encode as invite code         │
  │ 4. Display invite code           │
  │                                  │
  │ ←── 用户复制邀请码, 发给对方 ──→ │
  │                                  │
  │                                  │ 5. Decode invite code
  │                                  │ 6. Create PeerConnection
  │                                  │ 7. Set remote SDP (offer)
  │                                  │ 8. Generate SDP Answer
  │                                  │ 9. Display reply code
  │                                  │
  │ ←── 用户复制应答码, 发给创建者 ──→ │
  │                                  │
  │ 10. Set remote SDP (answer)      │
  │ 11. ICE completes                │
  │ 12. DataChannel opens            │
  │ 13. Send "joined" notification   │
  │                                  │
  │ ←───────── P2P Chat! ──────────→ │
```

For N > 2 participants: the process repeats pairwise for each existing participant when a new joiner enters.

## Risks / Trade-offs

- **[用户体验]** 复制粘贴 SDP 比自动连接繁琐 → 这是纯前端方案的根本约束，无服务器就无法自动发现。二维码扫码可缓解。
- **[扩展性]** Mesh 拓扑在 10+ 用户时连接数 O(n²) → 明确限定 3-8 人场景。8 人时 28 条总连接，每节点 7 条，性能可接受。
- **[连接失败]** 严格内网环境可能完全阻断 WebRTC → 提供 ICE 候选查看能力帮助调试。可设置 `iceTransportPolicy: relay` 或关闭 STUN 使用 host-only。
- **[单文件膨胀]** 所有代码在一个文件 → 保持模块化组织（函数分组 + 注释分区），不引入框架。
- **[浏览器兼容]** 旧版浏览器（< Chrome 56 / < Safari 11）不支持 WebRTC → 页面增加兼容性检测提示。

## Open Questions

<!-- 无待解决的设计问题 -->
