# 类型注册与预创建

摘录自示例工程的 TS 视图片段，用于对照"注册同时预创建"的写法。

```ts
/**
  * name: Model'RegisterType
  * sourcePath: map.map_/329/scripts/Model'RegisterType.code
  */
public X_Model_x27_RegisterType = BeScript({ name: "Model'RegisterType", color: [104, 0, 255], comment: "注册类型" })((modelKey: BeString, amount: BeLong) => {
  // @locals-begin
  var bullet: BeList
  var bulletUnused: BeList
  var exist: BeBool
  var created: BeLong
  var model: BeMech
  var core: BeDevice
  // @locals-end

  bullet = G.create.listN()
  bulletUnused = G.create.listN()
  exist = this.X_Model_x27_All.contains(modelKey)
  if (G.bool.not(exist)) {
    this.X_Model_x27_All.set(modelKey, bullet)
    this.X_Model_x27_Unused.set(modelKey, bulletUnused)
  }
  created = G.create.long(BeLong.fromBeConst("0"))
  while (G.long.lt(created, amount)) {
    model = Act.self<Device_Scenario_329>(this).X_Model_x27_SetModel(BeVector3.fromBeConst("0,-1e+06,0"), BeVector3.fromBeConst("1,0,0"), modelKey)
    core = model.getCoreDevice()
    core.callFun(BeString.fromBeConst("Public'AutoReclaim"), modelKey)
    created.addEqual(BeLong.fromBeConst("1"))
  }
  // --- 隐式返回勿修改
  return
});

/**
  * name: Wall'New
  * sourcePath: map.map_/329/scripts/Wall'New.code
  */
protected X_Wall_x27_New = BeScript({ name: "Wall'New", color: [8, 0, 255], private: true })((pos: BeVector3, direction: BeVector3) => {
  // @locals-begin
  var list: BeList
  var obj: BeMech
  // @locals-end

  list = this.X_Model_x27_All.get<"BeList">(BeString.fromBeConst("wall"), BeBool.fromBeConst("0"))
  obj = G.createAIMech(BeString.fromBeConst("边界"), BeFloat.fromBeConst("0"), pos, direction, BeEnum.fromBeConst("0"), BeBool.fromBeConst("0"))
  Act.self<Device_Scenario_329>(this).X_Generic_x27_MechPhysics(obj, BeBool.fromBeConst("1"), BeFloat.fromBeConst("1"), BeFloat.fromBeConst("0"), BeFloat.fromBeConst("0"))
  list.add(obj)
  // --- 隐式返回勿修改
  return obj
});

/**
  * name: Wall'SetWall
  * sourcePath: map.map_/329/scripts/Wall'SetWall.code
  */
public X_Wall_x27_SetWall = BeScript({ name: "Wall'SetWall", color: [8, 0, 255], retVar: "obj" })((pos: BeVector3, direction: BeVector3) => {
  // @locals-begin
  var exist: BeBool
  var list1: BeList
  var list2: BeList
  var unused: BeList
  var index: BeFloat
  var obj: BeMech
  // --- return-separator ---
  // @locals-end

  exist = this.X_Model_x27_All.contains(BeString.fromBeConst("wall"))
  if (G.bool.not(exist)) {
    list1 = G.create.listN()
    list2 = G.create.listN()
    this.X_Model_x27_All.set(BeString.fromBeConst("wall"), list1)
    this.X_Model_x27_Unused.set(BeString.fromBeConst("wall"), list2)
  }
  unused = this.X_Model_x27_Unused.get<"BeList">(BeString.fromBeConst("wall"), BeBool.fromBeConst("0"))
  index = unused.count()
  exist = G.float.gt(index, BeFloat.fromBeConst("0"))
  if (G.create.bool(exist)) {
    index.minusEqual(BeFloat.fromBeConst("1"))
    obj = unused.read<"BeMech">(index)
    obj.setActive(BeBool.fromBeConst("1"))
    obj.move(pos, BeEnum.fromBeConst("0"))
    obj.rot(direction, BeVector3.fromBeConst("0,1,0"))
    unused.remove(index)
  }
  if (G.bool.not(exist)) {
    obj = Act.self<Device_Scenario_329>(this).X_Wall_x27_New(pos, direction)
  }
  // --- 隐式返回勿修改
  return obj
});
```

要点：
- 使用 `BeScript({ name: "...", color: [...], comment: "..." })` 赋值格式
- 注册时创建两个列表：All 和 Unused
- 预创建时循环调用 SetModel 并立即回收
