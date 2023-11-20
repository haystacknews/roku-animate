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