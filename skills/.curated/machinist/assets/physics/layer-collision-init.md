# 层碰撞关系初始化

摘录自示例工程的 TS 视图片段，用于对照"机械/子弹设置 layer，以及按需提供碰撞检测层列表"的写法。

## 设置碰撞层

```ts
/**
  * name: Public'SetLayer
  * sourcePath: map.map_/329/scripts/Public'SetLayer.code
  */
@BeScript({ name: "Public'SetLayer" })
public Script__X_Public_x27_SetLayer(layer1: BeFloat, layer2: BeFloat): void {
  // @locals-begin
  // @locals-end

  Act.self<Device_anonymous_0>(this).setLayer(layer1)
  Act.byName<Device_攻击判定_8>("攻击判定").setLayer(layer2)
  // --- 隐式返回勿修改
  return
}
```

## 获取碰撞层列表

```ts
/**
  * name: 粒子'得到碰撞层列表
  * sourcePath: map.map_/329/scripts/粒子'得到碰撞层列表.code
  */
@BeScript({ name: "粒子'得到碰撞层列表" })
public Script__X_粒子_x27_得到碰撞层列表(): BeList {
  // @locals-begin
  var layer1: BeFloat
  var list: BeList
  var init: BeBool
  // @locals-end

  if (G.bool.not(init)) {
    layer1 = G.create.float(BeFloat.fromBeConst("0"))
    list = G.create.listN(layer1)
    init = G.create.bool(BeBool.fromBeConst("1"))
  }
  // --- 隐式返回勿修改
  return list
}
```

要点：
- 装饰器使用 `@BeScript({ name: "..." })` 格式
- `setLayer` 设置零件的碰撞层
- 碰撞层列表用于粒子检测等场景
