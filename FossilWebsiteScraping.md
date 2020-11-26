---
title: "Fossil Location HTML Scraping - processing tables into maps"
output:
  html_document:
    keep_md: true
---

Don Kelly (http://donaldkenney.x10.mx/FOSSINDX.HTM) has created a well curated website that contains, among other things, a list of 16000 fossil sites in the United States and Canada. This list is nicely formatted into a series of html tables on webpages for each state/province. The table includes various information (e.g., lat-long, formation, fossils) for each locality.

The structure and content of the website make it an interesting dataset to practice web scraping as well as producing geospatial visualizations. The goal of this project is to scrape the data from the multiple websites and html tables, format data into a single table representing all 16000 fossil localities, then produce some visualizations from the dataset.

Required libraries

```r
library(dplyr)
library(rvest)
library(leaflet)
library(qdapRegex)
```

### 1. Assemble data from the web
This is going to be done in by gathering a list of the URL's to the indivdual tables (listed on: http://donaldkenney.x10.mx/FOSSINDX.HTM). Then we will loop through the list of URL's scraping the html tables and appending them to a master table. Once we have a master table we will reformat and tidy the data. This first section will coerce many variables due to the structure and completeness of the html tables.


```r
#Get structure of main website and create list of links to individual tables
website <- "http://donaldkenney.x10.mx/FOSSINDX.HTM"
website_structure <- read_html(website)
website_structure <- website_structure %>% html_nodes("li")

#Set up some constants prior to entering the for loop
url_prefix <- "http://donaldkenney.x10.mx/"
fossil_table <- NULL

#start the for loop that will scrape individual tables on separate URL's
for (i in 1:length(website_structure)){
  #assemble the entire URL that we need to navigate to  
  url_suffex <- qdapRegex::ex_between(website_structure[i], '"', '"')
  url <- paste(url_prefix,url_suffex,sep="")
  
  #scrape the indvidual table
  fossils <- read_html(url)
  fossildf <- fossils %>% html_nodes("table") %>% html_table(fill=TRUE)
  fossildf <- fossildf[[1]]

  #bind indiviudal table to the master table
  fossil_table <- rbind(fossil_table,fossildf)
}

#rename some columns
names <- colnames(fossil_table)
names[10:15] <- c("10","LatLong","12","13","14","15")
colnames(fossil_table) <- names

#Mutate in lat/long columns and select the required columns  
fossil_table <- fossil_table %>% 
  mutate(Latitude = substr(LatLong,1,7), Longitude = substr(LatLong,9,17)) %>%
  select(Location,County,`State/Province`,`Directions,Notes`,Age,Formation,Fossils,Latitude,Longitude)

#Change lat/long to numeric from string.    
fossil_table$Latitude <- as.numeric(fossil_table$Latitude) 
fossil_table$Longitude <- as.numeric(fossil_table$Longitude)

#view final table
str(fossil_table)
```

```
## 'data.frame':	16882 obs. of  9 variables:
##  $ Location        : chr  "Central Alberta*" "Mount Dawson Creek[?]" "Southern Alberta" "Southern Alberta" ...
##  $ County          : chr  "[?]" "[?]" "[?]" "[?]" ...
##  $ State/Province  : chr  "AB" "AB" "AB" "AB" ...
##  $ Directions,Notes: chr  "" "" "Alberta in" "" ...
##  $ Age             : chr  "K" "Kl" "Ku" "Ku" ...
##  $ Formation       : chr  "Edmonton Group" "Commotion" "Oldman" "Judith River" ...
##  $ Fossils         : chr  "vertebrates-reptilia-dinosauria-Leptoceratops,ornithischia-Pachyrhinosaurus" "plants" "vertebrates-reptilia-dinosauria-Edmontosaurus,hadrosauroidea-Corythosaurus,Saurolophus;ornithischia-Monoclonius"| __truncated__ "vertebrates-reptilia-dinosauria-ornithischia-ankylosauria-Panoplosaurus;nodosauridae-Edmontonia,Euoplocephalus" ...
##  $ Latitude        : num  53.5 51.2 49.6 49.6 49.1 ...
##  $ Longitude       : num  -114 -117 -113 -113 -114 ...
```

### 2. Use leaflet to make an interactive map of fossil localities
It should be emphasized here that there are ~16000 observations in this dataframe, so trying to plot all the fossil localities in North America will be computationally taxing. It is recommended that you filter by state as done below.

```r
#Filter data to include one or more states/provinces. State/province codes are listed below:
#"AB" "AK" "AL" "AR" "AZ" "BA" "BC" "CA" "CO" "CT" "DC" "DE" "FL" "GA" "HI" "IA" "ID" "IL" "IN" "KS" "KY" "LA" "MA" "MB" "MD" "ME" "MI" "MN" "MO" "MS" "MT" "NB" "NC"
#"ND" "NE" "NH" "NJ" "NL" "NM" "NS" "NT" "NU" "NV" "NY" "OH" "OK" "ON" "OR" "PE" "PA" "QC" "RI" "SC" "SD" "SK" "TN" "TX" "UT" "VA" "VT" "WA" "WI" "WV" "WY" "YT"
SearchLocation <- c("BC")

fossil_map <- fossil_table %>% filter(`State/Province`==SearchLocation) # the master table will be filtered into a dataframe for mapping (fossil_map)

#Use leaflet package to make a map of fossil localtities within the fossil_map dataframe
m <- leaflet() %>%
  addTiles() %>%  # Add default OpenStreetMap map tiles
  addMarkers(lng=fossil_map$Longitude, lat=fossil_map$Latitude, 
            popup=paste("Fossils: ", fossil_map$Fossils, "<br/>", 
            "Formation: ", fossil_map$Formation))
m
```

<!--html_preserve--><div id="htmlwidget-83eb11d1b7005ac73cf7" style="width:672px;height:480px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-83eb11d1b7005ac73cf7">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addTiles","args":["//{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"attribution":"&copy; <a href=\"http://openstreetmap.org\">OpenStreetMap<\/a> contributors, <a href=\"http://creativecommons.org/licenses/by-sa/2.0/\">CC-BY-SA<\/a>"}]},{"method":"addMarkers","args":[[54.6961,54.7872,54.8266,54.8276,54.8254,54.8254,54.8149,54.4998,48.3297,48.3829,48.4268,48.4198,51.6236,52.9167,52.3332,52.3349,52.3349,52.3621,52.9826,49.083,51.398,51.3827,51.3954,51.3827,51.4528,51.1818,51.4537,51.4537,51.4652,51.4537,51.4584,51.3992,51.4219,51.3986,51.4611,49.5495,49.5495,49.6811,49.8329,49.6922,49.6925,49.5377,49.6805,48.8663,48.9829,50.3477,50.4329,49.4664,49.5077,49.5077,49.5077,51.4433,50.3117,49.5136,51.4537,51.4652,51.5169,53.8729,59.5163,54.5705,54.5705,54.5133,54.5705,58.633,49.5497,49.5497,49.5497,null,49.5497,49.5497,50.7831,49.083,49.3955,51.5542,49.1997,48.9886,49.2642,49.3286,49.3018,49.262,49.2742,50.0302,50.6076,49.1942,49.1663,49.1942,49.3209,49.3502,49.0664,49.0635,49.2982,49.4584,49.4585,49.4585,49.4585,49.4585,49.4585,49.4585,49.469,49.3538,49.3767,55.6976,56.0413,56.2456,50.5894,56.45,54.5381,null,56.0322,54.864,56.2636,56.1526,55.1347,49.4889,53.0495,52.9995,53.2328,50.5278,49.0497,57.5683,50.0042,49.4163,49.4163,50.7372,50.8831,50.0899,50.759,50.759,51.4254,50.7971,50.7331,50.0894],[null,-127.1695,-127.0188,-127.2821,-127.0222,-127.0222,-127.6041,-126.3034,null,-123.8693,-123.3622,-123.3673,-124.7262,-120.2527,-121.4194,-121.4161,-121.4161,-121.0685,-122.4921,-122.1859,-116.4919,-116.4479,-116.4822,-116.4479,-116.2869,-116.1162,-116.3342,-116.3342,-116.3753,-116.3342,-116.3603,-116.4372,-116.417,-116.4333,-116.3639,-124.6909,-124.6909,-124.9264,-125.3529,-124.9983,-124.9924,-124.7333,-125.0656,-124.2694,-124.5695,-116.6075,-127.1201,-123.0193,-115.7627,-115.7627,-115.7627,-116.342,-115.8651,-115.7767,-116.3342,-116.3753,-116.4015,-120.0374,-124.0866,-120.7755,-120.7755,-120.7027,-120.7755,-123.703,-121.8359,-121.8359,-121.8359,null,-121.8359,-121.8359,-121.1359,-121.7025,-123.2194,-124.4026,-122.5859,-123.0567,-123.1128,-123.1301,-123.1499,-123.2522,-123.1633,-115.6384,-127.7037,-124.0703,-123.936,-124.0703,-124.3136,-125.0079,-120.7858,-120.782,-119.5302,-120.509,-120.5054,-120.5054,-120.5054,-120.5054,-120.5054,-120.5054,-120.5048,-120.5447,-120.5921,-121.6296,-123.9807,-120.8354,-115.2003,-122.868,-120.7456,null,-121.9043,-121.5527,-123.4369,-120.6817,-120.9925,-124.3546,-131.7867,-132.0034,-132.0034,-122.9362,-120.9692,-128.8381,-125.1617,-123.386,-123.386,-121.2713,-121.4026,-121.4722,-120.6683,-120.6683,-120.2048,null,-120.1192,-121.5555],null,null,null,{"interactive":true,"draggable":false,"keyboard":true,"title":"","alt":"","zIndexOffset":0,"opacity":1,"riseOnHover":false,"riseOffset":250},["Fossils:  invertebrates-mollusks <br/> Formation:  ","Fossils:  invertebrates-mollusks-cephalopoda-ammonoidea <br/> Formation:  ","Fossils:  invertebrates-arthropoda-insecta;;plants,vertebrates-fish? <br/> Formation:  ","Fossils:  invertebrates-mollusks-cephalopoda-ammonoidea;;;plants(petrified wood) <br/> Formation:  ","Fossils:  plants <br/> Formation:  ","Fossils:  invertebrates-arthropoda-insecta-hemiptera-Cercopidae;;;;plants-angiosperms-sequoioidae-Metasequoia;;gymnospermopsida-Glyptostrobus;;vertebrates(?) <br/> Formation:  Smithers?","Fossils:  invertebrates(marine) <br/> Formation:  ","Fossils:  invertebrates-mollusks <br/> Formation:  ","Fossils:  - <br/> Formation:  ","Fossils:  invertebrates-mollusks-bivalvia,gastropoda <br/> Formation:  ","Fossils:  invertebrates(obscure) <br/> Formation:  ","Fossils:   <br/> Formation:  ","Fossils:  invertebrates-mollusks-cephalopoda-belemnoidea <br/> Formation:  ","Fossils:  invertebrates(Ediacaran) <br/> Formation:  ","Fossils:  invertebrates-arthropoda-insecta?;;plants?,vertebrates-fish-actinopterygii-Amyzon <br/> Formation:  ","Fossils:  vertebrates-fish-actinopterygii-Eohiodon <br/> Formation:  ","Fossils:  plants(leaves)(impressions),vertebrates-fish <br/> Formation:  ","Fossils:  invertebrates-arthropoda-insecta(55Taxa);;plants(41Taxa),vertebrates-fish(3Taxa) <br/> Formation:  ","Fossils:  plants(leaves),vertebrates-fish <br/> Formation:  ","Fossils:  plants <br/> Formation:  Huntington_BC","Fossils:   <br/> Formation:  ","Fossils:  invertebrates-arthropoda-trilobita-Zacanthoides <br/> Formation:  Stephen Shale","Fossils:  invertebrates-arthropoda-trilobita(diverse),(others) <br/> Formation:  ","Fossils:  invertebrates-arthropoda-trilobita-bathyuridae-Bathyuriscus;Burlingia,Neolenus <br/> Formation:  Stephen","Fossils:  invertebrates-arthropoda-trilobita-olenellidae-Olenellus(O gilberti) <br/> Formation:  ","Fossils:  invertebrates <br/> Formation:  Stephen","Fossils:  invertebrates-arthropoda-trilobita-Ogygopsis,(others) <br/> Formation:  Stephen","Fossils:  invertebrates-mollusks <br/> Formation:  Cathedral (Canada)","Fossils:  invertebrates-mollusks <br/> Formation:  Mount Whyte","Fossils:  invertebrates-brachiopoda-acrotretida-Acrothele <br/> Formation:  ","Fossils:  invertebrates-mollusks <br/> Formation:  Paget","Fossils:  invertebrates-arthropoda-trilobita-Ogygopsis,(others) <br/> Formation:  Stephen","Fossils:  invertebrates-mollusks <br/> Formation:  Mount Whyte","Fossils:  invertebrates-arthropoda-trilobita-Elrathia,Ogygopsis,Zacanthoides <br/> Formation:  ","Fossils:  invertebrates-mollusks <br/> Formation:  Paget","Fossils:  invertebrates-mollusks-bivalvia-Inoceramus,Nemodon;cephalopoda-ammonoidea-Gaudryceras,Hypophylloceras,Nostoceras,Pachydiscus,Phyllopachyceras,Pseudophyllites <br/> Formation:  Lambert_BC","Fossils:  plants(seeds)(cones) <br/> Formation:  Spray","Fossils:  plants <br/> Formation:  Nanaimo","Fossils:  plants(seeds)(cones) <br/> Formation:  Comox","Fossils:   <br/> Formation:  ","Fossils:  vertebrates-reptilia(marine)-plesiosauria,squamata-mosasauridae;turtles <br/> Formation:  Haslam,Pender","Fossils:  invertebrates-mollusks-bivalvia,cephalopoda-ammonoidea <br/> Formation:  ","Fossils:  invertebrates?,vertebrates-reptilia-Elasmosaurus,Tylosaurus <br/> Formation:  ","Fossils:  ? <br/> Formation:  ","Fossils:  invertebrates(55 taxa)-arthropoda-trilobita(3 taxa);brachiopoda(22 taxa) <br/> Formation:  Mount Mark","Fossils:  invertebrates-cnidaria-corals <br/> Formation:  Simla","Fossils:  invertebrates-mollusks <br/> Formation:  Mount Whyte","Fossils:  invertebrates-mollusks <br/> Formation:  Cathedral (Canada)","Fossils:  invertebrates-arthropoda-trilobita-olenellidae-Olenellus;Wanneria <br/> Formation:  Eager","Fossils:  invertebrates-mollusks <br/> Formation:  Eager","Fossils:  invertebrates-arthropoda-trilobita-Labiostria <br/> Formation:  ","Fossils:  invertebrates-brachiopoda-Obolus <br/> Formation:  Eldon","Fossils:  invertebrates-brachiopoda,cnidaria-corals;mollusks-cephalopoda-ammonoidea;;;protists-forams <br/> Formation:  ","Fossils:  invertebrates-arthropoda-trilobita-olenellidae-Olenellus;Wanneria <br/> Formation:  Eager","Fossils:  invertebrates-arthropoda-trilobita-Albertella,bathyuridae-Bathyuriscus;;;brachiopoda-acrotretida-Acrothele;Micromitra,Obolus,Wimanella <br/> Formation:  ","Fossils:  invertebrates-mollusks <br/> Formation:  Sherbrooke","Fossils:  invertebrates-mollusks <br/> Formation:  Paget","Fossils:  invertebrates-cnidaria-corals <br/> Formation:  Simla","Fossils:  vertebrates-fish-sarcopterygii-dipnoi-coelacanthiformes-Bobasatrania? <br/> Formation:  Grayling","Fossils:  vertebrates-fish-chondrichthyes-selachii(sharks)-holocephali-Listracanthus <br/> Formation:  Sulphur Mountain","Fossils:  vertebrates-fish(large)-sarcopterygii-dipnoi-coelacanthiformes-Albertonia,Bobasatrania;;;;reptilia-ichthyosauria <br/> Formation:  ","Fossils:  vertebrates-fish-actinopterygii-palaeoniscidae-Boreosomus,Pteronisculus;Perleidus,Saurichthys;chondrichthyes-selachii-holocephali-Listracanthus;;;sarcopterygii-dipnoi-coelacanthiformes-Albertonia,Bobasatrania,Whitea <br/> Formation:  Sulphur Mountain","Fossils:  vertebrates-fish-chondrichthyes-selachii-hybodontidae-Palaeobates;;;;reptilia-ichthyosauria-Shastasaurus <br/> Formation:  Sulphur Mountain","Fossils:  vertebrates-fish(bones) <br/> Formation:  Toad,Grayling","Fossils:  invertebrates-mollusks-bivalvia-Buchia;cephalopoda-belemnoidea <br/> Formation:  ","Fossils:  invertebrates-mollusks-bivalvia-Aucella <br/> Formation:  ","Fossils:  invertebrates-mollusks-cephalopoda-ammonoidea-Paracadoceras,Pseudocadoceras <br/> Formation:  Mysterious Creek","Fossils:  invertebrates-mollusks-bivalvia-Buchia;cephalopoda-ammonoidea,belemnoidea <br/> Formation:  Mysterious Creek","Fossils:  invertebrates-mollusks-bivalvia-Buchia;cephalopoda-belemnoidea <br/> Formation:  ","Fossils:  invertebrates-mollusks-cephalopoda-ammonoidea <br/> Formation:  ","Fossils:  invertebrates-arthropoda-insecta;;plants(leaves-ginkgophyta-Ginkgo(seed pods);;vertebrates-fish <br/> Formation:  Kamloops Group","Fossils:  invertebrates-echinoderms-crinoids(fragments) <br/> Formation:  Chilliwack Group","Fossils:  invertebrates-brachiopoda-Gigantoproductus;bryozoa,cnidaria-coelenterata <br/> Formation:  ","Fossils:  plants? <br/> Formation:  ","Fossils:  plants <br/> Formation:  Huntington_BC","Fossils:  invertebrates-mollusks(shells) <br/> Formation:  ","Fossils:  invertebrates-brachiopoda,echinoderms-echinoidea(sea_urchins)(fragments);mollusks;worms(tubes)-annelida-Serpula <br/> Formation:  ","Fossils:  plants(leaves)(wood)(fruits)(others)(poorly preserved) <br/> Formation:  Burrard","Fossils:  plants(leaves) <br/> Formation:  Burrard,Kitsilano","Fossils:   <br/> Formation:  ","Fossils:  plants(leaves)(coal) <br/> Formation:  Kitsilano","Fossils:  vertebrates-fish-sarcopterygii-dipnoi-coelacanthiformes <br/> Formation:  Grayling","Fossils:  plants(fragments)-gymnospermopsida-coniferophyta;pteridophyta-(ferns) <br/> Formation:  Longarm (Canada)","Fossils:  ? <br/> Formation:  ","Fossils:  plants <br/> Formation:  Nanaimo","Fossils:  plants(seeds)(cones) <br/> Formation:  Haslam","Fossils:  invertebrates-mollusks-cephalopoda-ammonoidea-Pachydiscus? <br/> Formation:  Haslam","Fossils:   <br/> Formation:  ","Fossils:  invertebrates(diverse)(marine) <br/> Formation:  ","Fossils:  plants-angiosperms-Ficus_fig,Magnolia,Platanus(sycamore);ginkgophyta-Ginkgo;gymnospermopsida-coniferophyta;pteridophyta-cycadopsida,(ferns) <br/> Formation:  ","Fossils:  plants(leaves)(pine_needles) <br/> Formation:  ","Fossils:   <br/> Formation:  ","Fossils:  invertebrates-arthropoda-insecta(rare);mollusks-gastropoda;;plants(well preserved),vertebrates-mammals(teeth) <br/> Formation:  Allenby_BC?","Fossils:  plants(leaves)(pine needles)(seeds)(roots casts) <br/> Formation:  ","Fossils:  vertebrates-fish-actinopterygii-Eohiodon <br/> Formation:  Allenby_BC","Fossils:  plants <br/> Formation:  ","Fossils:  plants <br/> Formation:  ","Fossils:  plants(coal) <br/> Formation:  Allenby_BC","Fossils:  plants(leaves)-gymnospermopsida-coniferophyta-pineacea(needles)(cones) <br/> Formation:  ","Fossils:  plants-angiosperms-Paleorosa <br/> Formation:  Allenby_BC","Fossils:  invertebrates-arthropoda-insecta;;plants(leaves) <br/> Formation:  ","Fossils:  plants(leaves)(impressions) <br/> Formation:  ","Fossils:  vertebrates-reptilia(marine)-rauisuchidae <br/> Formation:  Pardonet","Fossils:  vertebrates-reptilia-ichthyosauria <br/> Formation:  ","Fossils:  - <br/> Formation:  ","Fossils:  - <br/> Formation:  ","Fossils:  - <br/> Formation:  ","Fossils:  invertebrates-cnidaria-corals;;(others) <br/> Formation:  ","Fossils:  invertebrates-mollusks <br/> Formation:  ","Fossils:  ichnofossils-burrows_invertebrate-Skolithos <br/> Formation:  Monkman Quartzite","Fossils:  invertebrates-cnidaria-corals <br/> Formation:  Kakisa","Fossils:  invertebrates(marine)-mollusks-cephalopoda-ammonoidea <br/> Formation:  ","Fossils:   <br/> Formation:  ","Fossils:  invertebrates-mollusks <br/> Formation:  ","Fossils:  invertebrates-mollusks-cephalopoda-ammonoidea-Desmoceras <br/> Formation:  Haida","Fossils:  plants <br/> Formation:  ","Fossils:  invertebrates-mollusks-cephalopoda-ammonoidea-Teloceras <br/> Formation:  Yakoun","Fossils:  - <br/> Formation:  ","Fossils:  invertebrates-brachiopoda-Gigantoproductus;bryozoa,cnidaria-coelenterata <br/> Formation:  ","Fossils:  - <br/> Formation:  ","Fossils:  invertebrates-arthropoda-crustacea-decapoda-Longusorbis <br/> Formation:  Spray","Fossils:  invertebrates-mollusks-cephalopoda-ammonoidea <br/> Formation:  ","Fossils:  invertebrates-mollusks-cephalopoda-ammonoidea(sparse) <br/> Formation:  ","Fossils:  invertebrates-brachiopoda,conodonts,mollusks-bivalvia,cephalopoda-ammonoidea,belemnoidea;;;plants-(microflora),(pollen),pteridophyta-(ferns) <br/> Formation:  ","Fossils:  plants(amber) <br/> Formation:  ","Fossils:  invertebrates-mollusks-bivalvia-Aucella <br/> Formation:  ","Fossils:  vertebrates-fish-actinopterygii-Eohiodon <br/> Formation:  Tranquille","Fossils:  vertebrates-fish-actinopterygii-Eohiodon <br/> Formation:  ","Fossils:  plants(leaves)-pteridophyta-(ferns) <br/> Formation:  ","Fossils:  invertebrates-arthropoda-insecta(55Taxa);;plants(41Taxa),vertebrates-fish(3Taxa) <br/> Formation:  ","Fossils:  invertebrates-echinoderms-crinoids <br/> Formation:  ","Fossils:  plants <br/> Formation:  "],null,null,null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null]}],"limits":{"lat":[48.3297,59.5163],"lng":[-132.0034,-115.2003]}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

