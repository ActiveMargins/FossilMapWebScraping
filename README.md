## FossilMapWebScraping

Don Kelly (http://donaldkenney.x10.mx/FOSSINDX.HTM) has created a well curated website that contains, among other things, a list of 16000 fossil sites in the United States and Canada. This list is nicely formatted into a series of html tables on webpages for each state/province. The table includes various information (e.g., lat-long, formation, fossils) for each locality.

The structure and content of the website make it an interesting dataset to practice web scraping as well as producing geospatial visualizations. The goal of this project is to scrape the data from the multiple websites and html tables, format data into a single table representing all 16000 fossil localities, then produce some visualizations from the dataset.

The algroithm here scrapes the website, first by creating a list of URL's (one for each state/province), then loop through this list and appending them together. Once assembled the html is read into a table. The final code chunk turns this table into Leaflet map that displace the fossil localities. When clicked, each location displays the locality name, fossil at the locality, and formation name. 

I use this when I go out into the mountains to see if there are any localities nearby or on route.  
#### ALWAYS follow local laws and regulations for collecting fossils.

Below are examples of maps generated from this website for British Columbia, Montana, and Colorado.
