---
title: "Project organization"
subcategory: "GDNative"
---

The [Godot C example](https://docs.godotengine.org/en/stable/tutorials/plugins/gdnative/gdnative-c-example.html) suggests putting the native headers and source code at the root level with the Godot project created as a sibling:

```
/foo
  - godot-headers
  - src
    - my_native_code.c
  - foo
    - project.godot, ...
```

For projects where native code was not planned from the outset, it may be desirable to have the native code live inside the Godot project. It seems like this can be done with `.gdignore` files, but this may contaminate the final export action (or require filtering from there). 

A proof of concept that works for development (export untested as of this writing):

```
/foo
  - project.godot, ...
  - gdnative
    - bin
      - my_lib.gdnlib
      - my_type.gdns
      - lib
        - .gdignore
    - external
      - .gdignore
      - godot-headers (as a git submodule)
    - src
      - .gdignore
      - my_native_code.c
```

The `SConstruct` from the sample can then be modified to use the new structure.
