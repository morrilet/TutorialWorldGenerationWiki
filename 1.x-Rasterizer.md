First off, one must create a world generator to add our new facet generation to.  I copy/pasted from Core and stripped it down to its bare minimum.

```java
@RegisterWorldGenerator(id = "tutorialWorld", displayName = "Tutorial World")
public class TutorialWorldGenerator extends BaseFacetedWorldGenerator{

    @In
    private WorldGeneratorPluginLibrary worldGeneratorPluginLibrary;
    
    public WorldGenerator(SimpleUri uri) {
        super(uri);
    }

    @Override
    protected WorldBuilder createWorld() {
        return new WorldBuilder(worldGeneratorPluginLibrary);
    }

}
```

Next we need to add a rasterizer to put blocks into the world when a chunk is generated. And you end up with this:

```java
public class TutorialWorldRasterizer implements WorldRasterizer {
    @Override
    public void initialize() {
    }

    @Override
    public void generateChunk(CoreChunk chunk, Region chunkRegion) {
    }
}
```

In the generate chunk is where the magic will happen.  The ```chunk``` parameter interacts directly with chunk storage where you can place blocks.  The ```chunkRegion``` parameter holds metadata about the world.  There will be nothing in the ```chunkRegion``` right now,  but we will get there.

Next let us add some basic rasterization where everything below y=0 is dirt.

```java
public class TutorialWorldRasterizer implements WorldRasterizer {
    private Block dirt;

    @Override
    public void initialize() {
        dirt = CoreRegistry.get(BlockManager.class).getBlock("Core:Dirt");
    }

    @Override
    public void generateChunk(CoreChunk chunk, Region chunkRegion) {
        for(Vector3i position : chunkRegion.getRegion()) {
            if(position.y < 0) {
                chunk.setBlock(ChunkMath.calcBlockPos(position), dirt);
            }
        }
    }
}
```

Careful to change your world coordinates from the region into local coordinates when setting the block on the chunk.

And add it to our world builder:

```java
    @Override
    protected WorldBuilder createWorld() {
        return new WorldBuilder(worldGeneratorPluginLibrary)
                .addRasterizer(new WorldRasterizer());
    }
```

And after we enable the module in game and select the "Tutorial World Generator" voila...

```
java.lang.IllegalStateException: surface height facet not found
```

Not exactly what we wanted.  We need to provide surface information so that the spawn algorithm can find where the surface is and gently place new players there.  

[Next: Facet Production](1.x-Facet Production)

