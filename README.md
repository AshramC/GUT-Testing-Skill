# GUT Testing Skill for LLM

<p align="center">
  <img src="https://img.shields.io/badge/Godot-4.x-blue?logo=godot-engine" alt="Godot 4.x">
  <img src="https://img.shields.io/badge/GUT-9.x-green" alt="GUT 9.x">
  <img src="https://img.shields.io/badge/License-MIT-yellow" alt="MIT License">
</p>

A structured skill document that enables LLMs (Large Language Models) to write standardized unit tests for Godot 4 projects using the [GUT (Godot Unit Testing)](https://github.com/bitwes/Gut) framework.

一套结构化的技能文档，使 LLM（大语言模型）能够使用 [GUT](https://github.com/bitwes/Gut) 框架为 Godot 4 项目编写规范的单元测试代码。

---

## ✨ Features | 特性

- **Godot 4 Focused** - 专注于 Godot 4 版本
- **CLI-First** - 面向命令行测试运行，适合 CI/CD 集成
- **Comprehensive Coverage** - 涵盖 Doubles、Stubs、Spies、异步测试、参数化测试等核心功能
- **Ready-to-Use Templates** - 提供 10 个即用测试模板
- **Quick Reference** - 包含断言速查表和常见问题解决方案

---

## 📁 Structure | 文件结构

```
gut-testing-skill/
├── SKILL.md        # 核心技能文档 - 完整的GUT框架使用指南
├── TEMPLATES.md    # 测试模板集 - 10个常用场景的测试模板
└── CHECKLIST.md    # 检查清单 - 项目结构、CLI命令、问题速查
```

### SKILL.md
核心参考文档，包含：
- 测试脚本结构与生命周期
- 完整的断言方法速查
- Doubles/Stubs/Spies 详细用法
- 异步测试与等待方法
- 参数化测试
- 内存管理与泄漏检测
- 输入模拟 (InputSender)
- CLI 命令与配置文件

### TEMPLATES.md
10 个实用测试模板：
1. 基础单元测试
2. 带 Double 的依赖隔离测试
3. 信号测试
4. 异步测试
5. 参数化测试
6. 输入测试
7. 内部类分组测试
8. Partial Double 测试
9. 场景集成测试
10. 内存泄漏测试

### CHECKLIST.md
快速参考：
- 推荐项目结构
- 编写测试检查清单
- CLI 快速命令
- 常见问题速查表

---

## 🚀 Usage | 使用方法

### For Claude Desktop / Claude Code

将此技能文件夹放入 Claude 的 skills 目录：

```bash
# 克隆仓库
git clone https://github.com/AshramC/GUT-Testing-Skill.git

# 复制到 skills 目录
cp -r gut-testing-skill /path/to/your/skills/user/
```

### For Other LLM Tools

将文档内容作为系统提示或知识库的一部分注入到 LLM 上下文中。

### Direct Reference

在与 LLM 对话时直接引用：

```
请参考 gut-testing-skill 为我的 Player 类编写单元测试
```

---

## 💡 Example | 示例

**Prompt:**
```
为以下 Player 类编写 GUT 单元测试：

class_name Player extends CharacterBody2D

var health: int = 100
signal died

func take_damage(amount: int):
    health -= amount
    if health <= 0:
        died.emit()
```

**LLM Output (guided by this skill):**
```gdscript
extends GutTest

var _player: Player

func before_each():
    _player = add_child_autofree(Player.new())
    watch_signals(_player)

func test_initial_health_is_100():
    assert_eq(_player.health, 100)

func test_take_damage_reduces_health():
    _player.take_damage(30)
    assert_eq(_player.health, 70)

func test_death_emits_died_signal():
    _player.take_damage(100)
    assert_signal_emitted(_player, "died")

func test_overkill_still_emits_died():
    _player.take_damage(150)
    assert_signal_emitted(_player, "died")
    assert_lte(_player.health, 0)
```

---

## 🔧 CLI Quick Start | CLI 快速开始

```bash
# 运行所有测试
godot --headless -s addons/gut/gut_cmdln.gd -gexit

# 运行特定文件
godot --headless -s addons/gut/gut_cmdln.gd -gtest=res://test/unit/test_player.gd -gexit

# 生成 JUnit XML 报告 (CI/CD)
godot --headless -s addons/gut/gut_cmdln.gd -gjunit_xml_file=results.xml -gexit
```

---

## 📋 Requirements | 依赖

- Godot 4.x
- GUT 9.x (Godot Unit Testing addon)

---

## 🤝 Contributing | 贡献

欢迎贡献！请通过以下方式参与：

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/amazing-feature`)
3. 提交更改 (`git commit -m 'Add some amazing feature'`)
4. 推送到分支 (`git push origin feature/amazing-feature`)
5. 开启 Pull Request

### 贡献方向
- 补充更多测试模板
- 添加特定场景的最佳实践
- 修正文档错误
- 翻译为其他语言

---

## 📄 License | 许可证

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

本项目采用 MIT 许可证 - 详情请参阅 [LICENSE](LICENSE) 文件。

---

## 🔗 Related Links | 相关链接

- [GUT - Godot Unit Testing](https://github.com/bitwes/Gut)
- [GUT Documentation](https://gut.readthedocs.io/)
- [Godot Engine](https://godotengine.org/)

---

## ⭐ Acknowledgments | 致谢

- [bitwes/Gut](https://github.com/bitwes/Gut) - 优秀的 Godot 单元测试框架
- 本 Skill 文档基于 GUT 官方文档整理优化

---

<p align="center">
  <i>If this skill helps you write better tests, please consider giving it a ⭐!</i>
</p>
