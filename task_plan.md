# Task Plan: C++ RTSP 项目规划

## Goal
在 Windows 环境下规划并实现一个基于 C++ 的 RTSP 客户端项目，能够连接摄像头或 MediaMTX，完成 RTSP 会话建立、RTP 接收、H.264 解码、视频显示，并具备保活、断线重连和基础诊断能力。

## Phases
- [x] Phase 1: 明确项目目标、协议模式和技术选型
- [x] Phase 2: 建立项目规划文档
- [ ] Phase 3: 准备 C++、FFmpeg、MediaMTX 和测试环境
- [ ] Phase 4: 使用现成库跑通 RTSP/TCP 拉流
- [ ] Phase 5: 实现 RTSP 请求响应和会话管理
- [ ] Phase 6: 实现 TCP 交织 RTP/RTCP 解析
- [ ] Phase 7: 接入 FFmpeg 解码和视频显示
- [ ] Phase 8: 增加保活、重连、日志和错误处理
- [ ] Phase 9: 增加 UDP 传输模式并比较性能
- [ ] Phase 10: 测试、抓包分析和整理文档

## Key Questions
1. 第一版是否只支持单路 H.264 视频？
2. 第一版是否采用 RTSP over TCP，后续再扩展 UDP？
3. 视频显示使用 Qt，还是先使用 OpenCV/SDL 做最小验证？
4. 是否需要同时支持音频？

## Decisions Made
- 第一版语言：C++。
- 第一版传输：RTSP 控制消息使用 TCP，RTP/RTCP 采用 TCP interleaved。
- 第一版媒体范围：单路 H.264 视频，暂不优先实现音频。
- 协议学习范围：RTSP 1.0、SDP、RTP、RTCP、H.264 over RTP。
- 服务端/测试源：优先使用 MediaMTX，摄像头由 FFmpeg 或手机 RTSP 应用提供。
- 解码：使用 FFmpeg 库，不自行实现 H.264 解码器。
- UDP：作为第二阶段能力，用于低延迟和局域网场景。

## Errors Encountered
- 暂无。

## Status
**Currently in Phase 2** - 已创建项目目录和规划文档，尚未编写实现代码。
