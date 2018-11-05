---
layout: post
title: "On calculating a centroid of a polygon"
description: "Finding the best way to compute a centroid of a closed 2D surface, given its vertices"
category: Algorithms
tags: [GIS]
---
{% include JB/setup %}

The problem I am facing today is *"how to find the center of a polygon?"*. The first idea that came to my mind
was based
on the elementary plain geometry: calculation of the center of a regular polygon. But the problem is certainly more
complex. One could imagine to have a set of points given as latitude/longitude of a particular plain representation
of a portion of the Earth (a map with a specific projection - usually based on the common WGS84 ellipsoid), but how to
tell if this set of points form or not a closed surface; and if this set arranges one or more closed irregular polygons?
<!--more-->

![Finding the centroid of a closed poly-line](http://i.stack.imgur.com/sGcEX.png)

One problem at a time, and use the Cartesian analytical approach. Therefore, we assume an individual closed shape, modeled by a set of vertices that engage a closed [poly-line](http://www.webopedia.com/TERM/P/polyline.html). My GIS experiences
suggest me that I had to find the centroid of this "feature", so the first hint is search for the source code of a
[QGIS/Sextante plugin](http://code.google.com/p/sextante/source/browse/trunk/soft/bindings/qgis-plugin/src/sextante/ftools/Centroids.py) (written in Python) in order to find a pattern to infer an algorithm, and here start my real research.

The centroids algorithm for QGIS is too much bound to the implementation of the main software, so it had to be discarded.
The right formula behind this computation could be found [here](http://math.stackexchange.com/q/3177), where it is clearer the sense of the term *centroid*:

>The centroid of a polygon is indeed its center of mass -- but the mass of a polygon is uniformly distributed over its surface, not only at the vertices. You're right that **if the mass were split evenly among the vertices only, the centroid would be the arithmetic mean of the coordinates of the vertices**.

>It just so happens that both definitions are equivalent (mass evenly distributed over the surface vs mass at the vertices only) for simple shapes like triangles and rectangles.

In our case this assumption is not true, so the correct answer resides on the third response which drags in the [Green's
theorem](http://en.wikipedia.org/wiki/Green%27s_theorem). To me this solution is praticable but even too complex. The
KISS principle is my guiding light. And then I realized that the best solution is in the approximation of the problem:

1. find an enclosing polygon/circle
2. calculate the centroid in the arithmetic way

The best solution I found? It's [this one](http://gis.stackexchange.com/q/2128) (in JavaScript) based on the
[epoly.js](http://www.geocodezip.com/scripts/v3_epoly.js) library which extends the Google Maps API using [prototypal
inheritance](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain).

The resulting code is here:

```javascript
google.maps.Polygon.prototype.Centroid = function() {
var p = this;
var b = this.Bounds();
var c = new google.maps.LatLng((b.getSouthWest().lat()+b.getNorthEast().lat())/2,
            (b.getSouthWest().lng()+b.getNorthEast().lng())/2);
if (!p.Contains(c)){
    var fc = c; //False Centroid
    var percentages = [0.1,0.2,0.3,0.4,0.5,0.6,0.7,0.8,0.9]; //We'll check every 10% down each ray and see if we're inside our polygon
    var rays = [
        new google.maps.Polyline({path:[fc,new google.maps.LatLng(b.getNorthEast().lat(),fc.lng())]}),
        new google.maps.Polyline({path:[fc,new google.maps.LatLng(fc.lat(),b.getNorthEast().lng())]}),
        new google.maps.Polyline({path:[fc,new google.maps.LatLng(b.getSouthWest().lat(),fc.lng())]}),
        new google.maps.Polyline({path:[fc,new google.maps.LatLng(fc.lat(),b.getSouthWest().lng())]}),
        new google.maps.Polyline({path:[fc,b.getNorthEast()]}),
        new google.maps.Polyline({path:[fc,new google.maps.LatLng(b.getSouthWest().lat(),b.getNorthEast().lng())]}),
        new google.maps.Polyline({path:[fc,b.getSouthWest()]}),
        new google.maps.Polyline({path:[fc,new google.maps.LatLng(b.getNorthEast().lat(),b.getSouthWest().lng())]})
    ];
    var lp;
    for (var i=0;i<percentages.length;i++){
        var percent = percentages[i];
        for (var j=0;j<rays.length;j++){
            var ray = rays[j];
            var tp = ray.GetPointAtDistance(percent*ray.Distance()); //Test Point i% down the ray
            if (p.Contains(tp)){
                lp = tp; //It worked, store it
                break;
            }
        }
        if (lp){
            c = lp;
            break;
        }
    }
}
return c;}
```
Also consider the possibility of combining the previos code with
[this Google Maps API v3 extension](https://github.com/tparkin/Google-Maps-Point-in-Polygon).

[Another possible solution](http://www.cs.mcgill.ca/~cs507/projects/1998/jacob/solutions.html) was also pointed out to me by [Astrac](https://github.com/Astrac). I hope this post could possibly help someone for managing maps with vector layers polygons.
