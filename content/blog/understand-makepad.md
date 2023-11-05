+++
title = '理解Makepad中的布局系统'
date = 2023-11-05T22:23:31+08:00
draft = false
categories = ["Rust","GUI"]
+++

# 背景知识说明

当我尝试学习一个新的GUI框架的时候，我首先会关注其如何进行页面布局与多文件构建，这篇简单的文章主要与大家一起完成布局系统的学习。
当我们在讨论一个布局系统的时候，我们在讨论什么东西？
* 如何描述不同的组件之间的相对（绝对）位置约束？
* 如何描述组件自身的位置属性？
* 如何描述子组件？
![image](/images/2023-11-06-makepad-box-model.png)

上面的图片描述了一个UI框架应该具备的功能：

1. 通过margin描述外部边距
2. 通过border描述边框
3. 通过padding描述内边距

以上三点也是盒模型基本的功能，除了盒模型，另外一个重要的话题是组件的布局。所谓的布局就是描述组件之间的位置关系，通过相对位置或者绝对位置
描述不同的组件应该怎么排列。

为什么布局系统非常的重要？我们可以假设我们有一个框架，没有任何的布局方面的特性，那么我们的代码看上去就像是

```bash
ComponentA {x:0,y:100}
ComponentB {x:50,y:0}
ComponentC {x:200,y:200}
```
这样其实就是绝对布局，然后的子元素全部相对父元素进行布局，这样会造成应用编写异常的困难，也不具备跨设备的适配性。

# Makepad 中的盒模型

这一部分其实没有什么特别要说明，我们直接上代码即可
```rust
live_design! {
<View> {
        width: Fill,
        height:60,
        padding: {
            top: 20,
            left:0,
            right:0,
            bottom:20
        },
        margin: {
            left:20
            right:20
            top:20,
            bottom:20
        },
        show_bg: true,
        show_bg: true,
                flow: Down,
         draw_bg: {
            color: #222222,
            radius: vec2(5, 4.5)
        }
    }
}
```
上面的代码将得到下面的效果，这里与HTML基本一样，因此盒模型部分与主流框架基本一样，不会有任何理解上的困难。
![image](/images/2023-11-06-makepad-e1.jpg)

这里我们需要注意的是，上面没有讨论边框，尤其是常见的边框圆角，那么在makepad中应该如何实现圆角嘞？

```rust
live_design! {
     <RoundedAllView> {
        width: Fill,
        height:60,
        padding: {
            top: 20,
            left:0,
            right:0,
            bottom:20
        },
        margin: {
            left:20
            right:20
            top:20,
            bottom:20
        },
        show_bg: true,
        show_bg: true,
                flow: Down,
         draw_bg: {
            instance radius: vec4(10.0,20.0,30.0,40.0)
            color: #222222
        }
    }
}
```
![image](/images/2023-11-06-makepad-e2.jpg)

这样，我们就得到了一个自定义四个角的圆角矩形，这里我们要注意：

* 组件名称从`View`切换到了 `RoundedAllView`
* 在draw_bg属性中增加了 ` instance radius: vec4(10.0,20.0,30.0,40.0)` 这一行代码

与HTML比较的话，看上去更加的繁琐，一件反直觉的事情背后总是会有这样那样的原因，我们来讨论一下为什么不直接在`View`属性中
增加`radius`参数嘞？

我们先直接来看实现代码

在`makepad_widgets/src/base.rs`中

```rust
live_design! {
    View = <ViewBase> {}
}
```

```rust
live_design! {
    RoundedAllView = <ViewBase> {show_bg: true, draw_bg: {
        instance border_width: 0.0
        instance border_color: #0000
        instance inset: vec4(0.0, 0.0, 0.0, 0.0)
        instance radius: vec4(2.5, 2.5, 2.5, 2.5)
        
        fn get_color(self) -> vec4 {
            return self.color
        }
        
        fn get_border_color(self) -> vec4 {
            return self.border_color
        }
        
        fn pixel(self) -> vec4 {
            let sdf = Sdf2d::viewport(self.pos * self.rect_size)
            sdf.box_all(
                self.inset.x + self.border_width,
                self.inset.y + self.border_width,
                self.rect_size.x - (self.inset.x + self.inset.z + self.border_width * 2.0),
                self.rect_size.y - (self.inset.y + self.inset.w + self.border_width * 2.0),
                self.radius.x,
                self.radius.y,
                self.radius.z,
                self.radius.w
            )
            sdf.fill_keep(self.get_color())
            if self.border_width > 0.0 {
                sdf.stroke(self.get_border_color(), self.border_width)
            }
            return sdf.result;
        }
    }}
}
```

我们先跳过具体代码，就可以发现，`View` 与 `RoundedAllView` 中代码量就有本质的差异，通常情况下下，代码量也意味着运行成本，所以，实现圆角是具有成本的，
我们不应该在不需要圆角的地方为了方面使用支持圆角或其他没有用到的高级功能属性。

# 理解布局系统
大家先花30秒时间猜一下下面的代码会如何展示？
```rust
live_deisign! {
    <View> {
        width: Fill,
        height:600,
        show_bg: true,
         draw_bg: {
            color: #222222
        }
        <View> {
            width: 200
            height: 200
            show_bg: true
            draw_bg : {
                color: #999999
            }
        }

        <View> {
             width: 200
            height: 200
            show_bg: true
            draw_bg : {
                color: #aaaaaa
            }
        }
        <View> {
             width: 200
            height: 200
            show_bg: true
            draw_bg : {
                color: #eeeeee
            }
        }
    }
}
```

3... <br>
2.... <br>
1... <br>

![image](/images/2023-11-06-makepad-e3.jpg)

结果和大家想象中一致吗？ 我们可以看到不同元素从上大下一次排列，如果我们想要横向排列嘞？

![image](/images/2023-11-06-makepad-e4.jpg)

```rust
live_deisign! {
    <View> {
        width: Fill,
        height:600,
        show_bg: true,
        flow: Right
         draw_bg: {
            color: #222222
        }
        <View> {
            width: 200
            height: 200
            show_bg: true
            draw_bg : {
                color: #999999
            }
        }

        <View> {
             width: 200
            height: 200
            show_bg: true
            draw_bg : {
                color: #aaaaaa
            }
        }
        <View> {
             width: 200
            height: 200
            show_bg: true
            draw_bg : {
                color: #eeeeee
            }
        }
    }
}
```

我们只需要增加 `flow:Right` 这一行代码即可实现横向排列。 `flow` 属性支持 `Right` `Down` `Overlay` 3个属性。最后我们通过一个稍微复杂的例子
展现我们的布局代码。

```rust
live_design! {
    <View> {
        width: Fill,
        height:600,
        show_bg: true,
        flow: Right
         draw_bg: {
            color: #222222
        }
        <View> {
            width: 200
            height: 200
            flow: Right
            <View> {
            width: 100
            height: 200
            show_bg: true
            draw_bg : {
                color: #ccc999
            }
        }
            <View> {
            width: 100
            height: 200
            show_bg: true
            draw_bg : {
                color: #567acd
            }
        }
        }

       <View> {
            width: 200
            height: 200
            flow: Down
            <View> {
            width: 200
            height: 100
            show_bg: true
            draw_bg : {
                color: #345ade
            }
        }
            <View> {
            width: 200
            height: 100
            show_bg: true
            draw_bg : {
                color: #3e4f5a
            }
        }
        }
        <View> {
             width: 200
            height: 200
            flow: Overlay
            <View> {
            width: 200
            height: 200
            show_bg: true
            draw_bg : {
                color: #2a1d3c
            }
        }
            <View> {
            width: 100
            height: 100
            show_bg: true
            draw_bg : {
                color: #c1d2a3
            }
        }
        }
        }
}
```
![image](/images/2023-11-06-makepad-e5.jpg)
# 后记
从目前的实现中来看，并没有直接实现布局相关的既有组件，而是直接提供 `Layout` / `Walk` / `Flow` 3种能力实现相关功能。大家参考源码可以从
`makepad_widgets/src/base.rs` 与 `makepad/draw/src/trtle.rs` 文件入手理解组件的功能与属性。


















