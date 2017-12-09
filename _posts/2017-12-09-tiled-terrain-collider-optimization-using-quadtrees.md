---
layout: post
title: Tiled Terrain Collider Optimization Using QuadTrees
date: 2017-12-09 17:20
author: sanderman0
comments: true
tags: [game development]
---
One of my personal pet projects is a game developed in Unity which will include a large world composed of smalll 2D tiles, similar to games such as Terraria and Starbound. Normally, handling collision in such worlds is relatively easy, given the fact that all tiles are aligned to a grid. It becomes simple to index into an array-like datastructure to see if there is a tile in a certain space and thus to determine if an entity will be colliding.

That approach will not work well in the case of this particular project, since I intend to use my grid system not only for terrain, but also for vehicles, floating platforms, or other moving structures. This means I want my grids to be able to move and rotate much like any other game object might. Grid cells may no longer align with any world axes, and thus the former approach to collisions would become a lot more complicated. In addition, we will be dealing with a lot more potential collision points if we want two grids to be able to rotate and still collide with each other. For these reasons I have decided that in this project, using the regular 2D physics system and colliders available in Unity would be more appropriate. 

<h2>First Iteration: Collider for each solid cell.</h2>

A naive approach simply iterates over all the cells and create a new rectangle for each solid cell. These rectangles are then turned into BoxCollider2D components. In the following images, the normal tile renderers are disabled so that we can more easily inspect the resulting colliders, and the Unity profiler shows us almost exactly how much memory and cpu time we are using each frame:

<a href="{{ site.url }}/assets/images/gridgame/collider-cells.png"><img src="{{ site.url }}/assets/images/gridgame/collider-cells.png"/></a>

This first iteration introduces the problem that I want to talk about in this post. The pseudorandomly generated world that I am currently testing in has 311466 solid tiles and an equal number of colliders generated. It is worth thinking about ways to simplify this by grouping cells into fewer colliders, to save memory and cpu performance.

<h2>Second Iteration: Simplified colliders using Region QuadTree</h2>

If we take a step back and say that our empty cells are white, and our solid cells are black, then our grids starts to look a lot like a black and white image. Perhaps we can look at image compression techniques for inspiration? A [Region quadtree](https://en.wikipedia.org/wiki/Quadtree#Region_quadtree) turns out to be a very useful datastructure to deal with these types of problems. 

By keeping track of which nodes of the tree are fully or partially filled, we can combine many small cells into relatively few large colliders. This is demonstrated in the following sample. This code generates rectangles for one grid chunk, which will later be used to create the actual collider components. We'll operate on the assumption that all chunks have a width and height of 64 cells.

```cs
public class QuadTreeColliderGen
{
  public class TreeNode
  {
    public enum NodeValue { Empty, Solid, Partial }

    public NodeValue value;
    public int minX;
    public int minY;
    public int width;
    public TreeNode[] children;

    public TreeNode(NodeValue value, int minX, int minY, int width, TreeNode[] children)
    {
      this.value = value;
      this.minX = minX;
      this.minY = minY;
      this.width = width;
      this.children = children;
    }
  }

  public static List<Rect> GenerateColliders(BitBuffer solidBuffer)
  {
    Profiler.BeginSample("QuadTreeColliderGen.GenerateColliders");
    List<Rect> rects = new List<Rect>();
    TreeNode tree = CreateTree(solidBuffer, 0, 0, Grid2D.ChunkWidth);
    RenderTree(tree, rects);
    Profiler.EndSample();
    return rects;
  }

  private static TreeNode CreateTree(BitBuffer solidTileBuffer, int minX, int minY, int width)
  {
    if (width == 1)
    {
      // Base case.
      int index = Grid2D.GetChunkCellIndex(new Int2(minX, minY), Grid2D.ChunkWidth);
      bool solid = solidTileBuffer.GetValue(index);
      TreeNode.NodeValue value = solid ? TreeNode.NodeValue.Solid : TreeNode.NodeValue.Empty;
      return new TreeNode(value, minX, minY, width, null);
    }
    else
    {
      // Recursive case.
      int childWidth = width / 2;
      TreeNode[] children = new TreeNode[4]
      {
          CreateTree(solidTileBuffer, minX, minY, childWidth),
          CreateTree(solidTileBuffer, minX + childWidth, minY, childWidth),
          CreateTree(solidTileBuffer, minX, minY + childWidth, childWidth),
          CreateTree(solidTileBuffer, minX + childWidth, minY + childWidth, childWidth)
      };

      TreeNode.NodeValue value;
      if (Array.TrueForAll(children, item => item.value == TreeNode.NodeValue.Solid))
      {
        value = TreeNode.NodeValue.Solid;
      }
      else if (Array.TrueForAll(children, item => item.value == TreeNode.NodeValue.Empty))
      {
        value = TreeNode.NodeValue.Empty;
      }
      else
      {
        value = TreeNode.NodeValue.Partial;
      }
      return new TreeNode(value, minX, minY, width, children);
    }
  }

  private static void RenderTree(TreeNode tree, List<Rect> results)
  {
    switch (tree.value)
    {
      case TreeNode.NodeValue.Solid:
        results.Add(new Rect(tree.minX, tree.minY, tree.width, tree.width));
        break;
      case TreeNode.NodeValue.Partial:
        foreach (TreeNode node in tree.children) RenderTree(node, results);
        break;
      default:
        break;
    }
  }
}
```

<a href="{{ site.url }}/assets/images/gridgame/quadtree.png"><img src="{{ site.url }}/assets/images/gridgame/quadtree.png"/></a>

This results in a much more manageable number of 40658 box colliders in this particular case. Unfortunately, we are now also allocating a huge number of temporary objects, with heavy memory and performance costs. Is there any way we can allocate the memory we need once and then reuse it?

<h2>Third Iteration: Array based QuadTree</h2>

We can rewrite our code so that we do not need to create new objects all the time. Instead we can put our tree nodes in an array, and keep reusing that array. This requires using indexes into the array rather than references to objects. (We change our nodes into structs to turn them into value types instead of reference types) We pre-allocate the space we need in the array and result list. (Size is based on the number of cells in a single grid chunk, which is 64*64 cells.) 

A lot of performance is saved since we no longer need to allocate new memory for objects all the time. (GC Alloc shows 0 bytes, since the memory we are using was allocated just once and before starting these profiling samples) Unfortunately, all this has also made our code quite a bit more complex and harder to read.

```cs
public class QuadTreeColliderGenerator
{
  private List<Rect> rects = new List<Rect>(4096);
  private TreeNode[] nodes = new TreeNode[5461];

  public struct TreeNode
  {
    public enum NodeValue { Empty, Solid, Partial }

    public NodeValue value;
    public int minX;
    public int minY;
    public int width;
    public int childA;
    public int childB;
    public int childC;
    public int childD;
  }

  public List<Rect> GenerateColliderRects(BitBuffer solidBuffer)
  {
    Profiler.BeginSample("QuadTreeColliderGen.GenerateColliders");
    rects.Clear();
    int nodeCount = 0;
    CreateTree(nodes, ref nodeCount, solidBuffer, 0, 0, Grid2D.ChunkWidth);
    RenderTree(0, nodes, rects);
    Profiler.EndSample();
    return rects;
  }

  /// <summary>
  /// Creates a tree of nodes inside the specified array. Nodes reference their children by index into the array.
  /// </summary>
  private static int CreateTree(TreeNode[] nodes, ref int nodeCount, BitBuffer solidTileBuffer, int minX, int minY, int width)
  {
    int nodeIndex = nodeCount;
    nodeCount++;

    if (width == 1)
    {
      // Base case.
      int index = Grid2D.GetChunkCellIndex(new Int2(minX, minY), Grid2D.ChunkWidth);
      bool solid = solidTileBuffer.GetValue(index);
      TreeNode.NodeValue value = solid ? TreeNode.NodeValue.Solid : TreeNode.NodeValue.Empty;

      nodes[nodeIndex].value = value;
      nodes[nodeIndex].minX = minX;
      nodes[nodeIndex].minY = minY;
      nodes[nodeIndex].width = width;
      
      return nodeIndex;
    }
    else
    {
      // Recursive case.
      int childWidth = width / 2;
      int childA = nodes[nodeIndex].childA = CreateTree(nodes, ref nodeCount, solidTileBuffer, minX, minY, childWidth);
      int childB = nodes[nodeIndex].childB = CreateTree(nodes, ref nodeCount, solidTileBuffer, minX + childWidth, minY, childWidth);
      int childC = nodes[nodeIndex].childC = CreateTree(nodes, ref nodeCount, solidTileBuffer, minX, minY + childWidth, childWidth);
      int childD = nodes[nodeIndex].childD = CreateTree(nodes, ref nodeCount, solidTileBuffer, minX + childWidth, minY + childWidth, childWidth);

      if (nodes[childA].value == TreeNode.NodeValue.Solid && nodes[childB].value == TreeNode.NodeValue.Solid
        && nodes[childC].value == TreeNode.NodeValue.Solid && nodes[childD].value == TreeNode.NodeValue.Solid)
      {
        nodes[nodeIndex].value = TreeNode.NodeValue.Solid;
        nodes[nodeIndex].minX = minX;
        nodes[nodeIndex].minY = minY;
        nodes[nodeIndex].width = width;
        return nodeIndex;
      }
      else if (nodes[childA].value == TreeNode.NodeValue.Empty && nodes[childB].value == TreeNode.NodeValue.Empty
        && nodes[childC].value == TreeNode.NodeValue.Empty && nodes[childD].value == TreeNode.NodeValue.Empty)
      {
        nodes[nodeIndex].value = TreeNode.NodeValue.Empty;
        return nodeIndex;
      }
      else
      {
        nodes[nodeIndex].value = TreeNode.NodeValue.Partial;
        nodes[nodeIndex].minX = minX;
        nodes[nodeIndex].minY = minY;
        nodes[nodeIndex].width = width;
        return nodeIndex;
      }
    }
  }

  private static void RenderTree(int nodeIndex, TreeNode[] nodes, List<Rect> results)
  {
    switch (nodes[nodeIndex].value)
    {
      case TreeNode.NodeValue.Solid:
        results.Add(new Rect(nodes[nodeIndex].minX, nodes[nodeIndex].minY, nodes[nodeIndex].width, nodes[nodeIndex].width));
        break;
      case TreeNode.NodeValue.Partial:
        RenderTree(nodes[nodeIndex].childA, nodes, results);
        RenderTree(nodes[nodeIndex].childB, nodes, results);
        RenderTree(nodes[nodeIndex].childC, nodes, results);
        RenderTree(nodes[nodeIndex].childD, nodes, results);
        break;
      default:
        break;
    }
  }
}
```

<a href="{{ site.url }}/assets/images/gridgame/quadtree-optimized.png"><img src="{{ site.url }}/assets/images/gridgame/quadtree-optimized.png"/></a>

<h2>Fourth Iteration: Implicit QuadTree</h2>

Is there any way we can simplify our code further? Do we even need to allocate any data for our tree at all? After all, we are not really doing anything interesting with our tree after generating the collider rectangles we need. Do we really need all this bookkeeping?

Turns out we can actually use recursion to treat the instruction stack itself as an implicit tree. We can take a look at the results of our children before returning, and add another rectangle to the result list when appropriate. We only track the data we need in our local scope, so we no longer need to write to an array all the time. This saves us yet more memory and cpu time.

```cs
public class QuadTreeColliderGenerator
{
  private List<Rect> rects = new List<Rect>(Grid2D.ChunkWidth * Grid2D.ChunkWidth);

  public enum NodeType { Empty, Solid, Partial }

  public List<Rect> GenerateColliderRects(BitBuffer solidBuffer)
  {
    Profiler.BeginSample("ImplicitQuadTreeColliderGen.GenerateColliders");
    rects.Clear();
    NodeType node = RenderTree(rects, solidBuffer, 0, 0, Grid2D.ChunkWidth);
    if (node == NodeType.Solid) rects.Add(new Rect(0, 0, Grid2D.ChunkWidth, Grid2D.ChunkWidth));
    Profiler.EndSample();
    return rects;
  }

  /// <summary>
  /// Creates a tree of nodes inside the specified array. Nodes reference their children by index into the array.
  /// </summary>
  private static NodeType RenderTree(List<Rect> results, BitBuffer solidTileBuffer, int minX, int minY, int width)
  {
    if (width == 1)
    {
      // Base case.
      int index = Grid2D.GetChunkCellIndex(new Int2(minX, minY), Grid2D.ChunkWidth);
      bool solid = solidTileBuffer.GetValue(index);
      NodeType value = solid ? NodeType.Solid : NodeType.Empty;
      return value;
    }
    else
    {
      // Recursive case.
      int childWidth = width / 2;
      NodeType childA = RenderTree(results, solidTileBuffer, minX, minY, childWidth);
      NodeType childB = RenderTree(results, solidTileBuffer, minX + childWidth, minY, childWidth);
      NodeType childC = RenderTree(results, solidTileBuffer, minX, minY + childWidth, childWidth);
      NodeType childD = RenderTree(results, solidTileBuffer, minX + childWidth, minY + childWidth, childWidth);

      if (childA == NodeType.Solid && childB == NodeType.Solid
        && childC == NodeType.Solid && childD == NodeType.Solid)
      {
        return NodeType.Solid;
      }
      else if (childA == NodeType.Empty && childB == NodeType.Empty
        && childC == NodeType.Empty && childD == NodeType.Empty)
      {
        return NodeType.Empty;
      }
      else
      {
        if (childA == NodeType.Solid) results.Add(new Rect(minX, minY, childWidth, childWidth));
        if (childB == NodeType.Solid) results.Add(new Rect(minX + childWidth, minY, childWidth, childWidth));
        if (childC == NodeType.Solid) results.Add(new Rect(minX, minY + childWidth, childWidth, childWidth));
        if (childD == NodeType.Solid) results.Add(new Rect(minX + childWidth, minY + childWidth, childWidth, childWidth));
        return NodeType.Partial;
      }
    }
  }
}
```

<a href="{{ site.url }}/assets/images/gridgame/quadtree-implicit.png"><img src="{{ site.url }}/assets/images/gridgame/quadtree-implicit.png"/></a>

## Conclusion

We started with a result of roughly 310 thousand collider rectangles in this particular world and were able to reduce that to 40000. The final solution is able to do this without any tempory heap allocations, doing most of its work using local values which are allocated on the stack to keep things fast. This allows us to process 257 grid chunks in under 100ms, where each of these chunks has a height and width of 64 for 4096 cells total. (Not all these chunks contain actual solid tiles)

It is quite possible there could be other methods to reduce the rectangle count even further, but for now I am satisfied with what I have.

Finally, here is a sneak peek of what I am making, and why this was needed:

<a href="{{ site.url }}/assets/images/gridgame/collider-overlay.png"><img src="{{ site.url }}/assets/images/gridgame/collider-overlay.png"/></a>