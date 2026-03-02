# 下载弹幕数据和上传数据

摘录自示例工程的 TS 视图片段，用于对照"存档上传节流、配置下载与应用"的写法。

```ts
/**
  * name: 存档'上传
  * sourcePath: map.map_/54/scripts/存档'上传.code
  */
@BeScript({ name: "存档'上传" })
public Script__X_存档_x27_上传(强制: BeBool): void {
  // @locals-begin
  var 当前时间: BeFloat
  var 时间间隔: BeFloat
  var 可上传: BeBool
  var 满足强制间隔: BeBool
  // @locals-end

  当前时间 = G.time.time()
  时间间隔 = G.float.minus(当前时间, this.X_存档_x27_上次上传)
  可上传 = G.float.gte(时间间隔, BeFloat.fromBeConst("100"))
  满足强制间隔 = G.float.gte(时间间隔, BeFloat.fromBeConst("1"))
  if (G.bool.or(可上传, 强制)) {
    if (G.create.bool(满足强制间隔)) {
      this.X_存档_x27_上次上传 = G.time.time()
      G.map.save()
    }
    if (G.bool.not(满足强制间隔)) {
      G.delay(G.create.float(BeFloat.fromBeConst("2")))
      this.X_存档_x27_上次上传 = G.time.time()
      G.map.save()
    }
  }
  // --- 隐式返回勿修改
  return
}

/**
  * name: 功能设置持久化'下载攻略提示并应用
  * sourcePath: map.map_/54/scripts/功能设置持久化'下载攻略提示并应用.code
  */
@BeScript({ name: "功能设置持久化'下载攻略提示并应用" })
public Script__X_功能设置持久化_x27_下载攻略提示并应用(): void {
  // @locals-begin
  var data: BeDict
  // @locals-end

  this.X_功能开关_x27_攻略提示 = G.create.bool(BeBool.fromBeConst("1"))
  data = G.map.load()
  if (data.contains(BeString.fromBeConst("攻略提示存档标志位"))) {
    this.X_功能开关_x27_攻略提示 = data.read<"BeBool">(BeString.fromBeConst("攻略提示存档标志位"), BeBool.fromBeConst("0"))
  }
  Act.self<Device_弹幕_54>(this).Script__X_UI_功能面板_x27_应用并刷新攻略提示显示()
  // --- 隐式返回勿修改
  return
}

/**
  * name: 功能设置持久化'上传攻略提示
  * sourcePath: map.map_/54/scripts/功能设置持久化'上传攻略提示.code
  */
@BeScript({ name: "功能设置持久化'上传攻略提示" })
public Script__X_功能设置持久化_x27_上传攻略提示(): void {
  // @locals-begin
  var data: BeDict
  // @locals-end

  data = G.map.load()
  data.insert(BeString.fromBeConst("攻略提示存档标志位"), this.X_功能开关_x27_攻略提示)
  Act.self<Device_弹幕_54>(this).Script__X_存档_x27_上传(BeBool.fromBeConst("1"))
  // --- 隐式返回勿修改
  return
}
```

要点：
- 装饰器使用 `@BeScript({ name: "..." })` 格式
- `G.map.save()` 上传存档
- `G.map.load()` 下载存档
- 上传节流：检查时间间隔避免频繁上传
