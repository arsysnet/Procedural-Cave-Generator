This is a quick overview of the project's internals. Unity's workflow generally doesn't make use of assemblies the way that C# projects normally would, so I've mainly used namespaces to organize the various parts of the project. The folder structure in this directory matches the namespace structure. This should make it easy to extract a useful chunk of the project and export it to another project, without having to carry over everything. 

The following is an overview of the namespaces, their responsibilities, and their dependencies. 

### ArrayExtensions

Dependencies: None.

Extension methods for 2D arrays, with a focus on building a more expressive API for grids. Instead of writing code that looks like this:

```cs
for (int y = 0; y < arr.GetLength(1); y++){
    for (int x = 0; x < arr.GetLength(0); x++){
        arr[x, y] = SomeFunction(x, y);
    }
}
for (int y = 1; y < arr.GetLength(1) - 1; y++){
    for (int x = 1; x < arr.GetLength(0) - 1; x++){
        arr[x, y] = SomeOtherFunction(x, y, otherData);
    }
}
```

We can instead write code like this:
```cs
arr.Transform(SomeFunction);
arr.TransformInterior((x, y) => SomeOtherFunction(x, y, otherData));
```

### CaveGeneration

Dependencies: Everything else.

The main class is CaveGenerator, which serves as the entry point to the entire system. Lots of plumbing, and a fair bit of editor scripting for the custom inspector to CaveGeneratorUI, but not much else.

### Editor

Dependencies: None.

Contains a variety of helper functions I wrote while working on the editor scripts in this project. 

### HeightMaps

Dependencies: None.

Offers various implementations of the IHeightMap interface, whose purpose is to supply a unique height value y for each pair of floats (x, z), exposed through the factory class HeightMapFactory.

### MapGeneration

Dependencies: Maps, ArrayExtensions.

Offers a variety of algorithms for generating and processing Map objects. MapBuilder offers most of the functionality (smoothing, connectivity, random fill, expanding floors, etc.). MapTunnelers offers strategies for tunneling between two points in the map.

### Maps

Dependencies: ArrayExtensions.

Offers a small number of core types for working with 2D grids. This includes Map, which represents a 2D grid of floors and walls, and Coord, a compact integer equivalent of Vector2. 

I often export this folder into other projects that make use of grids. 

### MeshGeneration

Dependencies: HeightMaps.

This provides the functionality to produce 3D meshes out of 2D grids, making heavy use of the Marching Squares algorithm. Marching Squares converts a 2D grid into a flat mesh. The outline generator traces out the outlines (edges) of a flat mesh produced by the marching squares algorithm, which are used to build walls between ceilings and floors, though it could also be used to generate 2D polygon colliders for a 2D implementation.

### Modules

Dependencies: Editor.

Contains the core module system explained at length in the other readmes. Most of its power comes from editor scripting: a custom inspector which exposes editors to all of its submodules, and a context menu item that offers a deep copy for modules, a feature absent for ScriptableObjects. 

### Modules - HeightMaps

Dependencies: Modules, HeightMaps.

Offers heightmap modules that can be socketed into a cave generator to define the height of its floors and ceilings. 

### Modules - MapGeneration

Dependencies: Modules, MapGeneration, Maps, Editor.

Offers map generation modules that can be socketed into a cave generator to define the layout of floors and walls. Also contains a visual editor to build map modules out of other modules, and a custom asset postprocessor that takes control of the import process for image files with "_map" in their name.

The map module system has quite a few potential applications outside of this project. If you need to generate binary 2D grids for a project, you may find this system useful.

### Modules - Outlines

Dependencies: Modules.

Offers outline modules that can be socketed into the Rock Outline generator to lay out rocks along the outlines. 

### Threading

Dependencies: None.

Offers a few methods for multi-threading, most notably a parallel foreach. Will become entirely redundant once Unity is updated to a more modern version of C# and .Net (which is on the roadmap). 