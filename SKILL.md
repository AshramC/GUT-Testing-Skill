# GUT (Godot Unit Testing) - Godot 4 测试技能

## 概览
GUT 是 Godot 的单元测试框架。本技能说明如何为 Godot 4 项目编写规范的单元测试，并通过 CLI 运行。

## 约定
1. 测试文件必须以 `test_` 开头，扩展名为 `.gd`。
2. 测试方法必须以 `test_` 开头。
3. 推荐目录：`res://test/unit/` 与 `res://test/integration/`。
4. 所有测试脚本必须 `extends GutTest`。

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

class TestFeatureA:
    extends GutTest

    func before_each():
        pass

    func test_feature_a_works():
        assert_true(true)

class TestFeatureB:
    extends GutTest

    func test_feature_b_works():
        assert_true(true)
```

---

## 2. 断言速查

### 基础断言
```gdscript
assert_eq(got, expected, "optional message")
assert_ne(got, not_expected)
assert_true(value)
assert_false(value)
assert_null(value)
assert_not_null(value)
```

### 数值断言
```gdscript
assert_gt(value, threshold)
assert_lt(value, threshold)
assert_gte(value, threshold)
assert_lte(value, threshold)
assert_between(value, low, high)
assert_almost_eq(got, expected, tolerance)
assert_almost_ne(got, expected, tolerance)
```

### 字符串断言
```gdscript
assert_string_contains(text, search)
assert_string_starts_with(text, prefix)
assert_string_ends_with(text, suffix)
```

### 数组/字典断言
```gdscript
assert_has(array_or_dict, item_or_key)
assert_does_not_have(array_or_dict, item_or_key)
assert_eq_deep(got, expected)
assert_ne_deep(got, expected)
assert_same(a, b)
assert_not_same(a, b)
```

### 信号断言
```gdscript
watch_signals(obj)
assert_signal_emitted(obj, "signal_name")
assert_signal_not_emitted(obj, "signal_name")
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

### 错误断言（Godot 4.5+）
```gdscript
assert_engine_error()
assert_engine_error("expected text")
assert_push_error()
assert_push_error("expected text")
```

---

## 3. Doubles（测试替身）

### 创建 Double
```gdscript
var MyClass = load("res://my_class.gd")
var dbl = double(MyClass).new()

var MyScene = load("res://my_scene.tscn")
var dbl_scene = double(MyScene).instantiate()

var partial = partial_double(MyClass).new()
```

### 内部类替身
```gdscript
var SomeScript = load("res://some_script.gd")

func before_all():
    register_inner_classes(SomeScript)

func test_inner_class():
    var dbl = double(SomeScript.InnerClass).new()
```

### 原生类替身
```gdscript
var dbl_node = double(Node2D).new()
var dbl_raycast = partial_double(RayCast2D).new()
```

### Double Strategy
```gdscript
# 默认: SCRIPT_ONLY - 只包含脚本定义的方法
# INCLUDE_NATIVE - 包含原生方法 (如 set_position)

# 脚本级别设置
func before_all():
    set_double_strategy(DOUBLE_STRATEGY.INCLUDE_NATIVE)

# 单个 double 设置
var dbl = double(MyClass, DOUBLE_STRATEGY.INCLUDE_NATIVE).new()
```

### 静态方法处理
```gdscript
# 静态方法不能被 double，需要忽略
func before_each():
    ignore_method_when_doubling(MyClass, "static_method")
```

---

## 4. Stubbing

### 基本 stub
```gdscript
var dbl = double(MyClass).new()

stub(dbl.some_method).to_return(42)
stub(dbl.some_method).to_do_nothing()
stub(dbl.some_method).to_call_super()
stub(dbl.some_method).to_call(func(arg): return arg * 2)
```

### 条件 stub
```gdscript
stub(dbl.some_method).to_return("world").when_passed("hello")
stub(dbl.some_method).to_return("bar").when_passed("foo", 123)

stub(dbl.some_method.bind("hello")).to_return("world")
```

### 脚本级别 stub
```gdscript
var MyClass = load("res://my_class.gd")
stub(MyClass, "some_method").to_return(100)

var inst1 = double(MyClass).new()
var inst2 = double(MyClass).new()
```

### 参数默认值 stub
```gdscript
stub(dbl, "method_with_defaults").param_defaults([1, "default"])
```

---

## 5. Spies（调用观察）
```gdscript
var dbl = double(MyClass).new()

dbl.some_method(1, 2, 3)

assert_called(dbl, "some_method")
assert_called(dbl, "some_method", [1, 2, 3])
assert_not_called(dbl, "other_method")
assert_call_count(dbl, "some_method", 1)

var params = get_call_parameters(dbl, "some_method")
var params0 = get_call_parameters(dbl, "some_method", 0)
var count = get_call_count(dbl, "some_method")
```

---

## 6. 异步测试（Await）

### 等待时间
```gdscript
func test_async():
    await wait_seconds(2.0)
    await wait_seconds(1.0, "waiting for animation")
```

### 等待帧
```gdscript
func test_frames():
    await wait_physics_frames(5)
    await wait_idle_frames(10)
    await wait_process_frames(10)
```

### 等待信号
```gdscript
func test_signal():
    var obj = MyClass.new()
    var received = await wait_for_signal(obj.my_signal, 5)
    assert_true(received, "Signal should be emitted")
```

### 等待条件
```gdscript
func test_condition():
    var obj = MyClass.new()
    await wait_until(func(): return obj.is_ready, 10, 0.5)
    await wait_while(func(): return obj.is_loading, 10)
```

### 调试暂停
```gdscript
func test_debug():
    pause_before_teardown()
```

---

## 7. 参数化测试

### 基本
```gdscript
var add_params = [[1, 2, 3], [10, 20, 30], [-1, 1, 0]]

func test_add(p=use_parameters(add_params)):
    var result = p[0] + p[1]
    assert_eq(result, p[2])
```

### 命名参数
```gdscript
var params = ParameterFactory.named_parameters(
    ["a", "b", "expected"],
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
```gdscript
func test_with_autofree():
    var node = autofree(Node.new())
    var node2 = autoqfree(Node.new())
    var node3 = add_child_autofree(Node2D.new())
    var node4 = add_child_autoqfree(Sprite2D.new())
```

```gdscript
func test_no_leaks():
    var obj = MyClass.new()
    obj.free()
    assert_no_new_orphans()
```

注意事项：
1. Double 与 Partial Double 会自动释放。
2. 不要在 `before_all` 里使用 `autofree`。
3. `autofree` 对象不在场景树中，但仍会被 `assert_no_new_orphans` 检测。

---

## 9. 模拟处理循环
```gdscript
func test_process():
    var obj = add_child_autofree(MyNode.new())
    gut.simulate(obj, 20, 0.1)
    gut.simulate(obj, 20, 0.1, true)
```

---

## 10. 输入模拟（InputSender）
```gdscript
extends GutTest

var _sender = InputSender.new(Input)

func after_each():
    _sender.release_all()
    _sender.clear()
```

```gdscript
func test_jump():
    var player = add_child_autofree(Player.new())
    _sender.action_down("jump")
    await wait_frames(2)
    assert_true(player.is_jumping)
```

---

## 11. CLI 运行测试

```bash
# 基本运行
godot --headless -s addons/gut/gut_cmdln.gd

# 带调试信息
godot -d --headless -s addons/gut/gut_cmdln.gd

# 指定项目路径
godot --headless -s addons/gut/gut_cmdln.gd --path /path/to/project
```

常用选项：
```bash
-gdir=res://test/unit,res://test/integration
-ginclude_subdirs
-gtest=res://test/unit/test_player.gd
-gselect=player
-ginner_class=TestMovement
-gunit_test_name=jump
-gexit
-gexit_on_success
-glog=2
-gjunit_xml_file=res://test_results.xml
```

---

## 12. 配置文件 (.gutconfig.json)

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

---

## 13. 常见模式与最佳实践

### 实例化被测类
判断脚本是否有 `class_name`：
```gdscript
# 有 class_name
var player = Player.new()

# 无 class_name
var Helper = load("res://helper.gd")
var helper = Helper.new()
```

### 创建 Double（无论有无 class_name）
```gdscript
# Double 始终需要 load() + double()
var Script = load("res://player.gd")
var dbl = double(Script).new()
```

### 使用 Double 隔离依赖
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
    assert_eq(enemy.health, 50)
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

---

## 14. Godot 4 常见坑（务必规避）

### 1. `load().new()` 与 `class_name`
- 对有 `class_name` 的脚本使用 `load().new()` 可能报：`Nonexistent function 'new' in base 'GDScript'`。
- 推荐 `ClassName.new()` 或 `preload("res://...").new()`。

### 2. 类型标注导致解析失败
- 错误示例：`var foo: SomeScript` 或 `func bar(x: SomeScript)`。
- 若 `SomeScript` 未被注册或未 `preload`，会出现 `Identifier not declared`。
- 解决：移除类型标注，或用 `const SomeScript = preload("...")` 后再标注。

### 3. `.tscn` 编码问题
- `.tscn` 带 BOM 会导致 `Parse Error: Expected '['`。
- 建议统一为 **UTF-8 无 BOM**。

### 4. `@onready` 未执行
- Scene instantiate 后立刻访问 `@onready` 变量可能为 `null`。
- 测试中建议 `await wait_idle_frames(1)` 再断言。

### 5. Double 相关错误

| 错误信息 | 原因 | 解决方案 |
|----------|------|----------|
| `Nonexistent function 'new' in base 'GDScript'` | 对有 class_name 的脚本用 load().new() | 直接用 `ClassName.new()` |
| `Nonexistent function 'new' in base 'Nil'` | double 传入字符串或 load 路径错误 | 先 `load()` 再 `double()` |
| `Invalid operands 'int' and 'Nil'` | 参数默认值变 null | 使用 `param_defaults()` |
| `Nonexistent function 'xxx' in base 'Node'` | 脚本编译失败未附加 | 检查脚本及依赖是否有编译错误 |

---

## 15. 集成测试入口推荐

1. 运行时脚本优先通过 `PackedScene` 实例化，而不是直接脚本对象。
2. 最小模板：
```gdscript
func test_scene_integration():
    var scene = load("res://path/to/RuntimeScene.tscn")
    assert_true(scene is PackedScene)
    var inst = scene.instantiate()
    add_child_autofree(inst)
    await wait_idle_frames(1)
    
    # 现在可以安全访问 @onready 变量和子节点
    var runtime = inst.get_node("Runtime")
    assert_true(runtime.has_method("start"))
```

---

## 16. 快速诊断模板

### 脚本/场景是否能加载
```gdscript
func test_script_loads():
    var script = load("res://path/to/script.gd")
    assert_not_null(script, "Script should load without errors")
```

### 依赖是否可加载
```gdscript
func test_dependencies_load():
    var dep1 = load("res://path/to/dep1.gd")
    var dep2 = load("res://path/to/dep2.gd")
    assert_not_null(dep1)
    assert_not_null(dep2)
```

### 场景脚本是否正确附加
```gdscript
func test_scene_scripts_attached():
    var scene = load("res://my_scene.tscn")
    var inst = add_child_autofree(scene.instantiate())
    await wait_idle_frames(1)
    var node = inst.get_node("SomeNode")
    assert_true(node.has_method("expected_method"), "Script should be attached")
```

### 快速验证 class_name
```gdscript
# 如果能直接写类名且编辑器不报错，说明有 class_name
var player = Player.new()  # ✓ Player 有 class_name

# 如果编辑器报错 "Identifier not found"，说明没有 class_name
var helper = Helper.new()  # ✗ 需要用 load()
```

---

## 17. GDScript 语法注意事项

### 1. 三元表达式
```gdscript
# ❌ 错误 - GDScript 不支持
var result = condition ? a : b

# ✓ 正确
var result = a if condition else b
```

### 2. 类型推断与 null
```gdscript
# ❌ 错误 - 无法从 null 推断类型
var result := null

# ✓ 正确
var result: Variant = null
var result = null
```

### 3. Mock 类中的 Variant
```gdscript
class MockComponent:
    extends RefCounted
    var collected := {}
    var restored: Variant = null  # 不要用 := null
    
    func collect_state() -> Dictionary:
        return collected
    
    func restore_state(state: Dictionary) -> void:
        restored = state
```

---

## Quick Reference

```
结构:
  extends GutTest
  before_all / before_each / after_each / after_all
  func test_xxx():

实例化:
  有 class_name  → ClassName.new()
  无 class_name  → load("...").new()
  创建 Double    → double(load("...")).new()

断言:
  assert_eq / ne / true / false / null / not_null
  assert_gt / lt / gte / lte / between
  assert_has / does_not_have / eq_deep
  assert_called / not_called / call_count
  watch_signals → assert_signal_emitted

Double & Stub:
  double(Script).new()
  partial_double(Script).new()
  stub(dbl.method).to_return(val)
  stub(dbl.method).to_call_super()
  stub(dbl.method).when_passed(args)
  ignore_method_when_doubling(Class, "static_method")

Await:
  await wait_seconds(n)
  await wait_for_signal(sig, timeout)
  await wait_idle_frames(n)
  await wait_physics_frames(n)

Memory:
  autofree(obj) / autoqfree(obj)
  add_child_autofree(obj)
  assert_no_new_orphans()

GDScript 语法:
  三元表达式  → a if cond else b
  Null 类型   → var x: Variant = null

CLI:
  godot --headless -s addons/gut/gut_cmdln.gd -gexit
```
