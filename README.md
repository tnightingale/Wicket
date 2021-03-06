##########
# Wicket #
##########

Updated **April 8, 2012** by K. Arthur Endsley

################
## Motivation ##
################

Wicket was created out of the need for a lightweight Javascript library that can translate Well-Known Text (WKT) strings into geographic features. This problem arose in the context of [OpenClimateGIS](https://github.com/arthur-e/OpenClimateGIS), a web framework for accessing and subsetting online climate data.

OpenClimateGIS emits WKT representations of user-defined geometry. Our [API explorer](http://www.openclimategis.org/builder/) allows users to define arbitrary areas-of-interest (AOIs) and view predefined AOIs on a Google Maps API instance. So, initially, the problem was converting between WKT strings and Google Maps API features. While other mapping libraries, such as [OpenLayers](http://www.openlayers.org), have very nice WKT libraries built-in, the Google Maps API, as of this writing, does not. In the (apparent) absence of lightweight, easy-to-use WKT library in Javascript, I set out to create one.

That is what Wicket aspires to be: lightweight, framework-agnostic, and useful. I hope it achieves these goals. If you find it isn't living up to that and you have ideas on how to improve it, please fork the code or [drop me a line](mailto:kaendsle@mtu.edu).

##############
## Colophon ##
##############

########################
### Acknowledgements ###
########################

The following open sources were borrowed from; they retain all their original rights:

* The OpenLayers 2.7 WKT module (OpenLayers.Format.WKT)
* Chris Pietshmann's [article on converting Bing Maps shapes (VEShape) to WKT](http://pietschsoft.com/post/2009/04/04/Virtual-Earth-Shapes-%28VEShape%29-to-WKT-%28Well-Known-Text%29-and-Back-using-JavaScript.aspx)
* Charles R. Schmidt's and the Python Spatial Analysis Laboratory's (PySAL) WKT writer

###################
### Conventions ###
###################

The conventions I've adopted in writing this library:

* The Crockford-ian module pattern with a single global (namespace) variable
* The most un-Crockford-ian use of new to instantiate new Objects (when this is required, the Object begins with a capital letter e.g. new Wkt())
* The namespace is the only name beginning with a capital letter that doesn't need to and shouldn't be preceded by new
* The namespace is the result of a function allowing for private members
* Tricky operators (++ and --) and type-coercing operators (== and !=) are not used

The base library, wicket.js, contains the Wkt.Wkt base object. This object doesn't do anything on its own except read in WKT strings, allow the underlying geometry to be manipulated programmatically, and write WKT strings. By loading additional libraries, such as wicket-gmap3.js, users can transform between between WKT and the features of a given framework (e.g. google.maps.Polygon instances). The intent is to add support for new frameworks as additional Javascript files that alter the Wkt.Wkt prototype.

##############
## Concepts ##
##############

WKT geometries are stored internally using the following convention. The atomic unit of geometry is the coordinate pair (e.g. latitude and longitude) which is represented by an Object with x and y properties. An Array with a single coordinate pair represents a a single point (i.e. POINT feature)

    [ {x: -83.123, y: 42.123} ]

An Array of multiple points (an Array of Arrays) specifies a "collection" of points (i.e. a MULTIPOINT feature):

    [
        [ {x: -83.123, y: 42.123} ],
        [ {x: -83.234, y: 42.234} ]
    ]

An Array of multiple coordinates specifies a collection of connected points in an ordered sequence (i.e. LINESTRING feature):

    [
        {x: -83.12, y: 42.12},
        {x: -83.23, y: 42.23},
        {x: -83.34, y: 42.34}
    ]

An Array can also contain other Arrays. In these cases, the contained Array(s) can each represent one of two geometry types. The contained Array might reprsent a single polygon (i.e. POLYGON feature):

    [
        [
            {x: -83, y: 42},
            {x: -83, y: 43},
            {x: -82, y: 43},
            {x: -82, y: 42},
            {x: -83, y: 42}
        ]
    ]

It might also represent a LINESTRING feature. Both POLYGON and LINESTRING features are internally represented the same way. The difference between the two is specified elsewhere (in the Wkt instance's type) and must be retained. In this particular example (above), we can see that the first coordinate in the Array is repeated at the end, meaning that the geometry is closed. We can therefore infer it represents a POLYGON and not a LINESTRING even before we plot it. Wicket retains the *type* of the feature and will always remember which it is.

Similarly, multiple nested Arrays might reprsent a MULTIPOLYGON feature:

    [ 
        [
            [
                {x: -83, y: 42},
                {x: -83, y: 43},
                {x: -82, y: 43},
                {x: -82, y: 42},
                {x: -83, y: 42}
            ]
        ],
        [ 
            [
                {x: -70, y: 40},
                {x: -70, y: 41},
                {x: -69, y: 41},
                {x: -69, y: 40},
                {x: -70, y: 40}
            ]
        ]
    ]

Or a POLYGON with inner rings (holes) in it where the outer ring is the polygon envelope and comes first; subsequent Arrays are inner rings (holes):

    [ 
        [
            {x: 35, y: 10},
            {x: 10, y: 20},
            {x: 15, y: 40},
            {x: 45, y: 45},
            {x: 35, y: 10}
        ],
        [
            {x: 20, y: 30},
            {x: 35, y: 35},
            {x: 30, y: 20},
            {x: 20, y: 30}
        ]
    ]

Or they might represent a MULTILINESTRING where each nested Array is a different LINESTRING in the collection. Again, Wicket remembers the correct *type* of feature even though the internal representation is ambiguous.
