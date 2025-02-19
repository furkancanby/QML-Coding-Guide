# Table of Contents

- [Item 1: Code Style](#item-1-code-style)
    - [Signal Handler Ordering](#signal-handler-ordering)
    - [Property Ordering](#property-ordering)
    - [Function Ordering](#function-ordering)
    - [Animations](#animations)
    - [Giving Components `id`s](#giving-components-ids)
    - [Property Assignments](#property-assignments)
    - [Import Statements](#import-statements)
        + [Import Order](#import-order)
    - [Full Example](#full-example)
- [Item 2: Bindings](#item-2-bindings)
    - [Prefer Bindings over Imperative Assignments](#prefer-bindings-over-imperative-assignments)
    - [Making `Connections`](#making-connections)
    - [Use `Binding` Object](#use-binding-object)
        + [Transient Bindings](#transient-bindings)
    - [KISS It](#kiss-it)
    - [Be Lazy](#be-lazy)
    - [Avoid Unnecessary Re-Evaluations](#avoid-unnecessary-re-evaluations)
- [Item 3: C++ Integration](#item-3-c-integration)
    - [Prefer Context Properties for Primitive Data Types](#prefer-context-properties-for-primitive-data-types)
    - [Prefer Singletons Over Context Properties](#prefer-singletons-over-context-properties)
    - [Prefer Instantiated Classes Over Singletons and Context Properties](#prefer-instantiated-classes-over-singletons-and-context-properties)
    - [Watch Out for Object Ownership Rules](#watch-out-for-object-ownership-rules)
- [Item 4: Memory Management](#item-4-memory-management)
    - [Reduce the Number of Implicit Types](#reduce-the-number-of-implicit-types)
- [Item 5: Signal Handling](#item-5-signal-handling)
    - [Try to Avoid Using connect Function in Models](#try-to-avoid-using-connect-function-in-models)
- [Item 6: Javascript](#item-6-javascript)
    - [Use Arrow Functions](#use-arrow-functions)
    - [Use the Modern Way of Declaring Variables](#use-the-modern-way-of-declaring-variables)


# Item 1: Code Style

This section provides details about how to format the order of properties, signals,
and functions to make things easy on the eyes and quickly switch to related code block.

[QML object attributes](https://doc.qt.io/qt-5/qtqml-syntax-objectattributes.html)
are always structured in the following order:

- id
- Property declarations
- Signal declarations
- Object properties
- Attached properties and signal handlers
- States
- Transitions
- Signal handlers
- Child objects
  + Visual Items
  + Qt provided non-visual items
  + Custom non-visual items
- `QtObject` for encapsulating private members[1](https://bugreports.qt.io/browse/QTBUG-11984)
- JavaScript functions

The main purpose for this order is to make sure that the most intrinsic properties of a type is
always the most visible one in order to make the interface easier to digest at a first glance.
Although it could be argued that the JavaScript functions are also part of the interface, the ideal
is to have no functions at all.

## Signal Handler Ordering

When handling the signals attached to an `Item`, make sure to always leave
`Component.onCompleted` to the last line.

```qml
// Wrong
Item {
    Component.onCompleted: {
    }
    onSomethingHappened: {
    }
}

// Correct
Item {
    onSomethingHappened: {
    }
    Component.onCompleted: {
    }
}
```

This is because it mentally makes for a better picture because
`Component.onCompleted` is expected to be fired when the components construction
is complete.

------

If there are multiple signal handlers in an `Item`, then the ones with least amount
of lines may be placed at the top. As the implementation lines increases, the handler
also moves down. The only exception to this is `Component.onCompleted` signal, it
is always placed at the bottom.

```qml
// Wrong
Item {
    onOtherEvent: {
        // Line 1
        // Line 2
        // Line 3
        // Line 4
    }
    onSomethingHappened: {
        // Line 1
        // Line 2
    }
}

// Correct
Item {
    onSomethingHappened: {
        // Line 1
        // Line 2
    }
    onOtherEvent: {
        // Line 1
        // Line 2
        // Line 3
        // Line 4
    }
}
```

## Property Ordering

The first property assignment must always be the `id` of the component. If you
want to declare custom properties for a component, the declarations are always
above the first property assignment.

```qml
// Wrong
Item {
    someProperty: false
    property int otherProperty: -1
    id: myItem
}

// Correct
Item {
    id: myItem

    property int otherProperty: -1

    someProperty: false
}
```

There's also a bit of predefined order for property assignments. The order goes
as follows:

- id
- x
- y
- width
- height
- anchors

The goal here is to put the most obvious and defining properties at the top for
easy access and visibility. For example, for an `Image` you may decide to also
put `sourceSize` above `anchors`.

------

If there are also property assignments along with signal handlers, make sure to
always put property assignments above the signal handlers.

```qml
// Wrong
Item {
    onOtherEvent: {
    }
    someProperty: true
    onSomethingHappened: {
    }
    x: 23
    y: 32
}

// Correct
Item {
    x: 23
    y: 32
    someProperty: true
    onOtherEvent: {
    }
    onSomethingHappened: {
    }
}
```

It is usually harder to see the property assignments If they are mixed with
signal handlers. That's why we are putting the assignments above the signal
handlers.

### Function Ordering

Although there are no private and public functions in QML, you can provide a
similar mechanism by wrapping the properties and functions that are only supposed
to be used internally in `QtObject `.

Public function implementations are always put at the very bottom of the file.

```qml
// Wrong
Item {

    function someFunction() {
    }

    someProperty: true
}

// Correct
Item {
    someProperty: true
    onOtherEvent: {
    }
    onSomethingHappened: {
    }

    function someFunction() {
    }
}
```

### Animations

When using any subclass of `Animation`, especially nested ones like
`SequentialAnimation`, try to reduce the number of properties in one line.
More than 2-3 assignments on the same line becomes harder to reason with after
a while. Or maybe you can keep the one line assignments to whatever line length
convention you have set up for your project.

Since animations are harder to imagine in your mind, you will benefit from
keeping the animations as simple as possible.

```qml
// Bad
NumberAnimation { target: root; property: "opacity"; duration: root.animationDuration; from: 0; to: 1 }

// Depends on your convention. The line does not exceed 80 characters.
PropertyAction { target: root; property: "visible"; value: true }

// Good.
SequentialAnimation {
    PropertyAction {
        target: root
        property: "visible"
        value: true
    }

    NumberAnimation {
        target: root
        property: "opacity"
        duration: root.animationDuration
        from: 0
        to: 1
    }
}
```

### Giving Components `id`s

If a component does not need to be accessed for a functionality, avoid setting
the `id` property. This way you don't clutter the namespace with unused `id`s and
you'll be less likely to run into duplicate `id` problem. Also, having an id for
a type puts additional cognitive stress because it now means that there's
additional relationships that we need to care for.

If you want to mark the type with a descriptor but you don't intend to reference
the type, you can use `objectName` instead old just plain old comments.

Make sure that the top most component in the file always has `root` as its `id`.
Qt will make unqualified name look up deprecated in QML 3, so it's better to
start giving IDs to your components now and use qualified look up.

See [QTBUG-71578](https://bugreports.qt.io/browse/QTBUG-71578) and
[QTBUG-76016](https://bugreports.qt.io/browse/QTBUG-76016) for more details
on this.

### Property Assignments

When assigning grouped properties, always prefer the dot notation If you are only
altering just one property. Otherwise, always use the group notation.

```qml
Image {
    anchors.left: parent.left // Dot notation
    sourceSize { // Group notation
        width: 32
        height: 32
    }
}
```

When you are assigning the component to a `Loader`'s `sourceComponent` in different
places in the same file, consider using the same implementation. For example, in
the following example there are two instances of the same component. If both of
those `SomeSpecialComponent` are meant to be identical it is a better idea to
wrap `SomeSpecialComponent` in a `Component`.

```qml
// BEGIN bad.
Loader {
    id: loaderOne
    sourceComponent: SomeSpecialComponent {
        text: "Some Component"
    }
}

Loader {
    id: loaderTwo
    sourceComponent: SomeSpecialComponent {
        text: "Some Component"
    }
}
// END bad.

// BEGIN good.
Loader {
    id: loaderOne
    sourceComponent: specialComponent
}

Loader {
    id: loaderTwo
    sourceComponent: specialComponent
}

Component {
    id: specialComponent

    SomeSpecialComponent {
        text: "Some Component"
    }
}
// END good.
```

This ensures that whenever you make a change to `specialComponent` it will take
effect in all of the `Loader`s. In the bad example, you would have to duplicate
the same change.

When in a similar situation without the use of `Loader`, you can use inline
components.

```qml
component SomeSpecialComponent: Rectangle {

}
```

### Import Statements

If you are importing a JavaScript file, make sure to not include the same
module in both the QML file and the JavaScript file. JavaScript files share the
imports from the QML file so you can take advantage of that. If the JavaScript
file is meant as a library, this does not apply.

If you are not making use of the imported module in the QML file, consider moving
the import statement to the JavaScript file. But note that once you import something
in the JavaScript file, the imports will no longer be shared. For the complete
rules see [here](https://doc.qt.io/qt-5/qtqml-javascript-imports.html#imports-within-javascript-resources).

Alternatively, you can use `Qt.include()` which copies the contents of the
included file and you will not have to worry about the import sharing rules.

As a general rule, you should avoid having unused import statements.

#### Import Order

When importing other modules, use the following order;

- Qt modules
- Third party modules
- Local C++ module imports
- QML folder imports

### Full Example

```qml
// First Qt imports
import QtQuick 2.15
import QtQuick.Controls 2.15
// Then custom imports
import my.library 1.0

Item {
    id: root

    // ----- Property Declarations

    // Required properties should be at the top.
    required property int radius: 0

    property int radius: 0
    property color borderColor: "blue"

    // ----- Signal declarations

    signal clicked()
    signal doubleClicked()

    // ----- In this section, we group the size and position information together.

    x: 0
    y: 0
    z: 0
    width: 100
    height: 100
    anchors.top: parent.top // If a single assignment, dot notation can be used.
    // If the item is an image, sourceSize is also set here.
    // sourceSize: Qt.size(12, 12)

    // ----- Then comes the other properties. There's no predefined order to these.

    // Do not use empty lines to separate the assignments. Empty lines are reserved
    // for separating type declarations.
    enabled: true
    layer.enabled: true

    // ----- Then attached properties and attached signal handlers.

    Layout.fillWidth: true
    Drag.active: false
    Drag.onActiveChanged: {

    }

    // ----- States and transitions.

    states: [
        State {

        }
    ]
    transitions: [
        Transitions {

        }
    ]

    // ----- Signal handlers

    onWidthChanged: { // Always use curly braces.

    }
    // onCompleted and onDestruction signal handlers are always the last in
    // the order.
    Component.onCompleted: {

    }
    Component.onDestruction: {

    }

    // ----- Visual children.

    Rectangle {
        height: 50
        anchors: { // For multiple assignments, use group notation.
            top: parent.top
            left: parent.left
            right: parent.right
        }
        color: "red"
        layer: {
            enabled: true
            samples: 4
        }
    }

    Rectangle {
        width: parent.width
        height: 1
        color: "green"
    }

    // ----- Qt provided non-visual children

    Timer {

    }

    // ----- Custom non-visual children

    MyCustomNonVisualType {

    }

    QtObject {
        id: privates

        property int diameter: 0
    }

    // ----- JavaScript functions

    function collapse() {

    }

    function setCollapsed(value: bool) {
        if (value === true) {
        }
        else {
        }
    }
}
```

# Item 2: Bindings

Bindings are a powerful tool when used responsibly. Bindings are evaluated
whenever a property it depends on changes and this may result in poor performance
or unexpected behaviors. Even when the binding is simple, its consequence can be
expensive. For instance, a binding can cause the position of an item to change
and every other item that depends on the position of that item or is anchored to
it will also update its position.

So consider the following rules when you are using bindings.

## Prefer Bindings over Imperative Assignments

See the related section on [Qt Documentation](https://doc.qt.io/qt-5/qtquick-bestpractices.html#prefer-declarative-bindings-over-imperative-assignments).

The official documentation explains things well, but it is also important to
understand the performance complications of bindings and understand where the
bottlenecks can be.

If you suspect that the performance issue you are having is related to
excessive evaluations of bindings, then use the QML profiler to confirm your
suspicion and then opt-in to use imperative option.

Refer to the [official documentation](https://doc.qt.io/qtcreator/creator-qml-performance-monitor.html)
on how to use QML profiler.

## Making `Connections`

A `Connections` object is used to handle signals from arbitrary `QObject` derived
classes in QML. One thing to keep in mind when using connections is the default
value of `target` property of the `Connections` is its parent if not explicitly
set to something else. If you are setting the target after dynamically creating
a QML object, you might want to set the `target` to `null` otherwise you might
get signals that are not meant to be handled.

```qml
// Bad
Item {
    id: root
    onSomethingHappened: {
        // Set the target of the Connections.
    }

    Connections {
        // Notice that target is not set so it's implicitly set to root.
        onWidthChanged: {
            // Do something. But since Item also has a width property we may
            // handle the change for root until the target is set explicitly.
        }
    }
}

// Good
Item {
    id: root
    onSomethingHappened: {
        // Set the target of the Connections.
    }

    Connections {
        target: null // Good. Now we won't have the same problem.
        onWidthChanged: {
            // Do something. Only handles the changes for the intended target.
        }
    }
}
```

## Use `Binding` Object

`Binding`'s `when` property can be used to enable or disable a binding expression
depending on a condition. If the binding that you are using is complex and does
not need to be executed every time a property changes, this is a good idea to
reduce the binding execution count.

Using the same example above, we can rewrite it as follows using a `Binding` object.

```qml
Rectangle {
    id: root

    Binding on color {
        when: mouseArea.pressed
        value: mouseArea.pressed ? "red" : "yellow"
    }

    MouseArea {
        id: mouseArea
        anchors.fill: parent
    }
}
```

Again, this is a really simple example to get the point out. In a real life
situation, you would not get more benefit from using `Binding` object in this
case unless the binding expression is expensive (e.g It changes the item's
`anchor` which causes a whole chain reaction and causes other items to be
repositioned.).

`Binding` objects can also be used to provide bidirectional binding for
properties without the risk of breaking the bindings. Consider the following
example:

```qml
Rectangle {
    id: rect
    width: 50
    height: 50
    anchors.centerIn: parent
    color: cd.color

    MouseArea {
        anchors.fill: parent
        acceptedButtons: Qt.LeftButton | Qt.RightButton
        onClicked: {
            if (mouse.button === Qt.LeftButton) {
                cd.visible = true;
            }
            else {
                parent.color = "black"
            }
        }
    }
}

ColorDialog {
    id: cd
    color: rect.color
}
```

The binding to `color` properties of `ColorDialog` and `Rectangle` will be broken
once those proeprties are set from outside. If you play around with the example,
you'll see that `parent.color = "black"` breaks the binding.

Now, see the following example and you'll find that bindings are not broken.

```qml
Rectangle {
    id: rect
    width: 50
    height: 50
    anchors.centerIn: parent
    color: "red"

    Binding on color {
        value: cd.color
    }

    MouseArea {
        anchors.fill: parent
        acceptedButtons: Qt.LeftButton | Qt.RightButton
        onClicked: {
            if (mouse.button === Qt.LeftButton) {
                cd.visible = true;
            }
            else {
                parent.color = "black"
            }
        }
    }
}

ColorDialog {
    id: cd

    Binding on color {
        value: rect.color
    }
}
```

### Transient Bindings

There may be cases where you have to end up using an imperative assignment. But
naturally this will break the binding. In that case, you can create transient
`Binding` objects to safely set the new property without breaking the existing
binding.

```qml
Item {
    property var contentItem

    onContentItemChanged: {
        contentItem.width = 100 // This will break the binding.
        // -----
        const temp = cmpBinding.createObject(root, {
            "target": contentItem,
            "property": "width",
            "value": 100
        })
        // Now the width property is safely updated to 100 without breaking
        // any existing bindings.
        temp.destroy() // Don't forget to destroy it.
    }

    Component {
        id: cmpBinding

        Binding { }
    }
}
```

## KISS It

You are probably already familiar with the [KISS principle](https://en.wikipedia.org/wiki/KISS_principle).
QML supports optimization of binding expressions. Optimized bindings do not require
a JavaScript environment hence it runs faster. The basic requirement for optimization
of bindings is that the type  information of every symbol accessed must be known at
compile time.

So, avoid accessing `var` properties. You can see the full list of prerequisites
of optimized bindings [here](https://doc.qt.io/qt-5/qtquick-performance.html#bindings).

## Be Lazy

There may be cases where you don't need the binding immediately but when a certain
condition is met. By lazily creating a binding, you can avoid unnecessary executions.
To create a binding during runtime, you can use `Qt.binding()`.

```qml
Item {
    property int edgePosition: 0

    Component.onCompleted: {
        if (checkForSomeCondition() == true) {
            // bind to the result of the binding expression passed to Qt.binding()
            edgePosition = Qt.binding(function() { return x + width })
        }
    }
}
```

You can also use `Qt.callLater` to reduce the redundant calls to a function.

## Avoid Unnecessary Re-Evaluations

If you have a loop or process where you update the value of the property, you may
want to use a temporary local variable where you accumulate those changes and only
report the last value to the property. This way you can avoid triggering re-evaluation
of binding expressions during the intermediate stages of accumulation.

Here's a bad example straight from Qt documentation:

```qml
import QtQuick 2.3

Item {
    id: root

    property int accumulatedValue: 0

    width: 200
    height: 200
    Component.onCompleted: {
        const someData = [ 1, 2, 3, 4, 5, 20 ];
        for (let i = 0; i < someData.length; ++i) {
            accumulatedValue = accumulatedValue + someData[i];
        }
    }

    Text {
        anchors.fill: parent
        text: root.accumulatedValue.toString()
        onTextChanged: console.log("text binding re-evaluated")
    }
}
```

And here is the proper way of doing it:

```qml
import QtQuick 2.3

Item {
    id: root

    property int accumulatedValue: 0

    width: 200
    height: 200
    Component.onCompleted: {
        const someData = [ 1, 2, 3, 4, 5, 20 ];
        let temp = accumulatedValue;
        for (let i = 0; i < someData.length; ++i) {
            temp = temp + someData[i];
        }

        accumulatedValue = temp;
    }

    Text {
        anchors.fill: parent
        text: root.accumulatedValue.toString()
        onTextChanged: console.log("text binding re-evaluated")
    }
}
```

# Item 3: C++ Integration

QML can be extended with C++ by exposing the `QObject` classes using the `Q_OBJECT`
macro or custom data types using `Q_GADGET` macro.
It always should be preferred to use C++ to add functionality to a QML application.
But it is important to know which is the best way to expose your C++ classes, and
it depends on your use case.

## Prefer Context Properties for Primitive Data Types

Context properties are registered using

```cpp
rootContext()->setContextProperty("someProperty", QVariant());
```

Context properties always takes in a `QVariant`, which means that whenever you
access the property it is re-evaluated because in between each access the property
may be changed as `setContextProperty()` can be used at any moment in time.

If you are exposing context properties, set them before loading the `main.qml`
file otherwise your UI would be blocked. Doing it before loading the window at
least hides a window freeze from the user.

If you really have to use context properties and you access them repeatedly in
the same scope, consider assigning the context to a `var` so that it is only
evaluated once.

```js
function expensiveOperation() { // Bad
    for (var index in aList) {
        // contextProperty is re-evaluated each time the for loop resets.
        // For long operations, this may significantly affect the performance
        // and block the UI.
        contextProperty.someOperation(aList[index]);
    }
}

function expensiveOperation() { // Less Bad
    var context = contextProperty; // Only evaluated once.
    for (var index in aList) {
        context.someOperation(aList[index]);
    }
}
```

Since the cost of accessing context properties is expensive, calling a method
from a context property is even more expensive. So, keep your context properties
for only primitive types If you really have to use context properties. An example
use case could be adding the macro equivalents for QML code. For example, If you
want to have different behaviors based on the build type, you could do something
like the following.

```cpp
#ifdef QT_DEBUG
    rootContext->setContextProperty("QT_DEBUG", QVariant(true));
#else
    rootContext->setContextProperty("QT_DEBUG", QVariant(false));
#endif
```

And then have different behavior in QML.

```qml
MyItem {
    someProperty: QT_DEBUG ? 32 : 23
}
```

This will not have a significant impact If you have an infrequent use of the context
property or for a small app. But If you are concerned with performance (e.g when
writing a game.), either avoid from context properties or use them for primitive
types, or types that are inexpensive to convert.
See [here](https://doc.qt.io/qt-5/qtqml-cppintegration-data.html#conversion-between-qt-and-javascript-types)
for a list of data types that support conversation and their impact on performance.

Context properties will be deprecated in Qt 6.
See [QTBUG-73064](https://bugreports.qt.io/browse/QTBUG-73064).

## Prefer Singletons Over Context Properties

There are bound to be cases where you have to provide a single instance for a
functionality or common data access. In this situation, resort to using a singleton
as it will have a better performance and be easier to read. Singletons are also
a good option to expose enums to QML.

```cpp
class MySingletonClass : public QObject
{
public:
    static QObject *singletonProvider(QQmlEngine *qmlEngine, QJSEngine *jsEngine)
    {
        if (m_Instance == nullptr) {
            m_Instance = new MySingletonClass(qmlEngine);
        }

        Q_UNUSED(jsEngine);
        return m_Instance;
    }
};

// In main.cpp
qmlRegisterSingletonType<SingletonTest>("MyNameSpace", 1, 0, "MySingletonClass", MySingletonClass::singletonProvider);
```

## Prefer Instantiated Classes Over Singletons and Context Properties

Context properties and singletons are sufficient solution If you are not concerned
about milking every CPU cycle you can. They are good and standard solutions to a
problem and there will be cases where they make more sense. But instantiated classes
in QML outperform both of those significantly. Also, since instantiated classes
have parents, you don't have to dispose them manually. But with context properties
and singletons it will not be the case.

So, analyze your situation and try to stick to the solution that most suits the
problem at hand. Don't over use context properties or singletons or instantiated
classes unwarranted performance concerns.

Go [here](https://github.com/Furkanzmc/QML-Cpp-Access-Speed-Test) to see a
comparison of the three exposing methods compare against each other.

## Watch Out for Object Ownership Rules

When you are exposing data to QML from C++, you are likely to pass around custom
data types as well. It is important to realize the implications of ownership when
you are passing data to QML. Otherwise you might end up scratching your head trying
to figure out why your app crashes.

If you are exposing custom data type, prefer to set the parent of that data to the
C++ class that transmits it to QML. This way, when the C++ class gets destroyed
the custom data type also gets destroyed and you won't have to worry about releasing
memory manually.

There might also be cases where you expose data from a singleton class without a
parent and the data gets destroyed because QML object that receives it will take
ownership and destroy it. And you will end up accessing data that doesn't exist.
Ownership is **not** transferred as the result of a property access. For data
ownership rules see [here](https://doc.qt.io/qt-5/qtqml-cppintegration-data.html#data-ownership).

To learn more about the real life implications of this read [this blog post](https://www.embeddeduse.com/2018/04/02/qml-engine-deletes-c-objects-still-in-use/).

# Item 4: Memory Management

Most applications are not likely to have memory limitations. But in case you are
working on a memory limited hardware or you just really care about memory allocations,
follow these steps to reduce your memory usage.

## Reduce the Number of Implicit Types

If a type defines custom properties, that type becomes an implicit type to the JS
engine and additional type information has to be stored.

```qml
Rectangle { } // Explicit type because it doesn't contain any custom properties

Rectangle {
    // The deceleration of this property makes this Rectangle an implicit type.
    property int meaningOfLife: 42
}
```

You should follow the advice from the [official documentation](http://doc.qt.io/qt-5/qtquick-performance.html#avoid-defining-multiple-identical-implicit-types)
and split the type into its own component If it's used in more than one place.
But sometimes, that might not make sense for your case. If you are using a lot of
custom properties in your QML file, consider wrapping the custom properties of
types in a `QtObject`. Obviously, JS engine will still need to allocate memory
for those types, but you already gain the memory efficiency by avoiding the
implicit types. Additionally, wrapping the properties in a `QtObject` uses less
memory than scattering those properties to different types.

Consider the following example:

```qml
Window {
    Rectangle { id: r1 } // Explicit type. Memory 64b, 1 allocation.

    // Implicit type. Memory 128b, 3 allocations.
    Rectangle { id: r2; property string nameTwo: "" }

    QtObject { // Implicit type. Memory 128b, 3 allocations.
        id: privates
        property string name: ""
    }
}
```

In this example, the introduction of a custom property to added additional 64b
of memory and 2 more allocations. Along with `privates`, memory usage adds up to
256b. The total memory usage is 320b.

You can use the QML profiler to see the allocations and memory usage for each
type. If we change that example to the following, you'll see that both memory
usage and number of allocations are reduced.

```qml
Window {
    Rectangle { id: r1 } // Explicit type. Memory 64b, 1 allocation.

    Rectangle { id: r2 } // Explicit type. Memory 64b, 1 allocation.

    QtObject { // Implicit type. Memory 160b, 4 allocations.
        id: privates

        property string name: ""
        property string nameTwo: ""
    }
}
```

In the second example, total memory usage is 288b. This is really a minute
difference in this context, but as the number of components increase in a
project with memory constrained hardware, it can start to make a difference.

# Item 5: Signal Handling

Signals are a very powerful mechanism in Qt/QML. And the fact that you can
connect to signals from C++ makes it even better. But in some situations, If you
don't handle them correctly you might end up scratching your head.

## Try to Avoid Using connect Function in Models

You can have signals in the QML side, and the C++ side. Here's an example for
both cases.

QML Example.

```qml
// MyButton.qml
import QtQuick.Controls 2.3

Button {
    id: root

    signal rightClicked()
}
```

C++ Example:

```cpp
class MyButton
{
    Q_OBJECT

signals:
    void rightClicked();
};
```

The way you connect to signals is using the syntax

```qml
item.somethingChanged.connect(function() {})
```

When this method is used, you create a function that is connected to the
`somethingChanged` signal.

Consider the following example:

```qml
// MyItem.qml
Item {
    id: root

    property QtObject customObject

    objectName: "my_item_is_alive"
    onCustomObjectChanged: {
        customObject.somethingChanged.connect(() => {
            console.log(root.objectName)
        })
    }
}
```

This is a perfectly legal code. And it would most likely work in most scenarios.
But, if the life time of the `customObject` is not managed in `MyItem`, meaning
if the `customObject` can keep on living when the `MyItem` instance is destroyed,
you run into problems.

The connection is created in the context of `MyItem`, and the function naturally
has access to its enclosing context. So, as long as we have the instance of
`MyItem`, whenever `somethingChanged` is emitted we'd get a log saying
`my_item_is_alive`.

Here's a quote directly from [Qt documentation](https://doc.qt.io/qt-5/qml-qtquick-listview.html):

> Delegates are instantiated as needed and may be destroyed at any time. They
> are parented to `ListView`'s `contentItem`, not to the view itself. State
> should never be stored in a delegate.

So you might be making use of an external object to store state. But what If
`MyItem` is used in a `ListView`, and it went out of view and it was destroyed
by `ListView`?

Let's examine what happens with a more concrete example.

```qml
ApplicationWindow {
    id: root

    property list<QtObject> myObjects: [
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        },
        QtObject {
            signal somethingHappened()
        }
    ]

    width: 640
    height: 480

    ListView {
        anchors {
            top: parent.top
            left: parent.left
            right: parent.right
            bottom: btn.top
        }
        // Low enough we can resize the window to destroy buttons.
        cacheBuffer: 1
        model: root.myObjects.length
        delegate: Button {
            id: self

            readonly property string name: "Button #" + index

            text: "Button " + index
            onClicked: {
                root.myObjects[index].somethingHappened()
            }
            Component.onCompleted: {
                root.myObjects[index].somethingHappened.connect(() => {
                    // When the button is destroyed, this will cause the following
                    // error: TypeError: Type error
                    console.log(self.name)
                })
            }
            Component.onDestruction: {
                console.log("Destroyed #", index)
            }
        }
    }

    Button {
        id: btn
        anchors {
            bottom: parent.bottom
            horizontalCenter: parent.horizontalCenter
        }
        text: "Emit Last Signal"
        onClicked: {
            root.myObjects[root.myObjects.length - 1].somethingHappened()
        }
    }
}
```

In this example, once one of the buttons is destroyed we still have the object
instance. And then object instance still contains the connection we made in
`Component.onCompleted`. So, when we click on `btn`, we get an error:
`TypeError: Type error`. But once we expand the window so that the button is
created again, we don't get that error. That is, we don't get that error for the
newly created button. But the previous connection still exists and still causes
error. But now that a new one is created, we end up with two connections on the
same object.

This is obviously not ideal and should be avoided. But how do you do it?

The simplest and most elegant solution (That I have found) is to simply use a
`Connections` object and handle the signal there. So, If we change the code to
this:

```qml
delegate: Button {
    id: self

    readonly property string name: "Button #" + index

    text: "Button " + index
    onClicked: {
        root.myObjects[index].somethingHappened()
    }

    Connections {
        target: root.myObjects[index]
        onSomethingHappened: {
            console.log(self.name)
        }
    }
}
```

Now, whenever the delegate is destroyed so is the connection. This method can
be used even for multiple objects. You can simply put the `Connections` in a
`Component` and use `createObject` to instantiate it for a specific object.

```qml
Item {
    id: root
    onObjectAdded: {
        cmp.createObject(root, {"target": newObject})
    }

    Component {
        id: cmp

        Connections {
            target: root.myObjects[index]
            onSomethingHappened: {
                console.log(self.name)
            }
        }
    }
}
```

# Item 6: Javascript

It is the prevalent advice that you should avoid using JavaScript as much as possible
in your QML code and have the C++ side handle all the logic. This is a sound advice
and should be followed, but there are cases where you can't avoid having JavaScript
code for your UI. In those cases, follow these guidelines to ensure a good use of
JavaScript in QML.

## Use Arrow Functions

Arrow functions were introduced in ES6. Its syntax is pretty close to C++ lambdas
and they have a pretty neat feature that makes them most comfortable to use
when you are using the `connect()` function to create a binding. If there's no
block within the arrow function, it has an implicit return statement.

Let's compare the arrow function version with the old way.

```qml
Item {
    property int value: -1

    Component.onCompelted: {
        // Arrow function
        root.value = Qt.binding(() => root.someOtherValue)
        // The old way.
        root.value = Qt.binding(function() { return root.someOtherValue })
    }
}
```

The arrow function version is easier on the eyes and cleaner to write.
For more information about arrow functions, head over to the [MDN Blog](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

## Use the Modern Way of Declaring Variables

With ES6, there are 3 ways of delcaring a variable: `var`, `let`, and `const`.

You should leverage `let` and `const` in your codebase and avoid using `var`.
`let` and `const` enables a scope based naming wheras `var` only knows about one
scope.

```qml
Item {
    onClicked: {
        const value = 32;
        let valueTwo = 42;
        {
            // Valid assignment since we are in a different scope.
            const value = 32;
            let valueTwo = 42;
        }
    }
}
```

Much like in C++, prefer using `const` If you don't want the variable to be assigned.
But keep in mind that `const` variables in JavaScript are not immutable. It just
means they can't be reassigned, but their contents can be changed.

```js
const value = 32;
value = 42; // ERROR!

const obj = {value: 32};
obj.value = 42; // Valid.
```

See the MDN posts on [const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)
and [let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)
