# Variants

Follow up on the fundamental building block of Godot's data types.

## Overview

You will often find the type ``Variant`` in Godot source code.   Variants are
Godot's way of passing around certain data types.  They are similar to Swift's
`Any` type, but they can only hold Godot types (most structures and classes
that derive from ``GodotObject``). 

## Creating Variant values
You can create Variants from types that conform to the VariantStorable 
protocol. 

This includes the following types:

* Godot's native types: GString, Vector, Rect, Transform, Plane, Quaternion,
  AABB,  Basis, Projection, Int64, NodePaths, RIDs, Callable, GDictionary, Array
  and PackedArrays. 
* Swift types that SwiftGodot adds convenience conformances for: Bool, Int, String and Float
* Godot's objects: e.g. Node, Area2D
* Your own subclasses of SwiftGodot.Object type.
* Other types that you can manually conform to VariantStorable.

You wrap your data type by calling one of the ``Variant`` constructors, and then
you can pass this variant to Godot functions that expect a ``Variant``.

For example, to pass the value `true`:

```swift
let trueVaraint = Variant (true)
```

You can get a string representation of a variant by calling ``Variant``'s
``Variant/description`` method:

```swift
print (trueVariant.description)
```

## Extracting values from Variants

If you know the kind of return that a variant will return, you can invoke the
failing initializer for that specific type for most structures. Every VariantStorable
will have an `init(_ variant: Variant)` implementation.

For example, this is how you could get a Vector2 from a variant:

```swift
func distance (variant: Variant) -> Float? {
    guard let vector = Vector2 (variant) else {
	return nil
    }
    return vector.length ()
}
```

Notice that this might return `nil`, which would be the case if the variant
contains a different type than the one you were expecting. You can check the
type of the variant by accessing the `.gtype` property of the variant.

## Extracting Godot-derived objects from Variants

Godot-derived objects are slightly different. If you know you have a
``GodotObject`` stored in the variant, you can call the ``Variant/asObject(_:)``
instead.  This is a generic method, so you would invoke it like this:

```swift
func getNode (variant: Variant) -> Node? {
    guard let node = variant.asObject (Node.self)) else {
	  return nil
    }
    return node
}
```

Swift type inference can also be used, so you can avoid specifying the type if
the compiler can infer the type, like here:

```swift
func getNode (variant: Variant) -> Node? {
    var node: Node?
    node = variant.asObject ()
    return node
}
```

The reason to rely on calling the `asObject` method rather than having a
constructor for the type that takes a variant (like the case for the non-object
types) is that the method can ensure that only one instance of your objects is surfaced to Swift. 

## Accessing Array Elements

Some of the variant types contain arrays, either objects, or a particular
packed version of those.   You can access the individual elements of the
those with a convenient subscript provided on the array.

## Calling Variant Methods

It is possible to invoke the built-in variant methods that exist in the Godot
universe by using the `call` method on a Variant.   This is very similar to the
call method on an Object.

For example, you can invoke this generic "size" method on an array to get the
size of an array, regardless of the specific type of array:

```
func printSize (myArray: Variant) {
    switch variant.call(method: "size") {
    case .failure(let err):
        print (err)
        return 0
    case .success(let val):
        return Int (val) ?? 0
    }
}
```