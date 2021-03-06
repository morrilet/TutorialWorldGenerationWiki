Modifying existing facets is where this generation system gains its power.  In this section,  we will attempt to create mountains from our little hilly terrain.
```java
@Updates(@Facet(SurfaceHeightFacet.class))
public class MountainsProvider implements FacetProvider {

    private Noise mountainNoise;

    @Override
    public void setSeed(long seed) {
        mountainNoise = new SubSampledNoise(new BrownianNoise(new PerlinNoise(seed + 2), 8), new Vector2f(0.001f, 0.001f), 1);
    }

    @Override
    public void process(GeneratingRegion region) {
        SurfaceHeightFacet facet = region.getRegionFacet(SurfaceHeightFacet.class);
        float mountainHeight = 400;
        // loop through every position on our 2d array
        Rect2i processRegion = facet.getWorldRegion();
        for (BaseVector2i position : processRegion.contents()) {
            // scale our max mountain height to noise (between -1 and 1)
            float additiveMountainHeight = mountainNoise.noise(position.x(), position.y()) * mountainHeight;
            // dont bother subtracting mountain height, that will allow unaffected regions
            additiveMountainHeight = TeraMath.clamp(additiveMountainHeight, 0, mountainHeight);

            facet.setWorld(position, facet.getWorld(position) + additiveMountainHeight);
        }
    }
}
```

Dont miss the ```@Updates``` magic that allows this provider to be ordered correctly in relation to other facet providers.

And we add it to our world builder:
```java
    @Override
    protected WorldBuilder createWorld() {
        return new WorldBuilder(worldGeneratorPluginLibrary)
                .addProvider(new SurfaceProvider())
                .addProvider(new SeaLevelProvider(0))
                .addProvider(new MountainsProvider())
                .addRasterizer(new TutorialWorldRasterizer());
    }
```

And now we have the cliffs of insanity!

![image](https://raw.githubusercontent.com/Terasology/TutorialWorldGeneration/master/images/Facet%20Modification.png)

[Next: Facet Configuration](Facet-Configuration)
