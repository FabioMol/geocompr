
# (PART) Extensions {-}

# Making maps with R {#adv-map}

## Prerequisites {-}

- This chapter requires the following packages that we have already been using:


```r
library(sf)
library(raster)
library(tidyverse)
library(spData)
library(spDataLarge)
```

- In addition it uses the following visualization packages:


```r
library(tmap)    # for static and interactive maps
library(leaflet) # for interactive maps
library(mapview) # for interactive maps
library(shiny)   # for web applications
```

## Introduction

A satisfying and important aspect of geographic research is producing and communicating the results in the form of maps.
Map making --- the art of Cartography --- is an ancient skill that involves precision, consideration of the map-reader and often an element of creativity.
Basic plotting of geographic data is straightforward in R with `plot()` (see section \@ref(basic-map)), and it is possible to create advanced maps using base R methods [@murrell_r_2016].

This chapter focuses on dedicated map-making packages, especially **tmap**, for reasons that are explained in section \@ref(static-maps).
There are a multitude of options but when learning a new skill (in this case map making), it makes sense to gain depth-of-knowledge in one package before branching out, a concept that also applies to the choice of programming language as we saw in Chapter \@ref(intro).
It is worth developing advanced map making skills not only because it is a fun activity that can produce beautiful results. 
Map making also has important practical applications.
A carefully crafted map can help ensure that time spent in the analysis phases of geocomputational projects --- for example using methods covered in chapters \@ref(spatial-class) to \@ref(reproj-geo-data) --- are communicated effectively [@brewer_designing_2015]:

> Amateur-looking maps can undermine your audience’s ability to understand important information and weaken the presentation of a professional data investigation.

<!-- Todo: consider adding footnote saying it's good to focus on visualization early as in R4DS but that we cover it later because there's a risk of getting distracted by pretty pictures to the detriment of good analysis. -->

Maps have been used for several thousand years for a wide variety of purposes.
Historic examples include maps of buildings and land ownership in the Old Babylonian dynasty more than 3000 years ago and Ptolemy's world map in his masterpiece *Geography* nearly 2000 years ago [@talbert_ancient_2014].
However, maps have historically been out of reach for everyday people.

Modern computing has the potential to change this.
Map making skills can also help meet research and public engagement objectives.
From a research perspective clear maps are often the best way to present the results of geocomputational research.
From policy and 'citizen science' perspectives, attractive and engaging maps can help change peoples' minds, based on the evidence.
Map making is therefore a critical part of geocomputation and its emphasis not only on describing, but also *changing* the world (see Chapter \@ref(intro)).

<!-- info about relation between efficiency and editability -->
<!-- intro to the chapter structure -->

## Static maps

Static maps are the most common type of visual output from geocomputation.
Fixed images for printed outputs, common formats for static maps include `.png` and `.pdf`, for raster and vector outputs, respectively (interactive maps are covered in section \@ref(interactive-maps)).
Initially static maps were the *only* type of map that R could produce.
Things have advanced greatly since **sp** was realeased [see @pebesma_classes_2005].
Although many new techniques for geographic data visualization have been developed and demonstrated since then, a decade later static plotting (and some animations) was still the emphasis of geographic data visualisation in R [@cheshire_spatial_2015].

Despite the innovation of interactive mapping in R, static maps are still the foundation of mapping in R.
The generic `plot()` function is often the fastest way to create static maps from vector and raster spatial objects, as shown in sections \@ref(basic-map) and \@ref(basic-map-raster).
Sometimes simplicity and speed are priorities, especially during the development phase of a project, and this is where `plot()` excels.
The base R approach is also extensible, with `plot()` offering dozens of arguments and the **grid** providing functions for low-level control of graphical outputs, --- see *R Graphics* [@murrell_r_2016], especially  Chapter [14](https://www.stat.auckland.ac.nz/~paul/RG2e/chapter14.html).
The focus of this section, however, is making static maps with **tmap**.

Why **tmap**?
It is a powerful and flexible map-making package with sensible defaults.
It has a concise syntax that allows for the creation of attractive maps with minimal code, that will be familiar to **ggplot2** users.
Furthermore, **tmap** has a unique capability to generate static and interactive maps using the same code via `tmap_mode()`.
It accepts a wider range of spatial classes (including `raster` objects) than alternatives such as **ggplot2**, as documented in vignettes [`tmap-nutshell`](https://cran.r-project.org/web/packages/tmap/vignettes/tmap-nutshell.html) and [`tmap-modes`](https://cran.r-project.org/web/packages/tmap/vignettes/tmap-modes.html) and an academic paper on the subject [@tennekes_tmap_2018].
This section teaches how to make static maps with **tmap**, emphasizing the important aesthetic and layout options.

### tmap basics

Like **ggplot2**, **tmap** is based on the idea of a 'grammar of graphics' [@wilkinson_grammar_2005].
This involves a separation between the input data and the aesthetics (how data are visualised): each input dataset can be 'mapped' in a range of different ways including location on the map (defined by data's `geometry`), color, and other visual variables.
The basic building block is `tm_shape()` (which defines input data, raster and vector objects), followed by one or more layer elements such as `tm_fill()` and `tm_dots()`.
This layering is demonstrated in the chunk below, which generates the maps presented in Figure \@ref(fig:tmshape):


```r
# Add fill layer to nz shape
tm_shape(nz) +
  tm_fill() 
# Add border layer to nz shape
tm_shape(nz) +
  tm_borders() 
# Add fill and border layers to nz shape
tm_shape(nz) +
  tm_fill() +
  tm_borders() 
```

<div class="figure" style="text-align: center">
<img src="figures/tmshape-1.png" alt="New Zealand's shape plotted with fill (left), border (middle) and fill and border (right) layers added using tmap functions." width="576" />
<p class="caption">(\#fig:tmshape)New Zealand's shape plotted with fill (left), border (middle) and fill and border (right) layers added using tmap functions.</p>
</div>

The object passed to `tm_shape()` in this case is `nz`, an `sf` object representing the regions of New Zealand (see section \@ref(intro-sf) for more on `sf` objects).
Layers are added to represent `nz` visually, with `tm_fill()` and `tm_borders()` creating shaded areas (left panel) and border outlines (middle panel) in Figure \@ref(fig:tmshape), respectively.

This is an intuitive approach to map making:
the common task of *adding* new layers is undertaken by the addition operator `+`, followed by `tm_*()`.
The asterisk (\*) refers to a wide range of layer types which have self-explanatory names including `fill`, `borders` (demonstrated above), `bubbles`, `text` and `raster`
(see the help page [`tmap-element`](https://www.rdocumentation.org/packages/tmap/versions/2.0/topics/tmap-element) for a full list).
This layering is illustrated in the right panel of Figure \@ref(fig:tmshape), the result of adding a border *on top of* the fill layer:
note the `tm_borders()` call *after* `tm_fill() +` in the previous code chunk.

\BeginKnitrBlock{rmdnote}<div class="rmdnote">`qtm()` is a handy function for **q**uickly creating **t**map **m**aps (hence the snappy name).
It is concise and provides a good default visualization in many cases:
`qtm(nz)`, for example, is equivalent to `tm_shape(nz) + tm_fill() + tm_borders()`.
Further, layers can be added concisely using multiple `qtm()` calls, such as `qtm(nz) + qtm(nz_height)`.
The disadvantage is that it makes aesthetics of individual layers harder to control, explaining why we avoid teaching it in this chapter.</div>\EndKnitrBlock{rmdnote}

### Map objects {#map-obj}

A useful feature of **tmap** is its ability to store *objects* representing maps.
The code chunk below demonstrates this by saving the last plot in Figure \@ref(fig:tmshape) as an object of class `tmap` (note the use of `tm_polygons()` which condenses `tm_fill()  + tm_borders()` into a single function):


```r
map_nz = tm_shape(nz) + tm_polygons()
class(map_nz)
#> [1] "tmap"
```

`map_nz` can be plotted later, for example by adding additional layers (as shown below) or simply running `map_nz` in the console, which is equivalent to `print(map_nz)`.

New *shapes* can be added with `+ tm_shape(new_obj)`.
In this case `new_obj` represents a new spatial object to be plotted on top of preceding layers.
When a new shape is added in this way all subsequent aesthetic functions refer to it, until another new shape is added.
This syntax allows the creation of maps with multiple shapes and layers, as illustrated in the next code chunk which uses the function `tm_raster()` to plot a raster layer (with `alpha` set to make the layer semi-transparent):


```r
map_nz1 = map_nz +
  tm_shape(nz_elev) + tm_raster(alpha = 0.7)
```

Building on the previously created `map_nz` object, the preceding code creates a new map object `map_nz1` that contains another shape (`nz_eleve`) representing average elevation across New Zealand (see Figure \@ref(fig:tmlayers), left).
More shapes and layers can be added, as illustrated in the code chunk below which creates `nz_water`, representing New Zealand's [territorial waters](https://en.wikipedia.org/wiki/Territorial_waters), and adds the resulting lines to an existing map object.


```r
nz_water = st_union(nz) %>% st_buffer(22200) %>% 
  st_cast(to = "LINESTRING")
map_nz2 = map_nz1 +
  tm_shape(nz_water) + tm_lines()
```

There is no limit to the number of layers or shapes that can be added to `tmap` objects.
The same shape can even be used multiple times.
The final map illustrated in Figure \@ref(fig:tmlayers) is created by adding a layer representing high points (stored in the object `nz_height`) onto the previously created `map_nz2` object with `tm_dots()` (see `?tm_dots` and `?tm_bubbles` for details on **tmap**'s point plotting functions).
The resulting map, which has four layers, is illustrated in the right-hand panel of:


```r
map_nz3 = map_nz2 +
  tm_shape(nz_height) + tm_dots()
```

A useful and little known feature of **tmap** is that multiple map objects can be arranged in a single 'metaplot' with `tmap_arrange()`.
This is demonstrated in the code chunk below which plots `map_nz1` to `map_nz3`, resulting in Figure \@ref(fig:tmlayers).


```r
tmap_arrange(map_nz1, map_nz2, map_nz3)
```

<div class="figure" style="text-align: center">
<img src="figures/tmlayers-1.png" alt="Maps with additional layers added to the final map of Figure 9.1." width="576" />
<p class="caption">(\#fig:tmlayers)Maps with additional layers added to the final map of Figure 9.1.</p>
</div>

Additional elements can also be added with the `+` operator.
Aesthetic settings, however, are controlled by arguments to layer functions.

### Aesthetics

The plots in the previous section demonstrate **tmap**'s default aesthetic settings.
Gray shades are used for `tm_fill()` and  `tm_bubbles()` layers and a continuous black line is used to represent lines created with `tm_lines()`.
Of course, these default values and other aesthetics can be overridden.
The purpose of this section is to show how.

There are two main types of map aesthetics: those that change with the data and those that are constant.
Unlike **ggplot2**, which uses the helper function `aes()` to represent variable aesthetics, **tmap** accepts aesthetic arguments that are either variable fields (based on column names) or constant values.^[
If there is a clash between a fixed value and a column name the column name takes precedence. This can be verified by running the next code chunk after running `nz$red = 1:nrow(nz)`.
]
The most commonly used aesthetics for fill and border layers include color, transparency, line width and line type, set with `col`, `alpha`, `lwd`, and `lty` arguments respectively.
The impact of setting these with fixed values is illustrated in Figure \@ref(fig:tmstatic).


```r
ma1 = tm_shape(nz) + tm_fill(col = "red")
ma2 = tm_shape(nz) + tm_fill(col = "red", alpha = 0.3)
ma3 = tm_shape(nz) + tm_borders(col = "blue")
ma4 = tm_shape(nz) + tm_borders(lwd = 3)
ma5 = tm_shape(nz) + tm_borders(lty = 2)
ma6 = tm_shape(nz) + tm_fill(col = "red", alpha = 0.3) +
  tm_borders(col = "blue", lwd = 3, lty = 2)
tmap_arrange(ma1, ma2, ma3, ma4, ma5, ma6)
```

<div class="figure" style="text-align: center">
<img src="figures/tmstatic-1.png" alt="The impact of changing commonly used fill and border aesthetics to fixed values." width="576" />
<p class="caption">(\#fig:tmstatic)The impact of changing commonly used fill and border aesthetics to fixed values.</p>
</div>




Like base R plots, arguments defining aesthetics can also receive values that vary.
Unlike the base R code below (which generates the left panel in Figure \@ref(fig:tmcol)), **tmap** aesthetic arguments will not accept a numeric vector:


```r
plot(st_geometry(nz), col = nz$Land_area)  # works
tm_shape(nz) + tm_fill(col = nz$Land_area) # fails
#> Error: Fill argument neither colors nor valid variable name(s)
```

Instead `col` (and other aesthetics that can vary such as `lwd` for line layers and `size` for point layers) requires a character string naming an attribute associated with the geometry to be plotted.
Thus, one would achieve the desired result as follows (plotted in the right-hand panel of Figure \@ref(fig:tmcol)):


```r
tm_shape(nz) + tm_fill(col = "Land_area")
```

<div class="figure" style="text-align: center">
<img src="figures/tmcol-1.png" alt="Comparison of base (left) and tmap (right) handling of a numeric color field." width="45%" /><img src="figures/tmcol-2.png" alt="Comparison of base (left) and tmap (right) handling of a numeric color field." width="45%" />
<p class="caption">(\#fig:tmcol)Comparison of base (left) and tmap (right) handling of a numeric color field.</p>
</div>

An important argument in functions defining aesthetic layers such as `tm_fill()` is `title`, which sets the title of the associated legend.
The following code chunk demonstrates this functionality by providing a more attractive name than the variable name `Land_area` (note the use of `expression()` to create superscript text):


```r
legend_title = expression("Area (km"^2*")")
map_nza = tm_shape(nz) +
  tm_fill(col = "Land_area", title = legend_title) + tm_borders()
```

### Color settings

Color settings are an important part of map design.
They can have a major impact on how spatial variability is portrayed as illustrated in Figure \@ref(fig:tmpal).
This shows four ways of coloring regions in New Zealand depending on median income, from left to right (and demonstrated in the code chunk below):

- The default setting uses 'pretty' breaks, described in the next paragraph
- `breaks` which allows you to manually set the breaks
- `n` which sets the number of bins into which numeric variables are categorized
- `palette` which defines the color scheme, for example `RdBu`


```r
breaks = c(0, 3, 4, 5) * 10000
tm_shape(nz) + tm_polygons(col = "Median_income")
tm_shape(nz) + tm_polygons(col = "Median_income", breaks = breaks)
tm_shape(nz) + tm_polygons(col = "Median_income", n = 10)
tm_shape(nz) + tm_polygons(col = "Median_income", palette = "RdBu")
```

<div class="figure" style="text-align: center">
<img src="figures/tmpal-1.png" alt="Illustration of settings that affect color settings. The results show (from left to right): default settings, manual breaks, n breaks, and the impact of changing the palette." width="576" />
<p class="caption">(\#fig:tmpal)Illustration of settings that affect color settings. The results show (from left to right): default settings, manual breaks, n breaks, and the impact of changing the palette.</p>
</div>

Another way to change color settings is by altering color break (or bin) settings.
In addition to manually setting `breaks` **tmap** allows users to specify algorithms to automatically create breaks with the `style` argument.
Six of the most useful break styles are illustrated in Figure \@ref(fig:break-styles) and described in the bullet points below:

- `style = pretty`, the default setting, rounds breaks into whole numbers where possible and spaces them evenly
- `style = equal` divides input values into bins of equal range, and is appropriate for variables with a uniform distribution (not recommended for variables with a skewed distribution as the resulting map may end-up having little color diversity)
- `style = quantile` ensures the same number of observations fall into each category (with the potential down side that bin ranges can vary widely)
- `style = jenks` identifies groups of similar values in the data and maximizes the differences between categories
- `style = cont` (and `order`) present a large number of colors over continuous color field, and are particularly suited for continuous rasters (`order` can help visualize skewed distributions)
- `style = cat` was designed to represent categorical values and assures that each category receives a unique color

<div class="figure" style="text-align: center">
<img src="figures/break-styles-1.png" alt="Illustration of different binning methods set using the style argument in tmap." width="576" />
<p class="caption">(\#fig:break-styles)Illustration of different binning methods set using the style argument in tmap.</p>
</div>

\BeginKnitrBlock{rmdnote}<div class="rmdnote">Although `style` is an argument of **tmap** functions it in fact originates as an argument in `classInt::classIntervals()` --- see the help page of this function for details.</div>\EndKnitrBlock{rmdnote}

Palettes define the color ranges associated with the bins and  determined by the `breaks`, `n`, and `style` arguments described above.
The default color palette is specified in `tm_layout()` (see section \@ref(layouts) to learn more), however, it could be quickly changed using the `palette` argument.
It expects a vector of colors or a new color palette name, which can be selected interactively with `tmaptools::palette_explorer()`.
You can add a `-` as prefix to reverse the palette order.

There are three main groups of color palettes - categorical, sequential and diverging (Figure \@ref(fig:colpal)), and each of them serves a different purpose.
Categorical palettes consist of easily distinguishable colors and are most appropriate for categorical data without any particular order such as state names or land cover classes.
Colors should be intuitive: rivers should be blue, for example, and pastures green.
Avoid too many categories: maps with large legends and many colors can be uninterpretable.^[
`col = "MAP_COLORS"` can be used in maps with a large number of individual polygons (for example a map of individual countries) to create unique colors for adjacent polygons.
] 

The second group is sequential palettes.
These follow a gradient, for example from light to dark colors (light colors tend to represent lower values), and are appropriate for continuous (numeric) variables.
Sequential palettes can be single (`Blues` go from light to dark blue for example) or multi-color/hue (`YlOrBr` is gradient from light yellow to brown via orange, for example), as demonstrated in the code chunk below --- output not shown, run the code yourself to see the results!


```r
tm_shape(nz) + tm_polygons("Population", palette = "Blues")
tm_shape(nz) + tm_polygons("Population", palette = "YlOrBr")
```

The last group, diverging palettes, typically range between three distinct colors (purple-white-green in Figure \@ref(fig:colpal)) and are usually created by joining two single color sequential palettes with the darker colors at each end.
Their main purpose is to visualize the difference from an important reference point, e.g. a certain temperature, the median household income or the mean probability for a drought event.
The reference point's value can be adjusted in **tmap** using the `midpoint` argument.

<div class="figure" style="text-align: center">
<img src="figures/colpal-1.png" alt="Examples of categorical, sequential and diverging palettes." width="50%" />
<p class="caption">(\#fig:colpal)Examples of categorical, sequential and diverging palettes.</p>
</div>

There are two important principles for consideration when working with colors - perceptibility and accessibility.
Firstly, colors on maps should match our perception. 
This means that certain colors are viewed through our experience and also cultural lenses.
For example, green colors usually represent vegetation or lowlands and blue is connected with water or cool.
Color palettes should also be easy to understand to effectively convey information.
It should be clear which values are lower and which are higher, and colors should change gradually.
This property is not preserved in the rainbow color palette, therefore we suggest avoiding it in spatial data visualization [@borland_rainbow_2007].
Instead [the viridis color palettes](https://cran.r-project.org/web/packages/viridis/), also available in **tmap**, can be used.
Secondly, changes in colors should be accessible to the largest number of people.
Therefore, it is important to use colorblind friendly palettes as often as possible.^[See the "Color blindness simulator" options in `tmaptools::palette_explorer()`.]

### Layouts

The map layout refers to the combination of all map elements into a cohesive map.
Map elements include among others the objects to be mapped, the title, the scale bar, margins and aspect ratios, while the color settings covered in the previous section relate to the palette and break-points used to affect how the map looks.
Both may result in subtle changes that can have an equally large impact on the impression left by your maps.

Additional elements such as north arrows and scale bars have their own functions - `tm_compass()` and `tm_scale_bar()` (Figure \@ref(fig:na-sb)).


```r
map_nz + 
  tm_compass(type = "8star", position = c("left", "top")) +
  tm_scale_bar(breaks = c(0, 100, 200), size = 1)
```

<div class="figure" style="text-align: center">
<img src="figures/na-sb-1.png" alt="Map with additional elements - a north arrow and scale bar." width="576" />
<p class="caption">(\#fig:na-sb)Map with additional elements - a north arrow and scale bar.</p>
</div>

**tmap** also allows a wide variety of layout settings to be changed, some of which are illustrated in Figure \@ref(fig:layout1), produced using the following code (see `args(tm_layout)` or `?tm_layout` for a full list):


```r
map_nz + tm_layout(title = "New Zealand")
map_nz + tm_layout(scale = 5)
map_nz + tm_layout(bg.color = "lightblue")
map_nz + tm_layout(frame = FALSE)
```

<div class="figure" style="text-align: center">
<img src="figures/layout1-1.png" alt="Layout options specified by (from left to right) title, scale, bg.color and frame arguments." width="576" />
<p class="caption">(\#fig:layout1)Layout options specified by (from left to right) title, scale, bg.color and frame arguments.</p>
</div>

The other arguments in `tm_layout()` provide control over many more aspects of the map in relation to the canvas on which it is placed.
Some useful layout settings are listed below (see Figure \@ref(fig:layout2) for illustrations of a selection of these):

- Frame width (`frame.lwd`) and an option to allow double lines (`frame.double.line`).
- Margin settings including `outer.margin` and `inner.margin`.
- Font settings, controlled by `fontface` and `fontfamily`.
- Legend settings including binary options such as `legend.show` (whether or not to show the legend) `legend.only` (omit the map) and `legend.outside` (should the legend go outside the map?), as well as multiple choice settings such as `legend.position`.
- Default colors of aesthetic layers (`aes.color`), map attributes such as the frame (`attr.color`).
- Color settings controlling `sepia.intensity` (how yellowy the map looks) and `saturation` (a color-grayscale).

<div class="figure" style="text-align: center">
<img src="figures/layout2-1.png" alt="Illustration of selected layout options." width="576" />
<p class="caption">(\#fig:layout2)Illustration of selected layout options.</p>
</div>

The impact of changing the color settings listed above is illustrated in Figure \@ref(fig:layout3) (see `?tm_layout` for a full list).

<div class="figure" style="text-align: center">
<img src="figures/layout3-1.png" alt="Illustration of selected color-related layout options." width="576" />
<p class="caption">(\#fig:layout3)Illustration of selected color-related layout options.</p>
</div>

<!-- Maybe reverse order and progress from the high- to the low-level control. Overall, this section reads a bit like a vignette. Is this the intention. -->
Beyond the low-level control over layouts and colors, **tmap** also offers high-level styles, using the `tm_style()` function (representing the second meaning of 'style' in the package).
Some styles such as `tm_style("cobalt")` result in stylized maps while others such as `tm_style("gray")` make more subtle changes, as illustrated in Figure \@ref(fig:tmstyles), created using code below (see `09-tmstyles.R`):


```r
map_nza + tm_style("bw")
map_nza + tm_style("classic")
map_nza + tm_style("cobalt")
map_nza + tm_style("col_blind")
```

<div class="figure" style="text-align: center">
<img src="figures/tmstyles-1.png" alt="Selected tmap styles: bw, classic, cobalt and col_blind (from left to right)." width="576" />
<p class="caption">(\#fig:tmstyles)Selected tmap styles: bw, classic, cobalt and col_blind (from left to right).</p>
</div>

\BeginKnitrBlock{rmdnote}<div class="rmdnote">A preview of predefined styles can be generated by executing `tmap_style_catalogue()`.
This creates a folder called `tmap_style_previews` containing nine images.
Each image, from `tm_style_albatross.png` to `tm_style_white.png` shows a faceted map of the world in the corresponding style.
Note: `tmap_style_catalogue()` takes some time to run.</div>\EndKnitrBlock{rmdnote}

### Faceted maps

Faceted maps, also referred to as 'small multiples', are composed of many maps arranged side-by-side, and sometimes stacked vertically.
Facets enable the visualization of how spatial relationships change with respect to another variable, such as time.
The changing populations of settlements, for example, can be represented in a faceted map with each panel representing the population at a particular moment in time.
The time dimension could be represented via another *aesthetic* such as color.
However, this risks cluttering the map because it will involve multiple overlapping points (cities do not tend to move over time!).

Typically all individual facets in a faceted map contain the same geometry data repeated multiple times, once for each column in the attribute data (this is the default plotting method for `sf` objects, see \@ref(spatial-class)).
However, facets can also represent shifting geometries such as as the evolution of a point pattern over time.
This use case of faceted plot is illustrated in Figure \@ref(fig:urban-facet).

<!-- todo: describe data type (long) and nrow vs ncol -->


```r
urb_1970_2030 = urban_agglomerations %>% 
  filter(year %in% c(1970, 1990, 2010, 2030))
tm_shape(world) + tm_polygons() + 
  tm_shape(urb_1970_2030) + tm_symbols(col = "black", border.col = "white",
                                       size = "population_millions") +
  tm_facets(by = "year", nrow = 2, free.coords = FALSE)
```

<div class="figure" style="text-align: center">
<img src="figures/urban-facet-1.png" alt="Faceted map showing the top 30 largest 'urban agglomerations' from 1970 to 2030 based on population projects by the United Nations." width="576" />
<p class="caption">(\#fig:urban-facet)Faceted map showing the top 30 largest 'urban agglomerations' from 1970 to 2030 based on population projects by the United Nations.</p>
</div>

The preceding code chunk demonstrates key features of faceted maps created with **tmap**:

- Shapes that do not have a facet variable are repeated (the countries in `world` in this case).
- The `by` argument which varies depending on a variable (`year` in this case).
- `nrow`/`ncol` setting specifying the number of rows and columns that facets should be arranged into.
- The `free.coords`-parameter specifying if each map has its own bounding box.

In addition to their utility for showing changing spatial relationships, faceted maps are also useful as the foundation for animated maps (see \@ref(animated-maps)).

### Inset maps

An inset map is a smaller map rendered within or next to the main map. 
It could serve many different purposes, including providing a context (Figure \@ref(fig:insetmap1)) or bringing some non-contiguous regions closer to ease their comparison (Figure \@ref(fig:insetmap2)).
They could be also used to focus on a smaller area in more detail or to cover the same area as the map but representing a different topic.

In the example below, we create a map of the central part of the New Zealand's Southern Alps.
Our inset map will show where the main map is in relation to the whole New Zealand.
The first step is to define the area of interest, which can be done by creating a new spatial object, `nz_region`.
<!--# mapview::mapview(nz_height, native.crs = TRUE) or mapedit??-->


```r
nz_region = st_bbox(c(xmin = 1340000, xmax = 1450000,
                      ymin = 5130000, ymax = 5210000)) %>% 
  st_as_sfc() %>% 
  st_set_crs(st_crs(nz_height))
```

In the second step, we create a base map showing the New Zealand's Southern Alps area. 
This is a place where the most important message is stated. 


```r
nz_height_map = tm_shape(nz_elev, bbox = tmaptools::bb(nz_region)) +
  tm_raster(style = "cont", palette = "YlGn", legend.show = TRUE) +
  tm_shape(nz_height) + tm_symbols(shape = 2, col = "red", size = 1) +
  tm_scale_bar(position = c("left", "bottom"))
```

The third step consists of the inset map creation. 
It gives a context and helps to locate the area of interest. 
Importantly, this map needs to clearly indicate the location of the main map, for example by stating its borders.


```r
nz_map = tm_shape(nz) + tm_polygons() +
  tm_shape(nz_height) + tm_symbols(shape = 2, col = "red", size = 0.1) + 
  tm_shape(nz_region) + tm_borders(lwd = 3) 
```

Finally, we combine the two maps.
A viewport from the **grid** package can be used by stating a center location (`x` and `y`) and a size (`width` and `height`) of the inset map.


```r
library(grid)
nz_height_map
print(nz_map, vp = viewport(0.8, 0.27, width = 0.5, height = 0.5))
```

<div class="figure" style="text-align: center">
<img src="figures/insetmap1-1.png" alt="Inset map providing a context - location of the central part of the Southern Alps in New Zealand." width="576" />
<p class="caption">(\#fig:insetmap1)Inset map providing a context - location of the central part of the Southern Alps in New Zealand.</p>
</div>

Inset map can be save to file either by using a graphic device (see section \@ref(visual-outputs)) or the `tmap_save()` function and its arguments - `insets_tm` and `insets_vp`.

Inset maps are also used to create one map of non-contiguous areas.
Probably, the most often used example is a map of United States, which consists of the contiguous United States, Hawaii and Alaska.
It is very important to find the best projection for each individual inset in these type of cases (see section \@ref(reproj-geo-data) to learn more).
We can use US National Atlas Equal Area for the map of the contiguous United States by putting its EPSG code in the `projection` argument of `tm_shape()`.


```r
us_states_map = tm_shape(us_states, projection = 2163) + tm_polygons() + 
  tm_layout(frame = FALSE)
```

The rest of our objects, `hawaii` and `alaska`, already have proper projections, therefore we just need to create two separate maps:


```r
hawaii_map = tm_shape(hawaii) + tm_polygons() + 
  tm_layout(title = "Hawaii", frame = FALSE, bg.color = NA, 
            title.position = c("left", "bottom"))
alaska_map = tm_shape(alaska) + tm_polygons() + 
  tm_layout(title = "Alaska", frame = FALSE, bg.color = NA)
```

The final map is created by combining and arranging these three maps:


```r
us_states_map
print(hawaii_map, vp = viewport(x = 0.4, y = 0.1, width = 0.2, height = 0.1))
print(alaska_map, vp = viewport(x = 0.15, y = 0.15, width = 0.3, height = 0.3))
```

<div class="figure" style="text-align: center">
<img src="figures/insetmap2-1.png" alt="Map of the United States." width="576" />
<p class="caption">(\#fig:insetmap2)Map of the United States.</p>
</div>

The code presented above is very compact and allows for creation of many similar maps, however the map do not represent sizes and locations of Hawaii and Alaska well.
See the [`us-map`](https://geocompr.github.io/geocompkg/articles/us-map.html) vignette from the **geocompkg** package for an alternative approach.

<!-- extended info about using tm_layout to show legend in main plot and remove it in the others -->
<!-- The main goal of this section is to present how to generate and arrange inset maps. -->
<!-- The next step is to use the knowledge from the previous sections to improve the map style or to add another data layers. -->
<!-- Moreover, the same skills can be applied to combine maps and plots. -->

## Animated maps

Faceted maps, described in \@ref(faceted-maps), provide a way of showing how spatial relationships vary but the approach has disadvantages.
Facets become tiny when many of them are squeezed into a single plot, potentially hiding *any* spatial relationships.
Furthermore the fact that each facet is separated by space on the screen or page means that subtle differences between facets can be hard to detect.

With an increasing proportion of communication happening via digital screens, animated maps are becoming more popular.
Animated maps can even be useful for paper reports: you can always link readers to a web-page containing an animated (or interactive) version of a printed map to help make it come alive.

Figure \@ref(fig:urban-animated) is a simple example of an animated map.
Unlike the faceted plot it does not squeeze multiple maps into a single screen and allows the reader to see how the spatial distribution of the worlds most populous agglomerations evolve over time (see the book's website for the animated version).

<div class="figure" style="text-align: center">
<img src="figures/urban-animated.gif" alt="Animated map showing the top 30 largest 'urban agglomerations' from 1950 to 2030 based on population projects by the United Nations."  />
<p class="caption">(\#fig:urban-animated)Animated map showing the top 30 largest 'urban agglomerations' from 1950 to 2030 based on population projects by the United Nations.</p>
</div>



The animated map illustrated in Figure \@ref(fig:urban-animated) can be created using the same **tmap** techniques that generates faceted maps, demonstrated in section \@ref(faceted-maps).
There are two differences, however, related to arguments in `tm_facets()`:

- `along = "year"` is used instead of `by = "year"`
- `free.coords = FALSE`, which maintains the map extent for each map iteration.

These additional arguments are demonstrated in the subsequent code chunk:


```r
urb_anim = tm_shape(world) + tm_polygons() + 
  tm_shape(urban_agglomerations) + tm_dots(size = "population_millions") +
  tm_facets(along = "year", free.coords = FALSE)
```

The resulting `urb_anim` represents a set of separate maps for each year.
The final stage is to combine them and save the result as a `.gif` file with `tmap_animation()`.
The following command creates the animation illustrated in Figure \@ref(fig:urban-animated), with a few elements missing, that we will add-in during the exercises:


```r
tmap_animation(urb_anim, filename = "urb_anim.gif", delay = 25)
```

Another illustration of the power of animated maps is provided in Figure \@ref(fig:animus).
This shows the development of states in United States, which first formed in the East and then incrementally to the West and finally into the interior.
Code to reproduce this map can be found in the script `09-usboundaries.R`.



<div class="figure" style="text-align: center">
<img src="https://user-images.githubusercontent.com/1825120/38543030-5794b6f0-3c9b-11e8-9da9-10ec1f3ea726.gif" alt="Animated map showing population growth and state formation and boundary changes in the United States, 1790-2010."  />
<p class="caption">(\#fig:animus)Animated map showing population growth and state formation and boundary changes in the United States, 1790-2010.</p>
</div>

## Interactive maps

Static and animated maps can breathe life into geographic datasets.
Interactive maps can take reader participation to a whole new level, providing results with a life of their own that can be explored in myriad ways.
Interactivity can take many forms, including the appearance of popup messages when users click or mouse-over geographic features and maps dynamically linked to non-geographic plots.^[
A good example of such a dynamically linked visualization is a map of Nigeria linked to a bar chart of fertility rates by Kyle Walker which can be found online.
<!-- A good example of such a dynamically linked visualization is a map of Nigeria linked to a bar chart of fertility rates by Kyle Walker, published on [Twitter](https://twitter.com/kyle_e_walker/status/985948444966768641) and (for the source code) as a GitHub [Gist](https://gist.github.com/walkerke/5fe9a198a30270e2fcb8120a7fc8242a). -->
]

The most important type of interactivity, however, is the display of geographic data on an interactive or 'slippy' web maps.
The release of the **leaflet** package in 2015 revolutionized interactive web map creation from within R and a number of packages have built on these foundations adding new features (e.g. **leaflet.extras**) and making the creation of web maps as simple as creating static maps (e.g. **mapview** and **tmap**).
This section illustrates each approach in the opposite order.
We will explore how to make slippy maps with **tmap** (the syntax of which we have already learned), **mapview** and finally **leaflet** (which provides low level control over interactive maps).

A unique feature of **tmap** mentioned in section \@ref(static-maps) is its ability to create static and interactive maps using the same code.
Maps can be viewed interactively at any point by switching to view mode, using the command `tmap_mode("view")`.
This is demonstrated in the code below, which creates an interactive map of New Zealand based on the `tmap` object `map_nz`, created in section \@ref(map-obj), and illustrated in Figure \@ref(fig:tmview):


```r
tmap_mode("view")
map_nz
```

<div class="figure" style="text-align: center">
<!--html_preserve--><div id="htmlwidget-6365b7f0d3a261f1c13a" style="width:576px;height:355.968px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-6365b7f0d3a261f1c13a">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addProviderTiles","args":["Esri.WorldGrayCanvas",null,"Esri.WorldGrayCanvas",{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"pane":"tilePane"}]},{"method":"addProviderTiles","args":["OpenStreetMap",null,"OpenStreetMap",{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"pane":"tilePane"}]},{"method":"addProviderTiles","args":["Esri.WorldTopoMap",null,"Esri.WorldTopoMap",{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"pane":"tilePane"}]},{"method":"createMapPane","args":["overlayPane01",401]},{"method":"addPolygons","args":[[[[{"lng":[174.616638800656,174.562809050564,174.481748333157,174.338965900234,174.214573266902,174.128772750479,174.045997583891,174.095972133855,174.155368483074,174.181866683395,174.135350500256,174.040527550541,173.983675883808,173.86604691657,173.688098133553,173.365180583183,173.356613017331,173.054243634031,173.130559999733,173.168093182624,173.167447217184,173.081005749918,172.851222917828,172.653880649817,172.69995103369,172.793065450767,172.869035233581,173.014442316542,172.984791449998,173.025130983722,173.120204417268,173.199324916413,173.280772883247,173.214814899913,173.288398249192,173.340087699738,173.286914883417,173.429163483407,173.389709633848,173.43403079967,173.535566349955,173.562634800269,173.644032483284,173.701142200014,173.74008500064,173.841272884062,173.931277716701,173.963284550175,174.078539632729,174.079958382619,174.125462949978,174.28978491699,174.350169531639,174.339655300127,174.468392016881,174.474719383801,174.536753999552,174.50237205005,174.557601466643,174.555188500026,174.485563450384,174.42268665044,174.404830266529,174.500809283377,174.460144966703,174.510000500399,174.589780900475,174.616638800656],"lat":[-36.1175604665931,-36.1790001827569,-36.2282769999411,-36.3168297168814,-36.2587116660842,-36.1736123994487,-36.1701695501245,-36.2448879334642,-36.2760972327256,-36.3630358998114,-36.3920499169095,-36.3928830666096,-36.2337049996773,-36.0912001162441,-35.8924174497112,-35.5474016995122,-35.5228177334182,-35.1912381663484,-35.1789323328505,-35.1364273327493,-35.0363686494584,-34.8923919496606,-34.6297133663182,-34.4780076502037,-34.4263823994855,-34.4539942827601,-34.4145187168343,-34.4241368998054,-34.5278073326595,-34.6535190997575,-34.81668376714,-34.870968849818,-34.8925261172536,-34.9612557667465,-35.0132316332065,-34.9582312164506,-34.8634985660914,-34.8251486160696,-34.9534963329236,-34.9859361497159,-34.9984530665273,-34.921567966528,-34.9661940829229,-34.9524767161151,-35.0372064834482,-34.9943093161935,-35.0472986498438,-35.1188625998023,-35.1161315001253,-35.2730977500874,-35.2940340332248,-35.2751480001439,-35.3264187489317,-35.3768799004023,-35.490753766877,-35.5563464661221,-35.5876817834851,-35.6408338001947,-35.7240167331044,-35.8608910000564,-35.786374216361,-35.784832099836,-35.8310155171749,-35.8410631829848,-35.9145349498182,-36.0304654495667,-36.0481898660492,-36.1175604665931]}]],[[{"lng":[175.291304133085,175.152091033439,175.141120333576,175.048242617196,174.990258082555,174.868951232796,174.815488049849,174.659628367495,174.538637882794,174.627048466479,174.690634167279,174.73950153409,174.857652450143,174.826874817006,174.744100215986,174.681552100084,174.583827667051,174.48303651745,174.468611433661,174.424401866458,174.273615083721,174.16870360025,174.187919333768,174.235835399823,174.312745950162,174.404776383887,174.43132198402,174.414885550516,174.25745998445,174.257569883856,174.373455166346,174.481748333157,174.562809050564,174.616638800656,174.657346033553,174.745751216322,174.818184083076,174.734791817454,174.723020317237,174.6928154505,174.740610167352,174.775982149355,174.735215883483,174.880847016882,174.95133831739,175.055796349565,175.093513100132,175.194032000506,175.28332898404,175.291304133085],"lat":[-37.0240384665172,-37.0215583499183,-37.175603383291,-37.1822711996281,-37.2165854996348,-37.2449866002391,-37.2286308828648,-37.2924320832073,-37.0766324001782,-37.0367340832541,-37.1366490000563,-37.1262254668378,-37.0510942830302,-37.0129061168963,-36.9843338828504,-36.9338147160346,-37.0073111163499,-37.0480030494167,-36.9540757993866,-36.8242195164721,-36.6113436505545,-36.4944308500355,-36.4344940164907,-36.4280906168391,-36.5364925994457,-36.6242364999802,-36.5110054662043,-36.4036616161859,-36.3809034491112,-36.3259556823754,-36.3194753664587,-36.2282769999411,-36.1790001827569,-36.1175604665931,-36.1775323830102,-36.2569215827706,-36.2852678662079,-36.4030681168164,-36.5128217502045,-36.5757347661745,-36.6480035493648,-36.7735064498795,-36.8411249497158,-36.8523293329412,-36.8953845498422,-36.882600599391,-36.9457113831059,-36.934308932921,-36.9885057501472,-37.0240384665172]}],[{"lng":[175.397381334664,175.42270826654,175.48369956673,175.479485900625,175.5432121837,175.525421999774,175.437765849463,175.433490317009,175.350243184351,175.364852117959,175.326193733308,175.341795483261,175.397381334664],"lat":[-36.0497132993229,-36.1291887990059,-36.1921515500045,-36.2444036833781,-36.2952009834002,-36.3476168495328,-36.3075231663546,-36.2724128496224,-36.228536449767,-36.182237616875,-36.138272981851,-36.0685923493946,-36.0497132993229]}],[{"lng":[175.172594933364,175.14511235048,175.066499350382,175.038030716822,175.172594933364],"lat":[-36.7386591493628,-36.8474120331354,-36.8337360501076,-36.7957700665624,-36.7386591493628]}]],[[{"lng":[175.940104000404,175.913492216935,175.832775867558,175.809667417205,175.833586717065,175.927714949317,175.905873816583,175.97718783329,175.940945834189,176.075603717482,176.076019150589,176.150334834187,176.232057866463,176.297547798202,176.390623467124,176.429143533873,176.505324767157,176.450980499877,176.255113766944,176.25838093416,176.257562950888,176.147549683454,176.172753900527,176.085669116788,175.989206866079,175.96436554996,175.885970716459,175.81256880052,175.669211766575,175.557854216472,175.634656366175,175.617972132813,175.682012417338,175.562633650262,175.579969833543,175.546438183897,175.574620149734,175.523386533046,175.632458999384,175.612185166049,175.480939533802,175.383212017291,175.317760016592,175.213231183619,175.10922668413,174.954390867085,174.937845033609,174.862896367393,174.791960217277,174.655191183282,174.615147466571,174.64557498284,174.634641616934,174.715704749268,174.712538232634,174.690774649571,174.777037133266,174.895569600305,174.874875616366,174.77780156729,174.792085916113,174.75679855049,174.843696184123,174.956284466653,174.951308116723,174.837134500703,174.768715216871,174.700024633059,174.659628367495,174.815488049849,174.868951232796,174.990258082555,175.048242617196,175.141120333576,175.152091033439,175.291304133085,175.294966566687,175.323006283065,175.3992043827,175.544902117386,175.514084716697,175.51974208322,175.491002617547,175.408398483484,175.490180517217,175.494676733872,175.429445151764,175.447073067326,175.405834634533,175.344544616922,175.328268300532,175.36819123307,175.520191916267,175.522613315882,175.583391316677,175.578198899777,175.646650566193,175.788850933098,175.708633783134,175.78130515032,175.856004517198,175.842233483927,175.888322383074,175.878370950751,175.940104000404],"lat":[-37.3732415493795,-37.4255867167681,-37.4716019831595,-37.5138131497333,-37.6274871173195,-37.7760034327579,-37.8285516669453,-37.9136730327822,-37.929422499792,-38.0495643996454,-38.1214420994107,-38.2181863994961,-38.2390825826552,-38.2295556169363,-38.3230470167692,-38.3134151167029,-38.4022137992992,-38.5616572327478,-38.8081781832656,-38.8899908831226,-38.9478759665031,-39.0376737001404,-39.0784858666215,-39.1294486168462,-39.11554808317,-39.1798030167947,-39.2058951834786,-39.3003262164755,-39.2969341494137,-39.2759983494818,-39.1583628832729,-39.0812491668439,-38.9797105002169,-38.986829699848,-38.9106842496516,-38.8598730996322,-38.8163002497041,-38.750091966835,-38.5917750671975,-38.4876116161861,-38.478880750128,-38.542295983231,-38.5560422670079,-38.6393990331454,-38.6245203666478,-38.6675035326548,-38.7501585666532,-38.7745840666622,-38.7317766330023,-38.7332825998921,-38.7066484493564,-38.4563665331151,-38.3871955160275,-38.3065037997069,-38.2005980662173,-38.1176242502238,-38.1325016493274,-38.0859713163665,-38.0485370834827,-38.0815316832065,-38.0072839664253,-37.8638128167999,-37.8039134660842,-37.7924118832134,-37.7515311835327,-37.7936627827116,-37.5608502999349,-37.4167737834823,-37.2924320832073,-37.2286308828648,-37.2449866002391,-37.2165854996348,-37.1822711996281,-37.175603383291,-37.0215583499183,-37.0240384665172,-37.1211146497524,-37.1906320164282,-37.2270412666345,-37.1573665495214,-37.0930035993838,-37.0341053993446,-36.9622664333688,-36.857511400251,-36.8083106669846,-36.7619687494618,-36.7001410820276,-36.6600482663062,-36.580563183098,-36.5530231170595,-36.4808005320916,-36.4685975823769,-36.5407973488788,-36.5918639497124,-36.6272173993718,-36.6802999496706,-36.7363723666233,-36.7041192832818,-36.8325179663713,-36.8224126992097,-36.9228085834167,-36.9474531172303,-37.0391867663686,-37.2003710496673,-37.3732415493795]}]],[[{"lng":[178.085284050728,178.107554900656,177.986683351023,177.996621683506,178.073006950406,177.972291633859,177.860967450149,177.739984233557,177.692067683864,177.604788567261,177.499270867144,177.463687900819,177.236922317683,177.257394033283,177.122170799827,177.045979500377,176.991448066662,176.87866763316,176.766746733651,176.717283066468,176.695970650021,176.601167800205,176.520452884071,176.468164783255,176.382140266511,176.303565400194,176.25838093416,176.255113766944,176.450980499877,176.505324767157,176.429143533873,176.390623467124,176.297547798202,176.232057866463,176.150334834187,176.076019150589,176.075603717482,175.940945834189,175.97718783329,175.905873816583,175.927714949317,175.833586717065,175.809667417205,175.832775867558,175.913492216935,175.940104000404,175.963443900663,175.93480529982,175.975048966786,176.053118300075,176.186754767108,176.250032000857,176.419017433436,176.477600450903,176.601950900424,176.736143750078,176.931668933866,177.083869783407,177.160565467584,177.383685833914,177.487534484142,177.5467901337,177.619619200774,177.683952433749,177.727635733791,177.868536417845,177.958640534434,178.006622349832,178.085284050728],"lat":[-37.5423911653952,-37.5987159651277,-37.6087571325392,-37.7602764990791,-37.7635896985385,-37.8450054654632,-38.1116561657724,-38.0420469662422,-38.1082775994784,-38.1866207660274,-38.2184667490582,-38.3335362661665,-38.4263417993583,-38.5082771658461,-38.5839209332306,-38.6208022327955,-38.693694583043,-38.6671346491991,-38.689309683067,-38.7245242994195,-38.8215917661477,-38.825559132932,-38.7968662995226,-38.9415194660224,-38.9887032828112,-38.9017194499202,-38.8899908831226,-38.8081781832656,-38.5616572327478,-38.4022137992992,-38.3134151167029,-38.3230470167692,-38.2295556169363,-38.2390825826552,-38.2181863994961,-38.1214420994107,-38.0495643996454,-37.929422499792,-37.9136730327822,-37.8285516669453,-37.7760034327579,-37.6274871173195,-37.5138131497333,-37.4716019831595,-37.4255867167681,-37.3732415493795,-37.4333808167205,-37.5368533661666,-37.6239334661018,-37.6595265664723,-37.634790017279,-37.6800395995794,-37.7484934665627,-37.7541410499537,-37.8295142667608,-37.8795955829941,-37.9168966999474,-37.9788140826354,-37.9898454662543,-37.9836964661637,-37.9495595659166,-37.9075796825578,-37.8107282995693,-37.7601537155194,-37.6801879490526,-37.6533825659818,-37.6053283322934,-37.5531278659688,-37.5423911653952]}]],[[{"lng":[177.898992867322,177.834796917381,177.799025000975,177.660163933681,177.660490067302,177.597522800508,177.49421208424,177.447906750195,177.353425167645,177.307560650033,177.122170799827,177.257394033283,177.236922317683,177.463687900819,177.499270867144,177.604788567261,177.692067683864,177.739984233557,177.860967450149,177.972291633859,178.073006950406,177.996621683506,177.986683351023,178.107554900656,178.085284050728,178.174634083746,178.276301851602,178.363710601162,178.482593484053,178.550373600413,178.415858017631,178.391299368031,178.337261951072,178.373593883376,178.321955168218,178.338370117511,178.312409501413,178.344519151294,178.281772317114,178.293040667099,178.198843133985,178.050040402505,177.9727296172,177.911783384436,177.898992867322],"lat":[-38.9718330659402,-38.9527794154328,-38.8943902994202,-38.9047009160675,-38.8489419157559,-38.8070414992073,-38.7779742826049,-38.6359438824164,-38.6245578660679,-38.659704199614,-38.5839209332306,-38.5082771658461,-38.4263417993583,-38.3335362661665,-38.2184667490582,-38.1866207660274,-38.1082775994784,-38.0420469662422,-38.1116561657724,-37.8450054654632,-37.7635896985385,-37.7602764990791,-37.6087571325392,-37.5987159651277,-37.5423911653952,-37.5346582985723,-37.5544868652179,-37.6310041980111,-37.6410965144209,-37.6920842484305,-37.8580439155172,-37.9613996652351,-38.0095467654096,-38.0944882317616,-38.1221914152638,-38.1917070816724,-38.2415486150003,-38.4184646319201,-38.4721755149837,-38.5269283319884,-38.6055091147978,-38.7020828317471,-38.6812606823481,-38.8279248659345,-38.9718330659402]}]],[[{"lng":[177.898992867322,177.901799650219,178.001674400697,177.924318566662,177.906174350275,177.850842016855,177.839268633525,177.870901817579,177.755965216565,177.636631149881,177.421363584153,177.259161701129,177.046117517326,176.989527483471,176.919394050427,176.876375366274,176.92417755026,176.922247883049,176.966353650084,177.075499034874,177.011022417483,177.005106232773,176.932128366633,176.888223216891,176.868997500466,176.794396933421,176.687031283282,176.622569049879,176.44329526638,176.359488317331,176.378605866561,176.323214066837,176.204970833785,176.136627349808,176.093831400639,176.106847067352,176.176852599749,176.195937933827,176.142726417603,176.183756817499,176.136827417622,176.111859467522,176.121760383432,176.080882450312,176.115201750033,176.085669116788,176.172753900527,176.147549683454,176.257562950888,176.25838093416,176.303565400194,176.382140266511,176.468164783255,176.520452884071,176.601167800205,176.695970650021,176.717283066468,176.766746733651,176.87866763316,176.991448066662,177.045979500377,177.122170799827,177.307560650033,177.353425167645,177.447906750195,177.49421208424,177.597522800508,177.660490067302,177.660163933681,177.799025000975,177.834796917381,177.898992867322],"lat":[-38.9718330659402,-39.0729589155023,-39.1049364824401,-39.1701287656029,-39.2262764647389,-39.2334497985144,-39.1523278981441,-39.0876976326241,-39.0586759157535,-39.0504614660947,-39.0626328989449,-39.1006631827984,-39.193936749739,-39.294002783693,-39.3400230828642,-39.4226607993692,-39.4759893488304,-39.549583649216,-39.6235120824413,-39.6732594152758,-39.7427146829718,-39.8359419163268,-39.9352899662693,-40.0255260159481,-40.1313829663584,-40.2053349995806,-40.2563932326417,-40.4280165160345,-40.4073888329473,-40.1971025164432,-40.1581407166381,-40.0937170161921,-40.0189951167237,-40.0285781338199,-39.9955025832323,-39.9172052496743,-39.7440490994983,-39.6635709665703,-39.584682783093,-39.5389751330716,-39.4752775332614,-39.3849908334062,-39.3263642668049,-39.2591636993616,-39.2176892164469,-39.1294486168462,-39.0784858666215,-39.0376737001404,-38.9478759665031,-38.8899908831226,-38.9017194499202,-38.9887032828112,-38.9415194660224,-38.7968662995226,-38.825559132932,-38.8215917661477,-38.7245242994195,-38.689309683067,-38.6671346491991,-38.693694583043,-38.6208022327955,-38.5839209332306,-38.659704199614,-38.6245578660679,-38.6359438824164,-38.7779742826049,-38.8070414992073,-38.8489419157559,-38.9047009160675,-38.8943902994202,-38.9527794154328,-38.9718330659402]}]],[[{"lng":[174.615147466571,174.655191183282,174.791960217277,174.862896367393,174.819869750297,174.798771383129,174.82803073341,174.764415417467,174.788411450134,174.675169549981,174.755642432479,174.747802983227,174.891668949943,174.871353583229,174.928495867377,174.967386967073,174.972628816641,174.930618333552,174.784081849931,174.716569950035,174.558874867369,174.444514817259,174.352709617041,174.201535516873,174.076819100748,173.910845583182,173.794988949217,173.75158500083,173.794817016681,173.909613299173,174.088957305084,174.200815833298,174.390964834124,174.559947483204,174.615147466571],"lat":[-38.7066484493564,-38.7332825998921,-38.7317766330023,-38.7745840666622,-38.7875171166264,-38.8652667999868,-38.9237043663331,-39.0047909500156,-39.0678331830497,-39.1298299493575,-39.2336436334958,-39.275799016305,-39.3582403665597,-39.4010654665755,-39.4761503161921,-39.4862796832709,-39.6531461001073,-39.7244787998992,-39.8579774497761,-39.8672062167726,-39.8174444664465,-39.7506608664393,-39.6590780832903,-39.5880837831519,-39.577339966687,-39.5148036332182,-39.4150641328021,-39.2803068998566,-39.1924945673089,-39.1211085998919,-39.0509301723833,-38.9883909501815,-38.988858083009,-38.8559135835186,-38.7066484493564]}]],[[{"lng":[176.085669116788,176.115201750033,176.080882450312,176.121760383432,176.111859467522,176.136827417622,176.183756817499,176.142726417603,176.195937933827,176.176852599749,176.106847067352,176.093831400639,176.136627349808,176.204970833785,176.323214066837,176.378605866561,176.359488317331,176.44329526638,176.622569049879,176.620793584036,176.493457017505,176.332762533486,176.211253483369,176.0823341834,176.022449299869,175.871636834195,175.764739183997,175.77010965014,175.690650115948,175.653222466794,175.473443566666,175.439643550552,175.371593716648,175.266658932695,175.136486700712,175.190707066476,175.227057750243,175.223560650608,175.198135366129,175.162540683897,175.045463016742,174.927877449905,174.784081849931,174.930618333552,174.972628816641,174.967386967073,174.928495867377,174.871353583229,174.891668949943,174.747802983227,174.755642432479,174.675169549981,174.788411450134,174.764415417467,174.82803073341,174.798771383129,174.819869750297,174.862896367393,174.937845033609,174.954390867085,175.10922668413,175.213231183619,175.317760016592,175.383212017291,175.480939533802,175.612185166049,175.632458999384,175.523386533046,175.574620149734,175.546438183897,175.579969833543,175.562633650262,175.682012417338,175.617972132813,175.634656366175,175.557854216472,175.669211766575,175.81256880052,175.885970716459,175.96436554996,175.989206866079,176.085669116788],"lat":[-39.1294486168462,-39.2176892164469,-39.2591636993616,-39.3263642668049,-39.3849908334062,-39.4752775332614,-39.5389751330716,-39.584682783093,-39.6635709665703,-39.7440490994983,-39.9172052496743,-39.9955025832323,-40.0285781338199,-40.0189951167237,-40.0937170161921,-40.1581407166381,-40.1971025164432,-40.4073888329473,-40.4280165160345,-40.4917772328785,-40.5305003326431,-40.6979007993168,-40.659210282768,-40.7081527159945,-40.7114886668177,-40.7779047333065,-40.7680905666746,-40.6988883998071,-40.6965623329528,-40.7424894826732,-40.6933970997204,-40.7441521327034,-40.7268607829925,-40.7686105330731,-40.7003334997973,-40.5669227160378,-40.3980753833572,-40.2930346163099,-40.1765422327651,-40.1071686333906,-39.9854570661082,-39.8993730493272,-39.8579774497761,-39.7244787998992,-39.6531461001073,-39.4862796832709,-39.4761503161921,-39.4010654665755,-39.3582403665597,-39.275799016305,-39.2336436334958,-39.1298299493575,-39.0678331830497,-39.0047909500156,-38.9237043663331,-38.8652667999868,-38.7875171166264,-38.7745840666622,-38.7501585666532,-38.6675035326548,-38.6245203666478,-38.6393990331454,-38.5560422670079,-38.542295983231,-38.478880750128,-38.4876116161861,-38.5917750671975,-38.750091966835,-38.8163002497041,-38.8598730996322,-38.9106842496516,-38.986829699848,-38.9797105002169,-39.0812491668439,-39.1583628832729,-39.2759983494818,-39.2969341494137,-39.3003262164755,-39.2058951834786,-39.1798030167947,-39.11554808317,-39.1294486168462]}]],[[{"lng":[176.332762533486,176.26722840018,176.224487017581,176.154899234063,176.10692563345,176.062479416918,176.003310849803,175.961835883779,175.899294816757,175.811865650044,175.675583900734,175.585185949959,175.533100033117,175.433381416302,175.325406533484,175.236423100173,175.195284183578,175.216735033403,175.043268483566,174.924173882771,174.847847867082,174.912076183578,174.844356367019,174.785921437228,174.839045832862,174.705109599961,174.66078746668,174.623012999722,174.711288199763,174.794672150245,174.867167799271,174.846312016586,174.938811199179,174.989772533759,175.035492733743,175.136486700712,175.266658932695,175.371593716648,175.439643550552,175.473443566666,175.653222466794,175.690650115948,175.77010965014,175.764739183997,175.871636834195,176.022449299869,176.0823341834,176.211253483369,176.332762533486],"lat":[-40.6979007993168,-40.7848635999896,-40.9064747167747,-40.9481505494893,-41.015220899261,-41.1310157497801,-41.1717069160291,-41.2447749000527,-41.2632956998994,-41.359626999291,-41.4137952330452,-41.4873728326723,-41.4965452334319,-41.5688357168968,-41.6059589661651,-41.6051656500414,-41.5238026326864,-41.4315767334039,-41.3733172995629,-41.4347028666894,-41.3594841327896,-41.259709699891,-41.2288581497332,-41.2651837993373,-41.3252582662544,-41.3571104667103,-41.3403752330376,-41.2603256333459,-41.219934017264,-41.124690299725,-41.0867387663858,-41.0499113660342,-41.000301466556,-40.8786943168829,-40.8516626997137,-40.7003334997973,-40.7686105330731,-40.7268607829925,-40.7441521327034,-40.6933970997204,-40.7424894826732,-40.6965623329528,-40.6988883998071,-40.7680905666746,-40.7779047333065,-40.7114886668177,-40.7081527159945,-40.659210282768,-40.6979007993168]}]],[[{"lng":[172.479030949819,172.444256900204,172.348260516018,172.235513800302,172.1330007499,172.094375383646,171.974953333456,171.915639166648,171.841774433461,171.763848766853,171.746369750712,171.69066966728,171.44922653414,171.411630450039,171.316264849941,171.237669015979,171.197689267427,171.079330766508,170.993271149153,170.937214367317,170.830348166594,170.70569891615,170.592312017226,170.528889117364,170.395291850712,170.32286143405,170.23842121714,170.098131417276,170.091407749956,169.965530866422,169.900034783341,169.699336634174,169.592780550647,169.591375933139,169.37295708232,169.275178699696,169.188798133344,169.147102432497,168.975078451222,168.963356499608,168.849166000645,168.836083533033,168.728392349424,168.625502800125,168.460041532958,168.334527917476,168.407363599041,168.349075433281,168.347349199204,168.224958950513,168.131928965928,168.052080299808,168.245224000369,168.327532100509,168.386086599287,168.512572866295,168.59603308294,168.709528882891,168.815458849998,168.888444333017,168.969334415799,169.209384000748,169.426815166877,169.591670650551,169.690794066995,169.793601633191,169.981233500258,170.050894783885,170.26610145036,170.257309000666,170.34165461676,170.420541400347,170.524388600502,170.720963766414,170.867846066505,170.947971800697,171.051014333038,171.167346766778,171.218172099769,171.304561750154,171.321142133862,171.417931516559,171.455980199127,171.462321733037,171.587717983286,171.670754701712,171.828431383484,171.882088466121,171.987652299441,172.063382449494,172.092597800296,172.111628817585,172.094501582949,172.133988150577,172.217804650849,172.229036100221,172.370078216958,172.310335033614,172.493938299216,172.485968450507,172.618004983595,172.657937583202,172.600312034113,172.584361567505,172.46311268413,172.418018282578,172.34245996745,172.320665484129,172.238130649319,172.247078967484,172.129322299744,172.115730567145,172.052126816993,172.135203666157,172.206730749948,172.26099479982,172.331950216583,172.335875833233,172.369593532568,172.479030949819],"lat":[-42.2770294330973,-42.3659640998669,-42.3913898493633,-42.4567272663168,-42.5552061664139,-42.6204476501481,-42.6389925162904,-42.6817304496134,-42.7745563165321,-42.7917224165001,-42.8740807000281,-42.9008274327434,-42.9092283835179,-42.9507592166428,-42.9474959831847,-43.0025114832403,-43.0742574837236,-43.1023006501349,-43.2153140831952,-43.2040572826411,-43.2835111333203,-43.3207865163813,-43.4340141666261,-43.4103522167365,-43.4980506000427,-43.4903493000688,-43.5138270161525,-43.5954793832756,-43.6603731667768,-43.7917623165151,-43.8129979661406,-43.9635462830298,-43.9580761661512,-44.0125360167186,-44.1169906665293,-44.0985810000811,-44.1105468990999,-44.0802167656571,-44.1440294836209,-44.1999363326238,-44.2466723492225,-44.3399177497125,-44.4194781165777,-44.4339354660492,-44.4953348155298,-44.4967794321351,-44.3751147821832,-44.3304823991759,-44.2842383160426,-44.256824249116,-44.2811937815188,-44.2586477653813,-44.158056415825,-44.0494656159258,-44.0035227657058,-44.0018953495409,-43.9725449323756,-43.9942465493971,-43.9674074832386,-43.9053743496898,-43.8784611496635,-43.7192993998752,-43.6253148328527,-43.5990183994978,-43.5430910165578,-43.4215250501236,-43.3578198495134,-43.2918990001713,-43.1725636158292,-43.118815349405,-43.0966792499658,-43.0361614328878,-43.0173943501588,-42.9301012999008,-42.8228048660163,-42.7232656999317,-42.650688316626,-42.5078089999748,-42.381019766233,-42.2672141667491,-42.1532471998875,-41.9177566328862,-41.8824885829382,-41.7497772332867,-41.7299331833162,-41.7398889758282,-41.6470442500116,-41.6006758828465,-41.4457266661726,-41.3840702160756,-41.282779150182,-41.0257784662432,-40.916250083059,-40.850621782872,-40.7745427669185,-40.8558806664281,-40.9707524671654,-41.0135581163012,-41.0245242002173,-41.0542427327216,-41.1598956494134,-41.239168999544,-41.3018332326605,-41.3761944999689,-41.4043775665175,-41.4716592493225,-41.5131613498026,-41.6102873667673,-41.6395180999627,-41.6842177167088,-41.7140016329438,-41.8684047996669,-41.9451821329784,-42.1040439833797,-42.0983147832165,-42.1483124834312,-42.1506392829313,-42.2342079660356,-42.2806025168152,-42.2770294330973]}]],[[{"lng":[174.048786900106,173.966599600681,173.892544684064,173.697996616623,173.671647083024,173.594467415865,173.509134350058,173.479398800221,173.368425849937,173.241651783235,173.14498353365,173.080542450878,172.816906899328,172.73369503355,172.704087800548,172.71511108423,172.776007199763,173.040053516148,173.102133650692,173.130149783874,173.100509666792,172.997729132882,172.836626966534,172.728520416444,172.587602799177,172.31661868302,172.028047117075,171.614830016684,171.439070166985,171.333436083013,171.244871200256,171.258240632397,171.195791149222,171.159184499413,171.167954350424,171.144153632672,170.882433150378,170.658825416631,170.636873433878,170.542436216334,170.447188600346,170.397211283579,170.349440716735,170.373258400944,170.319353066232,170.242072667457,170.094098767271,169.996618083497,169.944216317567,169.836191183419,169.757151799335,169.64421074906,169.633685649884,169.545884367477,169.57710081663,169.571069933865,169.717037416418,169.699336634174,169.900034783341,169.965530866422,170.091407749956,170.098131417276,170.23842121714,170.32286143405,170.395291850712,170.528889117364,170.592312017226,170.70569891615,170.830348166594,170.937214367317,170.993271149153,171.079330766508,171.197689267427,171.237669015979,171.316264849941,171.411630450039,171.44922653414,171.69066966728,171.746369750712,171.763848766853,171.841774433461,171.915639166648,171.974953333456,172.094375383646,172.1330007499,172.235513800302,172.348260516018,172.444256900204,172.479030949819,172.614713433524,172.639183050117,172.721472999322,172.7216385824,172.823290099099,172.84243633332,172.914144833248,172.94139345023,172.992906316788,173.035789000756,173.189895532864,173.24452471748,173.312201366903,173.464463350147,173.499107499943,173.720210166329,173.744379950814,173.817253667021,173.955504883873,174.048786900106],"lat":[-41.9681092162602,-42.0447000162179,-42.1892206160492,-42.351055932752,-42.4129525826766,-42.4284450167963,-42.5056333005085,-42.5927161833235,-42.7988198827098,-42.9528653661342,-42.9864308666799,-43.0507309163767,-43.1332584499924,-43.2550360161747,-43.3903552166591,-43.5479150830848,-43.5827193327987,-43.6552001167613,-43.6956743333843,-43.7627759999547,-43.8318133828848,-43.8873741827115,-43.889760282991,-43.8276729498285,-43.833265216244,-43.8670884165103,-43.9583162998763,-44.1318285168766,-44.2233021661649,-44.295190716563,-44.3817173993977,-44.4457059831022,-44.5267523667591,-44.627834699927,-44.8761730664257,-44.9403990326591,-44.891009399824,-44.9092402835359,-44.9513015836188,-44.9850467672132,-45.0659566666071,-45.0809043667793,-45.0216976998576,-44.9527760160294,-44.9030489828245,-44.9307172500278,-44.9400199334535,-44.8215735833956,-44.6539842168145,-44.6688994160923,-44.6121328159957,-44.5858154997107,-44.4624052998829,-44.4226623995998,-44.3456657831464,-44.1772291499773,-44.0382773159809,-43.9635462830298,-43.8129979661406,-43.7917623165151,-43.6603731667768,-43.5954793832756,-43.5138270161525,-43.4903493000688,-43.4980506000427,-43.4103522167365,-43.4340141666261,-43.3207865163813,-43.2835111333203,-43.2040572826411,-43.2153140831952,-43.1023006501349,-43.0742574837236,-43.0025114832403,-42.9474959831847,-42.9507592166428,-42.9092283835179,-42.9008274327434,-42.8740807000281,-42.7917224165001,-42.7745563165321,-42.6817304496134,-42.6389925162904,-42.6204476501481,-42.5552061664139,-42.4567272663168,-42.3913898493633,-42.3659640998669,-42.2770294330973,-42.1954626498809,-42.1007946829437,-42.1034917994101,-42.1586981838795,-42.2047070167512,-42.2891840664509,-42.3086089000751,-42.3778250335143,-42.403849533931,-42.4834705328707,-42.4105463995019,-42.3031041161546,-42.2503815996964,-42.1956736999014,-42.0787366833871,-41.9607712162466,-41.9073832665075,-41.9329969833512,-41.9126783171065,-41.9681092162602]}]],[[{"lng":[169.699336634174,169.717037416418,169.571069933865,169.57710081663,169.545884367477,169.633685649884,169.64421074906,169.757151799335,169.836191183419,169.944216317567,169.996618083497,170.094098767271,170.242072667457,170.319353066232,170.373258400944,170.349440716735,170.397211283579,170.447188600346,170.542436216334,170.636873433878,170.658825416631,170.882433150378,171.144153632672,171.100528099082,170.973751149775,170.904456867088,170.831473232453,170.831145883379,170.75579789982,170.725243549953,170.653058832419,170.600894433298,170.658593699998,170.663784716406,170.511765316615,170.297526650458,170.196885316546,170.168560666665,169.923941867019,169.79808831661,169.797051965838,169.587586633505,169.47711816663,169.404843300523,169.279166266453,169.200940883195,169.250660233525,169.172959832967,169.195997516664,169.134143867521,169.190826583093,169.198060700805,169.1596198997,169.153328134266,169.067252783316,169.091515616612,169.005199499322,168.993372050566,169.129045084135,169.140231617244,169.211048582457,169.099293033043,169.065469000311,168.959565366312,168.886900132632,168.782457032387,168.75163494987,168.69221520076,168.674105117547,168.539401383192,168.323294050019,168.300759133521,168.217006499085,168.284206182761,168.292267949593,168.133993850181,168.117214416596,168.191001933458,168.172333383406,168.234509800005,168.334527917476,168.460041532958,168.625502800125,168.728392349424,168.836083533033,168.849166000645,168.963356499608,168.975078451222,169.147102432497,169.188798133344,169.275178699696,169.37295708232,169.591375933139,169.592780550647,169.699336634174],"lat":[-43.9635462830298,-44.0382773159809,-44.1772291499773,-44.3456657831464,-44.4226623995998,-44.4624052998829,-44.5858154997107,-44.6121328159957,-44.6688994160923,-44.6539842168145,-44.8215735833956,-44.9400199334535,-44.9307172500278,-44.9030489828245,-44.9527760160294,-45.0216976998576,-45.0809043667793,-45.0659566666071,-44.9850467672132,-44.9513015836188,-44.9092402835359,-44.891009399824,-44.9403990326591,-45.0030083668204,-45.0991354496319,-45.1744155169177,-45.3055600328457,-45.4735115163131,-45.533487850182,-45.5959489167521,-45.6310545499492,-45.7107696833387,-45.7588499501258,-45.9035087330227,-45.9091708334692,-45.9623933001436,-46.0625319333419,-46.1545419164646,-46.2983543337638,-46.3546976164324,-46.4513536993082,-46.5657448492693,-46.5613084166797,-46.6123262493729,-46.6224137665975,-46.5607098326622,-46.5143265831323,-46.4861186661636,-46.4044065824797,-46.3754392329164,-46.3264038991747,-46.2126295490888,-46.1756371497824,-46.0487447159179,-45.9804276161464,-45.8825561162821,-45.8634921996335,-45.7285384164748,-45.5203341824305,-45.4652520827241,-45.4167719991232,-45.319945749245,-45.269609899731,-45.3591220827466,-45.4628204656639,-45.4370390829242,-45.3675663491687,-45.3396697495697,-45.2430717491471,-45.3093267162083,-45.327543165947,-45.2246531159516,-45.1445409488356,-45.1061745325468,-45.0438246323005,-44.9414410656498,-44.8714499484676,-44.7667900988546,-44.7045591824468,-44.543195698814,-44.4967794321351,-44.4953348155298,-44.4339354660492,-44.4194781165777,-44.3399177497125,-44.2466723492225,-44.1999363326238,-44.1440294836209,-44.0802167656571,-44.1105468990999,-44.0985810000811,-44.1169906665293,-44.0125360167186,-43.9580761661512,-43.9635462830298]}]],[[{"lng":[168.334527917476,168.234509800005,168.172333383406,168.191001933458,168.117214416596,168.133993850181,168.292267949593,168.284206182761,168.217006499085,168.300759133521,168.323294050019,168.539401383192,168.674105117547,168.69221520076,168.75163494987,168.782457032387,168.886900132632,168.959565366312,169.065469000311,169.099293033043,169.211048582457,169.140231617244,169.129045084135,168.993372050566,169.005199499322,169.091515616612,169.067252783316,169.153328134266,169.1596198997,169.198060700805,169.190826583093,169.134143867521,169.195997516664,169.172959832967,169.250660233525,169.200940883195,169.279166266453,169.220365916036,169.112977633974,169.005877783165,168.847240616748,168.793966399727,168.602963450817,168.515573883457,168.352956367785,168.265940500354,168.263609712926,168.190226400186,168.064940999915,168.015834933966,167.93185776713,167.783985416635,167.697388534619,167.712351200166,167.611463133841,167.533283266975,167.403100267612,167.360417901081,167.21721350111,167.105668635801,166.852483269489,166.778670303253,166.657467519732,166.618782638105,166.68276642128,166.698540019365,166.615138537561,166.611146487314,166.555778437685,166.449630289541,166.426302922672,166.450324487655,166.518401055545,166.622173086985,166.804038752399,166.730721587317,166.751807818911,166.731030037296,166.671602286539,166.684135137044,166.844406784828,166.908979302293,167.007590901768,166.967797184775,167.002267434006,167.271321767837,167.500637450702,167.571723434454,167.751370433439,167.817211300363,167.827906550695,167.872684933483,167.973286216925,168.008558300361,168.097524283348,168.052080299808,168.131928965928,168.224958950513,168.347349199204,168.349075433281,168.407363599041,168.334527917476],"lat":[-44.4967794321351,-44.543195698814,-44.7045591824468,-44.7667900988546,-44.8714499484676,-44.9414410656498,-45.0438246323005,-45.1061745325468,-45.1445409488356,-45.2246531159516,-45.327543165947,-45.3093267162083,-45.2430717491471,-45.3396697495697,-45.3675663491687,-45.4370390829242,-45.4628204656639,-45.3591220827466,-45.269609899731,-45.319945749245,-45.4167719991232,-45.4652520827241,-45.5203341824305,-45.7285384164748,-45.8634921996335,-45.8825561162821,-45.9804276161464,-46.0487447159179,-46.1756371497824,-46.2126295490888,-46.3264038991747,-46.3754392329164,-46.4044065824797,-46.4861186661636,-46.5143265831323,-46.5607098326622,-46.6224137665975,-46.6562230996031,-46.6445697995773,-46.6743386327922,-46.6586199324697,-46.581126249587,-46.5519047491435,-46.5742984494303,-46.536278198365,-46.5559335316574,-46.4953184330149,-46.3853629156488,-46.3392979653123,-46.3838269816428,-46.3581014984449,-46.3890891153421,-46.3079614649503,-46.2576959484531,-46.1914227639541,-46.1624094643935,-46.1520754139566,-46.2369483973937,-46.2625064636042,-46.2536611290707,-46.208479478094,-46.2220721780732,-46.2022869439588,-46.1487916607235,-46.1155054774682,-46.0582340943939,-46.0600605774594,-45.9997223441552,-45.974646443428,-45.9949010433078,-45.9044508925509,-45.8189640767231,-45.7943900103485,-45.7910534108493,-45.7598318118677,-45.7281351117593,-45.6676932449173,-45.5947802620528,-45.5749929274641,-45.5132728112309,-45.2758819118717,-45.3074377122482,-45.3141696626352,-45.1937010798687,-45.1255542132859,-44.8704800972672,-44.748057897824,-44.6858730808973,-44.5806611478559,-44.5991209981649,-44.5075593480495,-44.4342743651316,-44.3866309813592,-44.3250568982168,-44.3271642985448,-44.2586477653813,-44.2811937815188,-44.256824249116,-44.2842383160426,-44.3304823991759,-44.3751147821832,-44.4967794321351]}],[{"lng":[167.869696717386,167.976547566978,168.001210732732,168.141792199565,168.141009318344,168.031908184531,167.978834600349,168.150523284284,168.21466859946,168.156390867238,167.954351351045,167.825699752283,167.710747199939,167.636122350701,167.64603149998,167.466622085851,167.466427318386,167.575995201712,167.579121551345,167.64975053312,167.694039967428,167.759027316278,167.756471367387,167.704121801107,167.754640633164,167.869696717386],"lat":[-46.6842310150475,-46.7237613973466,-46.7857626640377,-46.8618585317734,-46.9094082979053,-46.89688748132,-46.9288782319926,-46.9664681167307,-46.9944344827427,-47.1041435150793,-47.1292745634624,-47.191379982114,-47.1604565144884,-47.2102895310388,-47.2637216303122,-47.2828524134173,-47.2165166300209,-47.166243313737,-47.085338897629,-47.0364652468971,-46.9670829976916,-46.9370851803384,-46.8609822636435,-46.7544420145734,-46.6946847146376,-46.6842310150475]}],[{"lng":[166.678832287015,166.716482704068,166.723640187457,166.684653836759,166.588378970674,166.53680382096,166.678832287015],"lat":[-45.6003755947174,-45.6245835280225,-45.699235444444,-45.748173577695,-45.7207469609017,-45.6343713934598,-45.6003755947174]}],[{"lng":[166.962425851921,166.949393052829,166.987792335244,166.947416384981,166.868999985619,166.962425851921],"lat":[-45.1477982791214,-45.2031774956885,-45.2585408461444,-45.2941630622485,-45.2405394789924,-45.1477982791214]}]],[[{"lng":[173.198887399729,173.297443983366,173.250831384127,173.304149350489,173.168700533592,173.151614666983,173.086652616993,173.003789650797,172.926235200656,172.847570550194,172.86109313253,172.735298849099,172.721472999322,172.639183050117,172.614713433524,172.479030949819,172.369593532568,172.335875833233,172.331950216583,172.26099479982,172.206730749948,172.135203666157,172.052126816993,172.115730567145,172.129322299744,172.247078967484,172.238130649319,172.320665484129,172.34245996745,172.418018282578,172.46311268413,172.584361567505,172.600312034113,172.657937583202,172.618004983595,172.485968450507,172.493938299216,172.310335033614,172.370078216958,172.229036100221,172.217804650849,172.292207984059,172.447285383514,172.60562558292,172.642595651343,172.728820134899,172.688014218755,172.692504033943,172.798009667228,172.886338016923,172.990330217499,173.063517816826,173.012366349789,173.033520217139,173.009530717519,173.06578501716,173.104255083169,173.198887399729],"lat":[-41.3306196997753,-41.3931415162802,-41.4665801165558,-41.5060320835981,-41.5311546995304,-41.5854709831784,-41.6249252333668,-41.7248032669424,-41.7577336161519,-41.8742874663461,-41.9224629661687,-42.0578953163025,-42.1034917994101,-42.1007946829437,-42.1954626498809,-42.2770294330973,-42.2806025168152,-42.2342079660356,-42.1506392829313,-42.1483124834312,-42.0983147832165,-42.1040439833797,-41.9451821329784,-41.8684047996669,-41.7140016329438,-41.6842177167088,-41.6395180999627,-41.6102873667673,-41.5131613498026,-41.4716592493225,-41.4043775665175,-41.3761944999689,-41.3018332326605,-41.239168999544,-41.1598956494134,-41.0542427327216,-41.0245242002173,-41.0135581163012,-40.9707524671654,-40.8558806664281,-40.7745427669185,-40.7473438326411,-40.6295302334237,-40.5644542999212,-40.5081192334808,-40.5290822657801,-40.5827790489934,-40.7278832002206,-40.8162795663909,-40.8323796326954,-40.7811902663483,-40.9057494657357,-40.9921395165831,-41.0887589498798,-41.1437780995614,-41.1838260998815,-41.29015973291,-41.3306196997753]}]],[[{"lng":[173.297443983366,173.198887399729,173.221776466012,173.310618882688,173.32361723313,173.412776467452,173.461631133591,173.593891449768,173.568365816496,173.523007850712,173.415419299657,173.40678916666,173.297443983366],"lat":[-41.3931415162802,-41.3306196997753,-41.2930119000069,-41.2507582163535,-41.2037450165907,-41.1494512164136,-41.1668533163242,-41.0511199330305,-41.1897309661611,-41.1979918667477,-41.2891416995037,-41.3309583501746,-41.3931415162802]}]],[[{"lng":[174.048786900106,173.955504883873,173.817253667021,173.744379950814,173.720210166329,173.499107499943,173.464463350147,173.312201366903,173.24452471748,173.189895532864,173.035789000756,172.992906316788,172.94139345023,172.914144833248,172.84243633332,172.823290099099,172.7216385824,172.721472999322,172.735298849099,172.86109313253,172.847570550194,172.926235200656,173.003789650797,173.086652616993,173.151614666983,173.168700533592,173.304149350489,173.250831384127,173.297443983366,173.40678916666,173.415419299657,173.523007850712,173.568365816496,173.593891449768,173.673303050289,173.802376834404,173.85077169915,173.868127116147,173.93680894915,173.993088400438,174.079004183599,174.17580193353,174.219448550258,174.12449421688,174.133062266944,174.048447900196,174.033535683903,174.072046116608,174.150420350879,174.15696061672,174.198299267036,174.275035250512,174.196758333519,174.048786900106],"lat":[-41.9681092162602,-41.9126783171065,-41.9329969833512,-41.9073832665075,-41.9607712162466,-42.0787366833871,-42.1956736999014,-42.2503815996964,-42.3031041161546,-42.4105463995019,-42.4834705328707,-42.403849533931,-42.3778250335143,-42.3086089000751,-42.2891840664509,-42.2047070167512,-42.1586981838795,-42.1034917994101,-42.0578953163025,-41.9224629661687,-41.8742874663461,-41.7577336161519,-41.7248032669424,-41.6249252333668,-41.5854709831784,-41.5311546995304,-41.5060320835981,-41.4665801165558,-41.3931415162802,-41.3309583501746,-41.2891416995037,-41.1979918667477,-41.1897309661611,-41.0511199330305,-41.0342675989878,-40.972247532291,-41.022320383094,-41.1442067667604,-41.0834601826331,-41.1021083834367,-41.0162762162912,-40.9990567827089,-41.0713428828236,-41.2362538666885,-41.2853295661851,-41.3931097327546,-41.4763469339435,-41.5239591995602,-41.5594337667488,-41.647669716411,-41.7232352500773,-41.7428873668553,-41.8356043168166,-41.9681092162602]}],[{"lng":[173.956358883868,173.960139751383,173.905318948269,173.813983967132,173.772206384171,173.805729534265,173.93662750116,173.956358883868],"lat":[-40.6928282325857,-40.7644749495513,-40.8572309998704,-40.9304674331604,-40.8619594325105,-40.8027343164386,-40.7475060664494,-40.6928282325857]}]]],null,"nz",{"interactive":true,"className":"","pane":"overlayPane01","stroke":true,"color":"#666666","weight":1,"opacity":1,"fill":true,"fillColor":["#D9D9D9","#D9D9D9","#D9D9D9","#D9D9D9","#D9D9D9","#D9D9D9","#D9D9D9","#D9D9D9","#D9D9D9","#D9D9D9","#D9D9D9","#D9D9D9","#D9D9D9","#D9D9D9","#D9D9D9","#D9D9D9"],"fillOpacity":[1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],"smoothFactor":1,"noClip":false},["<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>Northland<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>Northland<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>North<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>12,501<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>175,500<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>23,400<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>0.942<\/td><\/tr><\/table><\/div>","<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>Auckland<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>Auckland<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>North<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>4,942<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>1,657,200<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>29,600<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>0.944<\/td><\/tr><\/table><\/div>","<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>Waikato<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>Waikato<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>North<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>23,900<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>460,100<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>27,900<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>0.952<\/td><\/tr><\/table><\/div>","<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>Bay of Plenty<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>Bay of Plenty<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>North<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>12,071<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>299,900<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>26,200<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>0.928<\/td><\/tr><\/table><\/div>","<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>Gisborne<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>Gisborne<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>North<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>8,386<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>48,500<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>24,400<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>0.935<\/td><\/tr><\/table><\/div>","<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>Hawke's Bay<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>Hawke's Bay<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>North<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>14,138<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>164,000<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>26,100<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>0.924<\/td><\/tr><\/table><\/div>","<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>Taranaki<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>Taranaki<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>North<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>7,254<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>118,000<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>29,100<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>0.957<\/td><\/tr><\/table><\/div>","<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>Manawatu-Wanganui<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>Manawatu-Wanganui<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>North<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>22,221<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>234,500<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>25,000<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>0.939<\/td><\/tr><\/table><\/div>","<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>Wellington<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>Wellington<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>North<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>8,049<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>513,900<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>32,700<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>0.934<\/td><\/tr><\/table><\/div>","<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>West Coast<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>West Coast<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>South<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>23,245<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>32,400<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>26,900<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>1.014<\/td><\/tr><\/table><\/div>","<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>Canterbury<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>Canterbury<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>South<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>44,504<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>612,000<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>30,100<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>0.975<\/td><\/tr><\/table><\/div>","<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>Otago<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>Otago<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>South<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>31,186<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>224,200<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>26,300<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>0.951<\/td><\/tr><\/table><\/div>","<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>Southland<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>Southland<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>South<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>31,196<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>98,300<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>29,500<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>0.979<\/td><\/tr><\/table><\/div>","<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>Tasman<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>Tasman<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>South<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>9,616<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>51,100<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>25,700<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>0.972<\/td><\/tr><\/table><\/div>","<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>Nelson<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>Nelson<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>South<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>422<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>51,400<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>27,200<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>0.926<\/td><\/tr><\/table><\/div>","<div style=\"max-height:10em;overflow:auto;\"><table>\n\t\t\t   <thead><tr><th colspan=\"2\"><b>Marlborough<\/b><\/th><\/thead><\/tr><tr><td style=\"color: #888888;\">Name<\/td><td>Marlborough<\/td><\/tr><tr><td style=\"color: #888888;\">Island<\/td><td>South<\/td><\/tr><tr><td style=\"color: #888888;\">Land_area<\/td><td>10,458<\/td><\/tr><tr><td style=\"color: #888888;\">Population<\/td><td>46,200<\/td><\/tr><tr><td style=\"color: #888888;\">Median_income<\/td><td>27,900<\/td><\/tr><tr><td style=\"color: #888888;\">Sex_ratio<\/td><td>0.958<\/td><\/tr><\/table><\/div>"],{"maxWidth":500,"minWidth":100,"autoPan":true,"keepInView":false,"closeButton":true,"className":""},["Northland","Auckland","Waikato","Bay of Plenty","Gisborne","Hawke's Bay","Taranaki","Manawatu-Wanganui","Wellington","West Coast","Canterbury","Otago","Southland","Tasman","Nelson","Marlborough"],{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null]},{"method":"addLayersControl","args":[["Esri.WorldGrayCanvas","OpenStreetMap","Esri.WorldTopoMap"],"nz",{"collapsed":true,"autoZIndex":true,"position":"topleft"}]}],"limits":{"lat":[-47.2828524134173,-34.4145187168343],"lng":[166.426302922672,178.550373600413]},"fitBounds":[-47.2828524134173,166.426302922672,-34.4145187168343,178.550373600413,[]]},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->
<p class="caption">(\#fig:tmview)Interactive map of New Zealand created with tmap in view mode.</p>
</div>

Now that the interactive mode has been 'turned on', all maps produced with **tmap** will launch (another way to create interactive maps is with the `tmap_leaflet` function) .
Notable features of this interactive mode include the ability to specify the basemap using the `basemaps` argument in the function `tm_view()` (also see `?tm_basemap`), as demonstrated below (the result, a map of New Zealand with an interactive topographic basemap, is not shown):


```r
basemap = leaflet::providers$OpenTopoMap
map_nz +
  tm_view(basemaps = basemap)
```

An impressive and little-known feature of **tmap**'s view mode is that it also works with faceted plots.
The argument `sync` in `tm_facets()` can be used in this case to produce multiple maps with synchronized zoom and pan settings, as illustrated in Figure \@ref(fig:sync) which was produced by the following code:


```r
world_coffee = left_join(world, coffee_data, by = "name_long")
facets = c("coffee_production_2016", "coffee_production_2017")
tm_shape(world_coffee) + tm_polygons(facets) + 
  tm_facets(nrow = 1, sync = TRUE)
```

<div class="figure" style="text-align: center">
<img src="https://user-images.githubusercontent.com/1825120/39561412-4dbba7ba-4e9d-11e8-885c-7973b351006b.png" alt="Faceted interactive maps of global coffee producing in 2016 and 2017 in 'sync', demonstrating tmap's view mode in action."  />
<p class="caption">(\#fig:sync)Faceted interactive maps of global coffee producing in 2016 and 2017 in 'sync', demonstrating tmap's view mode in action.</p>
</div>

Switch **tmap** back to plotting mode with the same function:


```r
tmap_mode("plot")
#> tmap mode set to plotting
```

If you are not proficient with **tmap**, the quickest way to create an interactive maps may be with **mapview**.
The following 'one liner' is a reliable way to interactively explore a wide range of spatial data formats:


```r
mapview::mapview(nz)
```

<div class="figure" style="text-align: center">
<img src="https://user-images.githubusercontent.com/1825120/39979522-e8277398-573e-11e8-8c55-d72c6bcc58a4.png" alt="Illustration of mapview in action."  />
<p class="caption">(\#fig:mapview)Illustration of mapview in action.</p>
</div>

**mapview** has a concise syntax yet is powerful. By default, it provides some standard GIS functionality such as mouse position information, attribute queries (via pop-ups), scale bar, zoom-to-layer buttons.
It offers advanced controls including the ability to 'burst' datasets into multiple layers and the addition of multiple layers with `+` followed by the name of a geographic object. Additionlaly, it provides automatic coloring of attributes (via argument `zcol`). In essence, it can be considered a data-driven **leaflet** API (see below for more information about **leaflet**). Given that **mapview** always expects a spatial object (sf, sp, raster) as its first argument, it works well at the end of piped expressions. Consider the following example where **sf** is used to intersect lines and polygons and then visualised with **mapview**.


```r
trails %>%
  st_transform(st_crs(franconia)) %>%
  st_intersection(franconia[franconia$district == "Oberfranken", ]) %>%
  st_collection_extract("LINE") %>%
  mapview(color = "red", lwd = 3, layer.name = "trails") +
  mapview(franconia, zcol = "district", burst = TRUE) +
  breweries
```

<div class="figure" style="text-align: center">
<img src="https://user-images.githubusercontent.com/1825120/39979271-5f515256-573d-11e8-9ede-e472ca007d73.png" alt="Using mapview at the end of a sf based pipe expression."  />
<p class="caption">(\#fig:mapview2)Using mapview at the end of a sf based pipe expression.</p>
</div>

One important thing to keep in mind is that **mapview** layers are added via the `+` operator (similar to **ggplot2**). This is a frequent [gotcha](https://en.wikipedia.org/wiki/Gotcha_(programming)) in piped workflows where the main binding operator is `%>%`.

For further information on **mapview** see the package's website at [r-spatial.github.io/mapview/](https://r-spatial.github.io/mapview/articles/).

There are other ways to create interactive maps with R not demonstrated here due to space constraints.
The **googleway** package, for example, provides an interactive mapping interface that is flexible and extensible with `google_map()`.
The command `google_map(key = key) %>% add_polygons(st_transform(nz, 4326))` plots an interactive map of New Zealand (it assumes a Google API key is saved as `key`).
Many other functions are provided by the package, providing an R interface to a wide range of mapping services including routing, traffic visualization and geocoding (see the [`googleway-vignette`](https://cran.r-project.org/web/packages/googleway/vignettes/googleway-vignette.html) for details).



Last but not least is **leaflet** which is the most mature and widely used interactive mapping package in R.
**leaflet** provides a relatively low level interface to the Leaflet JavaScript library and many of its arguments can be understood by reading the documentation of the original JavaScript library (see [leafletjs.com](http://leafletjs.com/reference-1.3.0.html)).

Leaflet maps are created with `leaflet()`, the result of which is a `leaflet` map object which can be piped to other **leaflet** functions.
This allows multiple map layers and control settings to be added interactively, as demonstrated in the code below which generates Figure \@ref(fig:leaflet) (see [rstudio.github.io/leaflet/](https://rstudio.github.io/leaflet/) for details).


```r
pal = colorNumeric("RdYlBu", domain = cycle_hire$nbikes)
leaflet(data = cycle_hire) %>% 
  addProviderTiles(providers$Stamen.TonerLite) %>% 
  addCircles(col = ~pal(nbikes), opacity = 0.9) %>% 
  addPolygons(data = lnd, fill = FALSE) %>% 
  addLegend(pal = pal, values = ~nbikes) %>% 
  setView(lng = -0.1, 51.5, zoom = 12) %>% 
  addMiniMap()
```

<div class="figure" style="text-align: center">
<!--html_preserve--><div id="htmlwidget-ea9e511342e2045036e7" style="width:576px;height:355.968px;" class="leaflet html-widget"></div>
<p class="caption">(\#fig:leaflet)The leaflet package in action, showing cycle hire points in London.</p>
</div>

## Mapping applications

The interactive web maps demonstrated in section \@ref(interactive-maps) can go far.
Careful selection of layers to display, base-maps and pop-ups can be used to communicate the main results of many projects involving geocomputation.
But the web mapping approach to interactivity has limitations:

- Although the map is interactive in terms of panning, zooming and clicking, the code is static, meaning the user interface is fixed.
- All map content is generally static in a web map, meaning that web maps cannot scale to handle large datasets easily.
- Additional layers of interactivity, such a graphs showing relationships between variables and 'dashboards' are difficult to create using the web-mapping approach.

Overcoming these limitations involves going beyond static web mapping and towards geospatial frameworks and map servers.
Products in this field include [GeoDjango](https://docs.djangoproject.com/en/2.0/ref/contrib/gis/) (which extends the Django web framework and is written in [Python](https://github.com/django/django)), [MapGuide](https://www.osgeo.org/projects/mapguide-open-source/) (a framework for developing web applications, largely written in [C++](https://trac.osgeo.org/mapguide/wiki/MapGuideArchitecture)) and [GeoServer](http://geoserver.org/) (a mature and powerful map server written in [Java](https://github.com/geoserver/geoserver)).
Each of these (particularly GeoServer) is scalable, enabling maps to be served to thousands of people daily --- assuming there is sufficient public interest in your maps!
The bad news is that such server-side solutions require much skilled developer time to set-up and maintain, often involving teams of people with roles such as a dedicated geospatial database administrator ([DBA](http://wiki.gis.com/wiki/index.php/Database_administrator)).

The good news is that web mapping applications can now be rapidly created using **shiny**, a package for converting R code into interactive web applications.
This is thanks to its support for interactive maps via functions such as `renderLeaflet()`, documented on the [Shiny integration](https://rstudio.github.io/leaflet/shiny.html) section of RStudio's **leaflet** website.
This section gives some context, teaches the basics of **shiny** from a web mapping perspective and culminates in a full-screen mapping application in less than 100 lines of code.

The way **shiny** works is well documented at [shiny.rstudio.com](https://shiny.rstudio.com/).
The two key elements of a **shiny** app reflect the duality common to most web application development: 'front end' (the bit the user sees) and 'back end' code.
In **shiny** apps these elements are typically created in objects named `ui` and `server` within an R script named `app.R`, that lives in an 'app folder'.
This allows web mapping applications to be represented in a single file, such as the [`coffeeApp/app.R`](https://github.com/Robinlovelace/geocompr/blob/master/coffeeApp/app.R) file in the book's GitHub repo.

\BeginKnitrBlock{rmdnote}<div class="rmdnote">In **shiny** apps these are often split into `ui.R` (short for user interface) and `server.R` files, naming conventions used by [`shiny-server`](https://github.com/rstudio/shiny-server), a server-side Linux application for serving shiny apps on public-facing websites.
`shiny-server` also serves apps defined by a single `app.R` file in an 'app folder'.</div>\EndKnitrBlock{rmdnote}

Before considering large apps it is worth seeing a minimal example, named 'lifeApp', in action.^[
The word 'app' in this context refers to 'web application' and should not be confused with smartphone apps, the more common meaning of the word.
]
The code below defines and launches --- with the command `shinyApp()` --- a lifeApp, which provides an interactive slider allowing users to make countries appear with progressively lower levels of life expectancy (see Figure \@ref(fig:lifeApp)):


```r
library(shiny)    # for shiny apps
library(leaflet)  # renderLeaflet function
library(spData)   # loads the world dataset 
ui = fluidPage(
  sliderInput(inputId = "life", "Life expectancy", 49, 84, value = 80),
      leafletOutput(outputId = "map")
  )
server = function(input, output) {
  output$map = renderLeaflet({
    leaflet() %>% addProviderTiles("OpenStreetMap.BlackAndWhite") %>%
      addPolygons(data = world[world$lifeExp < input$life, ])})
}
shinyApp(ui, server)
```

<div class="figure" style="text-align: center">
<img src="https://user-images.githubusercontent.com/1825120/39690606-8f9400c8-51d2-11e8-84d7-f4a66a477d2a.png" alt="Screenshot showing minimal example of a web mapping application created with shiny."  />
<p class="caption">(\#fig:lifeApp)Screenshot showing minimal example of a web mapping application created with shiny.</p>
</div>

The **user interface** (`ui`) of lifeApp is created by `fluidPage()`.
This contains input and output 'widgets' --- in this case, a `sliderInput()` (many other `*Input()` functions are available) and a `leafletOutput()`.
These are arranged row-wise by default, explaining why the slider interface is placed directly above the map in Figure \@ref(fig:lifeApp) (see `?column` for adding content column-wise).

The **server side** (`server`) is a function with `input` and `output` arguments.
`output` is a list of objects containing elements generated by `render*()` function --- `renderLeaflet()` which in this example generates `output$map`.
Inputs elements such as `input$life` referred to in the server must relate to elements that exist in the `ui` --- defined by `inputId = "life"` in the code above.
The function `shinyApp()` combines both the `ui` and `server` elements and serves the results interactively via a new R process.
When you move the slider in the map shown in Figure \@ref(fig:lifeApp), you are actually causing R code to re-run, although this is hidden from view in the user interface.

Building on this basic example and knowing where to find help (see `?shiny`), the best way forward now may be to stop reading and start programming!
The recommended next step is to open the previously mentioned [`coffeeApp/app.R`](https://github.com/Robinlovelace/geocompr/blob/master/coffeeApp/app.R) script in an IDE of choice, modify it and re-run it repeatedly.
The example contains some of the components of a web mapping application implemented in **shiny** and should 'shine' a light on how they behave.

The `coffeeApp/app.R` script contains **shiny** functions that go beyond those demonstrated in the simple 'lifeApp' example.
These include `reactive()` and `observe()` (for creating outputs that respond to the user interface --- see `?reactive`) and `leafletProxy()` (for modifying a `leaflet` object that has already been created).
Such elements are critical to the creation of web mapping applications implemented in **shiny**.
A range of 'events' can be programmed including advanced functionality such as drawing new layers or subsetting data, as described in the shiny section of RStudio's **leaflet** [website.](https://rstudio.github.io/leaflet/shiny.html)

<div class="rmdnote">
<p>There are a number of ways to run a <strong>shiny</strong> app. For RStudio users the simplest way is probably to click on the ‘Run App’ button located in the top right of the source pane when an <code>app.R</code>, <code>ui.R</code> or <code>server.R</code> script is open. <strong>shiny</strong> apps can also be initiated by using <code>runApp()</code> with the first argument being the folder containing the app code and data: <code>runApp(&quot;coffeeApp&quot;)</code> in this case (which assumes a folder named <code>coffeeApp</code> containing the <code>app.R</code> script is in your working directory). You can also launch apps from a Unix command line with the command <code>Rscript -e 'shiny::runApp(&quot;coffeeApp&quot;)'</code>.</p>
</div>

Experimenting with apps such as `coffeeApp` will build not only your knowledge of web mapping applications in R but your practical skills.
Changing the contents of `setView()`, for example, will change the starting bounding box that the user sees when the app is initiated.
Such experimentation should not be done at random, but with reference to relevant documentation, starting with `?shiny`, and motivated by a desire to solve problems such as those posed in the exercises.

**shiny** used in this way can make prototyping mapping applications faster and more accessible than ever before (deploying **shiny** apps is a separate topic beyond the scope of this chapter).
Even if your applications are eventually deployed using different technologies, **shiny** undoubtedly allows web mapping applications to be developed in relatively few lines of code (60 in the case of coffeeApp).
That does not stop shiny apps getting rather large.
The Propensity to Cycle Tool (PCT) hosted at [pct.bike](http://www.pct.bike/), for example, is a national mapping tool funded by the UK's Department for Transport.
The PCT is used by dozens of people each day and has multiple interactive elements based on more than 1000 lines of [code](https://github.com/npct/pct-shiny/blob/master/regions_www/m/server.R) [@lovelace_propensity_2017].

While such apps undoubtedly take time and effort to develop, **shiny** provides a framework for reproducible prototyping that should aid the development process.
One potential problem with the ease of developing prototypes with **shiny** is the temptation to start programming too early, before the purpose of the mapping application has been envisioned in detail.
For that reason, despite advocating **shiny**, we recommend starting with the longer established technology of a pen and paper as the first stage for interactive mapping projects.
This way your prototype web applications should be limited not by technical considerations but by your motivations and imagination.

<div class="figure" style="text-align: center">
<iframe src="https://bookdown.org/robinlovelace/coffeeapp/?showcase=0" width="576" height="400px"></iframe>
<p class="caption">(\#fig:coffeeApp)coffeeApp, a simple web mapping application for exploring global coffee production in 2016 and 2017.</p>
</div>

## Other mapping packages

**tmap** provides a powerful interface for creating a wide range of static maps (section \@ref(static-maps)) and also supports interactive maps (section \@ref(interactive-maps)).
But there are many other options for creating maps in R.
The aim of this section is to provide a taster of some of these and pointers for additional resources: map making is a surprisingly active area of R package development so there is more to learn than can be covered here.

The most mature option is to use `plot()` methods provided by core spatial packages **sf** and **raster**, covered in sections \@ref(basic-map) and \@ref(basic-map-raster) respectively.
What we have not mentioned in those sections was that plot methods for raster and vector objects can be combined when the results draw onto the same plot area (elements such as keys in **sf** plots and multi-band rasters will interfere with this).
This behavior is illustrated in the subsequent code chunk which generates Figure \@ref(fig:nz-plot).
`plot()` has many other options which can be explored by following links in the `?plot` help page and the **sf** vignette [`sf5`](https://cran.r-project.org/web/packages/sf/vignettes/sf5.html).


```r
g = st_graticule(nz, lon = c(170, 175), lat = c(-45, -40, -35))
plot(nz_water, graticule = g, axes = TRUE, col = "blue")
raster::plot(nz_elev / 1000, add = TRUE)
plot(st_geometry(nz), add = TRUE)
```

<div class="figure" style="text-align: center">
<img src="figures/nz-plot-1.png" alt="Map of New Zealand created with plot(). The legend to the right refers to elevation (1000 m above sea level)." width="576" />
<p class="caption">(\#fig:nz-plot)Map of New Zealand created with plot(). The legend to the right refers to elevation (1000 m above sea level).</p>
</div>

Since version [2.3.0](https://www.tidyverse.org/articles/2018/05/ggplot2-2-3-0/), the **tidyverse** plotting package **ggplot2** has supported `sf` objects with `geom_sf()`.
The syntax is similar to that used by **tmap**:
an initial `ggplot()` call is followed by one or more layers, that are added with `+ geom_*()`, where `*` represents a layer type such as `geom_sf()` (for sf objects) or `geom_points()` (for points).

**ggplot2** plots graticules by default.
The default settings for the graticules can be overridden using `scale_x_continuous()` and `scale_y_continuous()`.
Other notable features include the use of unquoted variable names encapsulated in `aes()` to indicate which aesthetics vary and switching data sources using the `data` argument, as demonstrated in the code chunk below which creates Figure \@ref(fig:nz-gg):


```r
library(ggplot2)
g1 = ggplot() + geom_sf(data = nz, aes(fill = Median_income)) +
  geom_sf(data = nz_height) +
  scale_x_continuous(breaks = c(170, 175))
g1
```

<div class="figure" style="text-align: center">
<img src="figures/nz-gg-1.png" alt="Map of New Zealand created with ggplot2." width="576" />
<p class="caption">(\#fig:nz-gg)Map of New Zealand created with ggplot2.</p>
</div>

An advantage of **ggplot2** is that it has a strong user-community and many add-on packages.
Good additional resources can be found in the open source [ggplot2 book](https://github.com/hadley/ggplot2-book) [@wickham_ggplot2_2016] and in the descriptions of the multitude of '**gg**packages' such as **ggrepel** and **tidygraph**.

Another benefit of maps based on **ggplot2** is that they can easily be given a level of interactivity when printed using the function `ggplotly()` from the **plotly** package.
Try `plotly::ggplotly(g1)` for example, and compare the result with other **plotly** mapping functions described at [blog.cpsievert.me](https://blog.cpsievert.me/2018/03/30/visualizing-geo-spatial-data-with-sf-and-plotly/).



At the same time, **ggplot2** has a few drawbacks.
The `geom_sf()` function is not always able to create a desired legend to use from the spatial [data](https://github.com/tidyverse/ggplot2/issues/2037).
Raster objects are also not natively supported in **ggplot2** and need to be converted into a data frame before plotting.

We have covered mapping with **sf**, **raster** and **ggplot2** packages first because these packages are highly flexible, allowing for the creation of a wide range of static maps.
<!-- Many other static mapping packages are more specific. -->
Before we cover mapping packages for plotting a specific type of map (in the next paragraph), it is worth considering alternatives to the packages already covered for general-purpose mapping (Table \@ref(tab:map-gpkg)).


Table: (\#tab:map-gpkg)Selected general-purpose mapping packages.

package       title                                                            
------------  -----------------------------------------------------------------
cartography   Thematic Cartography                                             
ggplot2       Create Elegant Data Visualisations Using the Grammar of Graphics 
googleway     Accesses Google Maps APIs to Retrieve Data and Plot Maps         
ggspatial     Spatial Data Framework for ggplot2                               
leaflet       Create Interactive Web Maps with Leaflet                         
mapview       Interactive Viewing of Spatial Data in R                         
plotly        Create Interactive Web Graphics via 'plotly.js'                  
rasterVis     Visualization Methods for Raster Data                            
tmap          Thematic Maps                                                    

Table \@ref(tab:map-gpkg) shows a range of mapping packages are available, and there are many others not listed in this table.
Of note is **cartography**, which generates a range of unusual maps including choropleth, 'proportional symbol' and 'flow' maps, each of which is documented in the vignette [`cartography`](https://cran.r-project.org/web/packages/cartography/vignettes/cartography.html).

Several R packages also allows for plotting specific map types (Table \@ref(tab:map-spkg)).
They prepare cartograms, create line maps, transform polygons into regular or hexagonal grids, and visualize complex data on grids representing geographic topologies.


Table: (\#tab:map-spkg)Selected specific-purpose mapping packages, with associated metrics.

package     title                                                    
----------  ---------------------------------------------------------
cartogram   Create Cartograms with R                                 
geogrid     Turn Geospatial Polygons into Regular or Hexagonal Grids 
geofacet    'ggplot2' Faceting Utilities for Geographical Data       
linemap     Line Maps                                                

All of the aforementioned packages, however, have different approaches for data preparation and map creation.
In the next paragraph, we focus solely on the **cartogram** package.
Therefore, we suggest to read the [linemap](https://github.com/rCarto/linemap), [geogrid](https://github.com/jbaileyh/geogrid) and [geofacet](https://github.com/hafen/geofacet) documentations to learn more about them.

A cartogram is a map in which the geometry is proportionately distorted to represent a mapping variable. 
Creation of this type of map is possible in R with **cartogram**, which allows for creating continuous and non-contigous area cartograms.
It is not a mapping package per se, but it allows for construction of distorted spatial objects that could be plotted using any generic mapping package.

The `cartogram_cont()` function creates continuous area cartograms.
It accepts an `sf` object and name of the variable (column) as inputs.
Additionally, it is possible to modify the `intermax` argument - maximum number of iterations for the cartogram transformation.
For example, we could represent median income in New Zeleand's regions as a continuous cartogram (the right-hand panel of Figure \@ref(fig:cartomap1)) as follows:


```r
library(cartogram)
nz_carto = cartogram_cont(nz, "Median_income", itermax = 5)
tm_shape(nz_carto) + tm_polygons("Median_income")
```

<div class="figure" style="text-align: center">
<img src="figures/cartomap1-1.png" alt="Comparison of standard map (left) and continuous area cartogram (right)." width="576" />
<p class="caption">(\#fig:cartomap1)Comparison of standard map (left) and continuous area cartogram (right).</p>
</div>

**cartogram** also offers creation of non-contiguous area cartograms using  `cartogram_ncont()` and Dorling cartograms using `cartogram_dorling()`.
Non-contiguous area cartograms are created by scaling down each region based on the provided weighting variable.
Dorling cartograms consist of circles with their area proportional to the weighting variable.
The code chunk below demonstrates creation of non-contiguous area and Dorling cartograms of US states' population (Figure \@ref(fig:cartomap2)):


```r
us_states2163 = st_transform(us_states, 2163)
us_states2163_ncont = cartogram_ncont(us_states2163, "total_pop_15")
us_states2163_dorling = cartogram_dorling(us_states2163, "total_pop_15")
```

<div class="figure" style="text-align: center">
<img src="figures/cartomap2-1.png" alt="Comparison of non-continuous area cartogram (top) and Dorling cartogram (bottom)." width="576" />
<p class="caption">(\#fig:cartomap2)Comparison of non-continuous area cartogram (top) and Dorling cartogram (bottom).</p>
</div>

New mapping packages are emerging all the time.
In 2018 alone, a number of mapping packages have been released on CRAN, including
**mapdeck**, **rayshader**, **muHVT** and **mapsapi**.
In terms of interactive mapping, **leaflet.extras** contains many functions for extending the functionality of **leaflet** (see the end of the [`point-pattern`](https://geocompr.github.io/geocompkg/articles/point-pattern.html) vignette in the **geocompkg** website for examples of heatmaps created by **leaflet.extras**).

## Exercises

For these exercises we will create a new object, `africa`, using the `world` and `worldbank_df` datasets from the **spData** package (see chapter \@ref(attr) to learn more about attribute operations) as follows:


```r
africa = world %>% 
  filter(continent == "Africa", !is.na(iso_a2)) %>% 
  left_join(worldbank_df, by = "iso_a2") %>% 
  dplyr::select(name, subregion, gdpPercap, HDI, pop_growth) %>% 
  st_transform("+proj=aea +lat_1=20 +lat_2=-23 +lat_0=0 +lon_0=25")
```

We will also use `zion` and `nlcd` datasets from **spDataLarge**:


```r
zion = st_read((system.file("vector/zion.gpkg", package = "spDataLarge")))
```

1. Create a map showing the geographic distribution of the Human Development Index (`HDI`) across Africa with 
-base graphics (hint: use `plot()`) and
-**tmap** (hint: use `tm_shape(africa) + ...`).
    - Name two advantages of each approach
    - What three other mapping packages could be used to show the same data?
    - Name an advantage of each
    - Bonus: create three more maps of Africa using the three packages mentioned previously
1. Extend the map of Africa created with **tmap** for the previous exercise so the legend has three bins: "High" (`HDI` above 0.7), "Medium" (`HDI` between 0.55 and 0.7) and "Low" (`HDI` below 0.55).
    - Bonus: improve the map aesthetics, for example by changing the legend title, class labels and color palette.

1. Represent `africa`'s subregions on the map. 
Change the default color palette and legend title.
Next, combine this map and the map created in the previous exercise into a single plot.

1. Create a land cover map of the Zion National Park.
    - Change the default colors to match your perception of the land cover categories
    - Add a scale bar and north arrow and change the position of both to improve the map's aesthetic appeal
    - Bonus: Add an inset map of the Zion National Park's location in the context of the Utah state. (Hint: an object representing Utah can be subsetted from the `us_states` dataset.) 

1. Create facet maps of countries in Eastern Africa:
    - with one facet showing HDI and the other representing population growth (hint: using variables `HDI` and `pop_growth` respectively)
    - with a 'small multiple' per country <!-- animated map, interactive map -->

1. Building on the previous facet map examples, create animated maps of East Africa:
    - showing first the spatial distribution of HDI scores then population growth
    - showing each country in order

1. Create an interactive map of Africa:
    - with **tmap**
    - with **mapview**
    - with **leaflet**
    - bonus: for each approach add a legend (if not automatically provided) and a scale bar
1. Sketch on paper ideas for a web mapping app that could be used to make transport or land-use policies more evidence based:
    - In the city you live in, for a couple of users per day
    - In the country you live in, for dozens of users per day
    - Worldwide for hundreds of users per day and large data serving requirements
1. How would app design, deployment and project management decisions change as the scale of map deployment increases?
1. Update the code in `coffeeApp/app.R` so that instead of centering on Brazil the user can select which country to focus on:
    - Using `textInput()`
    - Using `selectInput()`
1. Reproduce Figure \@ref(fig:tmshape) and the 1st and 6th panel of Figure \@ref(fig:break-styles) as closely as possible using the **ggplot2** package.

1. Join `us_states` and `us_states_df` together and calculate a poverty rate for each state using the new dataset.
Next, construct a continuous area cartogram based on total population. 
Finally, create and compare two maps of the poverty rate: (1) a standard choropleth map and (2) a map using the created cartogram boundaries.
What is the information provided by the first and the second map?
How do they differ from each other?

1. Visualize population growth in Africa. 
Next, compare it with the maps of a hexagonal and regular grid created using the **geogrid** package.
