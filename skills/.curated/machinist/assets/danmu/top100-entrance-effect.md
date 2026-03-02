# 百强入场动图特效

摘录自示例工程的 TS 视图片段，用于对照"玩家入场时按世界排名触发入场动图，并绑定头像与动图图标维护"的写法。

## 玩家入场判断

```ts
/**
  * name: UI'GamePlayerJoin
  * sourcePath: map.map_/54/scripts/UI'GamePlayerJoin.code
  */
@BeScript({ name: "UI'GamePlayerJoin", color: [156, 0, 255] })
public Script__X_UI_x27_GamePlayerJoin(player: BeStruct): BeBool {
  // @locals-begin
  var ok: BeBool
  var rank: BeLong
  var team: BeLong
  var uid: BeString
  var uidproxy: BeString
  var 未上榜: BeBool
  var 百名以外: BeBool
  var name: BeString
  var M: BeMech
  var core: BeDevice
  // @locals-end

  ok = G.create.bool(BeBool.fromBeConst("1"))
  rank = player.get<"BeLong">(BeString.fromBeConst("排行榜名次"), BeBool.fromBeConst("1"))
  team = player.get<"BeLong">(BeString.fromBeConst("team"), BeBool.fromBeConst("1"))
  uid = player.get<"BeString">(BeString.fromBeConst("id"), BeBool.fromBeConst("1"))
  uidproxy = player.get<"BeString">(BeString.fromBeConst("idproxy"), BeBool.fromBeConst("1"))
  if (G.string.equal(uidproxy, uid)) {
    未上榜 = G.long.lt(rank, BeLong.fromBeConst("0"))
    百名以外 = G.long.gte(rank, BeLong.fromBeConst("100"))
    name = player.get<"BeString">(BeString.fromBeConst("name"), BeBool.fromBeConst("1"))
    if (G.bool.or(未上榜, 百名以外)) {
      Act.self<Device_弹幕_54>(this).Script__X_播报_x27_非百强玩家加入(player)
      return ok
    }
    M = G.createAIMech(BeString.fromBeConst("玩家入场"), BeFloat.fromBeConst("99"), BeVector3.fromBeConst("0,0,0"), BeVector3.fromBeConst("0,0,1"), BeEnum.fromBeConst("0"), BeBool.fromBeConst("0"))
    core = M.getCoreDevice()
    core.callFun(BeString.fromBeConst("设置排名"), rank, player)
  }
  // --- 隐式返回勿修改
  return ok
}
```

## 入场动图播放

```ts
/**
  * name: 入场动图
  * sourcePath: 机械/玩家入场.mech_/312/scripts/入场动图.code
  */
@BeScript({ name: "入场动图" })
public Script__入场动图(rank: BeLong): BeBool {
  // @locals-begin
  var 动图: BeCanvas
  var 主UI: BeCanvas
  var layer2: BeUIRect
  var t: BeUIRawImage
  var b1: BeBool
  var b2: BeBool
  // @locals-end

  if (G.long.gte(rank, BeLong.fromBeConst("100"))) {
    return ok
  }
  G.ui._2dmode(BeBool.fromBeConst("1"), BeFloat.fromBeConst("0"))
  动图 = G.create.canvas(BeCanvas.fromBeConst("guid:fa10b4d5ee634d46836d86acca035d77"))
  主UI = G.findCanvas(BeString.fromBeConst("主UI"))
  layer2 = 主UI.read<"BeUIRect">(BeString.fromBeConst("root/layer1"))
  t = 动图.read<"BeUIRawImage">(BeString.fromBeConst("无名0"))
  layer2.addSon(t)
  // 根据排名播放不同动图
  if (G.long.equal(rank, BeLong.fromBeConst("0"))) {
    Act.self<Device_玩家入场_312>(this).Script__Script__X_Generic_x27_PlayVideo(BeString.fromBeConst("榜1.mp4"), t)
  }
  b1 = G.long.gt(rank, BeLong.fromBeConst("0"))
  b2 = G.long.lte(rank, BeLong.fromBeConst("2"))
  if (G.bool.and(b1, b2)) {
    Act.self<Device_玩家入场_312>(this).Script__Script__X_Generic_x27_PlayVideo(BeString.fromBeConst("榜2.mp4"), t)
  }
  // ... 其他排名段
  t.del(BeFloat.fromBeConst("0"))
  // --- 隐式返回勿修改
  return ok
}
```

## 视频播放

```ts
/**
  * name: Generic'PlayVideo
  * sourcePath: 机械/玩家入场.mech_/312/scripts/Generic'PlayVideo.code
  */
@BeScript({ name: "Generic'PlayVideo" })
public Script__X_Generic_x27_PlayVideo(name1: BeString, UI: BeUIRawImage): BeBool {
  // @locals-begin
  var ret: BeBool
  var 当前颜色: BeColor
  var 视频信息: BeStruct
  var duration: BeFloat
  var timeStart: BeFloat
  var X_continue: BeBool
  var time: BeFloat
  var timeUsed: BeFloat
  // @locals-end

  ret = G.create.bool(BeBool.fromBeConst("0"))
  if (Act.self<Device_玩家入场_312>(this).Script__callClientMapRet<"BeBool">(BeString.fromBeConst("弹幕.IsInGame"))) {
    当前颜色 = G.create.color(BeColor.fromBeConst("255,255,255,0"))
    视频信息 = UI.setVideo(name1)
    duration = 视频信息.get<"BeFloat">(BeString.fromBeConst("time"), BeBool.fromBeConst("0"))
    timeStart = G.time.time()
    X_continue = G.create.bool(BeBool.fromBeConst("1"))
    while (G.create.bool(X_continue)) {
      time = G.time.time()
      timeUsed = G.float.minus(time, timeStart)
      当前颜色 = 当前颜色.lerp(BeColor.fromBeConst("255,255,255,255"), BeFloat.fromBeConst("0.1"))
      UI.setColor(当前颜色)
      X_continue = G.float.lt(timeUsed, duration)
      G.delay(G.create.float(BeFloat.fromBeConst("0")))
    }
  }
  UI.del(BeFloat.fromBeConst("0"))
  // --- 隐式返回勿修改
  return ret
}
```

要点：
- 装饰器使用 `@BeScript({ name: "...", color: [...] })` 格式
- 根据排名判断是否触发入场特效
- `G.createAIMech` 创建临时机械用于播放入场动画
- `UI.setVideo(name)` 播放视频
- 动图播放完成后删除 UI 元素
