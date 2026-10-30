# wwise-unreal-voice-mod

一个用于制作 Unreal Engine + Wwise 游戏语音替换 Mod 的 Codex Skill。

## 功能

- Wwise Event / Media 映射
- WEM 音频准备、编码与解码校验
- 定长媒体槽替换与 `JUNK` 补齐
- Unreal IoStore `.pak` / `.utoc` / `.ucas` 打包
- 安装前备份、哈希校验与安全回滚
- 游戏内实机验证
- 发布包、来源声明和 SHA-256 校验

## 安装

将以下文件复制到个人 Skill 目录：

```text
~/.cursor/skills/wwise-unreal-voice-mod/
├── SKILL.md
└── release-template.md
```

也可以让 Codex 直接读取本仓库中的 `SKILL.md`。

## 使用

在 Codex 中说明：

> 按 `wwise-unreal-voice-mod` 流程制作这个游戏的语音 Mod。

适用于 Unreal Engine + Wwise 项目。它不是一键生成器，具体步骤仍取决于游戏版本、容器格式、语音库结构和素材使用权限。

## 重要边界

- 离线容器校验不等于游戏内播放成功，必须实际测试。
- 修改游戏文件前应先备份并记录 SHA-256。
- 不要把凭据、加密密钥、私有源文件或无关 Mod 放进公开仓库。
- 游戏、角色、配音和第三方素材的权利归各自权利人所有。
- 发布前请遵守游戏 EULA、素材来源条款和平台规则。

## 文件

- [`SKILL.md`](SKILL.md)：完整工作流程
- [`release-template.md`](release-template.md)：Mod 发布说明模板
