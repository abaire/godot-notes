---
title: "Object interaction"
subcategory: "GDNative"
---

## Godot Object interaction from GDNative (in C)

As of March 2021 the documentation for GDNative use with Godot 3.2 is fairly sparse. In particular, the sample application does not perform any interaction with Godot objects. There is some information to be found in issues (e.g., [#51](https://github.com/godotengine/godot-headers/issues/51) which explains the difference between `ptrcall` and `call`).

### Object APIs

Godot Object APIs are exposed via the `api.json` file contained in the [`godot-headers`](https://github.com/godotengine/godot-headers) repository. This file contains all of the exported types along with important metadata (e.g., whether a type is a singleton, whether a method has default values, etc...).

### Constructing a GDNative Object

The API exposes `godot_get_class_constructor` which can be used to construct a new instance of a type (it also exposes `godot_global_get_singleton` to get access to an existing singleton type).

```C
  // Constructs a new instance of SurfaceTool.
  godot_class_constructor base_constructor = api->godot_get_class_constructor("SurfaceTool");
  godot_object *st = base_constructor();
```

### Destructing an Object

`godot_object_destroy` can be used to destroy a created Object.

```C
  api->godot_object_destroy(st);
  st = NULL;  
```

### Calling methods on an Object

Methods are accessed via `godot_method_bind` variables populated via `godot_method_bind_get_method`. These bind variables are not tied to a specific instance of the Object, so they can be declared as static variables that are populated once and reused as desired.

```C
  static godot_method_bind *surfacetool_add_vertex = NULL;
  if (!surfacetool_add_vertex) {
    surfacetool_add_vertex = api->godot_method_bind_get_method("SurfaceTool", "add_vertex");
  }
```

#### Calling with `ptrcall` (fast, unsafe, no default parameter values)

`godot_method_bind_ptrcall` allows a previously bound `godot_method_bind` to be invoked on some `godot_object`. This call has less overhead than the `call` variant (see below) but requires careful examination of the API of the method being called to avoid crashes.

```C
  // Sample invocation of a method that takes 1 argument and has no return.
  
  godot_int primitive = MESH__PRIMITIVE_TRIANGLES;
  const void *method_args[] = {&primitive};
  api->godot_method_bind_ptrcall(surfacetool_begin, st, method_args, NULL);
```

```C
  // Sample invocation of a method that takes no arguments and has no return.
  const void *method_args[] = {};
  api->godot_method_bind_ptrcall(some_method_binding, some_object, method_args, NULL);
```

#### Calling with `call` (slower, error checked, default parameter values applied)

`godot_method_bind_call` is a closer mapping to the behavior of GDScript, allowing the caller to omit parameters with default values.

For example, the `commit` method on `SurfaceTool` has two arguments, but both have default values as can be seen in the `api.json` file:

```JSON
{
  "arguments": [
            {
              "name": "existing",
              "type": "ArrayMesh",
              "has_default_value": true,
              "default_value": "Null"
            },
            {
              "name": "flags",
              "type": "int",
              "has_default_value": true,
              "default_value": "97280"
            }
          ]
}
```

were this method to be called via `godot_method_bind_ptrcall` (see above), these values would need to be replicated in the C code.

```C
  // Invocation of SurfaceTool::commit()
  
  // Both arguments have default values and may be omitted.
  const godot_variant *method_args[] = {NULL};
  godot_variant_call_error err;

  godot_variant ret = api->godot_method_bind_call(st_commit, st, method_args, 0, &err);
  
  // `godot_method_bind_call` also returns error information that may be useful for debugging.
  if (err.error != GODOT_CALL_ERROR_CALL_OK) {
    char buf[128] = {0};
    snprintf(buf, 127, "Call failed with code %d", err.error);
    api->godot_print_error(buf, __FUNCTION__, __FILE__, __LINE__);
  }
```

### Using Enum values declared on an Object

There does not appear to be a way to get at Enum values other than the global constants (via `godot_get_global_constants`). The values will be present in the `api.json` file, however, and can either be copied or generated into a header file with straightforward scripting.
