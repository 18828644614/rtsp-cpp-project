# Notes: C++ RTSP 项目

## 项目边界

### 第一版包含
- 输入一个 RTSP URL。
- 通过 TCP 建立 RTSP 控制连接。
- 支持 OPTIONS、DESCRIBE、SETUP、PLAY、TEARDOWN。
- 解析 SDP 中的视频轨道、编码格式、时钟频率和控制地址。
- 支持 RTP over RTSP/TCP 的 $ 交织数据。
- 识别 RTP 序号、时间戳、Marker、Payload Type 和 SSRC。
- 提取 H.264 NAL 单元并交给 FFmpeg 解码。
- 显示视频、统计 FPS/延迟/丢包等基础信息。
- 支持 RTSP 保活、停止和断线重连。
- 输出可读的协议日志。

### 第一版暂不包含
- 自己实现 H.264/H.265 解码器。
- 同时处理复杂的多路音频编码。
- 完整实现所有 RTSP 方法和所有认证算法。
- 一开始就支持 UDP、组播、NAT 穿透和公网部署。
- 录像数据库、用户管理和 Web 前端。

## 推荐技术栈

| 领域 | 选择 | 用途 |
|---|---|---|
| 语言 | C++17 或 C++20 | 主程序 |
| 构建 | CMake | 跨环境构建和依赖管理 |
| 网络 | Windows Winsock2，后续可封装成跨平台 Socket 层 | TCP/UDP 通信 |
| 媒体格式 | FFmpeg libavformat/libavcodec/libavutil/libswscale | RTSP 输入、解码、像素转换 |
| 图像显示 | 第一阶段 SDL2 或 OpenCV；正式桌面程序可用 Qt | 显示解码后的画面 |
| 流媒体服务 | MediaMTX | 接收摄像头推流并提供 RTSP |
| 测试工具 | FFmpeg、VLC、MediaMTX | 生成、播放、验证视频流 |
| 抓包 | Wireshark | 观察 RTSP/RTP/RTCP 交互 |
| 测试 | Catch2 或 GoogleTest | 单元测试和协议解析测试 |
| 日志 | spdlog 或自建轻量日志层 | 调试和故障定位 |

## 关键协议知识

### RTSP
- 请求响应格式和 CRLF 行结束。
- CSeq 请求序号。
- Session 会话标识。
- Transport 传输参数。
- Content-Length 和 TCP 粘包/拆包。
- DESCRIBE 返回 SDP。
- SETUP 协商轨道和 TCP interleaved 通道。
- PLAY、TEARDOWN 和保活。
- Basic/Digest 认证。

### SDP
- m=video 视频媒体描述。
- a=rtpmap 载荷类型、编码和时钟频率。
- a=fmtp 编码参数。
- a=control 轨道控制 URL。

### RTP
- Version、Payload Type、Marker。
- Sequence Number：检测乱序和丢包。
- Timestamp：安排播放时间和媒体同步。
- SSRC：识别同步源。
- H.264 的单 NAL、STAP-A、FU-A 打包。
- 一帧、NAL 单元和 RTP 包之间的关系。

### RTCP
- Sender Report（SR）。
- Receiver Report（RR）。
- 丢包率、累计丢包、抖动。
- NTP 与 RTP 时间戳映射。
- 可选的 NACK、PLI、FIR。

## 推荐目录规划

    rtsp-cpp-project/
    ├── CMakeLists.txt
    ├── README.md
    ├── LICENSE
    ├── docs/
    │   ├── protocol.md
    │   ├── build.md
    │   ├── testing.md
    │   └── troubleshooting.md
    ├── include/
    │   ├── net/
    │   ├── rtsp/
    │   ├── rtp/
    │   ├── media/
    │   └── app/
    ├── src/
    │   ├── net/
    │   ├── rtsp/
    │   ├── rtp/
    │   ├── media/
    │   └── app/
    ├── tests/
    ├── third_party/
    ├── configs/
    └── tools/

## 目标架构

    RtspClient
    ├── TcpConnection
    ├── RtspParser / RtspSerializer
    ├── SdpParser
    ├── RtspSession
    ├── InterleavedFrameParser
    ├── RtpPacketParser
    ├── H264Depacketizer
    ├── FfmpegDecoder
    ├── VideoRenderer
    ├── KeepAliveManager
    ├── ReconnectManager
    └── Logger / Metrics

## 验收标准

### 阶段验收
- 能用 VLC 播放 MediaMTX 提供的 RTSP 流。
- 能打印完整的 RTSP 请求和响应。
- 能正确解析 SDP。
- 能获得并保存 Session 和 Transport 参数。
- 能从 TCP 字节流中区分 RTSP 消息、RTP 和 RTCP。
- 能检测 RTP 丢包和乱序。
- 能显示连续 H.264 视频。
- 断开网络后能重连并恢复播放。
- 能通过 Wireshark 与程序日志互相验证。

### 稳定性验收
- RTSP 地址错误时给出明确错误。
- 服务端返回 401 时能够处理认证失败。
- TCP 半包、粘包不会导致解析错位。
- 收到非法长度或异常 RTP 包时不会崩溃。
- 停止播放时能释放 Socket、线程、FFmpeg 对象和窗口资源。
- 高码率视频下内存队列不会无限增长。

## 主要风险

- TCP 交织解析和 RTSP 消息解析共用一条字节流，必须处理半包和粘包。
- H.264 分片丢包可能导致当前 NAL 单元无法解码。
- TCP 重传会增加实时延迟，需要限制缓冲和处理旧帧。
- FFmpeg 不同版本的库文件、头文件和 ABI 配置可能不一致。
- 摄像头厂商的 RTSP URL、认证方式、SDP 写法可能存在差异。
- 某些设备只支持 UDP 或只支持特定的保活方法。

## 参考规范
- RFC 2326：RTSP 1.0
- RFC 3550：RTP/RTCP
- RFC 6184：H.264 over RTP
- RFC 8866：SDP
