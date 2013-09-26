
===So what is Carto ?=== 

Mapnik is a popular tool to render, that is display geographic information into images. 

Mapnik's code instructs what map features (roads, rivers, buildings)) to display and what these features should look like (this is known as rendering), is in XML. It's very difficult to read. In 2009(ish?), cascadenik was made to help simplify this. 

Then, carto (also known as CartoCSS, but for the rest of this manual, it will be referred to as Carto) was set out to provide an easier way to write these instructions while taking full advantage of mapnik's rendering capabilities and is superceding cascadenik. 

It's like the CSS language (and less), it's used to style maps in the application, tilemill. 

There's a few concepts to keep in mind: 

== Organizing your data == 

= Format = 

Before you even start, organize your data however you'd like. tilemill supports postgis, SHP, geojson, CSV (must be formatted correctly - see http://www.mapbox.com/tilemill/docs/guides/google-docs/) and maybe something else. 
Make sure that its in the same projection used in tilemill, EPSG:4326. 

However, if you're really particular on performance or want to render large amount of data (like entire countries), go with postgis and shp instead of geojson or spreadsheets. 

SQL Queries: 

Additionally: if your use postgres data, you want your layers to be as specific as possible. Just as using select * FROM yourtablename is inefficient in your postgresql queries in general, because you're selecting everything, doing this in tilemill will also increase the time that . Simply put, you only want to queries so that load the data that you will want to display on your map. 

Go see your postgres query selector for more information. 

Here's a very sample SQL query for a layer: 

 (select shop from planet_osm_point where shop is not null) as points

 (if you're importing data from osm2pgsql see http://wiki.openstreetmap.org/wiki/Osm2pgsql/schema for the names of tables)

Shop is your column name, planet_osm_point is your table name and points is an arbitrary name. You can name it whatever you want. 

 What data should go in each layer ? 
 Should I make one for all roads ? one for railways ? Labels ? 
 - This question really confused me as a newbie tilemill user. 
 This is a question is a bit difficult to answer because it depends on the map that you're making. Generally, The more features that 
your map will include, the more layers that you'll want. 


If you're making a map that will feature distinct styling for bridges and tunnels and multiple types of highways, 
you'll likely have multiple layers there (one for bridges, one for highways, etc)


One applicable rule that is applicable for nearly every tilemill map is that you should have a layer solely dedicated to labels :)  
==== Styling === 


ohh, ahh, so how do I Make stuff look purdy ? 


Check out the comp-op page.  http://www.mapbox.com/tilemill/docs/guides/comp-op/


to select a type='specifictypeofstreetforexample'  [also double quotes works as well but the tilemill designers have 
use single quotes on a de facto basis. 

==Inheritance==
you'll want to be the field's name in single quotes. (if it's a text)

CSS saves you time by inheritance, so very basic: 
```
#place::small[type='neighbourhood'][zoom>=13][zoom<=20] {
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
```
 for zoom 14, it will have ALL of those characteristics that are included in the except, except for the text-size is now 11.  


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

Returning to our previous example, declaring the second rule will add a blue glow on top of the red line instead of replacing it:
```
#layer {
   line-color: red;
   line-width: 1;
}

#layer::glow {
   line-color: blue;
   line-opacity: 0.5;
   line-width: 4;
}  
```
This glow is only possible because of the line-opacity rule. Try removing that and then click save. You'll now only see a blue line. 


Attachment style: 

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
(And has been adopted by openstreetmap-carto led by andy allen, and ybon, math1985 agreed)

AJ Ashton - 
I don't really have a preference and will do either or both depending on the project and the data. But for OSM-Carto I think the no-repeat approach is a good fit, and it's good to have that kind of consistency on a collaborative project.


```
(What's the diff between: 
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




draw this stuff first, then draw this stuff first, then this stuff last. 

but this destroys a lot of layering :/  ? 

- (15mins of andy allen's talk)

- can also be intensive i/o for rendering, better to filter your queries at the source. 





====================
Selections/selectors within the Layer:

bFlood> how do you change the drawing order in Tilemill?
<ybon> bFlood: you can change the order of the layers
<bFlood> ok, how is that done from the UI? in the carto css or via the little layers popup window?
<ybon> the layers window
<ybon> drag the icon on the left



As stated earlier, you want your layers to only include the geo_data that you want, BUT you organize your 
by uses. 

In the following example, here's a postgresql query of a railway layer. 
 like the following with way, railway as your column names. 

 SELECT way, railway, CASE WHEN railway in ('spur','siding') THEN 'yard' WHEN railway='disused' THEN 'disused' WHEN railway='rail' THEN 'main' ELSE 'other' END AS type FROM planet_osm_line WHERE railway IS NOT NULL ORDER BY z_order) as rail",

 'Main' and 'yard' now become selectors that you can reference in your mss files. 


so you can reference yard, and as a selector and it will represent that selection you made in the SQL layer. (when railway row has values of spur or siding)


```
#railway[type='yard']
{
  your rules for yard
}
``` 

```
#railway[type='main']
{
  your rules for main
}
```

see: https://github.com/hotosm/HDM-CartoCSS/blob/master/roads.mss

a more advanced example:
```
 SELECT way, tunnel, bridge, railway, service, CASE WHEN railway in ('spur','siding') or (railway='rail' and service in ('spur','siding','yard')) THEN 'yard' WHEN railway='disused' THEN 'disused' WHEN railway='rail' THEN 'main' ELSE 'other' END AS type FROM planet_osm_line WHERE railway IS NOT NULL AND railway!='abandoned' ORDER BY z_order) as rail",
```

or: 
```
SELECT way, waterway AS type, CASE WHEN tags->'seasonal'='yes' OR tags->'intermittent'='yes' THEN 'yes' ELSE 'no' END AS seasonal FROM planet_osm_line WHERE waterway IN ('river', 'canal', 'stream', 'ditch', 'drain')) AS data",
```


AJ - 
    Adding a comma starts a whole new selector - meaning aspects of your previous selector (like the layer and zoom level) are not considered. So you need to repeat yourself a bit and include #parkpoint[zoom=10] after your comma. It's easier to think about the separation commas cause by always adding a new line after a comma, like this:

    #parkpoint[zoom=10][display_designation = 'National Park'],
    #parkpoint[zoom=10][display_designation = 'National Park & Preserve'] {


<bFlood> ajashton: is there a way to perform boolean ops between filters? so instead of #layer [field=foo][field=bar]  we could do [field1=foo] AND [field2=bar]?
<ajashton> #layer[field1=foo][field2=bar] is an AND operation
<ajashton> #layer[field1=foo], #layer[field2=bar] is an OR operation
<bFlood> sorry, meant OR
<bFlood> got it, thx
<ajashton> you can also use nesting, like #layer { [field1=foo],[field2=bar] { /* style */ } }

<bFlood> one more, for complex expressions can you use parens to separate the logic: #layer {[field3=baa],( [field1=foo][field2=bar])
<ajashton> bFlood, no parens. Everything between commas is totally separate
<ajashton> down side is that `x AND (y OR z)` has to instead be `(x AND Y) OR (x AND z)`
<ajashton> (not in that syntax, tho)

You can also use negatives, like: 
[display_designation != 'National Park'][display_designation != 'National Park & Preserve']
(display_designation is the name of a row in your data)

==Classes==: 


Opacity (and related comp-op) operates on a layer as a whole. However, it’s specified inline with the rest of the rules, which is confusing. This sort of thing shouldn’t work, or should perhaps throw a warning:

#foo[bar=1] { opacity: 0.2 }
#foo[bar=2] { opacity: 0.8 }

My experience has been that opacity gets set just once, for the whole layer, and I’m not sure how to predict which one would win in this case.



https://github.com/mapbox/carto/issues/249



=========

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

<style>
.map-tooltip {
 max-width:400px !important;
}
</style>

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
Extra Resources, Great tools/Examples:

Where are some great examples of carto in action ?! 

(this listed ended up getting posted at: )
see: http://gis.stackexchange.com/questions/62348/is-there-a-carto-css-gallery-which-also-contains-code/62824#62824

Cartocc: 

Cartocc is a npm (module?) that is designed for people to collaborate together on tilemill projects. In most cases, your database names, 
the path to your JSON files will be different than your collaborators. Using this, you can customize your project settings. 

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

