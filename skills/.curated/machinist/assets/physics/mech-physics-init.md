# 机械物理效果初始化

摘录自示例工程的 TS 视图片段，用于对照"机械/投射物物理参数初始化与通用入口"的写法。

```ts
/**
  * name: Generic'MechPhysics
  * sourcePath: map.map_/329/scripts/Generic'MechPhysics.code
  */
@BeScript({ name: "Generic'MechPhysics", color: [8, 0, 255] })
public Script__X_Generic_x27_MechPhysics(obj: BeMech, closePhysics: BeBool, elasticity: BeFloat, kFriction: BeFloat, sFriction: BeFloat): void {
  // @locals-begin
  // @locals-end

  if (G.create.bool(closePhysics)) {
    obj.enablePhysics(BeBool.fromBeConst("0"))
  }
  obj.setPhysicsMaterial(elasticity, kFriction, sFriction)
  obj.setDrag(BeFloat.fromBeConst("0"))
  obj.setAngularDrag(BeFloat.fromBeConst("1"))
  // --- 隐式返回勿修改
  return
}

/**
  * name: Generic'BulletInitialize
  * sourcePath: map.map_/329/scripts/Generic'BulletInitialize.code
  */
@BeScript({ name: "Generic'BulletInitialize", color: [8, 0, 255], comment: "通用子弹初始化接口再封装" })
public Script__X_Generic_x27_BulletInitialize(obj: BeMech, pos: BeVector3, direction: BeVector3, type: BeLong, faction: BeLong, player: BeStruct, HPMul: BeFloat, 额外暴击率: BeFloat, initHp: BeFloat, manualSpeed: BeFloat): void {
  // @locals-begin
  var core: BeDevice
  // @locals-end

  core = obj.getCoreDevice()
  core.callFun(BeString.fromBeConst("Public'Initialize"), pos, direction, type, faction, player, this.ScenarioOffset, HPMul, 额外暴击率, initHp, manualSpeed)
  // --- 隐式返回勿修改
  return
}

/**
  * name: Bullet'New
  * sourcePath: map.map_/329/scripts/Bullet'New.code
  */
@BeScript({ name: "Bullet'New", color: [104, 0, 255], comment: "创建一个普通子弹" })
private Script__X_Bullet_x27_New(modelKey: BeString, pos: BeVector3, dir: BeVector3, faction: BeLong): BeMech {
  // @locals-begin
  var modelFile: BeString
  var bulletList: BeList
  var obj: BeMech
  // @locals-end

  modelFile = G.string.add(modelKey, BeString.fromBeConst(".mech"))
  bulletList = this.X_Bullet_x27_All.get<"BeList">(modelKey, BeBool.fromBeConst("0"))
  Act.byName<Device_子弹_334>("子弹").changeMech(modelFile)
  obj = G.createAIMech(BeString.fromBeConst("子弹"), BeFloat.fromBeConst("0"), pos, dir, BeEnum.fromBeConst("1"), BeBool.fromBeConst("0"))
  if (G.long.equal(faction, BeLong.fromBeConst("1"))) {
    Act.self<Device_Scenario_329>(this).Script__X_Generic_x27_MechPhysics(obj, BeBool.fromBeConst("0"), BeFloat.fromBeConst("0.1"), BeFloat.fromBeConst("0.7"), BeFloat.fromBeConst("0.7"))
  }
  if (G.long.equal(faction, BeLong.fromBeConst("2"))) {
    Act.self<Device_Scenario_329>(this).Script__X_Generic_x27_MechPhysics(obj, BeBool.fromBeConst("0"), BeFloat.fromBeConst("1"), BeFloat.fromBeConst("0"), BeFloat.fromBeConst("0"))
  }
  bulletList.add(obj)
  obj.setCustom<"BeBool">(BeString.fromBeConst("areaCheck"), BeBool.fromBeConst("0"))
  // --- 隐式返回勿修改
  return obj
}
```

要点：
- 装饰器使用 `@BeScript({ name: "...", color: [...], comment: "..." })` 格式
- `enablePhysics(false)` 关闭物理
- `setPhysicsMaterial(弹性, 动摩擦, 静摩擦)` 设置物理材质
- `setDrag` / `setAngularDrag` 设置阻力
