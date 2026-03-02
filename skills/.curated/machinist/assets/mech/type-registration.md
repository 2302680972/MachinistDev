# 类型注册

摘录自示例工程的 TS 视图片段，用于对照"类型注册"的写法。

## 注册子弹类型

```ts
/**
  * name: Bullet'RegisterBulletType
  * sourcePath: map.map_/329/scripts/Bullet'RegisterBulletType.code
  */
@BeScript({ name: "Bullet'RegisterBulletType", comment: "注册子弹实体" })
public Script__X_Bullet_x27_RegisterBulletType($bulletType: BeString, $modelKey: BeString, $teamLimit: BeLong, $globalLimit: BeLong): void {
  // @locals-begin
  var $zero: BeLong
  var $bulletInfo: BeStruct
  // @locals-end

  $zero = G.create.long(BeLong.fromBeConst("0"))
  $bulletInfo = G.create.struct()
  $bulletInfo.set(BeString.fromBeConst("modelKey"), $modelKey)
  $bulletInfo.set(BeString.fromBeConst("teamLimit"), $teamLimit)
  $bulletInfo.set(BeString.fromBeConst("team1Current"), $zero)
  $bulletInfo.set(BeString.fromBeConst("team2Current"), $zero)
  $bulletInfo.set(BeString.fromBeConst("globalLimit"), $globalLimit)
  $bulletInfo.set(BeString.fromBeConst("globalCurrent"), $zero)
  this.BulletRegInfo.set($bulletType, $bulletInfo)
  // --- 隐式返回勿修改
  return
}
```

## 注册模型类型

```ts
/**
  * name: Bullet'RegisterModelType
  * sourcePath: map.map_/329/scripts/Bullet'RegisterModelType.code
  */
@BeScript({ name: "Bullet'RegisterModelType", comment: "注册模型" })
public Script__X_Bullet_x27_RegisterModelType($key: BeString): void {
  // @locals-begin
  var $bullet: BeList
  var $bulletUnused: BeList
  var $exist: BeBool
  // @locals-end

  $bullet = G.create.listN()
  $bulletUnused = G.create.listN()
  $exist = this.Bullet.contains($key)
  if (G.bool.not($exist)) {
    this.Bullet.set($key, $bullet)
    this.BulletUnused.set($key, $bulletUnused)
  }
  // --- 隐式返回勿修改
  return
}
```

## 注册礼物模型类型

```ts
/**
  * name: GiftModel'RegisterType
  * sourcePath: map.map_/329/scripts/GiftModel'RegisterType.code
  */
@BeScript({ name: "GiftModel'RegisterType", comment: "注册类型" })
public Script__X_GiftModel_x27_RegisterType($modelKey: BeString): void {
  // @locals-begin
  var $bullet: BeList
  var $bulletUnused: BeList
  var $exist: BeBool
  // @locals-end

  $bullet = G.create.listN()
  $bulletUnused = G.create.listN()
  $exist = this.Bullet.contains($modelKey)
  if (G.bool.not($exist)) {
    this.Bullet.set($modelKey, $bullet)
    this.BulletUnused.set($modelKey, $bulletUnused)
  }
  // --- 隐式返回勿修改
  return
}
```

要点：
- 装饰器使用 `@BeScript({ name: "...", comment: "..." })` 格式
- 注册时创建两个列表：一个存放所有实例，一个存放未使用的实例
- 使用 `contains` 检查是否已注册，避免重复注册
