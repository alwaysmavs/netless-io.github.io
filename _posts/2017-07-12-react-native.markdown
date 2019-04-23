---
layout:     post
title:      "ReactNative 研究：动画效果"
subtitle:   "ReactNative 动画的 api 使用"
author:     "beings"
header-img: "https://ohuuyffq2.qnssl.com/react-native-navigation.png"
header-mask: 0.3
catalog:    true
tags:
    - React Native
    - 前端
    - 动画效果
---

## 1. 动画的初始值设定

Animated提供了两种类型的值：

- `Animated.Value()` 用于单个值。
- `Animated.ValueXY()` 用于矢量值。

`Animated.Value` 可以绑定到样式属性或其他道具，也可以插值。单个`Animated.Value` 可以驱动任意数量的属性。

常见用法是，在初始化的时候指定好动画的初始值。然后在执行动画的时候再赋予目标值。

## 2. 创建动画的方法

React Native 创建动画主要是使用 `Animated` 的 API，`Animated` 提供了以下三个核心的方法：

1. `Animated.timing()` 
    推动一个值按照一个过渡曲线而随时间变化。 
    Config 参数有以下这些属性：
    
    - `duration`: 动画的持续时间（毫秒）。默认值为500.
    - `easing`: Easing function to define curve。默认值为Easing.inOut(Easing.ease)。
    - `delay`: 开始动画前的延迟时间（毫秒）。默认为0。
    - `useNativeDriver`: 使用原生动画驱动。默认不启用(false)。 
       
2. `Animated.decay()`
    推动一个值以一个初始的速度和一个衰减系数逐渐变为0。
    Config 参数有以下这些属性：
    
    - `velocity`: 初始速度。必填。
    - `deceleration`: 衰减系数。默认值0.997。
    - `useNativeDriver`: 使用原生动画驱动。默认不启用(false)。
    
3. `Animated.spring()`
    这是一个弹簧动画，主要通过张力，阻尼、初始速度等参数定义动画的形式。
    Config 参数有以下这些属性：
    
    - `friction`: 摩擦力。默认是 7
    - `tension`: 张力。默认是 40
    - `speed`: 速度。默认 12
    - `bounciness`: 反弹力。默认 8
    - `useNativeDriver`: 使用原生动画驱动。默认不启用(false)。

样例代码：

``` javascript

constructor(props) {
    super(props);
    this.state = {
        pan: new Animated.Value(0), // 初始化动画的值
    };
}

Animated.timing(                            
   this.state.pan, // 获取初始状态                   
   {
       toValue: 1, // 指定目标状态
   }
).start(); // 开始执行动画
                                   
```

**小结：**

- `Animated.timing()` 和 `Animated.spring()` 是比较常用的创建动画方法。
- `useNativeDriver`: 使用原生动画驱动会提高交互体验。通过使用本地驱动程序，我们在开始动画之前将所有关于动画的内容发送到本机，允许本机代码在UI线程上执行动画，而无需通过每个帧上的桥。一旦动画开始，JS线程可以被阻止而不影响动画。开发者可以通过在动画配置中指定 `useNativeDriver：true` 来使用本机驱动程序。


## 3. 创建动画的方法

通过在动画上调用 `start()` 来启动动画。 `start()` 完成回调，完成动画后将被调用。如果动画正常运行，则完成回调将使用 `{finished：true}` 进行调用。如果动画是完成的，因为在它完成之前调用了 `stop()`（例如因为它被手势或其他动画所打断），那么它将会收到 `{finished：false}`。

样例代码：

``` javascript

constructor(props) {
    super(props);
    this.state = {
        pan: new Animated.Value(0), // 初始化动画的值
    };
}

Animated.timing(                            
   this.state.pan, // 获取初始状态                   
   {
       toValue: 1, // 指定目标状态
   }
).start(
    (e) => {
        // 动画完成之后的回调
        e.finished // 获取动画是否完成的回调
    }
); // 开始执行动画
                                   
```
    
## 4. 组合动画调用动画的方式

除了以上三个创建动画的方法，对于每个独立的方法都有三种调用该动画的方式：

- `Animated.parallel(arrayOfAnimations)` 

    同时开始一个动画数组里的全部动画。默认情况下，如果有任何一个动画停止了，其余的也会被停止。你可以通过 `stopTogether` 选项来改变这个效果。

- `Animated.sequence(arrayOfAnimations)` 
    
    按顺序执行一个动画数组里的动画，等待一个完成后再执行下一个。如果当前的动画被中止，后面的动画则不会继续执行。

- `Animated.stagger(arrayOfAnimations)`
    
    一个动画数组，里面的动画有可能会同时执行（不设置 delay 时间的时候），不过会以指定的延迟来开始。与上述两个动画主要的不同点是 `Animated.Stagger` 的第一个参数，delay 会被应用到每一个动画。

## 5. 自定义动画组件

只有动画组件可以执行动画，这些特殊的组件可以将动画值绑定到属性，并进行有针对性的本机更新。如果每个组件都有动画属性，那么在每个框架上的反应渲染和对帐过程的成本会很高。它们还可以在卸载时进行清理，默认情况下它们是安全的。

- `createAnimatedComponent()` 用此接口创建的组件全部支持动画，但是比较少用。

动画使用上述包装器导出以下可动画组件：

- Animated.Image
- Animated.ScrollView
- Animated.Text
- Animated.View

## 6. 联动多个动画值

- `Animated.add(a: Animated, b: Animated)` 将两个动画值相加计算，得出一个新的动画值。
- `Animated.divide(a: Animated, b: Animated)` 将两个动画值相除计算，得出一个新的动画值。
- `Animated.multiply(a: Animated, b: Animated)` 将两个动画值相乘计算，得出一个新的动画值。
- `Animated.modulo(a: Animated, b: Animated)` 将两个动画值做取模（取余数）计算，得出一个新的动画值。
- `Animated.diffClamp(a, min, max) ` 创建一个限制在2个值之间的新动画值。它使用最后一个值之间的差异，所以即使该值远离边界，它将在值再次越来越近时开始变化。 `(value = clamp（value + diff，min，max))` 这对于滚动事件是有用的，例如，当向上滚动时显示导航栏，并在向下滚动时隐藏它。

## 7. 插值的原理和用法

`interpolate(config: InterpolationConfigType)`

在更新属性之前对值进行插值。譬如：把 0-1 **映射**到 0-10。

**样例代码**

``` javascript

constructor () {
  super()
  this.spinValue = new Animated.Value(0) // 设定初始值
}

componentDidMount () {
  this.spin() // 载入后调用
}

spin () {
  this.spinValue.setValue(0)  // 设置初始值
  Animated.timing(
    this.spinValue, // 获取初始值
    {
      toValue: 1,
      duration: 4000,
      easing: Easing.linear
    }
  ).start(() => this.spin()) // 循环执行
}

render () {
  const spin = this.spinValue.interpolate({
    inputRange: [0, 1],
    outputRange: ['0deg', '360deg']
  }) // 使用插值接口，将 [0,1] 影射到 ['0deg', '360deg']
  return (
    <View style={styles.container}>
      <Animated.Image/>
    </View>
  )
}

```



