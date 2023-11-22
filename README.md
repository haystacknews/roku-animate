# Animate for Roku

`animate` is a library that allows you to programmatically define SceneGraph animations through a simple
configuration object.

> *Translate the node with id `rectId` +200px (to the right) while scaling it to x1.5 and rotating it
120 degrees, for 1 second.*

```brs
m.animation = animate.create({
    targets: "rectId",
    translateX: 200,
    duration: 1,
    scale: 1.5,
    rotation: "120deg",
    autoplay: true
})
```

> Note: The resulting animation must live in the `m` scope or in the `m.top` scope to execute.

# Documentation

## Defining targets

Targets are the nodes that the animation will be applied to. Multiple targets can be defined for the
same animation.

```brs
' By string ID
m.animation = animate.create({
    targets: "rectId",
    translateX: 200
})

' Multiple IDs, separated by spaces
m.animation = animate.create({
    targets: "rect1 rect2",
    translateX: 200
})

' Direct reference to the node
targetNode = m.top.findNode("rect1")
m.animation = animate.create({
    targets: targetNode,
    translateX: 200
})

' Array of references
m.animation = animate.create({
    targets: m.top.findNode("container").getChildren(-1, 0), ' will return the array of all children
    translateX: 200
})
```

> When passing direct node references as targets, `animate` will insert IDs to the nodes if they don't
have them already. This is because the interpolator nodes require an ID to perform the animation.

## Parameters

### Animation and interpolator parameters

The following fields from [`Animation`](https://developer.roku.com/docs/references/scenegraph/animation-nodes/animation.md) and [`AnimationBase`](https://developer.roku.com/docs/references/scenegraph/abstract-nodes/animationbase.md) can be included directly in the configuration
object:

- `repeat`: Controls whether the animation stops when it finishes (false) or repeats from the beginning (true).
- `delay`: Delays the start of the animation by the specified number of seconds.
- `duration`: Sets the duration of the animation in seconds. The default is `1`.
- `easeFunction`: Specifies the interpolator function to be used for the animation. See the documentation for the `Animation` node for more details.
- `easeInPercent`
- `easeOutPercent`
- `optional`: Set to true to skip animations on lower performing Roku devices.

The following fields from `*FieldInterpolator` nodes can be also included:

- `fraction`: Specifies the percentage to be used to compute a value for the field.
- `reverse`: Enables animation to be played in reverse.

```brs
m.animation = animate.create({
    targets: "nodeId",
    translateX: 200,
    repeat: true,
    delay: 3,
    duration: 1,
    easeFunction: "linear",
    optional: true,
    reverse: false
})
```

### Node parameters

Any numeric parameter from a `Node` will be animated with a `FloatFieldInterpolator`. Parameters that
represent an array with two elements will be animated with a `Vector2DFieldInterpolator`. Parameters
named `color` or `blendColor` will be animated with a `ColorFieldInterpolator`. Any other parameters,
like strings, will be ignored.

```brs
m.animation = animate.create({
    target: "myRectangleId",
    opacity: 0.5, ' Will go from the current opacity to `0.5` using a `FloatFieldInterpolator`
    translation: [100, 400], ' Will go from the current translation to `[100, 400]` using a `Vector2DFieldInterpolator`
    color: "#ffffff" ' Will go from the current color to white using a `ColorFieldInterpolator`
})
```

### Special interpolator parameters

The parameter `x` will move the `Node` from it's current (x, y) to the absolute `x'` specified.

```brs
m.animation = animate({
    targets: "nodeId",
    x: 300 ' Will move the node from the current `(x, y)` to (`300`, y)
})
```

The parameter `y` works similarly.

```brs
m.animation = animate({
    targets: "nodeId",
    y: 500 ' Will move the node from the current `(x, y)` to `(x, 500)`
})
```

The parameters `translateX` and `translateY` will move the `Node` from the current `(x, y)` to `(x + x', y + y')`

```brs
m.animation = animate({
    targets: "nodeId",
    translateX: 200 ' Will move the node from (x, y) to (x + 200, y)
})
```

### Special interpolator values

The `rotation` parameter of a `Node` is usually defined in radians, but can be defined in degrees using
a string.

```brs
m.animation = animate.create({
    target: "nodeId",
    translateX: 200,
    rotation: "120deg" ' The string must contain a number and end in `"deg"`.
})
```

The `scale` parameter is usually defined as a `Vector2D`, but can be specified as a single number to
represent both the horizontal and vertical scale.

```brs
m.animation = animate.create({
    target: "nodeId",
    scale: 2 ' Will be scaled from `[currentXScale, currentYScale]` to `[2, 2]`
})
```

### Specific parameters per field

Each of the `Node` fields you wish to animate can have different animation and interpolator parameters.
The actual value must be included in the `value` key. Parameters defined at the root of the configuration
object will be inherited by the field-specific configuration, but can be overriden by them too.

```brs
m.animation = animate.create({
    targets: "nodeId",
    color: "#FAFA33",
    delay: 0.25,
    scale: {
        value: 2,
        duration: 1.6,
        ' all the other animations will inherit delay = 0.25
        ' except this one
        delay: 0.8,
        easeFunction: "inOutQuartic"
    },
    translateX: {
        value: 450,
        duration: 0.8,
        easeFunction: "linear"
    },
    rotation: {
        value: "-360deg",
        duration: 1.8
    }
})
```

### The `direction` parameter

The parameter `direction` can be used to reverse or alternate animations.

```brs
m.animation1 = animate.create({
    targets: "nodeId",
    translateX: 200,
    ' The default value, the node will be translated to `(x + 200, y)`.
    direction: "normal"
})

m.animation2 = animate.create({
    targets: "nodeId",
    translateX: 200,
    ' The animation will start at `(x + 200, y)` and will move the node to `(x, y)`
    direction: "reverse"
})

m.animation3 = animate.create({
    targets: "nodeId",
    translateX: 200,
    ' The animation will start at `(x, y)`, then go to `(x + 200, y)`, then go back to `(x, y)`
    direction: "alternate",
    ' Adding a `repeat` parameter will make the node go forwards and backwards forever
    repeat: true 
})
```

## Function based parameters

Any value for a node field, animation or interpolator parameter can be defined as a function of the current
target reference (`t`), the current target index `(i)` and the number of targets `(l)`.

```brs
m.animation = animate.create({
    targets: "rect1 rect2 rect3",
    translateX: function(t, i, l),
        ' Move each target +20px further to the right than the previous one
        return (i + 1) * 20
    end function,
    repeat: function(t, i, l)
        ' Only repeat the even targets
        return i mod 2 = 0
    end function,
    delay: function(t, i, l)
        ' Increase the delay one second for each target
        return i + 1
    end function,

})
```

The only exception to this is the `easeFunction` parameter, which can be implemented as a function
of a frame `(t)`. There are 60 frames in one second of `duration`.

```brs
m.animation = animate.create({
    target: "nodeId",
    translateX: 200,
    easeFunction: function(t)
        ' Accelerate resembling a sinusoidal curve (ease-in sine).
        radiansFactor = 0.01745329
        return -1 * cos(t * radiansFactor) + 1
    end function
})
```

> The value of `easeFunction` can also be a string, as defined in the [`Animation`](https://developer.roku.com/docs/references/scenegraph/animation-nodes/animation.md) docs.

### Penner functions

The `animate.penner` namespace contains some of Robert Penner's easing functions. See https://easings.net/
for more details.

The `*Sine` functions create animations that accelerate or decelerate according to a sinusoidal curve.

- `easeInSine`
- `easeOutSine`
- `easeInOutSine`
- `easeOutInSine`

The `*Circ` functions create an animation that accelerates and decelerates respectively in a manner
that resembles a quarter of a circle.

- `easeInCirc`
- `easeOutCirc`
- `easeInOutCirc`
- `easeOutInCirc`

The `*Elastic` functions create animations that overshoot the final state and then oscillate around it
before settling, creating an elastic effect.

- `easeInElastic`
- `easeOutElastic`
- `easeInOutElastic`
- `easeOutInElastic`

The `*Back` functions create an animation that overshoots the final state and then comes back.
This creates a "pullback" or "draw back" effect at the beginning or end of the animation, respectively.

- `easeInBack`
- `easeOutBack`
- `easeInOutBack`
- `easeOutInBack`

The `*Bounce` functions create animations that mimic a bouncing effect.

- `easeInBounce`
- `easeOutBounce`
- `easeInOutBounce`
- `easeOutInBounce`

## Timelines

Timelines can define a parallel or sequential flow of animations. The animation and interpolator parameters
from the first configuration object will be inherited by the subsequent steps, and can be overriden. The
exception to this is the `repeat` parameter, which can only be included once.

```brs
' Parallel timeline where one rectangle goes right and the other up, both at the same time.
m.timeline = animate.timeline({
    ' Animation and interpolator params
    duration: 2,
    repeat: true
}).add({
    ' Configuration for first target
    targets: "rect1",
    translateX: 200
}).add({
    ' Configuration for second target
    targets: "rect2",
    translateY: -200,
}).getAnimation()

m.timeline.control = "start"
```

> Don't forget to call `getAnimation()` at the end of the chain of `add()` calls.

A sequential timeline can be implemented by setting the `sequential` parameter to `true`.

```brs
' Sequential timeline where the node follows the path of a rectangle (move right, down, left, up)
m.turtle = animate.timeline({
    targets: "rect1",
    duration: 2
}).add({
    translateX: 200
}).add({
    translateY: 200
}).add({
    translateX: -200
}).add({
    translateY: 200
}).getAnimation()

m.turtle.control = "start"
```

# Contribute

You can see our issues list for things that need to be implemented. For any questions ping me ([@arturocuya](https://github.com/arturocuya)) on the Roku Developers [Slack channel](https://join.slack.com/t/rokudevelopers/shared_invite/zt-1fwucni9w-WizAcwB_rhL8nTctkO7trQ).
