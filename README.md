
<!-- README.md is generated from README.Rmd. Please edit that file -->

# ggsvg - Use SVG as points in ggplot

<!-- badges: start -->

![](https://img.shields.io/badge/cool-useless-green.svg)
<!-- badges: end -->

`ggsvg` is an extension to ggplot to use arbitrary SVG as points.

This SVG can be customised to respond to aesthetics e.g. element colours
can changes in response to fill and/or colour scales.

However, aesthetics are not limited to colour - any other SVG parameter
can be linked to any aesthetic which makes sense e.g. an aesthethic may
be used to control the corner radius on a rounded rectangle.

## What’s in the box

-   `geom_point_svg()` is equivalent to `geom_point()` except it also
    requires SVG text to be set (via the `svg` argument)
-   `scale_svg_*` a full(?) set of compatible scale functions for
    controlling the mapping of values to arbitrary named aesthetics.

## Installation

You can install from [GitHub](https://github.com/coolbutuseless/ggsvg)
with:

``` r
# install.package('remotes')
remotes::install_github('coolbutuseless/cssparser')
remotes::install_github('coolbutuseless/svgparser')
remotes::install_github('coolbutuseless/ggsvg')
```

#### Debugging

Set `options(GGSVG_DEBUG = TRUE)` for some verbose debugging.

This will cause `{ggsvg}` to output the final SVG for each and every
element it wants to put on the plot.

## Simple SVG Image

An SVG Christmas Tree with a hex star on top!

``` r
library(ggplot2)
library(ggsvg)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Define simple SVG
#   - Square with rounded corners and a circle inside it.
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
svg_text <- '
  <svg viewBox="0 0 100 100 ">
    <polygon points = "20,80 80,80 50,20" fill="darkgreen" />
    <polygon points = "40,83 60,83 60,95 40,95" fill="darkred" />
    <polygon points = "58.66 25.00 50.00 30.00 41.34 25.00 41.34 15.00 50.00 10.00 58.66 15.00 58.66 25.00" 
       fill="yellow3" />
  </svg>
  '


#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Render SVG with `{svgparser}`
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
grob <- svgparser::read_svg(svg_text)
grid::grid.newpage()
grid::grid.draw(grob)
```

<img src="man/figures/README-simple_svg-1.png" width="100%" />

## Simple SVG drawn for each point

Here, the simple SVG will be used as a plotting character.

The point locations are given by the capital “R” in the Hershey script
font.

``` r
# remotes::install_github('coolbutuseless/hershey')
library(hershey)
letter <- hershey[hershey$char == 'R' & hershey$font == 'scriptc',]

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Use 'geom_point_svg' to plot SVG image at each point
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ggplot(letter) +
  geom_path(aes(x, y, group = stroke), alpha = 0.5) +
  geom_point_svg(
    mapping  = aes(x, y),
    svg      = svg_text,
    size     = 10
  ) +
  theme_bw() + 
  coord_equal() + 
  labs(
    title    = "Merry Christmas #RStats",
    subtitle = "{ggsvg} Using SVG as points"
  )
```

<img src="man/figures/README-simple-1.png" width="100%" />

## Simple aesthetics

This is a basic example showing how ggplot aesthetics may be used to
control SVG image properties.

Some key things to note:

-   Locations for ggplot to insert aesthetics via the `glue` package are
    marked with **double** curly braces i.e. `{{...}}`.
-   One variable has been added to the SVG - `{{fill_tree}}`. This is
    used in place of the static colour that was defined in the original
    SVG
-   `fill_tree` must now appear as an aesthetic in the call to
    `geom_point_svg()`
-   We need to inform `ggplot2` of the default value for each new
    aesthetic by setting the `defaults` argument
-   For each aesthetic, you will also need to add a scale with
    `scale_svg_*()` to let ggplot know how it should turn the mapped
    variable into a value to insert in the SVG.
-   In this case, the variables is a `fill` variable, so use one of
    `scale_svg_fill_*()` family to ensure that the value is mapped to a
    colour.
-   The new `scale_svg_*()` scales are mostly identical to their
    `ggplot` counterparts except:
    -   The `aesthetics` argument no longer has a default value
    -   `colourbar` guides have been tweaked to allow for non-standard
        aesthetics.

``` r
library(ggplot2)
library(ggsvg)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Define some SVG
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
svg_text <- '
  <svg viewBox="0 0 100 100 ">
    <polygon points = "20,80 80,80 50,20" fill="{{fill_tree}}" />
    <polygon points = "40,83 60,83 60,95 40,95" fill="darkred" />
    <polygon points = "58.66 25.00 50.00 30.00 41.34 25.00 41.34 15.00 50.00 10.00 58.66 15.00 58.66 25.00" 
       fill="yellow3" />
  </svg>
  '

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Use 'geom_svg' to plot with this symbol
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ggplot(mtcars) +
  geom_point_svg(
    mapping     = aes(
      x           = mpg, 
      y           = wt, 
      size        = cyl, 
      fill_tree   = as.factor(cyl)
    ),
    svg         = svg_text,
    defaults    = list(fill_tree = 'darkgreen')
  ) +
  labs(
    title    = "{ggsvg} Simple SVG with fill aesthetic",
    subtitle = "`cyl` mapped to tree colour and size"
  ) + 
  theme_bw() +
  scale_size(range = c(5, 12), guide = 'none') + 
  scale_svg_fill_viridis_d('fill_tree', guide = guide_legend(override.aes = list(size = 7)))
```

<img src="man/figures/README-simpleaes-1.png" width="100%" />

## More complex example - Virus with multiple aesthetics

This is an SVG image of something virus-like from
[wikimedia](https://commons.wikimedia.org/wiki/File:Virus_black_white.svg)
- it is public domain.

``` r
svg_file <- system.file("Virus_black_white.svg", package = "ggsvg")
grob <- svgparser::read_svg(svg_file)
grid::grid.draw(grob)
```

<img src="man/figures/README-unnamed-chunk-2-1.png" width="100%" />

I have hand-edited the SVG to add two locations for `{glue}` to insert
text:

1.  `fill = "{{fill_inner}}"` has been added to all elements in the
    central body of the virus.
2.  `fill = "{{fill_outer}}"` has been added to the “arms” of the virus

``` r
library(ggplot2)
library(ggsvg)

fake_data <- readr::read_csv(
'county, month, count, majority
A, 1, 1.00, delta
A, 2, 1.40, delta
A, 3, 1.96, delta
A, 4, 2.74, delta
A, 5, 3.85, omicron
A, 6, 5.38, omicron
B, 1, 1.00, delta
B, 2, 1.10, delta
B, 3, 1.21, omicron
B, 4, 1.33, omicron
B, 5, 1.46, omicron
B, 6, 1.61, omicron
', show_col_types = FALSE)


#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Read in the SVG as text.
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
svg_file <- system.file("virus.svg", package = "ggsvg")
svg_text <- paste(readLines(svg_file), collapse = "\n")

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Use 'geom_svg' to plot with this symbol
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ggplot(fake_data) +
  geom_line(aes(month, count, group = county)) +
  geom_point_svg(
    mapping = aes(month, count, 
                  size = count, 
                  fill_inner = county, 
                  fill_outer = majority),
    svg         = svg_text,
    defaults    = list(fill_inner = '#aaaaaa80', fill_outer = '#aaaaaa80')
  ) +
  theme_bw() +
  scale_size_continuous(range = c(6, 20), guide = 'none') +
  scale_svg_fill_brewer(
    aesthetics = 'fill_inner', 
    palette = 'Dark2',
    guide = guide_legend(override.aes = list(size = 9))
  ) +
  scale_svg_fill_manual(
    aesthetics = 'fill_outer',
    values = c(delta = 'grey50', omicron = 'darkred'),
    guide = guide_legend(override.aes = list(size = 9))
  ) + 
  labs(
    title    = "{ggsvg} - Complex SVG with multiple fill aesthetics",
    subtitle = "Virus body coloured by county\nVirus tendrils coloured by delta/omicron majority status"
  )
```

<img src="man/figures/README-coronavirus-1.png" width="100%" />

## Mushroom plot

A style of plot consistning of two semi-circles which can be
sized/styled independently

#### Create a simple SVG

Note: Semicircles are drawn with path arcs in SVG and aren’t that
intuitive.

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Hand craft two semi-circles using SVG path arcs
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
semicircles_svg <- '
<svg width="100" height="100">
  <path d="M 0,50 a50,50 0 1,1 100,0" fill="#E79A16" />
  <path d="M 0,50 a50,50 0 0,0 100,0" fill="#D78590" />
</svg>
'

grob <- svgparser::read_svg(semicircles_svg)
grid::grid.newpage()
grid::grid.draw(grob)
```

<img src="man/figures/README-unnamed-chunk-3-1.png" width="100%" />

#### Introduce mappable locations

Add `{{}}` glue string locations and test it out

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Parameterise the radii for use with glue
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
semicircles_svg <- '
<svg width="100" height="100">
  <path d="M {{50-radius_top}},50 a{{radius_top}},{{radius_top}} 0 1,1 {{2*radius_top}},0" fill="#E79A16" />
  <path d="M {{50-radius_bot}},50 a{{radius_bot}},{{radius_bot}} 0 0,0 {{2*radius_bot}},0" fill="#D78590" />
</svg>
'

radius_top <- 40
radius_bot <- 30

final_svg <- glue::glue(semicircles_svg, .open = "{{", .close = "}}")


grob <- svgparser::read_svg(final_svg)
grid::grid.newpage()
grid::grid.draw(grob)
```

<img src="man/figures/README-unnamed-chunk-4-1.png" width="100%" />

#### Create a plot mapping 2 different quantities to the size

``` r
N <- 20
data <- data.frame(
  x  = runif(N),
  y  = runif(N),
  q1 = runif(N),
  q2 = runif(N),
  stringsAsFactors = FALSE
)

# options(GGSVG_DEBUG = TRUE)

ggplot(data) + 
  geom_point_svg(
    aes(x, y, radius_top = q1, radius_bot = q2), 
    svg = semicircles_svg, 
    size = 10,
    defaults = list(radius_top = 50, radius_bot = 50)) +
  scale_svg_size('radius_top', range = c(30, 50), guide = 'none') + 
  scale_svg_size('radius_bot', range = c(30, 50), guide = 'none') + 
  theme_bw() +
  labs(
    title = "{ggsvg} Multiple continuous aesthetics"
  )
```

<img src="man/figures/README-unnamed-chunk-5-1.png" width="100%" />

## Using an SVG icon from the web

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Read SVG from the web
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
car_url <- 'https://www.svgrepo.com/download/114837/car.svg'
car_svg <- paste(readLines(car_url), collapse = "\n")
```

``` r
car_grob <- svgparser::read_svg(car_svg)
grid::grid.newpage()
grid::grid.draw(car_grob)
```

<img src="man/figures/README-unnamed-chunk-8-1.png" width="100%" />

``` r
ggplot(mtcars) + 
  geom_point_svg(
    aes(mpg, wt),
    svg = car_svg,
    size = 8
  ) + 
  theme_bw()
```

<img src="man/figures/README-unnamed-chunk-9-1.png" width="100%" />

## Multiple different SVG

Since SVG is just a character string, you can set a different SVG for
every row of a data.frame, and then use this column as the `svg`
aesthetic.

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Read multiple SVG somehow
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
statue   <- paste(readLines("https://www.svgrepo.com/download/227885/statue-of-liberty.svg"), collapse = "\n")
building <- paste(readLines("https://www.svgrepo.com/download/129010/bank-sign.svg")        , collapse = "\n")
sign     <- paste(readLines("https://www.svgrepo.com/download/118570/exit-sign.svg")        , collapse = "\n")
```

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Define a data.frame mapping 'type' to actual 'svg'
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
icons_df <- data.frame(
  type = c('statue', 'building', 'sign'),
  svg  = c( statue ,  building ,  sign ),
  stringsAsFactors = FALSE
)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Create some Points-of-Interest and assign an svg to each
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
N <- 20
poi <- data.frame(
  lat = runif(N),
  lon = runif(N),
  type = sample(c('statue', 'building', 'sign'), N, replace = TRUE),
  stringsAsFactors = FALSE
)

poi <- merge(poi, icons_df)


#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# {ggsvg}
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ggplot(poi) + 
  geom_point_svg(
    aes(x = lon, y = lat, svg = I(svg)),
    size = 10
  ) + 
  labs(
    title = "{ggsvg} multiple SVG images"
  ) + 
  theme_bw()
```

<img src="man/figures/README-unnamed-chunk-12-1.png" width="100%" />

## Alternate approach to multiple different SVG

Instead of adding each `svg` to the `poi` data.frame, map `type` to the
`svg` aesthetic and use a `scale_svg_discrete_manual()` to map from
`type` to the actual SVG.

``` r
ggplot(poi) + 
  geom_point_svg(
    aes(x = lon, y = lat, svg = type),
    size = 10
  ) + 
  scale_svg_discrete_manual(
    aesthetics = 'svg', 
    values = c(statue = statue, building = building, sign = sign),
    guide = guide_legend(override.aes = list(size = 5))
  ) + 
  labs(
    title = "{ggsvg} multiple SVG images"
  ) + 
  theme_bw()
```

<img src="man/figures/README-unnamed-chunk-13-1.png" width="100%" />

## Mapping aesthetics to SVG transforms

Aesthethics are not limited to colour, fill, size etc.

*Anything* in SVG can be used as a target for an aesthetic mapping.

In this example the rotation of an SVG arrow is controlled via a value
in a data.frame.

#### Define the SVG

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Generic SVG arrow
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
arrow_text <- '
<svg viewBox="0 0 100 100">
  <g transform="rotate(45 50 50)">
    <line x1="10" y1="50" x2="90" y2="50" stroke="black" stroke-width="2" />
    <line x1="70" y1="30" x2="90" y2="50" stroke="black" stroke-width="2" />
    <line x1="70" y1="70" x2="90" y2="50" stroke="black" stroke-width="2" />
  </g>
</svg>
'

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Draw the arrow
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
svg <- svgparser::read_svg(arrow_text)
grid::grid.newpage()
grid::grid.draw(svg)
```

<img src="man/figures/README-unnamed-chunk-14-1.png" width="100%" />

#### Introduce mappable locations (glue strings!)

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# SVG arrow with controllable rotation angle
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
arrow_text <- '
<svg viewBox="0 0 100 100">
  <g transform="rotate({{angle}} 50 50)">
    <line x1="10" y1="50" x2="90" y2="50" stroke="black" stroke-width="2" />
    <line x1="70" y1="30" x2="90" y2="50" stroke="black" stroke-width="2" />
    <line x1="70" y1="70" x2="90" y2="50" stroke="black" stroke-width="2" />
  </g>
</svg>
'

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Test the rotation is changed as 'angle' is changed
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
angle <- -90
final_svg <- glue::glue(arrow_text, .open = "{{", .close = "}}")


#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Draw the rotated arrow
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
svg <- svgparser::read_svg(final_svg)
grid::grid.newpage()
grid::grid.draw(svg)
```

<img src="man/figures/README-unnamed-chunk-15-1.png" width="100%" />

#### Create a plot with rotatable SVG arrow

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Create a plausible vector field
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
set.seed(13)
value <- 360 * as.vector(ambient::normalise(ambient::noise_perlin(c(10, 10))))
data  <- cbind(expand.grid(x=1:10, y=1:10), value)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Use 'value' to control the arrow angle
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ggplot(data) + 
  geom_raster(aes(x, y, fill = value)) + 
  geom_point_svg(
    aes(x, y, angle = I(value)), 
    svg = arrow_text,
    size = 10,
    defaults = list(angle = 0)
  ) + 
  theme_bw() +
  theme(legend.position = 'none') + 
  scale_fill_viridis_c()
```

<img src="man/figures/README-unnamed-chunk-16-1.png" width="100%" />

## Acknowledgements

-   R Core for developing and maintaining the language.
-   CRAN maintainers, for patiently shepherding packages onto CRAN and
    maintaining the repository
