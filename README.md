# What's My Nearest 'L' Station?
What's My Nearest 'L' Station is an Edge Network Voronoi diagram partitioning Chicago's walkable street network into "regions" consisting of all paths with a shorter walking distance to a particular 'L' station than any other.

[**Look at the map here**](https://sabrinadchan.github.io/nearest-L-station/index.html). Be patient, as the map could take a couple of seconds to load and to zoom in and out.

Below I provide a conceptual overview of what an Edge Network Voronoi Diagram is and how to build one. Read my [blog post](https://sabrinadchan.github.io/data-blog/computing-a-network-voronoi-diagram.html) to learn the step-by-step process I used to compute the diagram in What's My Nearest 'L' Station with Python. 

## Overview
A [Voronoi diagram](https://en.wikipedia.org/wiki/Voronoi_diagram) is a partitioning of a plane with *n* points (called seeds or center nodes) into regions such that each region contains only one seed and all points closer to that seed than any other. Typically, the distance between points is measured using Euclidean distance, but other notions of distance, such as the Taxicab metric, may be used.

In the real world though, the notion of distance between two points is often restricted by a network. For example, a drive from one's home to work is confined to a city's system of highways and streets. Distance traveled is measured with respect to the path taken to get to work, not "how the crow flies." So, in order to find an accurate partitioning of Chicago into "regions" corresponding to the nearest 'L' station with respect to the street network, we need to take a different approach than a standard Voronoi diagram.

That's where Network Voronoi Diagrams (NVD) come in. A NVD is an extension of a regular Voronoi diagram from a planar region to a network of nodes and edges. The distance between two points along the network is determined by finding the shortest-path distance between the points. We can think of a Chicago's street network as a collection of edges (street segments) and nodes (intersections or dead ends) where each edge has a weight (distance in meters). 

Now, there are two types of NVDs: there are Node Network Voronoi Diagrams (NNVD) and Edge Network Voronoi Diagrams (ENVD). What's My Nearest 'L' Station is built using both an NNVD and ENVD. First, an NNVD is constructed: the shortest-path distance between each 'L' Station and each intersection (node) is calculated using [Dijkstra's algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm). The intersections are then partitioned based on which 'L' station is nearest. Next, the NNVD helps to construct an ENVD. If a street segment's pair of intersections are both associated with the same 'L' station, then that street segment is also associated with that 'L' station. If the intersections belong to different partitions, however, then the street segment is divided into two sub-segments, such that all of the points in each sub-segment are nearest to the same 'L' station.

The network used in this analysis includes not just streets, but any walkable path, including sidewalks, trails, alleyways, and parking lots, but not highways, private property, construction sites, etc. The analysis was conducted on a network including walkable path in Chicago and surrounding suburbs. The final map only includes the partitions of Chicago, Evanston, Skokie, Lincolnwood, Rosemont, Oak Park, Forest Park, and Cicero.

## Caveats and Future Work
The methods used to create the ENVD do not perfectly partition Chicago into regions of shortest walking distance to each 'L' station. A couple things to consider:
* The seeds used to partition the network do not precisely correspond to the entrance of each 'L' station, but instead correspond to the network node *nearest* to the 'L' station. It is possible in some cases that the chosen seed is actually from a street or alleyway behind the station entrance. This issue can be improved by manually selecting the nodes used as seeds, or going a step further, by adding new nodes (along existing edges) to the network that even more closely align with station entrances.
* While most 'L' stations have multiple entrances, sometimes blocks apart, the data set from the CTA includes one lat/lon per station. In reality, many people will prefer to use one station over another because an alternate entrance is closer—this is especially true downtown. We can improve this issue by adding seeds corresponding to additional station entrances. The resulting extra Voronoi cells can then be combined into a single cell for each 'L' station. I intend to address these first two issues in a future version of this map. 
* While it's true that a standard Voronoi partitioning of a planar representation of Chicago oversimplifies the analysis, a network Voronoi partitioning on Chicago's walkable paths is too restrictive. People are not always confined to walking along sidewalks, roads, alleys, etc. (that is, the shortest-path distance metric): we are free to cut through parks, parking lots, and other open spaces using the Euclidean metric! Not much can be done about this issue—I suppose one could introduce artificial edges into the network in especially egregious areas and remove them from the final product—but it is worth keeping this issue in mind and noticing where the Voronoi regions do not match up with reality.

## Acknowledgements
Street network spatial geometries were downloaded from OpenStreetMap's API using [OSMnx](https://github.com/gboeing/osmnx). Both OSMnx and [NetworkX](https://github.com/networkx/networkx) were instrumental in analyzing the street network data. 'L' Station Locations and 'L' Line geometries were accessed from the [Chicago Open Data Portal](https://data.cityofchicago.org/Transportation/CTA-List-of-CTA-Datasets/pnau-cf66). The final map was produced using [Leaflet](https://leafletjs.com) with map tiles by [Stamen](http://stamen.com/) and underlying map data from [OpenStreetMap](http://openstreetmap.org/).
