---
title: "C++"
subcategory: "GDNative"
---

## Defining classes without `using namespace godot`

The `GODOT_CLASS` macro needs to use the raw name of the Godot built-in baseclass or it will fail to generate. E.g.,
```c++

class MyClass : public godot::Reference {
  GODOT_CLASS(MyClass, Reference);

  ...
};
```

## Function '_init' hides a non-virtual function from class 'Object'

It is important to overload `_init` regardless of this warning, otherwise Godot will crash when the object is initialized. (Presumably unless the class directly subclasses `Object`).

## Singleton support

As of Godot 3.3 with `godot-cpp` at `1637975` there does not appear to be support for creating singleton classes in GDNative (though they can be made via native modules). As a workaround I've just created the singleton on the GDScript side.
