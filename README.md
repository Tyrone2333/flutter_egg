# flutter_egg

像安卓彩蛋一样的隐藏功能,通过多次点击可触发彩蛋

![d8bqNn.gif](https://upload-images.jianshu.io/upload_images/13417663-35919da410b57b2a.gif?imageMogr2/auto-orient/strip)


# 使用
```
            Egg(
              neededNum: 5,
              onTap: (int tapNum, int neededNum) {
                if (tapNum > 1 && tapNum < 5) {
                  $warn('再按 ${neededNum - tapNum}次');
                }
              },
              onTrigger: (int tapNum, int neededNum) {
                $warn('yo~靓仔你发现了隐藏功能');
              },
              child: Text(
                'You have pushed the\n button this many times:',
              ),
            )
```


# Getting Started

写项目的时候后端同学经常要我打一个测试包,但是又经常需要切换到线上地址查看效果,每次打个包要等几分钟实在太麻烦了.大家都知道安卓彩蛋是通过连续点击版本号来触发,于是便想做个像安卓彩蛋一样的隐藏功能,用来打开开发模式.

首先搭建简单的框架,可以点击,渲染 `child`
```
import 'package:flutter/material.dart';

class Egg extends StatefulWidget {
  final Widget child;

  Egg({
    this.child,
  });

  @override
  _EggState createState() => _EggState();
}

class _EggState extends State<Egg> {
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: handleOnTap,
      child: widget.child,
    );
  }

  void handleOnTap() {
    print(111);
  }
}

```

我们的组件想通用就不能写死数据,所以给类加上一些参数
```
class Egg extends StatefulWidget {
  final Widget child;

  /// 总共需要点击的次数
  final int neededNum;

  /// 两次点击的间隔
  final Duration interval;

  Egg({
    this.child,
    this.neededNum = 5,
    this.interval = const Duration(seconds: 1),
  });

  @override
  _EggState createState() => _EggState();
}
```

如何记录连续点击呢?只需要一个数组存储每次点击的时刻,通过对比本次和上次点击的时刻是否在容许间隔内,如果是则把本次时刻存起,用于下次对比;如果否就清空旧数据,只存本次点击时刻

下面是处理点击的逻辑.
```
  handleOnTap() {
    DateTime _lastPressedAt = tapTimeList.length > 0 ? tapTimeList.last : null;
    DateTime now = DateTime.now();

    print(_lastPressedAt);

    // 不超过点击间隔,tapTimeList 添加当前点击时刻
    if (_lastPressedAt != null && (now.difference(_lastPressedAt) < widget.interval)) {
      tapTimeList.add(now);
      $warn('再按 ${widget.neededNum - tapTimeList.length}次');
    } else {
      // 两次点击间隔超过 时长限制，重新计时
      print('超过 时长限制');
      tapTimeList.clear();
      tapTimeList.add(now);
    }

    print(tapTimeList);
    print('---------');
  }
```

到这一步我们已经可以实现连续点击,超时重置.

然后加一下外部传入点击回调

```
typedef OnTapCallBack = void Function(int tapNum, int neededNum);

class Egg extends StatefulWidget {
  
  ...
  
  final OnTapCallBack onTrigger;
  final OnTapCallBack onTap;

  Egg({
  
    ...
  
    this.onTrigger,
    this.onTap,
  });

  @override
  _EggState createState() => _EggState();
}
```

完善点击和触发彩蛋的处理
```
  handleOnTap() {
    DateTime lastPressedAt = tapTimeList.length > 0 ? tapTimeList.last : null;
    DateTime now = DateTime.now();

    // 不超过点击间隔,tapTimeList 添加当前点击时刻
    if (lastPressedAt != null && (now.difference(lastPressedAt) < widget.interval)) {
      tapTimeList.add(now);
//      $warn('再按 ${widget.neededNum - tapTimeList.length}次');

      // 到达条件,触发隐藏功能
      if (tapTimeList.length >= widget.neededNum) {
        if (widget.onTrigger != null) {
          widget.onTrigger(tapTimeList.length, widget.neededNum);
        }
        // 清空记录点击列表
        tapTimeList.clear();
      }
    } else {
      // 两次点击间隔超过 时长限制，重新计时
      tapTimeList.clear();
      tapTimeList.add(now);
    }

    // 每次都触发的 tap 事件
    if (widget.onTap != null) {
      widget.onTap(tapTimeList.length, widget.neededNum);
    }
  }
```

这样一个朴实无华的彩蛋组件就完成了

![d8bqNn.gif](https://upload-images.jianshu.io/upload_images/13417663-35919da410b57b2a.gif?imageMogr2/auto-orient/strip)

