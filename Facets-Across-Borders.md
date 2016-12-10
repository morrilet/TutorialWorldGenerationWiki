Many people have issues understanding what's going on with the facets across borders.
This is a short tutorial to help you understand it better, pictures included.
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
This is where it gets intresting. A rasterizer is what interprets the data.



