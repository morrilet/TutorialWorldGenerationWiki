There is an easy solution to our houses with missing walls.  Borders!  Borders will effectively extend the range of our facet data into the neighboring chunk.  That way when we are rasterizing our houses,  we can know where the locations of nearby houses are.

We need to change our ```HouseProvider' to grab a larger surface height area so that we can still put the houses at the right height.

```java
@Produces(HouseFacet.class)
@Requires(@Facet(value = SurfaceHeightFacet.class, border = @FacetBorder(bottom = 9, sides = 4)))
public class HouseProvider implements FacetProvider {
```

Now if you remember, we have this line further down ```Border3D border = region.getBorderForFacet(HouseFacet.class);``` which asks the region to get an appropriate border for the particular facet.  Luckily we have done this same procedure in the ```SurfaceProvider``` and the facet system knows that you are requesting some padding on the sides and bottom of the facet data.

Accomodating this newly found padding on our facet data in our ```HouseRasterizer``` is easy to do.  All we must do is iterate through the facet's region as apposed to the normal chunk region.
```java
 @Override
    public void generateChunk(CoreChunk chunk, Region chunkRegion) {
        HouseFacet houseFacet = chunkRegion.getFacet(HouseFacet.class);
        for (Vector3i position : houseFacet.getWorldRegion()) {
```

Woot!  Now, *all* our houses have no way of entry or exit... oh wait. <grabs pickaxe>

![image](https://raw.githubusercontent.com/Terasology/TutorialWorldGeneration/master/images/Borders.png)
