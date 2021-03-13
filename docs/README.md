# Adding a new category

1. Edit [_config.yml](https://github.com/abaire/godot-notes/blob/main/docs/_config.yml) and add the new collection name to the `collections` array.
    1. Set `output: true` so that the category pages are generated.
    1. Optionally set `show_in_nav: true` to have the category rendered in the side navigation bar.
    1. Optionally set `nav_label: <string>` to some string to override the collection name shown in the navigation bar.
 1. Create a new subdirectory with the collection name prefixed by `_` (e.g., a collection named "stuff" would have a directory named "_stuff").
 1. Add new files to the newly created subdirectory.
    1. Each new file should have a [Front Matter](https://jekyllrb.com/docs/front-matter/) block describing how to render the file.
        1. Set `title` to the title that should be displayed in the nav bar.
        1. Optionally set `subcategory` to cause the page to be rendered under a separate group. E.g., two pages in the "food" collection that are about vegetables ("carrots.md" and "lettuce.md") might share the subcategory "Vegetables".
