And now for some fun with metadata.  We need to provide surface height information so that our rasterizer can read this data and put blocks in the right place.  Luckily the engine provides us with a facet definition for this, ```SurfaceHeightFacet```.  This facet is basically a 2d representation of the surface of the world.  Each X,Y position holds the value of the height of the surface.

Start with a skeleton:
```java
@Produces(SurfaceHeightFacet.class)
public class SurfaceProvider implements FacetProvider {

    @Override
    public void setSeed(long seed) {
    }

    @Override
    public void process(GeneratingRegion region) {
    }
}
```

The key part of this is the annotation.  Without this, the world builder will not know how to organize facet providers together. Then we reimplement our existing rasterizer data.  But lets make it a little more interesting and make our surface at y=10.  Because,  you know danger happens at y=10.
```java
    @Override
    public void process(GeneratingRegion region) {
        // Create our surface height facet (we will get into borders later)
        Border3D border = region.getBorderForFacet(SurfaceHeightFacet.class);
        SurfaceHeightFacet facet = new SurfaceHeightFacet(region.getRegion(), border);

        // loop through every position on our 2d array
        Rect2i processRegion = facet.getWorldRegion();
        for(Vector2i position : processRegion) {
            facet.setWorld(position, 10f);
        }

        // give our newly created and populated facet to the region
        region.setRegionFacet(SurfaceHeightFacet.class, facet);
    }
```

Some key points to note is that ```facet.getWorldRegion()``` refers to world coordinates which happen to coincide with ```facet.setWorld(...)```.  There are also methods that deal with the local coordinate system.  But it is easier to stick with world positions.

We then add this to our world builder:
```java
    @Override
    protected WorldBuilder createWorld(long seed) {
        return new WorldBuilder(seed)
                .addProvider(new SurfaceProvider())
                .addProvider(new SeaLevelProvider(0))
                .addRasterizer(new TutorialWorldRasterizer());
    }
```

Oh, right.  Dont forget to put in the ```SeaLevelProvider``` so that the game doesnt spawn the player 100 meters underwater.

Now, when we run the rasterizer, we can access this facet data that we have provided.  Lets use it:
```java
    @Override
    public void generateChunk(CoreChunk chunk, Region chunkRegion) {
        SurfaceHeightFacet surfaceHeightFacet = chunkRegion.getFacet(SurfaceHeightFacet.class);
        for(Vector3i position : chunkRegion.getRegion()) {
            if(position.y < surfaceHeightFacet.getWorld(position.x, position.z)) {
                chunk.setBlock(ChunkMath.calcBlockPos(position), dirt);
            }
        }
    }
```

Be sure to get your x and z correct.  The surface height facet is 2d (x,y) and the ```Vector3i``` uses y as height.

Exciting times... lots of dirt... at y=10

![Facet Production](https://raw.githubusercontent.com/Terasology/TutorialWorldGeneration/master/images/Facet%20Production.png)

[Next: Noise Sampling](Noise Sampling)
