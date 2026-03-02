# 字符串拼接与翻译

摘录自示例工程的 TS 视图片段，用于对照"字符串拼接、占位符格式化、translate 与错误提示"的写法。

```ts
/**
  * name: 读取整数规则
  * sourcePath: map.map_/54/scripts/读取整数规则.code
  */
@BeScript({ name: "读取整数规则" })
public Script__读取整数规则(n: BeString): BeLong {
  // @locals-begin
  var ID: BeLong
  var 文字: BeString
  var s: BeString
  var ret: BeLong
  // @locals-end

  ID = G.findExcel2<"BeString">(BeString.fromBeConst("name"), n, BeString.fromBeConst("规则"))
  if (G.long.lt(ID, BeLong.fromBeConst("0"))) {
    文字 = G.create.string(BeString.fromBeConst("表格中找不到"))
    文字 = G.string.translate(文字)
    s = G.string.Format2(BeString.fromBeConst("%1 %2"), 文字, n)
    G.debug.print(s, BeColor.fromBeConst("255,50,50,205"), BeBool.fromBeConst("0"))
  }
  ret = G.readExcel2<"BeLong">(ID, BeString.fromBeConst("整数值"), BeString.fromBeConst("规则"))
  // --- 隐式返回勿修改
  return ret
}

/**
  * name: 倒计时时间显示
  * sourcePath: map.map_/54/scripts/倒计时时间显示.code
  */
@BeScript({ name: "倒计时时间显示" })
public Script__倒计时时间显示(秒: BeLong): BeString {
  // @locals-begin
  var r: BeString
  var day: BeFloat
  var 天: BeLong
  var 文字: BeString
  var hour: BeFloat
  var 小时: BeLong
  var min: BeFloat
  var 分钟: BeLong
  // @locals-end

  r = G.create.string(BeString.fromBeConst(""))
  if (G.long.gt(秒, BeLong.fromBeConst("86400"))) {
    day = G.long.division(秒, BeLong.fromBeConst("86400"))
    天 = day.toLong()
    文字 = G.create.string(BeString.fromBeConst(":"))
    r = G.string.add(天, 文字)
    秒 = G.long.mod(秒, BeLong.fromBeConst("86400"))
  }
  if (G.long.gt(秒, BeLong.fromBeConst("3600"))) {
    hour = G.long.division(秒, BeLong.fromBeConst("3600"))
    小时 = hour.toLong()
    if (G.long.gt(天, BeLong.fromBeConst("1"))) {
      if (G.long.lt(小时, BeLong.fromBeConst("10"))) {
        r = G.string.add(r, BeString.fromBeConst("0"))
      }
    }
    文字 = G.create.string(BeString.fromBeConst(":"))
    r = G.string.add(r, 小时, 文字)
    秒 = G.long.mod(秒, BeLong.fromBeConst("3600"))
  }
  min = G.long.division(秒, BeLong.fromBeConst("60"))
  分钟 = min.toLong()
  if (G.long.gt(小时, BeLong.fromBeConst("1"))) {
    if (G.long.lt(分钟, BeLong.fromBeConst("10"))) {
      r = G.string.add(r, BeString.fromBeConst("0"))
    }
  }
  文字 = G.create.string(BeString.fromBeConst(":"))
  r = G.string.add(r, 分钟, 文字)
  秒 = G.long.mod(秒, BeLong.fromBeConst("60"))
  if (G.long.lt(秒, BeLong.fromBeConst("10"))) {
    r = G.string.add(r, BeString.fromBeConst("0"))
  }
  r = G.string.add(r, 秒)
  // --- 隐式返回勿修改
  return r
}

/**
  * name: 读取文字规则
  * sourcePath: map.map_/54/scripts/读取文字规则.code
  */
@BeScript({ name: "读取文字规则" })
public Script__读取文字规则(n: BeString): BeString {
  // @locals-begin
  var ID: BeLong
  var 文字: BeString
  var s: BeString
  var r: BeString
  // @locals-end

  ID = G.findExcel2<"BeString">(BeString.fromBeConst("name"), n, BeString.fromBeConst("规则"))
  if (G.long.lt(ID, BeLong.fromBeConst("0"))) {
    文字 = G.create.string(BeString.fromBeConst("表格中找不到"))
    文字 = G.string.translate(文字)
    s = G.string.add(文字, n)
    G.debug.print(s, BeColor.fromBeConst("255,0,0,255"), BeBool.fromBeConst("0"))
  }
  r = G.readExcel2<"BeString">(ID, BeString.fromBeConst("文字"), BeString.fromBeConst("规则"))
  // --- 隐式返回勿修改
  return r
}

/**
  * name: 测试'发送礼物消息
  * sourcePath: map.map_/54/scripts/测试'发送礼物消息.code
  */
@BeScript({ name: "测试'发送礼物消息" })
public Script__X_测试_x27_发送礼物消息(玩家id: BeString, 礼物序号: BeLong, 礼物数量: BeLong, 头像url: BeString, 玩家昵称: BeString): void {
  // @locals-begin
  var 礼物id: BeString
  var 礼物消息: BeString
  // @locals-end

  礼物id = Act.self<Device_弹幕_54>(this).Script__X_测试_x27_得到抖音礼物id(礼物序号)
  礼物消息 = G.string.Format5(BeString.fromBeConst("[{\"msg_id\": \"1\",\"sec_openid\": \"%1\",\"sec_gift_id\": \"%2\",\"gift_num\": %3,\"gift_value\": 10000,\"avatar_url\": \"%4\",\"nickname\": \"%5\",\"timestamp\": 1}]"), 玩家id, 礼物id, 礼物数量, 头像url, 玩家昵称)
  G.simulate_comment(BeString.fromBeConst("live_gift"), 礼物消息)
  // --- 隐式返回勿修改
  return
}
```

要点：
- 装饰器使用 `@BeScript({ name: "..." })` 格式
- `G.string.translate(文字)` 翻译字符串
- `G.string.Format2("%1 %2", a, b)` 占位符格式化
- `G.string.add(a, b, c)` 拼接多个字符串
