Another neat way to use facets is to use the data from one or more other facets to create a unique dataset.  In this example we will attempt to create random stone houses on the surface of the world.  For this,  we will create a boolean 3d facet that will contain the bottom-center locations of each of the houses.  From there we can then create a rasterizer that loops through each of the values and creates a house wherever the facet value is true.

But first,  the facet class itself:
```java
public class HouseFacet extends BaseBooleanFieldFacet3D {

    public HouseFacet(Region3i targetRegion, Border3D border) {
        super(targetRegion, border);
    }
}
```
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
        for (Vector3i position : chunkRegion.getRegion()) {
            if (houseFacet.getWorld(position)) {
                // there should be a house here
                // create a couple 3d regions to help iterate through the cube shape, inside and out
                Vector3i centerHousePosition = position.clone();
                centerHousePosition.add(0,4,0);
                Region3i walls = Region3i.createFromCenterExtents(centerHousePosition, 4);
                Region3i inside = Region3i.createFromCenterExtents(centerHousePosition, 3);

                // loop through each of the positions in the cube, ignoring the inside ones
                for(Vector3i newBlockPosition : walls) {
                    if (chunkRegion.getRegion().encompasses(newBlockPosition)
                            && !inside.encompasses(newBlockPosition)) {
                        chunk.setBlock(TeraMath.calcBlockPos(newBlockPosition), stone);
                    }
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

[Borders!](Borders)