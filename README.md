# Optimal Scale for Decentralized Water Reuse

This repository consists of the code base for the web tool developed under the publication:

[Kavvada, Olga, Kara L. Nelson, and Arpad Horvath. "Spatial optimization for decentralized non-potable water reuse." Environmental Research Letters (2018).](http://iopscience.iop.org/article/10.1088/1748-9326/aabef0/meta)

The webtool can be found free of charge at: [https://water-reuse-map.herokuapp.com/](https://water-reuse-map.herokuapp.com/)

If you wish to use the tool for your own analysis please provide proper attribution by citing the work:
[Kavvada et al](http://iopscience.iop.org/article/10.1088/1748-9326/aabef0/meta)

## Background
The core and integral criterion for sustainable non-potable reuse (NPR) implantation is the issue of system scale. Larger facilities can benefit from the increased "economies of scale" in treatment but in the context of water reuse a tradeoff occurs as in a centralized system the water would be further away from the point of demand which exacerbates the upgradient conveyance and distribution impacts. This project defines a modeling method for decentralized water reuse systems to estimate their economic and environmental impacts under real topographical and demographic conditions. The developed tool estimates both the financial cost and environmental impacts of NPR as a function of treatment scale and optimized conveyance networks. It is based on a heuristic modeling approach using geospatial algorithms to determine the optimal degree of decentralization. The webtool is developed to assess and visualize alternative NPR system designs considering topography, economies of scale and building size.

The developed tool is currently based on the data of the city of San Francisco. However, it is a generalizable tool as long as the required input data is given.


## Input Data
The main input data required for the tool to run is a csv with building data of the area of interest. 
The required attributes in the csv file are:
- the lat, lon coordinates of each buildings (x_lon, y_lat), 
- a combined coordinates field in the form of `(lat, lon)`, 
- the building footprint area (Area_m2), 
- the number of floors (num_floor), 
- the building base elevation (ELEV_treat),
- the residential population of the building (SUM_pop_residential) and
- the commercial population of the building (SUM_pop_commercial)


## Algorithmic Process
To account for the spatially sensitive conditions and for the model to be able to assess multiple levels of decentralization, the developed algorithmic model expands its size through an iterative process with an input of the existing building location and characteristics. The model is capable of assessing different metrics of water reuse, namely economic cost, energy intensity and GHG emissions as defined by the user. As the model is capable of assessing different metrics each of the metric in question will be called "impact" from now on.

The algorithm involves the following steps:
- All the input buildings are loaded and a KD tree is constructed from their coordinate locations. The KD tree is used for optimal searching of the 500 closest buildings to the user query point and listed in ascending order based on the euclidean distance from the user query point. The 500 is an arbitrarely chosen value to provide a cutoff point for the analysis. It could be altered as see fit.
- The building located the closest to the point of interest is considered as the first building to enter the algorithm. 
- Impact calculation: Given this starting point the impact of water reuse is calculated with only this building in mind. This involves estimating the impact of treating the required volume of water (given the population of the building), the embodied impact for treatment (includes construction and transportation), the in-building impact for delivering the recycled water to all the building floors and finally the in-between building impact for sharing recycled water in the case of multiple buildings (in this first case this is zero). These operations are defined under the `metric == impact` operation in the module.
- After this first assessment, the second in-line building is considered. This building is added to a Graph along with the previous building. This system is now considered a cluster. Given this cluster the distance between the two buildings is calculated. 
- The process for impact calculation is repeated similarly as before but now considering the new cluster of buildings. Given this new cluster a new impact is defined.
- This new impact is compared to the previous impact of the cluster. If the new impact is smaller than the previous then the new building is kept in the cluster as it is beneficial for the overall systems performance. If not, then the building is discarded and the previous system's performance is kept.
- The process continues until all close (k=500) buildings are assessed. 


## Outputs
The output of the algorithmic process as described previously is a csv file located at the specified path when calling the model. The csv includes the locations of all the buildings in order of assessment along with information of the population served, the total impact of interest, and the breakdown of the total impact of interest to the pumping, treatment operational, treatment embodied and piping. 
It also outputs a json file with all the building locations and a tag of whether they are selected or not (accept = 'yes' or 'no') along with the total population served and the number of buildings for immediate visualization in the webtool. 


## Webtool
The webtool is connected to the python algorithm through Flask. The frontend is a javascript based webtool. The user can interact with the webpage and create certain events that are logged and passed to the python module. Specifically, the user can:
- Click on the map, to given the signal of the query point location, from where the algorithmic process is going to initiate.
- Click on a metric, "cost", "energy", "GHG" to define the metric of interest.
- Set the treatment equation parameters as floats in the corresponding boxes.
- Set the direct GHG emissions for treatment as a float in the corresponding box.
- Select the appropriate electricity mix from a dropdown menu or set a custom emission factor.
- Change the visualization by making the background map satellite, road or greyscale.

The outputs of the algorithmic process are going to appear on the screen after the computation is done. The outputs consist of the number of the total number of houses served and the served population and a visualization on the map of the selected and discarded buildings assessed.