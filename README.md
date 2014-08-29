# Norvig Web Data Science Award
## Entry Naward 2014: Where are the Bitcoins?

Naward Group 14

[Pieter Hameete](mailto:P.A.hameete@student.tudelft.nl)

[Tamis van der Laan](mailto:T.A.vanderLaan@student.tudelft.nl)

Delft University of Technology

IN4144 Data Sciene

### Idea

Bitcoin is a decentralized digital currency that operates through a network of privately owned computers. It is possible to connect to this network and request information regarding transactions and valuta. Furthermore the digital currency is anonymous in the sense that a bitcoin user is represented by an address based on its public key. Currently [over 13 million](https://blockchain.info/charts/total-bitcoins) bitcoins are in circulation, representing a value of [more than 6,5 billion USD](https://blockchain.info/charts/market-cap) at present. 

Our initial idea comes from our curiosity of where the owners of this digitual currency reside. To get a deeper insight into the users behind the bitcoins we aimed to retrieve more information about them by means of parsing the web pages available in the common crawl dataset. Because it is difficult to get information about individual bitcoin users based on their bitcoin address, we can look for additional information regarding the nationality and language of the bitcoin users in the web pages where we find their address. Our goal is to attribute bitcoin balance, ingoing transactions and outgoing transactions as a collective per country or language and to visualise this on a world map. This visualisation should then show an estimation of the relative amount of bitcoins owned by the citizens of each country, as well as provide insight into the relative amount of incoming and outgoing bitcoins per country.

### Method

#### Parsing Web Pages

Similarly to the provided `Href Extractor` from the warcexamples we have used the [warcutils library](https://github.com/norvigaward/warcutils) for parsing the sequence files to `WarcRecords`. These `WarcRecords`, each corresponding to a single web page, are then further processed by our MapReduce job that was written using Hadoop.000000000000000000

#### Detecting Bitcoin Addresses

Our first goal is to find any bitcoin addresses in the web page that correponds to the `WarcRecord` that is provided as input to the Mapper. These addresses may be found in any place on the webpage. The fastest method however is to look for so-called bitcoin URLs. These bitcoin URLs may be found in the _href_ property of hyperlinks and are formatted as _bitcoin:address_ to allow for easy usage by bitcoin applications installed on the pc of a visitor. These address are extracted from the raw html string of the web page by using a regular expression. To validate whether the extracted addresses are in fact valid addresses the [bitcoinj](http://code.google.com/p/bitcoinj/) library is used.

A majority of the bitcoin addresses present in the webpages can not be found in these nicely formatted bitcoin links however. A more complete  approach is to also check for each word in the website body text whether it is a valid bitcoin address using bitcoinj. We found that adding this detection to the url detection resulted in roughly 3 times more detected bitcoin addresses. Unfortunately this approach is computationally very expensive, causing the hadoop job to require 6 times more execution time. **Results used in this document are solely based on addresses found in bitcoin URLs. The run using a full content scan for bitcoin addresses is expected to complete on September 2nd and these results be swapped in later.**

#### Determining Webpage Country and Language

If for the web page that is being processed one or more bitcoin addresses are found we proceed with the next step: determining the country and language of the web page. We deduced the country by looking at the domain name in the URL of the webpage. The domain extension was extracted using a regular expression and then mapped to the corresponding country. using a predefined table. Only country dependent domain extensions could be used. General domain extensions such as .org or .com were mapped to an unknown country. Similarly domain extensions of webpages that are frequently used to generate short URLs (such ly for as Bit.ly) are mapped to an unknown country.

Detecting the language was done by making use of the [java language classifier langdetect by Nakatani](http://code.google.com/p/language-detection/). This language classifier with a precision of 99% was applied to the body text of the web page to determine the language of the webpage content. The mapper finally sends the <Address, Language> and <Address, Country> pairs to the reducers.

#### Removing Duplicates and Gathering Address Information

In the Reducer all information for a single Bitcoin address is gathered and duplicate addresses are removed. Now more information is extracted for the bitcoin address itself in the reducer. We used several blockchain.info web services to extract additional information from the bitcoin addresses. For this they have kindly provided us with an API key to avoid the request limit. The following web services are used to retrieve more information on balance and the amount of sent and received bitcoins:
Balance: “https://blockchain.info/q/addressbalance/<bitcoin address>” 
Outgoing transactions: “https://blockchain.info/q/getsentbyaddress/<bitcoin address>”
Incoming transactions: “https://blockchain.info/q/getrecievedbyaddress/<bitcoin address>”

Finally every bitcoin address and the extracted information is written to a json object by the Reducer to allow for easy further usage by our Javascript visualization. 

#### Visualizing Results

##### Postprocessing the Hadoop job output

As a final step we want to visualise the information that was extracted by the hadoop job. Because we wanted to visualize the information by country and by language the resulting json file was parsed using a python script. The final merged output file from the mapreduce application was small enough to allow for postprocessing on a single PC. Using a python script we generated two new json files: one containing information on a country basis and the other on a language basis. The python script computed for each country and language the total and relative amount of bitcoins owned, sent and received, as well as the total amount of bitcoins owned, sent and received worldwide amongst all crawled addresses.

##### Visualising on a World Map

We used the [d3 (Data-driven Documents)](http://d3js.org/) Javascript library to visualise the country and language data. The country data was visualized by coloring a world map which was drawn using the [topojson](https://github.com/mbostock/topojson) library according to one of the properties (balance, incoming transactions, outgoing transactions) which can be selected using a drop down menu. The relative amount is then indicated by coloring the country using a gradient multiple color scale. The language information is visualised in a pie chart. The pie chart also uses a dropdown menu to select the property to be visualized. The colored world map was made to show in full screen and is pannable as well as zoomable. Over this map the pie chart was superimposed in order to show all the information at once in an integrated interface.

### Results

Our final visualization can be found at the following webpage: http://naward14.uphero.com/ (sorry for the ad!).

### Discussion

### Future Work



