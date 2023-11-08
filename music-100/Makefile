BIN := ./node_modules/.bin

all: download map

# Fetch the data file
# process with a node version of jq (jqn) to grab the values
# could probably just format in jq to a geojson, but...
# easy enough to pipe to a csv 
# and then to a geojson with existing tools, specifying which fields to use for lat lon
# then write out the geojson for later use
download:
	curl https://www.johnnycashhasbeeneverywhere.com/america_keyed.json \
	| $(BIN)/jqn 'values' \
	| $(BIN)/json2csv \
	| $(BIN)/csv2geojson --lat lat --lon lng \
	> cash.geojson



# load in the countries shapefile from natural earth
# add the cash places geojson and style the circles
# all points are in the Americas so doing a orthographic projection to 
# just look at that part of the globe
# then write out the svg
map:
	$(BIN)/mapshaper -i ../shapefiles/ne_50m_admin_0_countries.zip name="countries" \
	-svg-style fill="#ccc" \
	-graticule interval=15 name=grat \
	-svg-style stroke="rgba(0,0,0,0.05)" \
	-i cash.geojson name="places" \
	-svg-style fill="#885599" \
	-svg-style stroke-width="0.5" \
	-svg-style fill-opacity=0.5 \
	-svg-style stroke="#0c0c0c" \
	-svg-style r='4' \
	-proj +proj=ortho +lat_0=10 +lon_0=-100 +x_0=0 +y_0=0 +a=6378140 +b=6356750 +units=m +no_defs target=* \
	-o cash.svg format=svg target=*