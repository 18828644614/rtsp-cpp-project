# Plan

这个项目将使用 C++ 构建一个可运行、可调试的 RTSP 拉流客户端。实施路线先用 MediaMTX 和 FFmpeg 验证视频链路，再逐步实现 RTSP 会话、SDP 解析、TCP 交织 RTP/RTCP 解析、H.264 解包和 FFmpeg 解码；第一版优先支持单路 H.264 视频和 RTSP over TCP，UDP 放到后续阶段。

## Scope
- In:
  - C++17/C++20、CMake、Winsock2。
  - RTSP 1.0 请求响应和会话管理。
  - SDP 解析。
  - RTSP over TCP 的 RTP/RTCP interleaved 解析。
  - H.264 over RTP 的 NAL 重组。
  - FFmpeg 解码和视频显示。
  - 保活、断线重连、日志、统计和测试。
  - 后续增加 RTP/RTCP over UDP。
- Out:
  - 自研 H.264/H.265 解码器。
  - 第一版的完整音频系统。
  - Web 前端、录像管理后台、用户权限系统。
  - 一开始支持所有 RTSP 扩展、组播和 NAT 穿透。

## Action items
[ ] 准备 Windows C++ 开发环境，确认 CMake、编译器、FFmpeg、MediaMTX 和 Wireshark 可用。

[ ] 使用 FFmpeg 采集电脑摄像头或测试视频，推送到 MediaMTX，并用 VLC 验证 RTSP 地址。

[ ] 建立 CMake 工程和基础模块边界，先划分网络、RTSP、RTP、媒体解码和显示模块。

[ ] 实现 Socket 抽象和可靠的 TCP 字节流读取，覆盖半包、粘包、超时、取消和资源释放。

[ ] 实现 RTSP 请求构造、响应解析和 CSeq 匹配，支持 OPTIONS、DESCRIBE、SETUP、PLAY、TEARDOWN。

[ ] 实现 SDP 解析，提取视频轨道、H.264 参数、时钟频率和控制 URL。

[ ] 实现 RTSP 会话状态机、Session 管理、Basic/Digest 认证和 GET_PARAMETER/OPTIONS 保活。

[ ] 实现 TCP interleaved 帧解析，区分 RTSP 文本消息、RTP 数据和 RTCP 数据。

[ ] 实现 RTP 头解析、序号检测、时间戳处理、乱序缓冲和 H.264 单 NAL/STAP-A/FU-A 重组。

[ ] 将重组后的 H.264 数据交给 FFmpeg 解码，并使用 SDL2、OpenCV 或 Qt 显示视频。

[ ] 增加日志、指标、异常处理、断线重连、停止播放和有限队列，验证长时间运行稳定性。

[ ] 使用 Wireshark、VLC、FFmpeg 和构造的协议单元测试验证 RTSP、RTP、RTCP 的边界情况。

[ ] 在 TCP 版本稳定后增加 UDP RTP/RTCP 传输，比较延迟、丢包、CPU 和部署复杂度。

## Open questions
- 第一版视频显示选 SDL2、OpenCV 还是 Qt？建议先用 SDL2/OpenCV 快速验证，后续再换 Qt。
- 是否需要音频？建议第一版暂不支持，先集中完成视频链路。
- 目标平台是否只考虑 Windows？建议第一版只做 Windows，网络层稳定后再抽象 Linux 支持。
