BIN := ./node_modules/.bin

all: install download map


install:
	npm install

# Fetch the data file
# process with a node version of jq (jqn) to grab the values
# could probably just format in jq to a geojson, but...
# easy enough to pipe to a csv 
# and then to a geojson with existing tools, specifying which fields to use for lat lon
# then write out the geojson for later use
download:
	curl https://www.johnnycashhasbeeneverywhere.com/america_keyed.json \
	| $(BIN)/jq 'to_entries | map(.value) | to_entries | map( .value + { idx: .key})' \
	| $(BIN)/json2csv \
	| $(BIN)/csv2geojson --lat lat --lon lng \
	> points.geojson



# load in the countries shapefile from natural earth
# add the cash places geojson and style the circles
# all points are in the Americas so doing a orthographic projection to 
# just look at that part of the globe
# then write out the svg
# you can play around with +lat_0=0 +lon_0=-90 to rotate the globe
map:
	$(BIN)/mapshaper \
	-i ../shapefiles/ne_50m_admin_0_countries/ne_50m_admin_0_countries.shp name=countries \
	-svg-style fill="rgba(233,218,195,.4)" \
	-svg-style stroke="rgba(35,35,48,0.4)" stroke-width=".5" \
	-i ../shapefiles/ne_50m_lakes/ne_50m_lakes.shp name=lakes \
	-filter "min_zoom < 1.1" \
	-erase lakes target=countries \
	-i ../shapefiles/ne_50m_coastline/ne_50m_coastline.shp name="coastline" \
	-svg-style stroke="rgba(195,177,151,0.5)" stroke-width=5 \
	-i ../shapefiles/ne_50m_ocean/ne_50m_ocean.shp name="ocean" \
	-svg-style fill-pattern="hatches 45deg 2px rgba(25,25,30,1) 2px rgba(35,35,50,1)" \
	-graticule interval=15 name=grat \
	-svg-style stroke="rgba(233,218,195,0.4)" stroke-width=.25 \
	-i points.geojson name="places_outline" \
	-each "type='center'" \
	-i points.geojson name="places" \
	-each "type='outline'" \
	-merge-layers target=places,places_outline name=points \
	-target points \
	-sort "+this.properties.idx" \
	-colorizer name=centerFill colors='#ff292f,#0c0c0c' categories='center,outline' \
	-colorizer name=centerStroke colors='none,#fff' categories='center,outline' \
	-colorizer name=rad colors='2,6' categories='center,outline' \
	-svg-style fill='centerFill(this.properties.type)' \
	-svg-style stroke='centerStroke(this.properties.type)' \
	-svg-style r='rad(type)' \
	-i points.geojson name="places_labels" \
	-filter "name==='Winnemucca' && idx<1" \
	-style label-text=this.properties.name fill="#0c0c0c" text-anchor=start dx=4 dy=-3 font-size=9 font-weight=400 \
	-proj +proj=ortho +lat_0=0 +lon_0=-90 +x_0=0 +y_0=0 +a=6378140 +b=6356750 +units=m +no_defs target=* \
	-o answer.svg format=svg target=*
