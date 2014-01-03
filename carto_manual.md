===So what is Carto ?=== 

Mapnik is a popular tool to render, that is display, geographic information into images. 

Mapnik's code instructs what features (roads, rivers, buildings) to display and what these features should look like (this is known as rendering), is in XML. It's very difficult to read. In 2009(ish?), cascadenik was made to help simplify this. 

Then, carto (also known as CartoCSS, but for the rest of this manual, it will be referred to as Carto) was set out to provide an easier way to write these instructions while taking full advantage of mapnik's rendering capabilities and is superceding cascadenik. 

It's like the CSS language (and less.js), it's used to style maps in the application, Tilemill. 


== Organizing your data == 

Before you even start making your map, organize your geospatial data however you'd like. Tilemill supports postgis, SHP, geojson, CSV (must be formatted correctly - see http://www.mapbox.com/tilemill/docs/guides/google-docs/) and maybe something else. 
Make sure that data is in the same projection used in Tilemill, EPSG:4326. 

If you're really particular on performance or want to render large amount of data (like entire countries), go with postgis and shp instead of geojson or spreadsheets. 

SQL Queries: 

Additionally: If you're using a postgis DB, you want your layers to be as specific as possible. Just as using ```select * FROM yourtablename``` is inefficient in your postgresql queries because you're selecting everything, using select * queries in Tilemill will increase the time needed to load your map. Simply put, your layer's query should only select the data that you will want to display on your map. 

Here's a very simple SQL query for a layer: 

```
 (select shop from planet_osm_point where shop is not null) as points
``` 

(if you're importing data from osm2pgsql see http://wiki.openstreetmap.org/wiki/Osm2pgsql/schema for the names of tables)

Let's show what this means: 

Shop is your column name, planet_osm_point is your table name and points is an arbitrary name. You can name it whatever you want. 


----
First things first (aka: things that in tilemill and that I really wish I knew when I started using tilemill:

 *What data should go in each layer?* 
 *Should I make one for all roads? railways? Labels?* 
 - This question really confused me as a newbie tilemill user. 
 This is a question is a bit difficult to answer because it depends on the map that you're making. 
Generally, The more features that your map will include, the more layers that you'll want. 
Do not try to put everything in one layer, even if all of your data is in one file. 

If you're making a map that will feature distinct styling for bridges and tunnels and multiple types of highways, 
you'll likely have multiple layers there (one for bridges, one for highways, one for tunnels, etc). 
Many popular stylesheets have 3 separate road stylesheets, one for lower zoom levels (like 10-13ish), medium (14-17ish)
and more detailed (18+), plus separate ones for all tunnels and bridges. 

One applicable rule that is applicable for nearly every tilemill map is that you should have a layer 
solely dedicated to labels :)  

*what programs do I use to edit my stylesheets (.mss) and my project file (.mml)?* 
- You can edit them within tilemill itself. You can also use your own preferred text editor (sublime, notepad, vim, emacs). Editing them within your own preferred text editor is is especially useful when you have multiple MSS files in your browser. If you edit and save your file in your text editor, tilemill will automatically detect the changes, refresh, and display the changes. 

* For writing selectors like ``` highway='specifictypeofstreetforexample' ``` use single quotes. Double quotes also work in Tilemill, it's just a standard practice by other tilemill users to use single quotes. 

* The order of the MSS files (from left to right) within tilemill does not matter. 
Neither do the names of the MSS files - they do not have to match the name of your layers. 
Use whatever names that are easy for you to remember. 

* Also, if you do not know by now, you can launch multiple instances of tilemill. This is useful if you're working on multiple projects at once or if you want to test out a piece of code real quick in a new project or context, and don't want to make a branch in git for whatever reason. 

* You can access Tilemill through your web browser. In the terminal, go to the directory where tilemill install is located (ADD this). In this directory, type in: ```./index.js --server=true```.  
Now, you can go to your web browser and type in localhost:20009 and your tilemill will be there. 

---

**Inheritance**

you'll want to be the field's name in single quotes. (if it's a text)

CSS saves you time by inheritance, so very basic: 

```
#place[type='neighbourhood'][zoom>=13][zoom<=20] {
  text-name:'[name]';
  text-face-name:@sans;
  text-placement:point;
  text-fill:@other_text;
  text-size:10;
  text-halo-fill:@other_halo;
  text-halo-radius:1;
  text-wrap-width: 30;
  text-min-distance: 100;
  text-avoid-edges: true;
  text-label-position-tolerance: 10;
  [zoom>=13] {
    text-min-distance: 50;
  }
  [zoom>=14] {
    text-size:11;
    }
}    
```
 Here, for zoom 13 and higher, the text-min-distance is now 50 and the rest of the characteristics that are included in first selection,
 for zoom 14 and higher, it will have ALL of those characteristics that are included in first selection, except for the text-size is now 11.  
 Also note that it's standard practice to use >=  when you write out your zoom queries and keep it consistent. Wish I knew why, but I haven't had any problems result from adapting this. 
 
---

==== Styling === 

ohh, ahh, so how do I Make stuff look purdy ? 


- Check out the comp-op page.  http://www.mapbox.com/tilemill/docs/guides/comp-op/

---

===Reading the API=== 

So, the api can be difficult to read if you're not familiar with reading APIs. 

For example, see [The polygon-pattern-file,](https://www.mapbox.com/carto/api/2.1.0/#polygon-pattern) 

``` #processed_p[zoom>=10] {
polygon-pattern-file:url('img/imagename.png'); 
} ```

This url path assumes that imagename.png is located in a subfolder of your project, named img. 

Tip: For images, images that you use for polygon-pattern-file or map-background should be "seamless." They should be  256x256 or 512x512 in size. If they aren't seemless, you will notice where one tile ends and the other begins (add ugly example)

--- 
===Attachments====

When you see the :: being used, they're called attachments. 

These are pretty damn dope, extend the capabilities of Tilemill, extend your mapmaking capabilities and wish I learned these earlier.

```
#layername[yourfilter-isthatwhatthisiscalled]::selector { 

}
```
you can name your selector whatever you want, it's arbitrary. 

 Also repeated in http://www.mapbox.com/tilemill/docs/manual/carto/ (include?)

 Note that newsymbol is not a special keyword but an arbitrary name chosen by the user. To help keep track of different symbolizers you can name additional symbolizers whatever makes sense for the situation. Some examples: #road::casing, #coastline::glow_inner, #building::shadow.

To recap what was written there, the second rule will add a blue glow on top of the red line instead of replacing it:
```
#layer {
   line-color: red;
   line-width: 4;
}

#layer::glow {
   line-color: blue;
   line-opacity: 0.5;
   line-width: 8;
}  
```
![Example 001](https://github.com/skorasaurus/carto-manual/raw/master/img/example_001.png)

This glow is only possible because of the line-opacity rule. Try removing that and then click save. 
You'll now only see a blue line, as shown in the image below: 

![Example 002](https://github.com/skorasaurus/carto-manual/raw/master/img/example_002.png)

Important! 
Remember the painter's algorithm ! One of the most important concepts that helped me understand Tilemill. 

Let's explain how this works with attachments and the previous example. As you saw in the most recent example, you only saw a blue line. 

but if we reverse the two colors, like the following : 

```
#layer {
   line-color: red;
   line-width: 8;
}

#layer::glow {
   line-color: blue;
   line-width: 4;
}
```

![Example 003](https://github.com/skorasaurus/carto-manual/raw/master/img/example_003.png)


In this case, you will see a blue line with a red outline or a red glow! Why? the red line with a width of 8, was drawn first, and then the blue line, with a width of 4, was drawn above the red line. 

With attachments, the line is drawn over the previous object! This is the painter's algorithm! 


However, if you do not use attachments, 
```
#countries {
   line-color: red;
   line-width: 8;
}

#countries {
   line-color: blue;
   line-width: 4;
}
```
you'll only end up with a blue line that is 4 pixels wide. Without attachments, the code in the second selector only
will be used. 

If you use attachments consecutively, like: 
```
#layer::first {
   line-color: red;
   line-width: 4;
}
#layer::glow {
   line-color: blue;
   line-width: 2;
}
``` 
They will have the same action as the early image (not immediately before, but the 2nd one before this example)... 
the blue glow will be atop the red line. For simplicity's sake, you shouldn't need to do this, just remove the attachment from the first layer/ 


(http://www.mapbox.com/tilemill/docs/guides/symbol-drawing-order/ explains pretty well, as well, wish this was there when I started)

And you already know, 
In addition to attachments, the painter's algorithm also applies to the entire mss file. Lines, points, and polygons that are written first in the MSS file will be drawn first. Any lines, polygons, and points that are written later in the file will be drawn over the earlier ones. 

There's also on last affect on the drawing order of features in Tilemill which is done by the layers (FIXME)

<ybon> bFlood: you can change the order of the layers
<bFlood> ok, how is that done from the UI? in the carto css or via the little layers popup window?
<ybon> the layers window
<ybon> drag the icon on the left


Now, Attachment style: 

there's no difference  between: 
```
#layer {
  ::outline {
    line-width: 6;
    line-color: black;
  }
  ::inline {
    line-width: 2;
    line-color: white;
  }
}
```
and 
```
#layer::outline {
    line-width: 6;
    line-color: black;
}
#layer::inline {
    line-width: 2;
    line-color: white;
}
```
first version is a little easier to read :) 
(And has been adopted by openstreetmap-carto, osm's standard stylesheet) 

AJ Ashton - 
I don't really have a preference and will do either or both depending on the project and the data. But for OSM-Carto I think the no-repeat approach is a good fit, and it's good to have that kind of consistency on a collaborative project.

D
(What's the diff between:
``` 
#place::small[type='neighbourhood'][zoom>=13][zoom<=20] 
and 
#place [type='neighbourhood'][zoom>=13][zoom<=20] ::small ? 
)
```

multiple textures, casings for lines (if you want a road to be outside with a different color than what the inside of it is...), and so much more can be done with attachments..

Like this example for a railroad. 

```
#rail [zoom>=12][zoom<=21]::perpendicular-dashes  {

line-cap: butt;
line-color: black;
[zoom>=12] { line-width: 5.25;
    line-dasharray: 0.45,10; }
[zoom>=14] { line-width: 5.75; 
    line-dasharray: 0.75,15;}
[zoom>=16] { line-width: 6.75; 
    line-dasharray: 1.25,20; }
[zoom>=18] { line-width: 7;
     line-dasharray: 1.5,25; }
[zoom>=20] { line-width: 7;
     line-dasharray: 1.75,35; }
}
#rail [zoom>=12][zoom<=21]::base-line {
 line-cap: butt;
line-color: black;
[zoom>=12] { line-width: 1; }
[zoom>=14] { line-width: 1.25; }
[zoom>=16] { line-width: 1.75; }
[zoom>=18] { line-width: 2; }
[zoom>=20] { line-width: 2.25; }
}
```

In the above rail instance, the perpendicular-dashes are drawn first, then the base-line are drawn. 


====================
Selections/selectors within the Layer:

There are times when even if you filter the objects that you want in your layer, 
you want to customize styling based on certain properties within your layer. 

Tips: 
- item

-  Adding a comma starts a whole new selector - meaning aspects of your previous selector 
-  like the layer and zoom level) are not considered. So you need to repeat yourself a bit 
-  and include #parkpoint[zoom=10] after your comma. It's easier to think about the separation commas cause by 
-  always adding a new line after a comma, like this:

```#parkpoint[zoom=10][display_designation = 'National Park'],
    #parkpoint[zoom=10][display_designation = 'National Park & Preserve'] {
    } ```
```
<bFlood> ajashton: is there a way to perform boolean ops between filters? so instead of #layer [field=foo][field=bar]  we could do [field1=foo] AND [field2=bar]?
<ajashton> #layer[field1=foo][field2=bar] is an AND operation
<ajashton> #layer[field1=foo], #layer[field2=bar] is an OR operation
<bFlood> sorry, meant OR
<bFlood> got it, thx
<ajashton> you can also use nesting, like ``` #layer { [field1=foo],[field2=bar] { /* style */ } } ``` 

<bFlood> one more, for complex expressions can you use parens to separate the logic: #layer {[field3=baa],( [field1=foo][field2=bar])
<ajashton> bFlood, no parens. Everything between commas is totally separate
<ajashton> down side is that `x AND (y OR z)` has to instead be `(x AND Y) OR (x AND z)`
<ajashton> (not in that syntax, tho)

You can also use negatives as a selector in carto, by using: !=

``` [display_designation != 'National Park'][display_designation != 'National Park & Preserve'] ``` 

(display_designation is the name of a row in your data )



As stated earlier, you want your layers to only include the data that you want to select. 

In the following example, here's a postgresql query of a railway layer. 
 like the following with way, railway as your column names. 

 ```SELECT way, railway, CASE WHEN railway in ('spur','siding') THEN 'yard' WHEN railway='disused' THEN 'disused' WHEN railway='rail' THEN 'main' ELSE 'other' END AS type FROM planet_osm_line WHERE railway IS NOT NULL ORDER BY z_order) as rail", ```

 'Main' and 'yard' now become selectors that you can reference in your layers!


so you can use 'yard' as a selector and it will represent that selection you made in the SQL layer. 
In the above (when railway row has values of spur or siding)


``` #railway[type='yard']
{
  your rules for yard
} ``` 

``` #railway[type='main']
{
  your rules for main
} ```

see: https://github.com/hotosm/HDM-CartoCSS/blob/master/roads.mss

a more advanced example:
```
 SELECT way, tunnel, bridge, railway, service, CASE WHEN railway in ('spur','siding') or (railway='rail' and service in ('spur','siding','yard')) THEN 'yard' WHEN railway='disused' THEN 'disused' WHEN railway='rail' THEN 'main' ELSE 'other' END AS type FROM planet_osm_line WHERE railway IS NOT NULL AND railway!='abandoned' ORDER BY z_order) as rail",
```


==== 
==Classes==: 


Opacity (and related comp-op) operates on a layer as a whole. However, it’s specified inline with the rest of the rules, which is confusing. 
This sort of thing shouldn’t work, or should perhaps throw a warning:
```
#foo[bar=1] { opacity: 0.2 }
#foo[bar=2] { opacity: 0.8 }
``` 
My experience has been that opacity gets set just once, for the whole layer, and I’m not sure how to predict 
which one would win in this case.



https://github.com/mapbox/carto/issues/249



====
Tilemill tooltips: 
(assume you already know what tooltips are...)

Yeah, you want awesome tooltips - (the content that is displayed when a user hover a marker with their cursor or when they click on a marker in your map) -

However, if you want something that is a little complicated, like custom images for each of your points, it can stil be 
done although I would do this through leaflet instead of plotting them on a map in tilemill. 

For example, I wanted to change the width of the tooltip so I could have a larger image display without 
the user having to scroll, as shown in the picture: 


What you'll need to do: 

So, the default styling for the Tooltips is located at: https://github.com/mapbox/wax/blob/master/theme/controls.css

Look for the relevant class and property within the CSS file, and place your edited code within the 
in this case, it's the class map-tooltip (shown below)

```<style>
.map-tooltip {
 max-width:400px !important;
}
</style>```

And insert the <style> and </style> surrounding the css code

As mentioned here- http://support.mapbox.com/discussions/tilemill/1460-customizing-the-tool-tips
The changed width of the tooltip will not appear in Tilemill after. once you upload them to mapbox's server and view the your map online, 
you will see your changes. 
Some changes, like the opacity of the tooltip box, will display within Tilemill. 

Tooltips: 
https://github.com/mapbox/wax/blob/master/theme/controls.css
- how to change the font ? font is not in there ?! 
- you cannot unless you have wax on your server, because of security concerns, see:
- 

One drawback that i'm immediately noticing too, to styling your markers in tilemill, is that if you're going to display them on a webpage (using mapbox.js), each time that you want to tweak them or debug it, you'll have to upload the entire project up to the web again, 

====

*My styling isn't displaying as I intended? Help?!* 

Most common culprit of this is that your data isn't included in the layer you're styling! 
Make sure your data is being included by checking the layer features (the little magnifying glass by the circle...)(NOTE: There may be some cases where this isn't correct - especially if you have many features in your layer) so follow the next step:  

you're using SQL, put your query into an SQL editor like pgadmin. 


==== 
**Extra Resources, Great tools/Examples**:

*Where are some great examples of carto in action?!* 

(this listed ended up getting posted at: )
see: http://gis.stackexchange.com/questions/62348/is-there-a-carto-css-gallery-which-also-contains-code/62824#62824

*Cartocc*: 

Cartocc is a npm (module?) that is designed for people to collaborate together on tilemill projects. In most cases, your database names, 
the path to your JSON files will be different than your collaborators. Using this, you can customize your project settings 

See for more details: https://github.com/yohanboniface/CartoCC

Also written by the author of cartocc, is the carto-sublime package in early development:
https://github.com/yohanboniface/Carto-sublime


=====
i need help?! checkout:

#mapbox on freenode (irc)
support.mapbox.com 
and the gis.stackexchange

for crashing: 

if you're using ubuntu, go to the directory where tilemill is installed in your terminal, , and then type in ./index.js 
- tilemill will startup. /usr/share/mapbox/

mac - i think yeah look for ~/Library/Logs/DiagnosticReports/node
(if its a node error)


ENHANCEMENT TICKET TO Make: 
I've made a change, for example, the color of a landuse in the stylesheet, and it'll zoom into where I last saved, great. So, I see how the change affected in my immediate area, but if I want to see quickly, this other location ((within my area)) that also has this landuse, I have to zoom out, and then zoom back into the other area... 
I've had this come up a couple time in my workflows, it'd be a nice enhancement although I have not idea if this is feasible in any way, I'm guessing not. 

OR a workaround could be, have your project open in server mode, two tabs on your internet browser, and have it open to the two affected areas.

Once you click on save, would the other one be refreshed as well ? verify. 

