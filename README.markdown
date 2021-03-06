EzSVG
=====

Lua library for easy SVG generation
-----------------------------------

This is a preview release of EzSVG. Interface is not yet finalized and some features are still missing. This document will first inform you about the core mechanics of EzSVG and then give an overview of its functions/methods. It will link to W3Cs SVG specification, as EzSVG just wraps the SVG language in a Lua way.

### What you can do with EzSVG

EzSVG can generate SVG documents generated by Lua code by creating and nesting SVG Elements. The elements are created by constructor functions (all functions attached to EzSVG starting with a capital letter) and represented as Lua tables. All elements, directly (or indirectly, via group or by reference) added to a main element, can be rendered with just one method call.

#### Renderings of the example files

* [example1.lua – Rotated numbers and some sine lines](http://www.cappel-nord.de/ezsvg-examples/example1.png)
* [example2.lua – Circular eye thing ...](http://www.cappel-nord.de/ezsvg-examples/example2.png)
* [example3.lua – Lyrics spiral](http://www.cappel-nord.de/ezsvg-examples/example3.png)
* [example4.lua – Cute rainbow buttons](http://www.cappel-nord.de/ezsvg-examples/example4.png)
* [example5.lua – Futuristic lines](http://www.cappel-nord.de/ezsvg-examples/example5.png)
* [example6.lua – A circular black/white pattern test](http://www.cappel-nord.de/ezsvg-examples/example6.png)
* [example7.lua – A hommage to the Caribou/Swim cover](http://www.cappel-nord.de/ezsvg-examples/example7.png)

### Good stuff in EzSVG

Besides generating the SVG document by the means of nested Lua tables some other features are implemented:

* Style properties can be written with an underscore instead of a dash (e.g. font_size instead of font-size) allowing for Lua literal table syntax when styling elements.
* Table methods return the table itself, allowing for method-call lists.
* Elements referenced by other elements are automagically inserted into the SVG document.
* Pure Lua library, small footprint: Mix it with every other library/framework – no collision of namespaces.

### Bad stuff in EzSVG

The main goal of EzSVG is generative artwork. EzSVG is neither about a complete set of features nor about output code quality. EzSVG can't read or render SVG. The user can create invalid SVG code with EzSVG.

### Lua example showing some features

```lua
 -- include the library
EzSVG = require "EzSVG"

 -- create a document
local doc = EzSVG.Document(1000, 1000)

 -- set styling for all new created elements
EzSVG.setStyle({
    stroke_width= 2,
    stroke= "black"
})

for d=0,1000,100 do
    -- create a circle
    local circle = EzSVG.Circle(d, d, d/10)
    -- add the circle to the doc
    doc:add(circle)
     -- set a fill color
    circle:setStyle("fill", EzSVG.rgb(d/4, 0, 0))
end

-- you can also set a single style
EzSVG.setStyle("stroke", "green")

-- create a group (very handy, also stays a group in Inkscape/Illustrator)
local group = EzSVG.Group()

for r=0,360, 10 do
    -- create a line and add a transform (rotation with r degrees, centered on 500/500)
    local line = EzSVG.Line(100, 500, 900, 500):rotate(r, 500, 500)
    -- add it to the group
    group:add(line)     
end

-- add the group to the document
doc:add(group)

-- clear the style
EzSVG.clearStyle()

-- create a path object and set its styling
local path = EzSVG.Path({
    stroke = "blue",
    stroke_width = 2,
    fill = EzSVG.RadialGradient():addStop(0, "black"):addStop(100, "yellow"),
    fill_opacity = "0.8"
})

-- draw the path
path:moveToA(500, 500)
    :sqCurveTo(300,300)
    :sqCurveTo(-200, 0)
    :sqCurveTo(-200, -400)
    :sqCurveToA(500, 500)
    
-- add path to the doc
doc:add(path)

-- add text to doc, add a path and style it
doc:add(EzSVG.Text(
    "It's so ez to draw some lines, it's so easy to draw some lines ..."
):setPath(path):setStyle({
    font_size= 60,
    font_family= "Arial",
    fill= "#CCCCCC",
    stroke= "black"
}))

 -- generate the svg and write to file
doc:writeTo("doc-example.svg")
```

[Here is the result rendered with Inkscape](http://www.cappel-nord.de/ezsvg-examples/doc-example.png)

Function/Method Reference
-------------------------

### Container Element Constructors
Containers contain other Elements which can be added with the add method:

**Container:add(element)**
Adds an *element* to the group. Be aware, that no copy of the *element* is made (all changes to the *element* are still affecting the *element* in the group).

#### EzSVG.Document(width, height, bgcolor, style)
Creates the root [&lt;svg&gt;](http://www.w3.org/TR/SVG/struct.html#SVGElement) document with specified *width* and *height*. If a *bgcolor* is specified a Rect of the same size is added automatically.

**Document:writeTo(filename)**
The *writeTo* method renders the SVG document to a file with the specified *filename*

**Example**
```lua
-- creates a SVG document with a red background and saves it
local doc = EzSVG.Document(100, 100, "red")
doc:writeTo("output.svg")
```

#### EzSVG.SVG(x, y, width, height, style)
Creates an empty [&lt;svg&gt;](http://www.w3.org/TR/SVG/struct.html#SVGElement) element. It's content can be embeded into another SVG document. Elements created with the SVG method can't be rendered to a file (use *EzSVG.Document* instead).

#### EzSVG.Group(style)
Creates an empty [&lt;g&gt;](http://www.w3.org/TR/SVG/struct.html#Groups) (group) element. Groups are very handy, because groups will also be groups in vector graphic programs such as Inkscape or Adobe Illustrator. Groups can be used in conjunction with *EzSVG.Use* to create elements that are repeated throughout the document.

### Shape Element Constructors

#### EzSVG.Circle(cx, cy, r, style)
Creates a [&lt;circle&gt;](http://www.w3.org/TR/SVG/shapes.html#CircleElement) with center point *cx/cy* and radius *r*.

#### EzSVG.Ellipse(cx, cy, rx, ry, style)
Creates a [&lt;ellipse&gt;](http://www.w3.org/TR/SVG/shapes.html#EllipseElement) with center point cx/cy and radii rx and ry.

#### EzSVG.Rect(x, y, width, height, rx, ry, style)
Creates a [&lt;rect&gt;](http://www.w3.org/TR/SVG/shapes.html#RectElement) with *x/y* specifying the side of the rectangle which has the smaller y-axis coordinate value in the current user coordinate system. *width/height* specify the dimensions of the rectangle. The *rx/ry* parameters can be used to give the rectangle round corners.

#### EzSVG.Line(x1, y1, x2, y2, style)
Creates a straight [&lt;line&gt;](http://www.w3.org/TR/SVG/shapes.html#LineElement) between *x1/y1* and *x2/y2*.

#### EzSVG.Polyline(points, style)
#### EzSVG.Polygon(points, style)
Creates a [&lt;polyline&gt;](http://www.w3.org/TR/SVG/shapes.html#PolylineElement) or [&lt;polygon&gt;](http://www.w3.org/TR/SVG/shapes.html#PolygonElement) with the table *points* as his vertices. The points table must have an even size, and has the layout {x1, y1, x2, y2, x3, y3, …}. The difference between a polygon and a polyline is basically, that a polygon is closing itself automatically (draws a line to its start point).

**Element:addPoint(x, y)**
Adds a point to the polygon/polyline element after creation.

### Other Element Constructors

#### EzSVG.Use(href, x, y, width, height, style)
Creates an [&lt;use&gt;](http://www.w3.org/TR/SVG/struct.html#UseElement) element with which you can place any other element (or a copy of that) on the canvas. The *href* can be either an element table or a direct href string. You can specify positions and dimensions with *x/y* and *width/height*.

#### EzSVG.Image(href, x, y, width, height, style)
Creates an [&lt;image&gt;](http://www.w3.org/TR/SVG/struct.html#ImageElement) element and places the image to the coordinates *x/y* with dimensions of *width/height*. The image *href* is specified by a String path (e.g. "img/flower1.png"). PNG and JPG are good choices for image file formats.

### Color Functions
Color functions return SVG compatible color notation as RGB strings. Range is from 0 to 255. In general you can use all color notations that are compatible with SVG, e.g. "rgb(255, 0, 0)", "#FF0000" and "red" are all the same.

#### EzSVG.rgb(red, green, blue)
#### EzSVG.gray(value)
#### EzSVG.hsv(hue, saturation, value)

### Styling Elements
EzSVG uses property values instead of the style property used for HTML/CSS. You can use the underscore character instead of the dash in all style methods, which is handy for literal table notation. You can use numbers and strings as values. In some cases you can also use elements as values (for example use a gradient for the fill property). EzSVG will turn it into a reference automagically.

There are many ways to set the style of an element:

* Set the style with EzSVG.setStyle before element creation.
* Use the style parameter at the elements creation.
* Use the setStyle method of an element after creation.
* Add a property directly into the table (you can't use underscores here!)

All created elements start with the current style table. These properties are overriden by the style parameter or with setStyle. Directly setting properties in the elements table override styles.

[Here is a list of all available style properties.](http://www.w3.org/TR/SVG/styling.html#SVGStylingProperties)

####Element:setStyle(table) or Element:setStyle(key, value)
Add/change style properties of the element, either with a key/value table or with just one key/value.

####Element:mergeStyle(table) or Element:mergeStyle(key, value)
Only add style properties to the element, already set properties aren't overwritten.

####Element:clearStyle()
Clears the style of the element. Styling attached as properties to the table stay.

####EzSVG.setStyle(table, tag) or Element.setStyle(key, value, tag)
Add/change style properties of the style table for the specified element tag. If no tag is specified the style for all tags is changed.

####EzSVG.pushStyle() / EzSVG.popStyle()
Push/pop the current style to the style stack. You can use this to save the current style and go back to this point.

### Experimental Functions
You can use them but I'll probably/surely change the interface and/or behaviour of them in the future. Currently they have too many arguments which makes them not as easy to use as I wish. I want to have ez defaults that work most of the time but I don't want to loose any features.

#### EzSVG.LinearGradient(x1, y1, x2, y1, userSpaceUnits, spread,  style)
#### EzSVG.RadialGradient(cx, cy, r, fx, fy, userSpaceUnits, spread, style)
#### EzSVG.Pattern(x, y, width, height, preserveAspectRatio, patternUnits, patternContentUnits, viewbox, style)
#### EzSVG.Mask(x, y, width, height, maskUnits, maskContentUnits, style)
