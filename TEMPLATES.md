# GUT Test Templates

## 模板1: 基础单元测试
```gdscript
# res://test/unit/test_calculator.gd
extends GutTest

var _calc: Calculator

func before_each():
    _calc = autofree(Calculator.new())

func test_add_positive_numbers():
    assert_eq(_calc.add(2, 3), 5)

func test_add_negative_numbers():
    assert_eq(_calc.add(-1, -1), -2)

func test_divide_by_zero_returns_error():
    var result = _calc.divide(10, 0)
    assert_null(result)
```

---

## 模板2: 带Double的测试
```gdscript
# res://test/unit/test_player_combat.gd
extends GutTest

var _player: Player
var _weapon_dbl

func before_each():
    var Weapon = load("res://scripts/weapon.gd")
    _weapon_dbl = double(Weapon).new()
    
    _player = add_child_autofree(Player.new())
    _player.set_weapon(_weapon_dbl)

func test_attack_calls_weapon_fire():
    stub(_weapon_dbl.fire).to_return(true)
    
    _player.attack()
    
    assert_called(_weapon_dbl, "fire")

func test_attack_with_no_ammo():
    stub(_weapon_dbl.has_ammo).to_return(false)
    stub(_weapon_dbl.fire).to_return(false)
    
    var result = _player.attack()
    
    assert_false(result)
    assert_not_called(_weapon_dbl, "fire")
```

---

## 模板3: 信号测试
```gdscript
# res://test/unit/test_health_component.gd
extends GutTest

var _health: HealthComponent

func before_each():
    _health = add_child_autofree(HealthComponent.new())
    _health.max_health = 100
    _health.current_health = 100
    watch_signals(_health)

func test_damage_emits_health_changed():
    _health.take_damage(30)
    
    assert_signal_emitted(_health, "health_changed")
    assert_signal_emitted_with_parameters(_health, "health_changed", [70, 100])

func test_death_emits_died_signal():
    _health.take_damage(100)
    
    assert_signal_emitted(_health, "died")

func test_heal_does_not_exceed_max():
    _health.take_damage(50)
    _health.heal(100)
    
    assert_eq(_health.current_health, 100)
```

---

## 模板4: 异步测试
```gdscript
# res://test/unit/test_async_loader.gd
extends GutTest

func test_resource_loads_within_timeout():
    var loader = add_child_autofree(AsyncLoader.new())
    
    loader.start_loading("res://scenes/level.tscn")
    
    var completed = await wait_for_signal(loader.loading_complete, 5)
    assert_true(completed, "Loading should complete within 5 seconds")
    assert_not_null(loader.loaded_resource)

func test_animation_completes():
    var sprite = add_child_autofree(AnimatedSprite2D.new())
    sprite.sprite_frames = load("res://sprites/player.tres")
    watch_signals(sprite)
    
    sprite.play("attack")
    
    await wait_for_signal(sprite.animation_finished, 2)
    assert_signal_emitted(sprite, "animation_finished")
```

---

## 模板5: 参数化测试
```gdscript
# res://test/unit/test_damage_calculator.gd
extends GutTest

var damage_params = ParameterFactory.named_parameters(
    ["base_damage", "armor", "expected_damage"],
    [
        [100, 0, 100],      # 无护甲
        [100, 50, 50],      # 50%护甲
        [100, 100, 0],      # 100%护甲
        [100, 150, 0],      # 超过100%护甲仍为0
    ]
)

func test_damage_reduction(p=use_parameters(damage_params)):
    var calc = autofree(DamageCalculator.new())
    
    var result = calc.calculate_damage(p.base_damage, p.armor)
    
    assert_eq(result, p.expected_damage, 
        "Base %d with armor %d should deal %d" % [p.base_damage, p.armor, p.expected_damage])
```

---

## 模板6: 输入测试
```gdscript
# res://test/unit/test_player_input.gd
extends GutTest

var _sender: InputSender
var _player: Player

func before_each():
    _sender = InputSender.new(Input)
    _player = add_child_autofree(Player.new())

func after_each():
    _sender.release_all()
    _sender.clear()

func test_move_right_on_input():
    var start_x = _player.position.x
    
    _sender.action_down("move_right").hold_for(0.5)
    await _sender.idle
    await wait_physics_frames(5)
    
    assert_gt(_player.position.x, start_x)

func test_jump_on_space():
    _sender.action_down("jump")
    await wait_physics_frames(2)
    
    assert_true(_player.is_jumping)
```

---

## 模板7: 内部类分组测试
```gdscript
# res://test/unit/test_inventory.gd
extends GutTest

class TestAddItem:
    extends GutTest
    
    var _inventory: Inventory
    
    func before_each():
        _inventory = autofree(Inventory.new())
        _inventory.max_slots = 10
    
    func test_add_item_to_empty_slot():
        var item = autofree(Item.new())
        var success = _inventory.add_item(item)
        
        assert_true(success)
        assert_eq(_inventory.get_item_count(), 1)
    
    func test_add_item_when_full():
        for i in range(10):
            _inventory.add_item(autofree(Item.new()))
        
        var extra = autofree(Item.new())
        var success = _inventory.add_item(extra)
        
        assert_false(success)

class TestRemoveItem:
    extends GutTest
    
    var _inventory: Inventory
    var _item: Item
    
    func before_each():
        _inventory = autofree(Inventory.new())
        _item = autofree(Item.new())
        _inventory.add_item(_item)
    
    func test_remove_existing_item():
        var success = _inventory.remove_item(_item)
        
        assert_true(success)
        assert_eq(_inventory.get_item_count(), 0)
    
    func test_remove_nonexistent_item():
        var other = autofree(Item.new())
        var success = _inventory.remove_item(other)
        
        assert_false(success)
```

---

## 模板8: Partial Double测试
```gdscript
# res://test/unit/test_enemy_ai.gd
extends GutTest

func test_enemy_attacks_when_player_in_range():
    var Enemy = load("res://scripts/enemy.gd")
    var enemy = partial_double(Enemy).new()
    add_child_autofree(enemy)
    
    # stub检测方法，但保留攻击逻辑
    stub(enemy.is_player_in_range).to_return(true)
    
    enemy.update_ai()
    
    assert_called(enemy, "attack")

func test_enemy_patrols_when_player_not_in_range():
    var Enemy = load("res://scripts/enemy.gd")
    var enemy = partial_double(Enemy).new()
    add_child_autofree(enemy)
    
    stub(enemy.is_player_in_range).to_return(false)
    
    enemy.update_ai()
    
    assert_called(enemy, "patrol")
    assert_not_called(enemy, "attack")
```

---

## 模板9: 场景测试
```gdscript
# res://test/integration/test_game_level.gd
extends GutTest

var _level

func before_each():
    var LevelScene = load("res://scenes/level_01.tscn")
    _level = add_child_autofree(LevelScene.instantiate())
    await wait_physics_frames(2)  # 等待场景初始化

func test_player_spawns_at_start_position():
    var player = _level.get_node("Player")
    var spawn = _level.get_node("SpawnPoint")
    
    assert_almost_eq(player.global_position, spawn.global_position, Vector2(1, 1))

func test_enemies_count():
    var enemies = _level.get_node("Enemies").get_children()
    
    assert_eq(enemies.size(), 5)
```

---

## 模板10: 内存泄漏测试
```gdscript
# res://test/unit/test_memory.gd
extends GutTest

class TestNoLeaks:
    extends GutTest
    
    func test_player_no_leak_without_tree():
        var Player = load("res://scripts/player.gd")
        var player = Player.new()
        player.free()
        
        assert_no_new_orphans()
    
    func test_player_no_leak_with_tree():
        var Player = load("res://scripts/player.gd")
        var player = Player.new()
        add_child(player)
        player.free()
        
        assert_no_new_orphans()
    
    func test_scene_no_leak():
        var Scene = load("res://scenes/enemy.tscn")
        var instance = Scene.instantiate()
        add_child(instance)
        instance.queue_free()
        
        await wait_seconds(0.1)
        assert_no_new_orphans()
```
