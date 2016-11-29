First off, create a new module that our world generator will be located in. A guide to creating and working with modules can be found at https://github.com/MovingBlocks/Terasology/wiki/Developing-Modules.

When the module is set up and running, add a new world generator class to it- for now, let's use this bare minimum template from the Core module:

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

Next, we need to add a rasterizer - this is the class that will put blocks into the world when a chunk is generated:

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

The `generateChunk()` method is where the magic will happen - the ```chunk``` parameter interacts directly with chunk storage where you can place blocks, while the ```chunkRegion``` parameter holds metadata about the world.  There's nothing in the ```chunkRegion``` method right now,  but we will get there soon enough :)

Now let's add some basic rasterization, replacing every block below y=0 with dirt.

```java
public class TutorialWorldRasterizer implements WorldRasterizer {
    private Block dirt;

    @Override
    public void initialize() {
        dirt = CoreRegistry.get(BlockManager.class).getBlock("Core:Dirt");
    }

    @Override
    public void generateChunk(CoreChunk chunk, Region chunkRegion) {
        for (Vector3i position : chunkRegion.getRegion()) {
            if (position.y < 0) {
                chunk.setBlock(ChunkMath.calcBlockPos(position), dirt);
            }
        }
    }
}
```

Note that you should always change your coordinates from regional to local when setting the block on the chunk.

Finally, let's add the rasterizer to the world builder:

```java
    @Override
    protected WorldBuilder createWorld() {
        return new WorldBuilder(worldGeneratorPluginLibrary)
                .addRasterizer(new WorldRasterizer());
    }
```

And after we enable the module in game and select the "Tutorial World Generator" - voila!

```
java.lang.IllegalStateException: surface height facet not found
```

...well, that's not quite what we wanted. To get our world generator working, we also need to provide surface information so that the spawn algorithm can find where the surface is and gently place new players there. 

[Next: Facet Production](Facet Production)