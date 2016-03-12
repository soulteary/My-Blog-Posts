# 使用前端方式挑战 Chrome 小恐龙游戏高分

今天看论坛发现有人发帖说 `Chrome周年庆祝` 在“小恐龙”游戏中埋入了新的菜单，可以触碰的 **蛋糕** 以及触碰之后获得的 **生日帽**。

看到现在“小恐龙”这个彩蛋的改进，往事历历在目。但是很快，帖子中有人晒高分引起了我的注意：这个游戏虽说简单，但是在分数达到一定程度后，加速度会让玩家**很难**继续游戏。

记得很早之前在机场玩过这个游戏，记忆中获取**三千以上的分数**还是挺困难的，更别提**八千分**，所以我这里推测这个玩家应该是有一些“特别的技巧”。

在游戏中取得令人叹为观止的高分，一般来说无非三种手段：

1. 练习：勤学苦练，熟能生巧，加上极致的耐心。
2. 工具：使用一些辅助设施来模拟人的操作。
3. 外挂：针对难度或者得分倍数进行改造或者直接对结果进行修改、甚至破坏游戏规则。

第一种方式我们暂且略过，后面如果实在闲，再进行尝试，我们直接开始后面两种方案的尝试。

## 编写辅助工具

![奔跑的小恐龙](https://attachment.soulteary.com/2018/09/13/dino.png)

> 君子性非异也，擅假于物也。

网上有个老外曾经编写过一个根据判断X轴方向小恐龙和前面物品距离进行自动跳跃的程序，在稍加改造后，将下面的代码贴到控制台中，即可完成小恐龙自动奔跑的神操作。

这里添加了代码避免小恐龙跳过蛋糕、避而不吃，以及调整了对障碍物需要跳跃的高度判断，防止小恐龙看到低飞的鸟儿迎面而上，当然，还有能够在代码执行后，自动开始游戏。

```js
function TrexRunnerBot() {

  const makeKeyArgs = (keyCode) => {
    const preventDefault = () => void 0;
    return {keyCode, preventDefault};
  };

  const upKeyArgs = makeKeyArgs(38);
  const downKeyArgs = makeKeyArgs(40);
  const startArgs = makeKeyArgs(32);

  if (!Runner().playing) {
    Runner().onKeyDown(startArgs);
    setTimeout(() => {
      Runner().onKeyUp(startArgs);
    }, 500);
  }

  function conquerTheGame() {
    if (!Runner || !Runner().horizon.obstacles[0]) return;

    const obstacle = Runner().horizon.obstacles[0];

    if (obstacle.typeConfig && obstacle.typeConfig.type === 'SNACK') return;

    if (needsToTackle(obstacle) && closeEnoughToTackle(obstacle)) tackle(obstacle);
  }

  function needsToTackle(obstacle) {
    return obstacle.yPos !== 50;
  }

  function closeEnoughToTackle(obstacle) {
    return obstacle.xPos <= Runner().currentSpeed * 18;
  }

  function tackle(obstacle) {
    if (isDuckable(obstacle)) {
      duck();
    } else {
      jumpOver(obstacle);
    }
  }

  function isDuckable(obstacle) {
    return obstacle.yPos === 50;
  }

  function duck() {
    Runner().onKeyDown(downKeyArgs);

    setTimeout(() => {
      Runner().onKeyUp(downKeyArgs);
    }, 500);
  }

  function jumpOver(obstacle) {
    if (isNextObstacleCloseTo(obstacle))
      jumpFast();
    else
      Runner().onKeyDown(upKeyArgs);
  }

  function isNextObstacleCloseTo(currentObstacle) {
    const nextObstacle = Runner().horizon.obstacles[1];

    return nextObstacle && nextObstacle.xPos - currentObstacle.xPos <= Runner().currentSpeed * 42;
  }

  function jumpFast() {
    Runner().onKeyDown(upKeyArgs);
    Runner().onKeyUp(upKeyArgs);
  }

  return {conquerTheGame: conquerTheGame};
}

let bot = TrexRunnerBot();
let botInterval = setInterval(bot.conquerTheGame, 2);
```

当然，为了你能够在移动端的复制便捷，我这里提供了压缩版的代码。

```js
function TrexRunnerBot(){function f(){Runner().onKeyDown(d);setTimeout(function(){Runner().onKeyUp(d)},500)}var b=function(a){return{keyCode:a,preventDefault:function(){}}},c=b(38),d=b(40),e=b(32);Runner().playing||(Runner().onKeyDown(e),setTimeout(function(){Runner().onKeyUp(e)},500));return{conquerTheGame:function(){if(Runner&&Runner().horizon.obstacles[0]){var a=Runner().horizon.obstacles[0];if((!a.typeConfig||"SNACK"!==a.typeConfig.type)&&50!==a.yPos&&a.xPos<=18*Runner().currentSpeed)if(50=== a.yPos)f();else{var b=Runner().horizon.obstacles[1];if(b&&b.xPos-a.xPos<=42*Runner().currentSpeed)Runner().onKeyDown(c),Runner().onKeyUp(c);else Runner().onKeyDown(c)}}}}}var bot=TrexRunnerBot(),botInterval=setInterval(bot.conquerTheGame,2);
```

如果你想在桌面端浏览器游玩小恐龙游戏，除了断网的操作外，还可以直接访问 `chrome://dino/` 这个地址。 

如果你想在手机上体验小恐龙自动奔跑，请参考下面的操作：

1. 飞行模式后打开 Chrome ，然后访问任意页面，呼出小恐龙界面。
2. 将上面的代码粘贴到地址栏。
3. 在地址栏前手动加上 `javascript:` ，然后确认。

你的小恐龙就自个去浪了，不过这里你获取分数的速度依然和正常游玩的玩家是一样的，小恐龙走一步记一分。

那么，接下来我们换个套路。

## 改写计分逻辑

与其说改写，不如说“劫持”。ES5 有一个很古老的 API， `Object.defineProperty()`，借助这个 API ，我们能够轻易的修改现有对象上的属性，配合重新定义对象具体内容的 `getter`、`setter` 描述符，可以做到对于属性的劫持操作，是不是很眼熟？没错，这个方案也是老生常谈的 MVVM 框架的双向数据绑定的实现方案之一。

```js
let hackScore = 0;

Object.defineProperty(Runner.instance_, 'distanceRan', {
  get: () => hackScore,
  set: (value) => hackScore = value + Math.floor(Math.random() * 1000),
  configurable: true,
  enumerable: true,
});
```

同样提供了压缩版本。

```js
var hackScore=0;Object.defineProperty(Runner.instance_,"distanceRan",{get:function(){return hackScore},set:function(a){return hackScore=a+Math.floor(1E3*Math.random())},configurable:!0,enumerable:!0});
```

将上面代码执行之后，再次运行程序，你会发现你获取分数的速度提升了一千倍。

如果你将第一个方案和这个方案的代码结合，会获得一个能够自动奔跑获得高分的“智能小恐龙”。

不过因为我们的“外挂”是基于计时器进行距离计算并模拟用户操作的，当你获得很高很高的分数之后，障碍物推进速度过快，一旦你进行窗口的来回切换，游戏进行暂停和游玩的状态切换，很大概率上“外挂”操作会延时，导致 `GAME OVER`。

如何能避免这个事情呢，我们来讲讲第三个套路。

## 破坏规则大法

如果说第一个方案起码还有付出时间成本，模拟玩家操作一步一步按部就班的获取分数；第二个方案偷天换日，一步当一千步使；那么接下来的方案就显得十分无耻了。

```js
Runner.instance_.gameOver=function(){};
```

下面代码在执行之后，会清空游戏的中断逻辑，配合第二个方案的快速获得分数的代码，你可以得到一只拥有穿越障碍物能力的小恐龙：勇往无前，永不停歇，分数不停的增长，直到报错。


## 其他

相关代码我推到了 [GitHub](https://github.com/soulteary/trex-runner-bot/tree/master/src) 上，有兴趣的同学也可以访问，以了解更多内容。

先写到这里，去看苹果发布会了。

--EOF