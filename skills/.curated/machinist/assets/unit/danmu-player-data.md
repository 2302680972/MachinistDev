# 弹幕玩家数据处理

摘录自示例工程的 TS 视图片段，用于对照"弹幕玩家数据读取与处理"的写法。

```ts
public Script__X_BarrageHandler_x27_Like($msgObj: BeDict): void {
  var $uid: BeString
  var $player: BeStruct
  var $likeCount: BeLong

  $uid = $msgObj.read<"BeString">(BeString.fromBeConst("uid"), BeBool.fromBeConst("0"))
  $player = this.PlayerMap.read<"BeStruct">($uid, BeBool.fromBeConst("0"))
  $player = Act.self<Device_弹幕_54>(this).Script__X_Player_x27_InfoAutoFill($player, $msgObj)
  $likeCount = $msgObj.read<"BeLong">(BeString.fromBeConst("likes"), BeBool.fromBeConst("0"))
  Act.self<Device_弹幕_54>(this).Script__X_得到礼物_x27_点赞($likeCount, $player)
}
```

要点：
- 弹幕玩家数据通常以 `uid` 为 key 存放在 `Dict` 或 `Struct` 中
- 从消息对象中读取 `uid`，再从 `PlayerMap` 中获取对应的玩家数据
- 推荐使用 `Dict` 类型，行为更可预测
