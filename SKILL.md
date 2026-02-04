# GUT (Godot Unit Testing) - Godot 4 Testing Skill

## Overview
GUT是Godot引擎的单元测试框架。本技能指导如何为Godot 4项目编写规范的单元测试代码并通过CLI运行。

## 约定
- 测试文件必须以 `test_` 开头，扩展名 `.gd`
- 测试方法必须以 `test_` 开头
- 测试目录推荐: `res://test/unit/` 和 `res://test/integration/`
- 所有测试脚本必须 `extends GutTest`

---

## 1. 测试脚本基础结构

### 最小测试脚本
```gdscript
extends GutTest

func test_example():
    assert_eq(1 + 1, 2, "1+1 should equal 2")
```

### 完整生命周期
```gdscript
extends GutTest

func before_all():
    # 所有测试开始前执行一次
    pass

func before_each():
    # 每个测试方法执行前调用
    pass

func after_each():
    # 每个测试方法执行后调用
    pass

func after_all():
    # 所有测试结束后执行一次
    pass

func test_something():
    assert_true(true)
```

### 内部测试类（分组测试）
```gdscript
extends GutTest

# 外层可以没有测试方法

class TestFeatureA:
    extends GutTest
    
    func before_each():
        pass  # 只对本类测试生效
    
    func test_feature_a_works():
        assert_true(true)

class TestFeatureB:
    extends GutTest
    
    func test_feature_b_works():
        assert_true(true)
```

---

## 2. 断言方法速查

### 基础断言
```gdscript
assert_eq(got, expected, "optional message")      # 相等
assert_ne(got, not_expected)                      # 不相等
assert_true(value)                                # 为真
assert_false(value)                               # 为假
assert_null(value)                                # 为null
assert_not_null(value)                            # 不为null
```

### 数值断言
```gdscript
assert_gt(value, threshold)                       # 大于
assert_lt(value, threshold)                       # 小于
assert_gte(value, threshold)                      # 大于等于
assert_lte(value, threshold)                      # 小于等于
assert_between(value, low, high)                  # 在范围内
assert_almost_eq(got, expected, tolerance)        # 近似相等(浮点数)
assert_almost_ne(got, expected, tolerance)        # 近似不等
```

### 字符串断言
```gdscript
assert_string_contains(text, search)              # 包含子串
assert_string_starts_with(text, prefix)           # 以...开头
assert_string_ends_with(text, suffix)             # 以...结尾
```

### 数组/字典断言
```gdscript
assert_has(array_or_dict, item_or_key)            # 包含元素/键
assert_does_not_have(array_or_dict, item_or_key)  # 不包含
assert_eq_deep(got, expected)                     # 深度比较(显示差异详情)
assert_ne_deep(got, expected)                     # 深度不等
assert_same(a, b)                                 # 引用相同
assert_not_same(a, b)                             # 引用不同
```

### 信号断言
```gdscript
watch_signals(obj)                                # 开始监视信号(必须先调用)
assert_signal_emitted(obj, "signal_name")         # 信号已发出
assert_signal_not_emitted(obj, "signal_name")     # 信号未发出
assert_signal_emitted_with_parameters(obj, "signal_name", [params])
assert_signal_emit_count(obj, "signal_name", count)
```

### 文件断言
```gdscript
assert_file_exists(path)
assert_file_does_not_exist(path)
assert_file_empty(path)
assert_file_not_empty(path)
```

### 错误断言 (Godot 4.5+)
```gdscript
# 断言引擎错误发生
assert_engine_error()                             # 任意引擎错误
assert_engine_error("expected text")              # 包含特定文本的错误

# 断言 push_error 发生
assert_push_error()
assert_push_error("expected text")
```

### 测试控制
```gdscript
pass_test("reason")                               # 手动通过测试
fail_test("reason")                               # 手动失败测试
pending("reason")                                 # 标记为待完成
```

---

## 3. Doubles (测试替身)

### 基本概念
- **Double**: 完全替身，所有方法默认不执行任何操作，返回null
- **Partial Double**: 部分替身，保留原有功能，可选择性stub

### 创建Double
```gdscript
# 脚本替身
var MyClass = load("res://my_class.gd")
var dbl = double(MyClass).new()

# 场景替身
var MyScene = load("res://my_scene.tscn")
var dbl_scene = double(MyScene).instantiate()

# 部分替身 (保留原有功能)
var partial = partial_double(MyClass).new()
```

### 内部类替身
```gdscript
var SomeScript = load("res://some_script.gd")

func before_all():
    register_inner_classes(SomeScript)  # 必须先注册

func test_inner_class():
    var dbl = double(SomeScript.InnerClass).new()
```

### 原生类型替身
```gdscript
var dbl_node = double(Node2D).new()
var dbl_raycast = partial_double(RayCast2D).new()
```

---

## 4. Stubbing (桩)

### 基本stub
```gdscript
var dbl = double(MyClass).new()

# 返回特定值
stub(dbl.some_method).to_return(42)

# 不执行任何操作
stub(dbl.some_method).to_do_nothing()

# 调用原实现
stub(dbl.some_method).to_call_super()

# 调用自定义Callable
stub(dbl.some_method).to_call(func(arg): return arg * 2)
```

### 条件stub
```gdscript
# 当传入特定参数时返回特定值
stub(dbl.some_method).to_return("world").when_passed("hello")
stub(dbl.some_method).to_return("bar").when_passed("foo", 123)

# 使用bind语法
stub(dbl.some_method.bind("hello")).to_return("world")
```

### 脚本级别stub (影响所有实例)
```gdscript
var MyClass = load("res://my_class.gd")
stub(MyClass, "some_method").to_return(100)

var inst1 = double(MyClass).new()
var inst2 = double(MyClass).new()
# inst1和inst2的some_method都返回100
```

### 参数默认值stub
```gdscript
# 解决 "Invalid operands 'int' and 'Nil'" 错误
stub(dbl, "method_with_defaults").param_defaults([1, "default"])
```

---

## 5. Spies (间谍)

### 断言方法被调用
```gdscript
var dbl = double(MyClass).new()
dbl.some_method(1, 2, 3)

assert_called(dbl, "some_method")                      # 被调用过
assert_called(dbl, "some_method", [1, 2, 3])          # 以特定参数调用
assert_not_called(dbl, "other_method")                 # 未被调用
assert_call_count(dbl, "some_method", 1)               # 调用次数
```

### 获取调用参数
```gdscript
var params = get_call_parameters(dbl, "some_method")       # 最后一次调用参数
var params = get_call_parameters(dbl, "some_method", 0)    # 第一次调用参数
var count = get_call_count(dbl, "some_method")             # 调用次数
```

---

## 6. 异步测试 (Await)

### 等待时间
```gdscript
func test_async():
    # 等待2秒
    await wait_seconds(2.0)
    
    # 等待并附带日志消息
    await wait_seconds(1.0, "waiting for animation")
```

### 等待帧
```gdscript
func test_frames():
    # 等待物理帧 (推荐至少2帧)
    await wait_physics_frames(5)
    
    # 等待处理帧
    await wait_idle_frames(10)
    # 或
    await wait_process_frames(10)
```

### 等待信号
```gdscript
func test_signal():
    var obj = MyClass.new()
    
    # 等待信号或超时(5秒)，返回bool表示是否收到信号
    var received = await wait_for_signal(obj.my_signal, 5)
    assert_true(received, "Signal should be emitted")
    
    # 直接在断言中使用
    assert_true(await wait_for_signal(obj.done, 3), "Should complete in 3s")
```

### 等待条件
```gdscript
func test_condition():
    var obj = MyClass.new()
    
    # 等待条件为真，最多10秒，每0.5秒检查一次
    await wait_until(func(): return obj.is_ready, 10, 0.5)
    
    # 等待条件变为假
    await wait_while(func(): return obj.is_loading, 10)
```

### 调试暂停
```gdscript
func test_debug():
    # 在teardown前暂停，等待手动点击继续
    pause_before_teardown()
```

---

## 7. 参数化测试

### 基本用法
```gdscript
# 参数数组，每个元素会执行一次测试
var add_params = [[1, 2, 3], [10, 20, 30], [-1, 1, 0]]

func test_add(p=use_parameters(add_params)):
    var result = p[0] + p[1]
    assert_eq(result, p[2])
```

### 命名参数
```gdscript
var params = ParameterFactory.named_parameters(
    ["a", "b", "expected"],  # 参数名
    [
        [1, 2, 3],
        [10, 20, 30],
        [-1, 1, 0]
    ]
)

func test_add_named(p=use_parameters(params)):
    assert_eq(p.a + p.b, p.expected)
```

---

## 8. 内存管理

### 自动释放
```gdscript
func test_with_autofree():
    # 测试结束后自动调用free()
    var node = autofree(Node.new())
    
    # 测试结束后自动调用queue_free()
    var node2 = autoqfree(Node.new())
    
    # 添加到场景树并自动free
    var node3 = add_child_autofree(Node2D.new())
    
    # 添加到场景树并自动queue_free
    var node4 = add_child_autoqfree(Sprite2D.new())
```

### 断言无内存泄漏
```gdscript
func test_no_leaks():
    var obj = MyClass.new()
    # ... 使用obj ...
    obj.free()
    
    assert_no_new_orphans()

func test_queue_free_leak():
    var obj = MyClass.new()
    obj.queue_free()
    # queue_free需要等待一帧
    await wait_seconds(0.1)
    assert_no_new_orphans()
```

### 注意事项
- Double和Partial Double会自动释放，无需手动处理
- 不要在`before_all`中使用autofree方法
- `autofree`的对象不在场景树中，仍会被`assert_no_new_orphans`检测为orphan

---

## 9. 模拟处理循环 (Simulate)

```gdscript
func test_process():
    var obj = add_child_autofree(MyNode.new())
    
    # 调用20次_process和_physics_process，delta为0.1
    gut.simulate(obj, 20, 0.1)
    
    # 检查是否在处理中 (第4个参数)
    gut.simulate(obj, 20, 0.1, true)
```

---

## 10. 输入模拟 (InputSender)

### 基本设置
```gdscript
extends GutTest

var _sender = InputSender.new(Input)

func after_each():
    _sender.release_all()  # 释放所有按键
    _sender.clear()        # 清除状态
```

### 发送输入
```gdscript
func test_jump():
    var player = add_child_autofree(Player.new())
    
    # 按下动作
    _sender.action_down("jump")
    await wait_frames(2)
    
    assert_true(player.is_jumping)

func test_key_sequence():
    _sender.key_down("W").hold_for(0.5)\
        .key_down("Space")\
        .wait(0.1)
    await _sender.idle
```

### 链式输入
```gdscript
func test_combo():
    _sender.action_down("down").hold_for("2f")\
        .action_down("forward").hold_for("2f")\
        .action_down("punch")
    await _sender.idle
    
    assert_true(player.did_special_move)
```

---

## 11. CLI运行测试

### 基本命令
```bash
# 基本运行
godot --headless -s addons/gut/gut_cmdln.gd

# 带调试信息运行
godot -d --headless -s addons/gut/gut_cmdln.gd

# 指定项目路径
godot --headless -s addons/gut/gut_cmdln.gd --path /path/to/project
```

### 常用选项
```bash
# 指定测试目录
-gdir=res://test/unit,res://test/integration

# 包含子目录
-ginclude_subdirs

# 运行特定测试文件
-gtest=res://test/unit/test_player.gd

# 运行文件名包含特定字符串的测试
-gselect=player

# 运行特定内部类
-ginner_class=TestMovement

# 运行特定测试方法 (名称包含)
-gunit_test_name=jump

# 测试完成后退出
-gexit

# 仅成功时退出
-gexit_on_success

# 日志级别 0-3
-glog=2

# 输出JUnit XML报告
-gjunit_xml_file=res://test_results.xml
```

### 完整示例
```bash
# 运行所有测试并退出
godot --headless -s addons/gut/gut_cmdln.gd -gexit

# 运行特定文件的特定测试
godot --headless -s addons/gut/gut_cmdln.gd \
    -gtest=res://test/unit/test_player.gd \
    -gunit_test_name=test_jump \
    -gexit

# 运行unit目录下所有测试，生成报告
godot --headless -s addons/gut/gut_cmdln.gd \
    -gdir=res://test/unit \
    -ginclude_subdirs \
    -gjunit_xml_file=res://reports/results.xml \
    -gexit
```

---

## 12. 配置文件 (.gutconfig.json)

在项目根目录创建 `res://.gutconfig.json`:

```json
{
    "dirs": [
        "res://test/unit/",
        "res://test/integration/"
    ],
    "include_subdirs": true,
    "prefix": "test_",
    "suffix": ".gd",
    "should_exit": true,
    "should_exit_on_success": false,
    "log_level": 2,
    "ignore_pause": true,
    "double_strategy": "SCRIPT_ONLY",
    "junit_xml_file": "",
    "junit_xml_timestamp": false,
    "pre_run_script": "",
    "post_run_script": ""
}
```

### 使用配置文件
```bash
# 默认使用 res://.gutconfig.json
godot --headless -s addons/gut/gut_cmdln.gd

# 指定配置文件
godot --headless -s addons/gut/gut_cmdln.gd -gconfig=res://custom.gutconfig.json

# 不使用配置文件
godot --headless -s addons/gut/gut_cmdln.gd -gconfig=
```

---

## 13. 常见模式与最佳实践

### 实例化被测类

#### 判断脚本是否有 class_name
查看被测脚本开头是否有 `class_name XxxClass` 声明，这决定了实例化方式。

#### 有 class_name 的脚本（推荐方式）
```gdscript
# 假设 player.gd 有 class_name Player

# ✓ 直接使用全局类名
var player = Player.new()
var player2 = add_child_autofree(Player.new())

# ❌ 不要这样做！会报错: Nonexistent function 'new' in base 'GDScript'
var Script = load("res://player.gd")
var player = Script.new()
```

#### 无 class_name 的脚本
```gdscript
# 假设 helper.gd 没有 class_name

# ✓ 使用 load().new()
var Helper = load("res://helper.gd")
var helper = Helper.new()
```

#### 创建 Double（无论有无 class_name）
```gdscript
# Double 始终需要 load() + double()，无论是否有 class_name
var Script = load("res://player.gd")
var dbl = double(Script).new()

# 部分替身同理
var partial = partial_double(Script).new()
```

### 测试被测类的基本模式
```gdscript
extends GutTest

var _player: Player  # 假设 Player 有 class_name

func before_each():
    _player = add_child_autofree(Player.new())

func test_initial_health():
    assert_eq(_player.health, 100)

func test_take_damage():
    _player.take_damage(30)
    assert_eq(_player.health, 70)
```

### 使用Double隔离依赖
```gdscript
extends GutTest

var _player: Player
var _weapon_dbl

func before_each():
    var Weapon = load("res://weapon.gd")
    _weapon_dbl = double(Weapon).new()
    stub(_weapon_dbl.get_damage).to_return(50)
    
    _player = add_child_autofree(Player.new())
    _player.weapon = _weapon_dbl

func test_attack_uses_weapon_damage():
    var enemy = add_child_autofree(Enemy.new())
    _player.attack(enemy)
    
    assert_called(_weapon_dbl, "get_damage")
    assert_eq(enemy.health, 50)  # 100 - 50
```

### 测试信号
```gdscript
extends GutTest

func test_emits_died_signal():
    var player = add_child_autofree(Player.new())
    watch_signals(player)
    
    player.health = 0
    player.check_death()
    
    assert_signal_emitted(player, "died")
```

### 测试异步行为
```gdscript
extends GutTest

func test_async_loading():
    var loader = ResourceLoader.new()
    
    var loaded = await wait_for_signal(loader.finished, 5)
    assert_true(loaded, "Should finish loading within 5 seconds")
```

---

## 14. 错误处理与调试

### 静态方法处理
```gdscript
# 静态方法不能被double，需要忽略
func before_each():
    ignore_method_when_doubling(MyClass, "static_method")
```

### Double Strategy
```gdscript
# 默认: SCRIPT_ONLY - 只包含脚本定义的方法
# INCLUDE_NATIVE - 包含原生方法 (如set_position)

# 脚本级别设置
func before_all():
    set_double_strategy(DOUBLE_STRATEGY.INCLUDE_NATIVE)

# 单个double设置
var dbl = double(MyClass, DOUBLE_STRATEGY.INCLUDE_NATIVE).new()
```

### 常见错误解决

**"Nonexistent function 'new' in base 'GDScript'"**
```gdscript
# 原因: 对有 class_name 的脚本使用 load().new()
# 解决: 直接使用全局类名

# ❌ 错误写法 (假设文件有 class_name Player)
var Script = load("res://player.gd")
var obj = Script.new()

# ✓ 正确写法
var obj = Player.new()
```

**"Nonexistent function 'new' in base 'Nil'"**
```gdscript
# 原因: double() 传入字符串路径，或 load() 路径错误
# 解决: 确保传入已加载的 Script 资源

# ❌ 错误写法
var dbl = double("res://my_class.gd").new()

# ✓ 正确写法
var Script = load("res://my_class.gd")
var dbl = double(Script).new()
```

**"Invalid operands 'int' and 'Nil'"**
```gdscript
# 原因: 方法参数默认值在double中变为null
# 解决: stub参数默认值
stub(dbl, "method").param_defaults([1, "default"])
```

### 场景加载后脚本方法不存在

**症状**: 
```
Invalid call. Nonexistent function 'xxx' in base 'Node'.
```

**原因**: 脚本有编译错误导致未能附加到节点，节点退化为普通 Node

**排查方法**:
```gdscript
# 1. 先单独测试脚本是否能加载
func test_script_loads():
    var script = load("res://path/to/script.gd")
    assert_not_null(script, "Script should load without errors")

# 2. 测试脚本依赖的其他脚本
func test_dependencies_load():
    var dep1 = load("res://path/to/dependency1.gd")
    var dep2 = load("res://path/to/dependency2.gd")
    assert_not_null(dep1)
    assert_not_null(dep2)

# 3. 检查场景实例化后节点类型
func test_scene_scripts_attached():
    var scene = load("res://my_scene.tscn")
    var instance = add_child_autofree(scene.instantiate())
    await wait_idle_frames(1)
    
    var node = instance.get_node("SomeNode")
    # 如果脚本正确附加，应该能调用脚本方法
    assert_true(node.has_method("expected_method"), "Script should be attached")
```

**常见导致脚本加载失败的原因**:
- 脚本中引用的其他类有编译错误
- 循环依赖
- 语法错误（如使用了 `?:` 三元运算符）

---

## 15. GDScript 语法注意事项

在编写测试代码时，注意以下 GDScript 4.x 特有的语法规则：

### 三元运算符

GDScript **不支持** C 风格的 `?:` 三元运算符，必须使用 `if-else` 表达式：

```gdscript
# ❌ 错误写法 - 会导致编译失败
var result = condition ? value_if_true : value_if_false

# ✓ 正确写法
var result = value_if_true if condition else value_if_false

# 示例
var name = player.name if player != null else "Unknown"
var damage = base_damage * 2 if is_critical else base_damage
```

### 类型推断与 null

使用 `:=` 进行类型推断时，不能直接赋值 `null`：

```gdscript
# ❌ 错误写法 - 无法从 null 推断类型
var result := null

# ✓ 正确写法 - 显式声明类型
var result: Variant = null
var result: Dictionary = {}
var result: Array = []

# 或者不使用类型推断
var result = null
```

### Mock 类中的常见模式
```gdscript
# 在测试中创建 Mock 类时的正确写法
class MockComponent:
    extends RefCounted
    var collected := {}
    var restored: Variant = null  # 注意这里用 Variant 而非 :=

    func collect_state() -> Dictionary:
        return collected

    func restore_state(state: Dictionary) -> void:
        restored = state
```

---

## Quick Reference Card

```
测试结构:
  extends GutTest
  before_all/before_each/after_each/after_all
  func test_xxx():

实例化:
  有 class_name: ClassName.new()
  无 class_name: load("...").new()
  Double: double(load("...")).new()

断言:
  assert_eq/ne/true/false/null/not_null
  assert_gt/lt/gte/lte/between
  assert_has/does_not_have
  assert_called/not_called/call_count

Double:
  double(Script).new()
  partial_double(Script).new()
  double(Scene).instantiate()

Stub:
  stub(dbl.method).to_return(val)
  stub(dbl.method).to_call_super()
  stub(dbl.method).to_do_nothing()
  stub(dbl.method).when_passed(args)

Spy:
  assert_called(dbl, "method", [args])
  get_call_parameters(dbl, "method")

Await:
  await wait_seconds(n)
  await wait_for_signal(sig, timeout)
  await wait_physics_frames(n)

Memory:
  autofree(obj) / autoqfree(obj)
  add_child_autofree(obj)

CLI:
  godot --headless -s addons/gut/gut_cmdln.gd -gexit

GDScript 语法:
  三元: value_if_true if cond else value_if_false
  Null 类型: var x: Variant = null (不要用 := null)
```
