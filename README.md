# NavMesh

A little demo for pathfinding in 2D space around convex-polygonal obstacles. This may be useful in GameDev.

Finds optimal paths from one source to one destination walking around obstacles at a given distance.

## Usage

You need to include ``PathFinder.*``, ``Vec.*``, ``Segment.*``, ``Polygon.*`` files in your project.

The main class is ``NavMesh::PathFinder``. There are also ``NavMesh::Vec``, ``NavMesh::Segment`` and ``NavMesh::Polygon`` for geometrical logic.

``PathFinder::AddPolygons`` should be called each time the map changes.
This is the slowest one, which takes ``O(n^3*k)`` time, where ``n``in the number of polygons, and ``k`` is average number of points in each.
This method consumes ``O(n^2*k)`` memory in the worst case (usually much less, as most of edges are not valid due to intersections).
Ideally, it should be called only if the map is updated.

``PathFinder::AddExternalPoints`` should be called afterh ``AddPolygons`` each time coordinates of external points change.
This method takes ``O(p*n^2)`` time and consumes ``O(n*p)`` memory, where ``p`` is the number of points added.

``PathFinder::FindPath`` should be called each time you need a path between two points. The points must be on of the external points.
This method takes ``O((n*k+p)*n*log(n*k+p))`` time and uses ``O((n+p)*n)`` memory.


## Details

This project constructs the graph around obstacles, using polygon sides and tangents to polygons as edges in the graph. 
Then it uses A* on the constructed graph to find the shortest path. This implementation takes 10ms to construct the graph with 100 polygons with upto 20 points each on i7-6700k CPU. 
The path calulations on the graph take less than 1ms. Already computed tangents are reused to check for intersections of potential edges and obstacles.

## Code structure
``source/`` directory contains the library, ``tests/`` directory contains tests and ``demo/`` directory contains a simple Windows demo application.
``point.cpp``, ``segment.cpp``, and ``polygon.cpp`` contain geometry primitives with necessary logic implemented on them.
``path_finder.cpp`` has main class, which invoces geometry calculations, constructs the graph and uses A* to find the path between two given points.

## Demo
A demo is a simple windows GUI application. It uses GDI+ to draw obstacles and a found path, as well as outputting some statistics.
The user interaction is mostly happening through the meny. Following items are available:

* Polygon
  * Add - click on the screen to add the point to the new polygon. Right click to add the polygon to the map.
  if you right click with empty current polygon you will exit the adding mode.
  * Delete - click inside the polygon you would like to delete. Right click to exit the mode.
* Move
  * Source - move mouse around to move the source point. Click to fix the source.
  * Destination - move mouse around to move the Destination point. Click to fix the source.
* Debug
  * Generate grid - will put on the map a grid of squares.
  * Generate polygons - will randomly generate 100 polygons on the map.
  * Generate circles - will randomly generate 100 circles on the map (more detailed obstacles).
  * Inflate polygons - toggles if the path should walk around the polygons or can stick to them. Inflation will double the amount of points, slowing the calculations down.
  * Show edges - toggle if the edges of the graph are shown.
  * Benchmark - Generate polygons and find path non-stop 200 times and output the average time spent.

You can also drag the source or the destination circles (if no action is selected). Current action is always shown at the top of the window. 
The upper left corner shows how much time all path related calulations took and how much geometrical calculations took specifically. There's also the result of the last benchmark.
Drawing is not included in any measurements and is quite slow, especially if you choose to draw edges.

## Building

The repository has Visual Studio 2019 project files necessary to build it in ``build_vs/`` directory.
Simply open NavMesh.sln in the Visual Studio 2019 or later. ``source`` project builds a static library, ``demo`` builds a demo and ``tests`` runs tests.

On other platforms you can execute:
``` mkdir build
cd build
cmake ..```

## Further work

The solution constructs all ``O(n)`` tangents for each point to add them as edges. Then it individually checks all edges for intersections besed on its relative position to all other tangents.
This ``O(n^2)`` part could in theory be optimized to ``O(n log n)`` by sorting all the segments from the source point by angle. Then a sweeping line algorithm should be able to find all the valid edges.
However, it couldn't dramatically improve the performance, as it's usually dominated by tangents calculations in case if most of the polygons are not intersecting (even inflated).

The interface could be updated to make it more flexible: 
Current solution run A* for each path between two points. If the map is static or very rarely updated, a Floyd-Warshall algorithm might be used instead to precalculate all possible paths.
If there's a single source and many destinations (or the other way around), a Dijkstra might be used instead of A* to get all paths in one go.

Also, the ``AddPolygons`` method could be updated to work with a timelimit and continue its work from the previous call (by splitting the loop on polygons). 
This way the caller may call this method on each frame and not loose fps. ``AddExternalPoints`` and ``GetPath`` should do nothing until ``AddPolygons`` has finished all the work.

Finally, a routine to split non-convex polygons to a unity of several convex polygons should be provided.
