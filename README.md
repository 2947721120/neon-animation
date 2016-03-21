# neon-animation

`neon-animation` 是一套元素和行为来实现可插入动画过渡的聚合物元件使用 [Web Animations](https://w3c.github.io/web-animations/).

*警告：原料药可能会改变。*

* [一个基本的动画元素](#basic)
* [动画配置](#configuration)
  * [动画类型](#configuration-types)
  * [配置属性](#configuration-properties)
  * [使用多个动画](#configuration-multiple)
  * [运行动画封装在子节点](#configuration-encapsulation)
* [页面转换](#page-transitions)
  * [共享元素的动画](#shared-element)
  * [声明页面过渡](#declarative-page)
* [包含的动画](#animations)
* [演示](#demos)

<a name="basic"></a>
## 一个基本的动画元素

可动画元素应实施 `Polymer.NeonAnimatableBehavior` behavior, or `Polymer.NeonAnimationRunnerBehavior` 如果他们也负责运行动画。

```js
Polymer({
  is: 'my-animatable',
  behaviors: [
    Polymer.NeonAnimationRunnerBehavior
  ],
  properties: {
    animationConfig: {
      value: function() {
        return {
          //霓虹灯动画/动画/比例缩小，animation.html提供
          name: 'scale-down-animation',
          node: this
        }
      }
    }
  },
  listeners: {
    // 当动画结束这个事件被触发
    'neon-animation-finish': '_onNeonAnimationFinish'
  },
  animate: function() {
    // 运行尺度缩小的动画
    this.playAnimation();
  },
  _onNeonAnimationFinish: function() {
    console.log('animation done!');
  }
});
```

[Live demo](http://morethanreal.github.io/neon-animation-demo/bower_components/neon-animation/demo/doc/basic.html)

<a name="configuration"></a>
## 动画配置

<a name="configuration-types"></a>
### 动画类型

一个元素可以运行不同的动画，比如它可能会做一些不同的事情，当它进入和退出时的视野。你可以设置 `animationConfig` 财产从一个动画式地图配置.

```js
Polymer({
  is: 'my-dialog',
  behaviors: [
    Polymer.NeonAnimationRunnerBehavior
  ],
  properties: {
    opened: {
      type: Boolean
    },
    animationConfig: {
      value: function() {
        return {
          'entry': {
            // 霓虹灯动画/动画/扩大规模，animation.html提供provided by neon-animation/animations/scale-up-animation.html
            name: 'scale-up-animation',
            node: this
          },
          'exit': {
            // 霓虹灯动画/动画/淡出，animation.html提供provided by neon-animation/animations/fade-out-animation.html
            name: 'fade-out-animation',
            node: this
          }
        }
      }
    }
  },
  listeners: {
    'neon-animation-finish': '_onNeonAnimationFinish'
  },
  show: function() {
    this.opened = true;
    this.style.display = 'inline-block';
    // 运行放大动画run scale-up-animation
    this.playAnimation('entry');
  },
  hide: function() {
    this.opened = false;
    // run fade-out-animation
    this.playAnimation('exit');
  },
  _onNeonAnimationFinish: function() {
    if (!this.opened) {
      this.style.display = 'none';
    }
  }
});
```

[Live demo](http://morethanreal.github.io/neon-animation-demo/bower_components/neon-animation/demo/doc/types.html)

你也可以使用方便的特性 `entryAnimation` 和 `exitAnimation` 到开始 `entry` 和离开 `exit`动画:

```js
properties: {
  entryAnimation: {
    value: 'scale-up-animation'
  },
  exitAnimation: {
    value: 'fade-out-animation'
  }
}
```

<a name="configuration-properties"></a>
### 配置属性

你可以通过额外的参数到动画中的配置对象设置动画，动画应该接受以下属性:

 * `name`: 一个动画的名字，即一个单元实施 `Polymer.NeonAnimationBehavior`.
 * `node`: 运用动画的目标节点。默认值为 `this`.
 * `timing`: 定时性能：采用动画。他们比赛 [Web Animations Animation Effect Timing interface](https://w3c.github.io/web-animations/#the-animationeffecttiming-interface). 它们包括以下：:
     * `duration`: T动画的毫秒时间
     * `delay`: 在毫秒动画的启动延迟。.
     * `easing`: 对于动画的计时功能。与CSS定时功能的价值。.

动画可以定义附加的配置和他们的文件列。

<a name="configuration-multiple"></a>
### 使用多个动画

设置动画配置阵列结合动画，像这样:

```js
animationConfig: {
  value: function() {
    return {
      // fade-in-animation is run with a 50ms delay from slide-down-animation
      'entry': [{
        name: 'slide-down-animation',
        node: this
      }, {
        name: 'fade-in-animation',
        node: this,
        timing: {delay: 50}
      }]
    }
  }
}
```

<a name="configuration-encapsulation"></a>
### 封装在子节点运行动画

你可以在被封装在一个孩子的元素，实现配置动画 `Polymer.NeonAnimatableBehavior` 与 `animatable` 财产.

```js
animationConfig: {
  value: function() {
    return {
      // run fade-in-animation on this, and the entry animation on this.$.myAnimatable
      'entry': [
        {name: 'fade-in-animation', node: this},
        {animatable: this.$.myAnimatable, type: 'entry'}
      ]
    }
  }
}
```

<a name="page-transitions"></a>
## 页面转换

*The artist formerly known as `<core-animated-pages>`*

The `neon-animated-pages` 元管理一组页面之间切换，运行动画页面之间的转换。它实现了 `Polymer.IronSelectableBehavior` 行为。每个子节点应实施 `Polymer.NeonAnimatableBehavior` 定义入口 `entry` 和 `exit` 动画一个页面的过渡期间， `entry` 动画运行的新的一页， `exit` 动画运行旧的页面.

<a name="shared-element"></a>
### 共享元素的动画

共享元素的动画作品在多个节点。例如，一个 "hero" 是一个动画的页面过渡到从单独的页面出现两元素的动画作为一个单一的元素中使用。共享元素动画配置有 `id` 属性，确定它们属于同一个动画。含有共同的元素也有一个 `sharedElements` 属性定义的映射 `id` 至元，与动画相关的元素。.

在输入页面：

```js
properties: {
  animationConfig: {
    value: function() {
      return {
        // the incoming page defines the 'entry' animation
        'entry': {
          name: 'hero-animation',
          id: 'hero',
          toPage: this
        }
      }
    }
  },
  sharedElements: {
    value: function() {
      return {
        'hero': this.$.hero
      }
    }
  }
}
```

在输出页面：

```js
properties: {
  animationConfig: {
    value: function() {
      return {
        // the outgoing page defines the 'exit' animation
        'exit': {
          name: 'hero-animation',
          id: 'hero',
          fromPage: this
        }
      }
    }
  },
  sharedElements: {
    value: function() {
      return {
        'hero': this.$.otherHero
      }
    }
  }
}
```

<a name="declarative-page"></a>
### 声明的网页过渡 `entry-animation` 和 `exit-animation` 退出动画属性 `<neon-animated-pages>`, 这些动画将适用于所有网页过渡。

例如：

```js
<neon-animated-pages id="pages" class="flex" selected="[[selected]]" entry-animation="slide-from-right-animation" exit-animation="slide-left-animation">
  <neon-animatable>1</neon-animatable>
  <neon-animatable>2</neon-animatable>
  <neon-animatable>3</neon-animatable>
  <neon-animatable>4</neon-animatable>
  <neon-animatable>5</neon-animatable>
</neon-animated-pages>
```

The new page will slide in from the right, and the old page slide away to the left.

<a name="animations"></a>
## Included animations

Single element animations:

 * `fade-in-animation` Animates opacity from `0` to `1`.
 * `fade-out-animation` Animates opacity from `1` to `0`.
 * `scale-down-animation` Animates transform from `scale(1)` to `scale(0)`.
 * `scale-up-animation` Animates transform from `scale(0)` to `scale(1)`.
 * `slide-down-animation` Animates transform from `translateY(-100%)` to `none`.
 * `slide-up-animation` Animates transform from `none` to `translateY(-100%)`.
 * `slide-left-animation` Animates transform from `none` to `translateX(-100%)`;
 * `slide-right-animation` Animates transform from `none` to `translateX(100%)`;
 * `slide-from-left-animation` Animates transform from `translateX(-100%)` to `none`;
 * `slide-from-right-animation` Animates transform from `translateX(100%)` to `none`;
 * `transform-animation` Animates a custom transform.

Note that there is a restriction that only one transform animation can be applied on the same element at a time. Use the custom `transform-animation` to combine transform properties.

Shared element animations

 * `hero-animation` Animates an element such that it looks like it scales and transforms from another element.
 * `ripple-animation` Animates an element to full screen such that it looks like it ripples from another element.

Group animations
 * `cascaded-animation` Applys an animation to an array of elements with a delay between each.

<a name="demos"></a>
## Demos

 * [Grid to full screen](http://morethanreal.github.io/neon-animation-demo/bower_components/neon-animation/demo/grid/index.html)
 * [Animation on load](http://morethanreal.github.io/neon-animation-demo/bower_components/neon-animation/demo/load/index.html)
 * [List item to detail](http://morethanreal.github.io/neon-animation-demo/bower_components/neon-animation/demo/list/index.html) (For narrow width)
 * [Dots to squares](http://morethanreal.github.io/neon-animation-demo/bower_components/neon-animation/demo/tiles/index.html)
 * [Declarative](http://morethanreal.github.io/neon-animation-demo/bower_components/neon-animation/demo/declarative/index.html)
