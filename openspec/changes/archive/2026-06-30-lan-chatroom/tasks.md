## 1. HTML 骨架与全局 CSS

- [x] 1.1 创建 `chat.html`，包含完整 HTML 结构（连接面板、消息区、输入区、成员列表区）
- [x] 1.2 编写全局 CSS：深色/浅色主题、自适应布局、消息气泡样式、连接面板样式、成员列表样式
- [x] 1.3 添加浏览器 WebRTC 兼容性检测，不兼容时显示提示信息

## 2. P2P 连接层

- [x] 2.1 实现 `createPeerConnection(peerId)` 函数：创建 RTCPeerConnection，配置 STUN 服务器，注册 ICE/连接状态回调
- [x] 2.2 实现 DataChannel 管理：创建 `chat` 标签的 DataChannel，注册 `onmessage`/`onopen`/`onclose` 回调
- [x] 2.3 实现 `peers` 状态管理：Map 存储所有 peer，提供 `addPeer`/`removePeer`/`updatePeerState` 操作
- [x] 2.4 实现连接状态反馈：将 peer 连接状态变化反映到 UI（connecting/connected/disconnected/failed）

## 3. 信令系统

- [x] 3.1 定义邀请码格式（JSON + base64 编码），实现 `encodeInviteCode()` 和 `decodeInviteCode()` 函数
- [x] 3.2 实现 SDP Offer 生成流程：创建 PeerConnection → 生成 Offer → 编码为邀请码 → 显示
- [x] 3.3 实现 SDP Answer 生成流程：解码邀请码 → 创建 PeerConnection → 设置远程 SDP → 生成 Answer → 编码为应答码 → 显示
- [x] 3.4 实现应答码解析流程：解码应答码 → 设置远程 SDP → 完成握手
- [x] 3.5 实现多 peer 信令：加入者与每个已有成员分别建立连接

## 4. 聊天室管理

- [x] 4.1 实现创建房间流程：输入昵称 → 生成邀请码 → 进入等待状态
- [x] 4.2 实现加入房间流程：粘贴邀请码 → 输入昵称 → 生成应答码 → 完成握手 → 进入聊天界面
- [x] 4.3 实现成员列表 UI：显示在线成员、连接状态、昵称
- [x] 4.4 实现成员加入/离开事件通知（系统消息广播）

## 5. 消息系统

- [x] 5.1 定义消息 JSON 格式，实现 `sendMessage(text)` 通过所有活跃 DataChannel 广播
- [x] 5.2 实现消息接收处理：解析 JSON、校验格式、渲染到消息列表
- [x] 5.3 实现消息输入框：Enter 发送、按钮发送、空消息拦截
- [x] 5.4 实现消息列表 UI：气泡样式、发送者/接收者区分、时间戳、系统消息样式
- [x] 5.5 实现自动滚动与新消息提示

## 6. 消息持久化

- [x] 6.1 实现 localStorage 消息存取：每次收到/发送消息时保存，最多保留 200 条
- [x] 6.2 实现页面加载时恢复消息历史

## 7. 二维码（可选增强）

- [x] 7.1 动态加载 qrcode.min.js CDN（按需，不影响核心功能）
- [x] 7.2 实现"显示二维码"按钮：将邀请码编码为二维码
- [x] 7.3 QRCode 加载失败时优雅降级（仅显示复制按钮）

## 8. 最终打磨

- [x] 8.1 端到端手动测试：创建房间 → 邀请加入 → 多用户消息收发 → 断开重连
- [x] 8.2 处理边缘情况：空消息、特殊字符、长消息截断、大消息性能
- [x] 8.3 美化 UI：消息时间格式化、头像首字母、连接状态色彩标识
