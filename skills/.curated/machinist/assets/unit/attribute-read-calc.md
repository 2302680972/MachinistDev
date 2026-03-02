# 属性读取与计算

摘录自示例工程的 TS 视图片段，用于对照"读表刷新等级属性、HP/伤害/暴击等链路"的写法。

```ts
/**
  * name: 属性'更新等级属性
  * sourcePath: map.map_/331/scripts/属性'更新等级属性.code
  */
@BeScript({ name: "属性'更新等级属性" })
public Script__X_属性_x27_更新等级属性(): BeLong {
  // @locals-begin
  var 下一级: BeLong
  var $ret: BeLong
  // @locals-end

  下一级 = G.long.add(this.X_属性_x27_等级, BeLong.fromBeConst("1"))
  $ret = G.readExcel2<"BeLong">(下一级, BeString.fromBeConst("升级总需求"), BeString.fromBeConst("升级需求"))
  // 攻方属性列表
  this.X_属性_x27_攻方HP倍数 = G.readExcel2<"BeFloat">(this.X_属性_x27_等级, BeString.fromBeConst("攻方HP倍数"), BeString.fromBeConst("升级需求"))
  this.X_属性_x27_攻方冷却 = G.readExcel2<"BeFloat">(this.X_属性_x27_等级, BeString.fromBeConst("攻方冷却"), BeString.fromBeConst("升级需求"))
  this.X_属性_x27_攻方发射数 = G.readExcel2<"BeLong">(this.X_属性_x27_等级, BeString.fromBeConst("攻方有效发射数"), BeString.fromBeConst("升级需求"))
  this.X_属性_x27_攻方暴击率 = G.readExcel2<"BeFloat">(this.X_属性_x27_等级, BeString.fromBeConst("攻方暴击率"), BeString.fromBeConst("升级需求"))
  this.X_属性_x27_攻方暴击倍数 = G.readExcel2<"BeFloat">(this.X_属性_x27_等级, BeString.fromBeConst("攻方暴击倍数"), BeString.fromBeConst("升级需求"))
  // 守方属性列表
  this.X_属性_x27_守方点赞乘数 = G.readExcel2<"BeLong">(this.X_属性_x27_等级, BeString.fromBeConst("守方点赞乘数"), BeString.fromBeConst("升级需求"))
  this.X_属性_x27_守方冷却 = G.readExcel2<"BeFloat">(this.X_属性_x27_等级, BeString.fromBeConst("守方冷却"), BeString.fromBeConst("升级需求"))
  this.X_属性_x27_守方发射数 = G.readExcel2<"BeLong">(this.X_属性_x27_等级, BeString.fromBeConst("守方有效发射数"), BeString.fromBeConst("升级需求"))
  this.X_属性_x27_守方暴击率 = G.readExcel2<"BeFloat">(this.X_属性_x27_等级, BeString.fromBeConst("守方暴击率"), BeString.fromBeConst("升级需求"))
  this.X_属性_x27_守方暴击倍数 = G.readExcel2<"BeFloat">(this.X_属性_x27_等级, BeString.fromBeConst("守方暴击倍数"), BeString.fromBeConst("升级需求"))
  // 模型相关
  this.X_属性_x27_攻方炮塔缩放 = G.readExcel2<"BeFloat">(this.X_属性_x27_等级, BeString.fromBeConst("攻方炮塔缩放"), BeString.fromBeConst("升级需求"))
  this.X_属性_x27_守方炮塔缩放 = G.readExcel2<"BeFloat">(this.X_属性_x27_等级, BeString.fromBeConst("守方炮塔缩放"), BeString.fromBeConst("升级需求"))
  this.X_属性_x27_边框颜色 = G.readExcel2<"BeColor">(this.X_属性_x27_等级, BeString.fromBeConst("边框颜色"), BeString.fromBeConst("升级需求"))
  this.X_属性_x27_边框自发光颜色 = G.readExcel2<"BeColor">(this.X_属性_x27_等级, BeString.fromBeConst("边框自发光"), BeString.fromBeConst("升级需求"))
  if (G.long.equal(this.X_Attr_x27_Type, BeLong.fromBeConst("11"))) {
    Act.self<Device_基地_331>(this).Script__X_属性_x27_攻方炮塔更新属性()
  }
  if (G.long.equal(this.X_Attr_x27_Type, BeLong.fromBeConst("21"))) {
    Act.self<Device_基地_331>(this).Script__X_属性_x27_守方炮塔更新属性()
  }
  // --- 隐式返回勿修改
  return $ret
}

/**
  * name: Public'GetHP
  * sourcePath: map.map_/331/scripts/Public'GetHP.code
  */
@BeScript({ name: "Public'GetHP", color: [8, 0, 255], comment: "得到此子弹的HP" })
public Script__X_Public_x27_GetHP(): BeFloat {
  // @locals-begin
  var hp: BeFloat
  // @locals-end

  hp = G.create.float(this.X_Attr_x27_HP)
  // --- 隐式返回勿修改
  return hp
}

/**
  * name: Public'Damage
  * sourcePath: map.map_/331/scripts/Public'Damage.code
  */
@BeScript({ name: "Public'Damage", color: [8, 0, 255], comment: "计算伤害并按需要回收" })
public Script__X_Public_x27_Damage(damage: BeFloat, faction: BeLong): BeFloat {
  // @locals-begin
  var hp: BeFloat
  var counter: BeFloat
  // @locals-end

  if (G.long.equal(faction, this.X_Attr_x27_Faction)) {
    return counter
  }
  hp = G.create.float(this.X_Attr_x27_HP)
  // 预先计算反击伤害
  counter = G.create.float(BeFloat.fromBeConst("0"))
  if (G.float.gte(hp, damage)) {
    counter = G.create.float(damage)
  }
  if (G.float.lt(hp, damage)) {
    counter = G.create.float(hp)
  }
  hp = G.float.minus(hp, damage)
  Act.self<Device_基地_331>(this).Script__X_Private_x27_UpdateHP(hp)
  // --- 隐式返回勿修改
  return counter
}

/**
  * name: Private'UpdateHP
  * sourcePath: map.map_/331/scripts/Private'UpdateHP.code
  */
@BeScript({ name: "Private'UpdateHP", color: [8, 0, 255] })
private Script__X_Private_x27_UpdateHP(newHP: BeFloat): void {
  // @locals-begin
  var currentHP: BeFloat
  // @locals-end

  currentHP = G.create.float(this.X_Attr_x27_HP)
  Act.self<Device_基地_331>(this).Script__X_Private_x27_BeforeChangeHP(currentHP, newHP)
  this.X_Attr_x27_HP = G.create.float(newHP)
  Act.self<Device_基地_331>(this).Script__X_Private_x27_AfterChangeHP(currentHP, newHP)
  // --- 隐式返回勿修改
  return
}

/**
  * name: Private'BeforeChangeHP
  * sourcePath: map.map_/331/scripts/Private'BeforeChangeHP.code
  */
@BeScript({ name: "Private'BeforeChangeHP", color: [8, 0, 255], comment: "HP变更前" })
private Script__X_Private_x27_BeforeChangeHP(oldHP: BeFloat, newHP: BeFloat): void {
  // @locals-begin
  // @locals-end

  if (G.float.lte(newHP, BeFloat.fromBeConst("0"))) {
    Act.self<Device_基地_331>(this).Script__X_Private_x27_OnDeath()
  }
  // --- 隐式返回勿修改
  return
}

/**
  * name: Private'AfterChangeHP
  * sourcePath: map.map_/331/scripts/Private'AfterChangeHP.code
  */
@BeScript({ name: "Private'AfterChangeHP", color: [8, 0, 255], comment: "HP变更后" })
private Script__X_Private_x27_AfterChangeHP(oldHP: BeFloat, newHP: BeFloat): void {
  // @locals-begin
  // @locals-end

  // 可在此处更新血条UI等
  // --- 隐式返回勿修改
  return
}
```

要点：
- 装饰器使用 `@BeScript({ name: "...", color: [...], comment: "..." })` 格式
- `G.readExcel2<类型>(行ID, 列名, 表名)` 读取表格数据
- HP 变更采用 Before/After 钩子模式
