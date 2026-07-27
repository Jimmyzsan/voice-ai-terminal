# 智能语音交互终端

一款以自然语音为主要操作方式、触屏为辅助方式的 Android 智能终端。用户可通过唤醒词调用不同 AI 助手、音频平台与内容应用，并使用连续对话完成菜单选择、播放控制、应用切换和退出操作。

> 当前状态：需求定义与方案设计阶段。

## 项目目标

本项目不是普通智能音箱，而是一个可扩展的语音交互平台：

- 通过固定唤醒词进入聆听状态；
- 理解自然语言和上下文指令；
- 调用豆包、ChatGPT 等 AI 助手；
- 打开并控制学习强国、汽水音乐、云听等 Android 应用；
- 通过语音播报菜单，支持按序号或名称选择；
- 对每次操作提供明确的声音或界面反馈；
- 通过插件式接口持续接入新的 AI 和内容服务。

## 典型交互

```text
用户：你好，小助手。
终端：我在，请说。
用户：打开学习强国。
终端：学习强国已打开。您可以选择：
      第一，新闻联播；
      第二，每日听新闻；
      第三，理论学习；
      第四，最近播放。
用户：选择第二个。
终端：正在播放每日听新闻。
用户：暂停。
终端：已暂停。
```

## 核心功能

### 语音唤醒与识别

- 支持固定唤醒词，例如“你好，小助手”
- 支持远场拾音、降噪、回声消除和打断播报
- 通过提示音、灯光或屏幕状态反馈“正在聆听”
- 支持在线语音识别，并为离线基础命令预留能力

### 自然语言与连续对话

- 同一意图可使用不同说法表达
- 记住当前应用、菜单层级和播放状态
- 理解“继续”“换一个”“返回”“退出”等上下文指令
- 对低置信度识别结果进行澄清，而不是直接误操作

### AI 助手

- 豆包
- ChatGPT
- 其他可通过 API、网页、深度链接或 Android 应用接入的助手

支持打开、退出、切换助手和重新开始对话。优先采用官方 API；应用自动化仅作为兼容方案。

### 音频与内容应用

计划支持：

- 学习强国
- 汽水音乐
- 云听
- 新闻、广播、有声书及其他音频平台

基础控制包括打开应用、搜索和播放、暂停/继续、上一首/下一首、音量、快进/后退、收藏、定时停止、返回、退出和跨应用切换。

### 语音菜单

- 播报当前可用栏目和操作
- 支持序号与栏目名称选择
- 支持“下一页”“重复一遍”“说慢一点”
- 支持“只播报前三个”“返回上一级”
- 菜单发生变化时同步更新屏幕和会话上下文

### 状态反馈

每个操作都必须返回清晰结果，例如：

- “正在打开 ChatGPT。”
- “音量已经调整到百分之五十。”
- “没有找到您说的节目，请重新选择。”
- “当前网络连接异常，请稍后再试。”

### 触屏辅助

屏幕可显示当前应用、播放内容、AI 对话、语音菜单、播放进度、音量、网络状态，以及返回、暂停和退出等常用按钮。语音始终是主要交互方式。

### 账户与隐私

- 支持应用账户登录、退出和用户切换
- 调用收藏、播放记录、订阅、推荐和历史对话
- 使用 Android Keystore 等安全机制加密保存令牌
- 提供实体麦克风静音键
- 摄像头隐私开关为可选硬件
- 明确展示麦克风、摄像头和网络使用状态

## 建议系统架构

```text
麦克风阵列
   │
唤醒词 / VAD / 降噪 / 回声消除
   │
语音识别 ASR
   │
对话编排器 ── 会话上下文与菜单状态
   │
意图识别与策略路由
   ├── AI 助手适配器
   ├── 音频平台适配器
   ├── Android 应用控制适配器
   └── 系统控制适配器
   │
执行结果与错误处理
   │
语音合成 TTS + 屏幕 UI + 灯光反馈
```

建议将每个外部服务实现为独立适配器，并提供统一接口：

```kotlin
interface ServiceAdapter {
    suspend fun open(context: SessionContext): ActionResult
    suspend fun execute(intent: UserIntent, context: SessionContext): ActionResult
    suspend fun getMenu(context: SessionContext): VoiceMenu?
    suspend fun close(context: SessionContext): ActionResult
}
```

接入优先级建议：

1. 官方 API 或 SDK
2. Android Intent / App Link / Deep Link
3. 授权后的网页版服务
4. Android 无障碍服务或自动化
5. 定制版合作应用

无障碍自动化容易受应用界面更新影响，也可能受到服务条款和权限政策限制，应逐个应用验证合规性与稳定性。

## 硬件建议

- Android 操作系统
- 高灵敏度远场麦克风阵列
- 硬件级降噪和回声消除
- 扬声器和可选音频输出接口
- 触摸显示屏
- Wi-Fi 与蓝牙
- 实体麦克风静音键
- 可选摄像头及物理隐私开关
- 长时间待机与自动唤醒能力

## 非功能要求

- 常用离线命令响应目标：小于 500 ms
- 在线命令首个反馈目标：小于 1.5 s
- 所有执行动作必须具备成功、失败或处理中反馈
- 网络中断后应保留暂停、音量和退出等本地控制
- 高风险操作（购买、支付、删除数据、发送内容）必须二次确认
- 记录匿名化诊断日志，不保存非必要的原始语音
- 支持适配器版本管理、灰度发布和失败回退

## 开发路线图

### 阶段 1：最小可行产品

- [ ] 唤醒词、ASR、TTS 和打断播报
- [ ] 统一意图模型与会话状态机
- [ ] 音量、暂停、继续、返回和退出
- [ ] 一个 AI 助手适配器
- [ ] 一个音频应用适配器
- [ ] 基础语音菜单和触屏界面
- [ ] 网络异常与识别失败反馈

### 阶段 2：多应用体验

- [ ] 豆包与 ChatGPT 切换
- [ ] 学习强国、汽水音乐和云听适配器
- [ ] 分页菜单、重复和语速控制
- [ ] 收藏、历史记录与定时停止
- [ ] 账户安全存储和多用户切换

### 阶段 3：产品化

- [ ] 离线基础命令
- [ ] 权限、隐私和合规审计
- [ ] 自动化回归测试和真机兼容矩阵
- [ ] 远程配置、适配器更新和诊断
- [ ] 无障碍及老年用户体验测试

详细验收标准见 [docs/acceptance-criteria.md](docs/acceptance-criteria.md)。

## 推荐项目结构

```text
voice-ai-terminal/
├── README.md
├── docs/
│   └── acceptance-criteria.md
├── app/                 # Android 主应用
├── core/
│   ├── conversation/    # 会话状态与上下文
│   ├── intent/          # 意图识别和路由
│   ├── speech/          # 唤醒、ASR、TTS
│   └── security/        # 凭据与隐私
├── adapters/
│   ├── ai/              # AI 助手适配器
│   ├── audio/           # 音频平台适配器
│   └── android/         # Intent、深度链接、无障碍
└── tests/
```

## 开始开发前需要确认

- 目标硬件型号、Android 版本和是否具备 GMS
- 首发地区及可用的语音识别/合成服务
- 各平台是否提供官方 API、SDK 或商业授权
- ChatGPT 与豆包采用 API、网页还是应用模式
- 首个 MVP 必须支持的两个应用
- 是否要求完全离线唤醒与基础控制
- 用户数据、语音日志和未成年人保护要求

## 贡献

项目目前处于需求拆解阶段。欢迎通过 Issue 提交应用接入需求、硬件适配信息和使用场景。正式开发后将补充构建、测试和代码贡献指南。

## 许可证

许可证尚未确定。在添加正式开源许可证之前，默认保留所有权利。


---

# Intelligent Voice Interaction Terminal

An Android-based intelligent terminal designed around natural voice interaction, with a touchscreen as a secondary control surface. Users can wake the device, talk continuously with different AI assistants, open audio and content applications, navigate spoken menus, and control playback without frequently using a phone or touching the screen.

> Current status: requirements definition and solution design.

## Project Vision

This project is more than a conventional smart speaker. It is intended to provide an extensible voice-first platform that can:

- wake up through a fixed phrase such as “Hello, Assistant”;
- understand natural language and context-dependent commands;
- connect to AI assistants such as Doubao and ChatGPT;
- open and control Android applications such as Xuexi Qiangguo, Soda Music, and Yunting;
- announce available menus and accept selections by number or name;
- provide clear spoken and visual feedback for every operation;
- support additional AI assistants and content services through modular adapters.

## Example Interaction

~~~text
User: Hello, Assistant.
Terminal: I’m listening.
User: Open Xuexi Qiangguo.
Terminal: Xuexi Qiangguo is open. You can choose:
          One, News Broadcast;
          Two, Daily News Audio;
          Three, Theory Learning;
          Four, Recently Played.
User: Choose the second option.
Terminal: Playing Daily News Audio.
User: Pause.
Terminal: Paused.
~~~

## Core Capabilities

### Voice Wake-Up and Continuous Conversation

- Fixed wake phrase with sound, light, or screen feedback
- Far-field microphones, noise reduction, echo cancellation, and barge-in
- Natural-language understanding with session context
- Clarification for ambiguous or low-confidence requests
- Online speech recognition with room for offline essential commands

### AI Assistants and Content Applications

Initial targets include Doubao, ChatGPT, Xuexi Qiangguo, Soda Music, Yunting, news, radio, audiobooks, and other services available through official APIs, SDKs, web applications, deep links, or Android applications.

Users can open, exit, and switch services by voice. Common playback controls include play, pause, resume, previous, next, volume adjustment, fast-forward, rewind, favorite, sleep timer, return, and exit.

### Spoken Menus and Feedback

The terminal announces available sections and actions. Users can select an option by its number or name and use commands such as “next page,” “repeat,” “speak more slowly,” “read only the first three,” and “go back.” Every operation must return a spoken success, failure, or processing message.

### Touchscreen, Accounts, and Privacy

The optional touchscreen displays the current application, playback content, AI conversation, spoken menu, progress, volume, network status, and essential controls. Credentials should be encrypted with Android Keystore or an equivalent mechanism. The device should include a physical microphone mute control, clear activity indicators, and confirmation for purchases, payments, deletion, or outbound communication.

## Proposed Architecture

~~~text
Microphone Array
   │
Wake Word / VAD / Noise Reduction / Echo Cancellation
   │
Automatic Speech Recognition
   │
Conversation Orchestrator ── Session Context and Menu State
   │
Intent Recognition and Policy Router
   ├── AI Assistant Adapters
   ├── Audio Platform Adapters
   ├── Android Application Control Adapters
   └── System Control Adapter
   │
Execution Results and Error Handling
   │
Text-to-Speech + Screen UI + Light Feedback
~~~

Recommended integration priority:

1. Official API or SDK
2. Android Intent, App Link, or deep link
3. Authorized web service
4. Android Accessibility Service or UI automation
5. Custom partner application

## Suggested Hardware

- Android operating system
- Far-field microphone array
- Noise reduction and acoustic echo cancellation
- Speaker and optional audio output
- Touch display
- Wi-Fi and Bluetooth
- Physical microphone mute switch
- Optional camera with a physical privacy switch
- Long standby time and automatic wake-up

## Development Roadmap

### Phase 1 — Minimum Viable Product

- [ ] Wake word, ASR, TTS, and barge-in
- [ ] Unified intent model and session state machine
- [ ] Essential playback and navigation controls
- [ ] One AI assistant adapter and one audio application adapter
- [ ] Basic spoken menu and touchscreen interface
- [ ] Network and recognition error handling

### Phase 2 — Multi-Application Experience

- [ ] Switching between Doubao and ChatGPT
- [ ] Xuexi Qiangguo, Soda Music, and Yunting adapters
- [ ] Menu pagination, repetition, and speech-rate controls
- [ ] Secure accounts, favorites, history, and sleep timer

### Phase 3 — Production Readiness

- [ ] Offline essential commands
- [ ] Privacy, permission, and compliance review
- [ ] Automated regression tests and device compatibility matrix
- [ ] Remote configuration, adapter updates, and diagnostics
- [ ] Accessibility and older-adult usability testing

See [docs/acceptance-criteria.md](docs/acceptance-criteria.md) for detailed acceptance criteria.

## License

No open-source license has been selected yet. All rights are reserved until a license is added.
