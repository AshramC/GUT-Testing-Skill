# GUT Testing - Project Structure & Checklist

## 推荐项目结构

```
project/
├── addons/
│   └── gut/                    # GUT插件 (已安装)
├── scenes/
├── scripts/
├── test/
│   ├── unit/                   # 单元测试
│   │   ├── test_player.gd
│   │   ├── test_enemy.gd
│   │   └── test_inventory.gd
│   ├── integration/            # 集成测试
│   │   ├── test_combat_system.gd
│   │   └── test_level_loading.gd
│   └── resources/              # 测试专用资源
│       └── mock_data.tres
└── .gutconfig.json             # GUT配置文件
```

---

## 编写测试检查清单

### 创建测试文件时
- [ ] 文件名以 `test_` 开头
- [ ] 文件放在 `res://test/unit/` 或 `res://test/integration/`
- [ ] 脚本 `extends GutTest`

### 编写测试方法时
- [ ] 方法名以 `test_` 开头
- [ ] 方法名描述测试目的 (如 `test_player_dies_when_health_zero`)
- [ ] 每个测试至少有一个断言
- [ ] 测试相互独立，不依赖执行顺序

### 使用Double时
- [ ] 先 `load()` 脚本/场景，再 `double()`
- [ ] 内部类需先 `register_inner_classes()`
- [ ] Double自动释放，无需手动free

### 内存管理
- [ ] 使用 `autofree()` 或 `add_child_autofree()` 管理对象
- [ ] 不在 `before_all` 中使用 autofree 方法
- [ ] 需要时使用 `assert_no_new_orphans()` 检测泄漏

### 异步测试
- [ ] 使用 GUT 的 `wait_*` 方法而非原生 `await`
- [ ] `wait_for_signal` 始终设置超时时间
- [ ] 物理帧等待至少2帧

### 输入测试
- [ ] `InputSender` 定义在类级别
- [ ] `after_each` 中调用 `release_all()` 和 `clear()`
- [ ] 发送输入后使用 `await` 等待处理

---

## CLI快速命令

```bash
# 运行所有测试
godot --headless -s addons/gut/gut_cmdln.gd -gexit

# 运行特定文件
godot --headless -s addons/gut/gut_cmdln.gd -gtest=res://test/unit/test_player.gd -gexit

# 运行名称匹配的测试
godot --headless -s addons/gut/gut_cmdln.gd -gselect=player -gexit

# 运行特定测试方法
godot --headless -s addons/gut/gut_cmdln.gd -gunit_test_name=test_jump -gexit

# 详细日志
godot --headless -s addons/gut/gut_cmdln.gd -glog=3 -gexit

# 生成报告
godot --headless -s addons/gut/gut_cmdln.gd -gjunit_xml_file=results.xml -gexit
```

---

## 常见问题速查

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| `Nonexistent function 'new' in base 'Nil'` | double传入字符串路径 | 先load()再double() |
| `Invalid operands 'int' and 'Nil'` | 参数默认值变null | 使用param_defaults() |
| 信号断言失败 | 未调用watch_signals | 断言前先watch_signals(obj) |
| 测试超时无响应 | await无超时 | 使用wait_for_signal设超时 |
| 原生方法无法stub | SCRIPT_ONLY策略 | 使用INCLUDE_NATIVE策略 |
| 内部类double失败 | 未注册 | 先register_inner_classes() |

---

## 最小配置文件

`.gutconfig.json`:
```json
{
    "dirs": ["res://test/unit/", "res://test/integration/"],
    "include_subdirs": true,
    "should_exit": true,
    "log_level": 2
}
```
