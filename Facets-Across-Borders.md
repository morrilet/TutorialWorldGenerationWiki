# Complex shapes and Facets across borders

Many people have issues understanding what's going on with the facets across borders.
This is a short tutorial to help you understand it better, pictures included in the following tutorials.
First of all, you need to understand the structure of how the world generation works. The Terasology developers have made a clear structure in this, so everyone is able to add new and great world generators. You have:

* Facets
* FacetProviders
* Rasterizers


**What is a Facet?**
A facet, is a side of a many-sided object. Take for instance a map of the Earth. You have a facet (or side) of the Earth that's all about population. Meanwhile, you also have a facet that's about differences in species across the globe!
A facet is one of these things. What does this have to do with our dear game Terasology you ask? Well, 3D-worlds are as you know not plain 3D fields with only dirt! The worlds contain objects, have properties and so on. One 'facet' of the world might be the biomes. Another one could be where that certain fruit grows.

**What is a FacetProvider?**
A FacetProvider is, as the name itself says, a class that providers the world with the appropriate facet for a certain region. What you typically do in here is scan through the blocks and check for a condition. If that condition is true, you do something, such as placing a facet on that particular coordinate which you scanned through.

**What is a Rasterizer?**
This is where it gets intresting. A rasterizer is what interprets the Facet data. The rasterizer is where you place the blocks in the world.

Simple enough? Well, let's get started with some tips on how to make some more complex shapes, such as pyramids.
Let us start with a problem definition, what do we want?
> We want to have a good looking pyramid to show off to my friends. It must have some sort of path to get to the center. No more than a few pyramids should spawn on a short distance, so it doesn't get too crowded. We also want to make sure that the pyramid isn't floating!

Okay. That seems a whole lot of work. First things first.
We should start out with creating our classes. (Try to find the solution on these issues yourself first before copying it from here! It's the best practise!)

**Our Facet**
```java
public class TempleFacet extends SparseObjectFacet3D<Temple> {

    public TempleFacet(Region3i targetRegion, Border3D border) {
        super(targetRegion, border);
    }
}
```



**Our FacetProvider**
```java
@Requires(@Facet(value = SurfaceHeightFacet.class, border = @FacetBorder(sides = 28, bottom = 28, top = 28)))
@Produces(TempleFacet.class)
public class TempleFacetProvider implements FacetProvider {
    private WhiteNoise noise;

    @Override
    public void process(GeneratingRegion region) {
    }

    @Override public void setSeed(long seed) {
    }
}
```

```java
public class TempleRasterizer implements WorldRasterizer {

    public static int getSize() {
    }

    @Override
    public void initialize() {
    }

    @Override
    public void generateChunk(CoreChunk chunk, Region chunkRegion) {
        
    }
}

```

Now we have set up you can continue.
We only have to change two classes basically, the provider and the rasterizer.

_**Adding the Meat!**_
**The TempleFacetProvider**

```java



@Requires(@Facet(value = SurfaceHeightFacet.class, border = @FacetBorder(sides = 28, bottom = 28, top = 28)))
@Produces(TempleFacet.class)
public class TempleFacetProvider implements FacetProvider {
    private WhiteNoise noise;

    @Override
    public void process(GeneratingRegion region) {
        //This gets the border of the TempleFacet and adds 30 by it on the sides/top/bottom.
        Border3D border = region.getBorderForFacet(TempleFacet.class).extendBy(30, 30, 30);
        //Creates a temple facet with the specified region and borders.
        TempleFacet templeFacet = new TempleFacet(region.getRegion(), border);
        //This takes the SurfaceHeightFacet from the region. It's used to get the correct surface height. (Obviously!)
        SurfaceHeightFacet facet = region.getRegionFacet(SurfaceHeightFacet.class);
        //Takes a 2D representation of the world. Like a map of the Earth!! (Important piece!!)
        Rect2i worldRegion = facet.getWorldRegion();
        //Loops through the contents of this rectangle.
        for (int wz = worldRegion.minY(); wz <= worldRegion.maxY(); wz++) {
            for (int wx = worldRegion.minX(); wx <= worldRegion.maxX(); wx++) {
                //Gets the surface height (Also obvious)
                int surfaceHeight = TeraMath.floorToInt(facet.getWorld(wx, wz));
                //Checks if the surface height is within the boundries.
                if (surfaceHeight >= templeFacet.getWorldRegion().minY() &&
                    surfaceHeight <= templeFacet.getWorldRegion().maxY()) {
                    //TODO: add explanation for Noise.
                    if (noise.noise(wx, wz) > 0.9999) {
                        templeFacet.setWorld(wx, surfaceHeight, wz, new Temple());
                    }
                }
            }
        }
        //Adds the TempleFacet to the region.
        region.setRegionFacet(TempleFacet.class, templeFacet);    
    }

    @Override public void setSeed(long seed) {
        //TODO ADD LINK TO NOISE-explanation
        noise = new WhiteNoise(seed);
    }
}
```







