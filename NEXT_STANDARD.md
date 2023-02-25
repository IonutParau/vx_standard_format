# Next Standard

This is the current progress on the next STANDARD.MD that adds some proposals.

# Proposals added

None yet.

# Current details:

# Structure

Like most level codes, VX starts with the `VX;` header to mark the level code as a VX level code.
The structure of the level is like this: `VX;<title?>;<description?>;<cell data?>;<grid data?>;<width?>;<height?>;<tag?>;`

# A note on those ending with a ?

The "segments" (parts seperated by `;`) that end with a `?>` mean that they are option, and may be empty.
Each segment's definition should represent what happens when the segment is empty.

# A note on trailing ;

A level code is allowed to remove empty segments if they are at the end.
This means the level code `VX;hello;there;;;;;;` can be turned into `VX;hello;there;`.

# Dynamic and Fixed grids

VX supports both dynamic and fixed grids.
However, implementations of VX are allowed to not support dynamic grids, in which case, they would error out if the level code required a dynamic grid.
However, fixed grids must be supported in some way (when decoding) for an implementation to be considered valid.

# The segments

## title

When empty, it means the level has no title. When not empty, the content of this segment is the title.

## description

When empty, it means the level has no description. When not empty, the content of this segment is the description.

## cell data

This is a base64-encoded bytestream of zlib-compressed json-encoded list of lists of objects representing the cells.

If it is JSON-decoded into something other than a list of cells, it should be treated as if the cell data segment was not specified!

If it is not specified (or decoded to something other than a list), it means the grid is empty.

### The grid format

In a fixed grid, every cell, empty or not, has an entry. In a dynamic grid, each non-empty cell has an entry (though entries with empty cells are allowed).

### The cell format (for a cell on a fixed grid)

Each cell is a Layer-List.
This means its a sequence of layers.

The format is a list containing a repeating sequence of id, rotation, data.

The id would be the id, either a number or string.

The id might be ahead-of-time translated to a standardized ID or to use the non-standardize `@` header.

The rotation must be a number.

The data must be an object.

There are typically 2 layers: foreground and background.

This would mean that the cell is encoded as: `[foreground.id, foreground.rot, foreground.data, background.id, background.rot, background.data]`.

A layer that is considered "useless" can optionally not be put there. If the amount of layers minimum is not met, the unspecified layers are allowed to be treated as containing an empty cell.

### The cell format (for a cell on an infinite grid)

Similar to the fixed-grid cell format, except at the start of the list is the x and y.

### grid data
  
This is a base64-encoded bytestream of a zlib-compressed json-encoded object.
  
This object should have 2 fields (though each one can be optionally not mentioned and instead replaced with a default):
- `A`, which is an indentifier for the remake that encoded this. Used for preprocessors (more on that later). Should default to the string `modularcm`.
- `GT`, which means grid-type. The string `fixed` means a fixed grid, while `dynamic` means a dynamic grid. It should default to `fixed`.

If it is not mentioned, the default one is used, which has the defaults of those 2 fields.

### width

If empty, it means `100`.
If not empty, it should be a raw number (no high bases, just the number) meaning the width of the grid if it is a fixed grid.

### height

If empty, it means that the fixed grid should be a square, and thus the height is equal to the width.
If it is `=`, it means that the fixed grid should be a square, and thus the height is equal to the width.
Otherwise, it's just a raw number meaning the height of the grid if it is a fixed grid.

# Preprocessors

Preprocessors may not always be supported, but they are meant to look at the `A` grid data field and do some pre-processing on the cell data, such as changing IDs or adding rotation.

There should at least be one preprocessor however, and that is the Standardized IDs preprocessor, which is meant to modify the cell based on its standardized ID to achieve the standardized functionality.

# Standardized IDs

Some IDs are standardized and meant to be used across all remakes, and when encoded, have their respective internal IDs converted to these standardized ones.

If any of these IDs would conflict with a pre-existing cell in the remake that does something else, you must put for that ID a `@` at the start.

An `@` at the start of the ID means that the ID is not standard, and thus the following characters must be turned into a character.

Suppose `gen` already is taken and isn't the generator. In that case, you would save it as `@gen`. Then, when decoding, `@gen` would be converted to `gen`.

The standardized IDs are:
- `mover`, it means the default mover
- `gen`, it means the default generator
- `rot_cw`, it means a clockwise rotator
- `rot_ccw`, it means a counter-clockwise rotator
- `push`, it means the push cell, aka a cell that does nothing and can be pushed from all directions
- `slide`, it means the normal slide cell
- `wall`, it means a normal wall cell
- `trash`, it means a normal trash cell
- `enemy`, it means a normal enemy
- `place`, it means a placeable
- `empty`, it means an empty cell

## The extended Standardized IDs

There are IDs of common cells that may not be implemented in all remakes
These IDs are often requested by remake authors.

The extended standardized IDs are:
- `onedir`, it means a one directional, aka a cell that can only be moved from one direction, that direction being its back (extra rotation may be applied to the cell in this case).
- `ghost`, it means a wall that can't be generated by a generator.
- `ungeneratable`, it means a push cell that can't be generated.
