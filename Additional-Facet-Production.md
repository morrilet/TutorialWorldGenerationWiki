Another neat way to use facets is to use the data from one or more other facets to create a unique dataset.  In this example we will attempt to create random stone houses on the surface of the world.  For this,  we will create a boolean 3d facet that will contain the bottom-center locations of each of the houses.  From there we can then create a rasterizer that loops through each of the values and creates a house wherever the facet value is true.

But first,  the facet class itself:
```java
public class HouseFacet extends BaseBooleanFieldFacet3D {

    public HouseFacet(Region3i targetRegion, Border3D border) {
        super(targetRegion, border);
    }
}
```
There are multiple other options to choose from, but generally they are grouped on their amount of dimensions.
An amount of dimensions in this case is just 2D or 3D. You have a whole lot of these to choose from!
Check it out here: https://github.com/MovingBlocks/Terasology/tree/develop/engine/src/main/java/org/terasology/world/generation/facets/base
As the name suggests, these are base classes. Interfaces are also in that package.


Easy when immortius has already created the plumbing!  

Now the ```FacetProvider```:
```java
@Produces(HouseFacet.class)
@Requires(@Facet(SurfaceHeightFacet.class))
public class HouseProvider implements FacetProvider {

    private Noise2D noise;

    @Override
    public void setSeed(long seed) {
        noise = new SimplexNoise(seed);
    }

    @Override
    public void process(GeneratingRegion region) {

        //Don't forget you sometimes have to extend the borders.
        //extendBy(top, bottom, sides) is the method used for this.

        Border3D border = region.getBorderForFacet(HouseFacet.class);
        HouseFacet facet = new HouseFacet(region.getRegion(), border);
        SurfaceHeightFacet surfaceHeightFacet = region.getRegionFacet(SurfaceHeightFacet.class);

        for (Vector2i position : surfaceHeightFacet.getWorldRegion()) {
            int surfaceHeight = (int) surfaceHeightFacet.getWorld(position);

            if (facet.getWorldRegion().encompasses(position.x, surfaceHeight, position.y)
                    && noise.noise(position.x, position.y) > 0.99) {
                facet.setWorld(position.x, surfaceHeight, position.y, true);
            }
        }

        region.setRegionFacet(HouseFacet.class, facet);
    }
}
```
You are probably wondering why the getWorldRegion() method returns an iterable 'list' of Vector2i's. The reason behind this was explained above. Compare the getWorldRegion() to a map of the Earth. The map has coordinates, x and y. The y-coordinate does NOT specify the height of the terrain. Here in this example it's the 3D-world z-coordinate.
An instance of SurfaceHeightFacet always uses this system, and you use SurfaceHeightFacet.getWorld(x, y); to get the height of the terrain on that particular coordinate. Again, the coordinate passed in here is the 3D-world Z coordinate! Not the height! The method returns the height. I hope that helps to clarify that.

Here we are discarding 99% of the positions and setting the remaining values to true at surface height when in the range of the facet.

The last remaining piece is to rasterize these "houses" into the world and then plug all them in to the world builder.
```java
public class HouseRasterizer implements WorldRasterizer {
    Block stone;

    @Override
    public void initialize() {
        stone = CoreRegistry.get(BlockManager.class).getBlock("Core:Stone");
    }

    @Override
    public void generateChunk(CoreChunk chunk, Region chunkRegion) {
        HouseFacet houseFacet = chunkRegion.getFacet(HouseFacet.class);

        for (Entry<BaseVector3i, House> entry : houseFacet.getWorldEntries().entrySet()) {
            // there should be a house here
            // create a couple 3d regions to help iterate through the cube shape, inside and out
            Vector3i centerHousePosition = new Vector3i(entry.getKey());
            int extent = entry.getValue().getExtent();
            centerHousePosition.add(0, extent, 0);
            Region3i walls = Region3i.createFromCenterExtents(centerHousePosition, extent);
            Region3i inside = Region3i.createFromCenterExtents(centerHousePosition, extent - 1);

            // loop through each of the positions in the cube, ignoring the is
            for (Vector3i newBlockPosition : walls) {
                if (chunkRegion.getRegion().encompasses(newBlockPosition)
                        && !inside.encompasses(newBlockPosition)) {
                    chunk.setBlock(ChunkMath.calcBlockPos(newBlockPosition), stone);
                }
            }
        }
    }
}
```

```java
    @Override
    protected WorldBuilder createWorld(long seed) {
        return new WorldBuilder(seed)
                .addProvider(new SurfaceProvider())
                .addProvider(new SeaLevelProvider(0))
                .addProvider(new MountainsProvider())
                .addProvider(new HouseProvider())
                .addRasterizer(new TutorialWorldRasterizer())
                .addRasterizer(new HouseRasterizer());
    }
```

Bingo.  The world is now a village of boring stone dwelling hermits!

![image](https://raw.githubusercontent.com/Terasology/TutorialWorldGeneration/master/images/RequiresFacetProduction1.png)![image](https://raw.githubusercontent.com/Terasology/TutorialWorldGeneration/master/images/RequiresFacetProduction2.png)

Notice though that there is problems. Some houses are missing walls and roofs.  This will happen at the chunk boundary where the neighboring chunk does not know that it should be generating the edge of the house in its chunk.

[Next: Borders](Borders)