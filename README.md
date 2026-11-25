# MultiWorldPlatform

**该项目由 AI 生成，README编写有AI参与。**

MultiWorldPlatform 是一个运行于 Sponge 平台的世界管理插件。开发期间内部名称 CommonWorldPlatform。

该插件可在以下服务端运行：

- SpongeVanilla 1.21.1
- SpongeNeo 1.21.1

## 命令

命令是`/mwp`，同时`/cwp`和`/multiworldplatform`也可使用。

```text
/mwp list
/mwp create <world_name> <environment> [args]
/mwp import <world_name>
/mwp export <world_name>
/mwp load <world_name>
/mwp unload <world_name> [--tp-player=<world_name>]
/mwp destroy <world_name> [--auto]
/mwp tp <world_name> [player]
/mwp gamerule <world_name> <rule> [value]
```

`environment` 支持 `minecraft:overworld`、`minecraft:the_nether`、
`minecraft:the_end`，也接受`overworld`、`the_nether`、`the_end`这样的简写形式。

创建参数：

```text
--seed=<int> //生成种子
--world-type=normal|flat|void //创建类型：正常/超平坦/虚空
--no-structures //不生成任何结构
--no-adjust-spawn //忽略且不调整危险世界出生点，也就是玩家可以出生在岩浆中
```

默认世界为 `world`，下界和末地显示为 `DIM-1`、`DIM1`，与世界目录
命名一致。插件创建的世界显示为 `multiworldplatform:<world_name>`。

`/mwp unload` 会先把玩家传送到目标世界，默认是 `minecraft:overworld`。
默认主世界不能被卸载或删除。删除已加载世界必须使用 `--auto`：

```text
/mwp destroy arena_01
# 已加载时拒绝

/mwp destroy arena_01 --auto
# 先踢出玩家、卸载，再删除
```

## 导入与导出

导入时按以下顺序查找 ZIP：

1. `world/<world_name>.zip`
2. `config/multiworldplatform/import/<world_name>.zip`

ZIP 必须由本插件导出。导出位置为：

```text
config/multiworldplatform/export/<world_name>.zip
```

导出 ZIP 的内部结构如下，其中世界文件夹名必须与导入名一致：

```text
<world_name>/
  .multiworldplatform-world.properties
  multiworldplatform-template.json
  multiworldplatform-world.conf
  level.dat
  region/
  entities/
  ...
```

导出要求目标世界中没有玩家。若世界当前已加载，插件会先保存，再短暂卸载并生成导出文件，完成后恢复原来的加载状态。

## 配置

全局配置：

```text
config/multiworldplatform/config.conf
```

每个世界一个世界单元文件：

```text
config/multiworldplatform/worlds/<world_name>.conf
```

示例：

```hocon
world {
    name="arena_01"
    key="multiworldplatform:arena_01"
    environment="minecraft:overworld"
    type=normal
    seed=12345
    generate-structures=true
    adjust-spawn=true
}
game-rules {
    do_daylight_cycle="false"
    random_tick_speed="3"
}
```

可以运行以下命令即时修改游戏规则：

```text
/mwp gamerule arena_01 doDaylightCycle false
/mwp gamerule arena_01 doDaylightCycle
```

手动修改世界配置后，在下一次加载该世界时生效。插件创建的世界默认
`load-on-startup=false`，重启后需再次`/mwp load`。


## 权限

文件权限节点：

```text
multiworldplatform.list
multiworldplatform.create
multiworldplatform.import
multiworldplatform.export
multiworldplatform.load
multiworldplatform.unload
multiworldplatform.destroy
multiworldplatform.tp
multiworldplatform.tp.other
multiworldplatform.gamerule
```

同时接受`commonworldplatform.*` 节点。安装
LuckPerms 时使用权限节点；未安装 LuckPerms 时使用原版 3 级 OP 的指令权限。

## API 与热重载

公开 API 和调用示例见 [docs/API.md](docs/API.md)。

维度热重载验证结论见
[docs/RELOAD-VALIDATION.md](docs/RELOAD-VALIDATION.md)。对于小游戏地图恢复，
推荐流程是：清空玩家、保存、卸载、替换维度目录、以同一 world key 加载。该流程在本次
SpongeNeo 12.0.4-RC0 实测中稳定；不支持在保持维度加载时原地替换其文件。
