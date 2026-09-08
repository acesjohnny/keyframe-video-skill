# 关键帧视频制作 — 一个 Agent Skill

一个 [Agent Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills)，用于制作多镜头 AI 短视频，其中每个镜头都从一张生成的关键帧图片开始。

本工作流适用于任何关键帧驱动的视频模型。它是基于 Agnes 视频 API（`agnes-video-2.5-flash`）编写并验证的。

[English](README.md) · **中文**

## 它真正解决的问题

价值不在「怎么调一个视频 API」，而在调用周围的纪律：

- **四道审批闸**——分镜、图片提示词、出图、托管——让用户在**花钱之前**批准方案，而不是拿到一个他并不想要的成品再来评价。
- **逐字复用的角色设定**，贯穿每一条镜头提示词。这才是真正维系角色连续性的东西。
- **诚实的验证。** Agent 听不见音频。这个 skill 教你如何拿到「确实有语音」的真实证据（RMS 包络的离散爆发、以及那些时间点上口型的变化），同时绝不让这种证据冒充「语种和台词正确」的结论——那个判定始终属于人。
- **真正浪费过时间的故障模式**，都写下来了：下载路径从错误的嵌套层级读取素材 URL、字幕被烧进画面、对白比正常低 35 dB、第二轮静默覆盖第一轮、以及一个残留的 git 锁挡住之后所有写操作。

## 安装

把 skill 复制到你的 agent 的 skills 目录，例如：

```sh
mkdir -p ~/.claude/skills/keyframe-video-production
cp SKILL.md ~/.claude/skills/keyframe-video-production/
```

中文版则复制 `SKILL.zh.md`。之后向你的 agent 提出「用某个场景做个短视频 demo」，它应该就能识别到这个 skill。

## 来历

从两次真实的生产运行中提炼。第二次验证了 `agnes-video-2.5-flash` 能直接从关键帧生成可听懂的中文普通话语音并带匹配口型，无需外挂 TTS——该结论由人工收听确认，不是自动分析得出。

## 许可

MIT，见 [LICENSE](LICENSE)。
