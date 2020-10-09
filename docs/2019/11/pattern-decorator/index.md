# 装饰者模式

本文介绍在 go 语言中使用装饰者模式.

<!-- more -->

## 一、前言

Decorator Pattern（装饰模式）

{{< admonition type=note title="维基百科" >}}
This pattern creates a decorator class which wraps the original class and provides additional functionality keeping class methods signature intact.
{{< /admonition >}}

白话文:

这个模式我们需要创建一个新的装饰类,然后装饰类会拥有一个属性是被装饰类,并且会有“一个”方法和需要使用的被装饰类的方法签名完全一致,调用装饰类的方法会执行装饰类内容并调用被装饰类的被装饰方法.

故事:

我买了一本书《go编程思想》,小明也买了一本书《go编程思想》并给它带上了一个黄金封面,两本书内容一摸一样,只是小明的变成金灿灿的土豪版.

## 二、实例

### 2.1 对于某个接口的方法装饰

代码:

```go
package decorator

import (
	"fmt"
	"testing"
)

type Shape interface {
	draw()
}

type Circle struct {
}

func (shape Circle) draw() {
	fmt.Println("Shape: Circle")
}

type Rectangle struct {
}

func (shape Rectangle) draw() {
	fmt.Println("Shape: Rectangle")
}

type ShapeDecorator struct {
	decoratorShape Shape
}

func (shape ShapeDecorator) draw() {
	shape.decoratorShape.draw()
}

type RedShapeDecorator struct {
	ShapeDecorator
}

func NewRedShapeDecorator(s Shape) *RedShapeDecorator {
	d := new(RedShapeDecorator)
	d.decoratorShape = s
	return d
}

func (shape RedShapeDecorator) draw() {
	shape.ShapeDecorator.draw()
	fmt.Println("red")
}

type BlueShapeDecorator struct {
	ShapeDecorator
}

func NewBlueShapeDecorator(s Shape) *BlueShapeDecorator {
	d := new(BlueShapeDecorator)
	d.decoratorShape = s
	return d
}

func (shape BlueShapeDecorator) draw() {
	shape.ShapeDecorator.draw()
	fmt.Println("blue")
}

func TestName(t *testing.T) {
	redShapedDecorator := NewRedShapeDecorator(Circle{})
	redShapedDecorator.draw()

	redShapedDecorator = NewRedShapeDecorator(Rectangle{})
	redShapedDecorator.draw()

	blueShapedDecorator := NewBlueShapeDecorator(Circle{})
	blueShapedDecorator.draw()

	blueShapedDecorator = NewBlueShapeDecorator(Rectangle{})
	blueShapedDecorator.draw()
}
```

输出结果:

```text
Shape: Circle
red
Shape: Rectangle
red
Shape: Circle
blue
Shape: Rectangle
blue
```

### 2.2 直接装饰方法

代码:

```go
type drawFunc func()

func CircleDraw() {
	fmt.Println("Shape: Circle")
}

func RedCircleDraw(d drawFunc) drawFunc {
	return func() {
		d()
		fmt.Println("red")
	}
}

func TestFunc(t *testing.T) {
	var drawFunc drawFunc
	drawFunc = CircleDraw
	drawFunc()

	drawFunc = RedCircleDraw(drawFunc)
	drawFunc()
}
```

输出结果:

```Text
Shape: Circle
Shape: Circle
red
```

## 三、参考

https://www.tutorialspoint.com/design_pattern/decorator_pattern.htm

