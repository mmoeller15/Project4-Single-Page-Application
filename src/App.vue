<script>
import $ from 'jquery'

export default {
    data() {
        return {
            view: 'map',
            codes: [],
            neighborhoods: [], 
            incidents: [],
            errors: [],
            new_incident:{
                case_number: null,
                date: null,
                time: null,
                code: null,
                incident: null,
                police_grid: null,
                neighborhood_number: null,
                block: null
            },
            searchbar: {
                address: null
            },
            filter: {
                incident_type: [],
                neighborhood_number: [],
                startDate: null,
                endDate: null,
                limit: null,
            }, 
            leaflet: {
                map: null,
                center: {
                    lat: 44.955139,
                    lng: -93.102222,
                    address: ""
                },
                zoom: 12,
                bounds: {
                    nw: {lat: 45.008206, lng: -93.217977},
                    se: {lat: 44.883658, lng: -92.993787}
                },
                neighborhood_markers: [
                    {location: [44.942068, -93.020521], marker: null, name: 'Conway/Battlecreek/Highwood'},
                    {location: [44.977413, -93.025156], marker: null, name: 'Greater East Side'},
                    {location: [44.931244, -93.079578], marker: null, name: 'West Side'},
                    {location: [44.956192, -93.060189], marker: null, name: 'Dayton\'s Bluff'},
                    {location: [44.978883, -93.068163], marker: null, name: 'Payne/Phalen'},
                    {location: [44.975766, -93.113887], marker: null, name: 'North End'},
                    {location: [44.959639, -93.121271], marker: null, name: 'Thomas/Dale(Frogtown)'},
                    {location: [44.947700, -93.128505], marker: null, name: 'Summit/University'},
                    {location: [44.930276, -93.119911], marker: null, name: 'West Seventh'},
                    {location: [44.982752, -93.147910], marker: null, name: 'Como'},
                    {location: [44.963631, -93.167548], marker: null, name: 'Hamline/Midway'},
                    {location: [44.973971, -93.197965], marker: null, name: 'St. Anthony'},
                    {location: [44.949043, -93.178261], marker: null, name: 'Union Park'},
                    {location: [44.934848, -93.176736], marker: null, name: 'Macalester-Groveland'},
                    {location: [44.913106, -93.170779], marker: null, name: 'Highland'},
                    {location: [44.937705, -93.136997], marker: null, name: 'Summit Hill'},
                    {location: [44.949203, -93.093739], marker: null, name: 'Capitol River'}
                ]
            }
        };
    },
    methods: {
        viewMap(event) {
            this.view = 'map';
        },

        viewNewIncident(event) {
            this.view = 'new_incident';
        },

        viewAbout(event) {
            this.view = 'about';
        },
        
        getJSON(url) {
            return new Promise((resolve, reject) => {
                $.ajax({
                    dataType: 'json',
                    url: url,
                    success: (response) => {
                        resolve(response);
                    },
                    error: (status, message) => {
                        reject({status: status.status, message: status.statusText});
                    }
                });
            });
        },

        uploadJSON(method, url, data) {
            return new Promise((resolve, reject) => {
                $.ajax({
                    type: method,
                    url: url,
                    contentType: 'application/json; charset=utf-8',
                    data: JSON.stringify(data),
                    dataType: 'text',
                    success: (response) => {
                        resolve(response);
                    },
                    error: (status, message) => {
                        reject({status: status.status, message: status.statusText});
                    }
                });
            });
        },

        getIncidentType(code) {
            let i = 0;
            //console.log(code);
             while (code != this.codes[i].code) {
                i++;
            }
            //console.log(this.codes[i].type);
            return this.codes[i].type;
        },

        newIncident(event){
            console.log(event);
            
            let url = "http://localhost:8005/new-incident";
            let i;
            let j;
            let counter = 0;
            if (this.new_incident.case_number != null){
                $('#case_number').css('border-color', 'green');
                counter = counter + 1;
            } else{
                $("#case_number").css('border-color', 'red');
            }

            if (this.new_incident.date != null){
                $('#date').css('border-color', 'green');
                counter = counter + 1;
            } else{
                $('#date').css('border-color', 'red');
            }

            if (this.new_incident.time != null){
                $('#time').css('border-color', 'green');
                counter = counter + 1;
            } else{
                $('#time').css('border-color', 'red');
            }

            if (this.new_incident.code != null){
                $('#code').css('border-color', 'green');
                counter = counter + 1;
            } else{
                $('#code').css('border-color', 'red');
            }

            if (this.new_incident.incident != null){
                $('#incident').css('border-color', 'green');
                counter = counter + 1;
            } else{
                $('#incident').css('border-color', 'red');
            }

            if (this.new_incident.police_grid != null){
                $('#police_grid').css('border-color', 'green');
                counter = counter + 1;
            } else{
                $('#police_grid').css('border-color', 'red');
            }

            if (this.new_incident.neighborhood_number != null){
                counter = counter + 1;
                $("#neighborhood_number").css('border-color', 'green');
            } else{
                $("#neighborhood_number").css('border-color', 'red');
            
            }

            if (this.new_incident.block != null){
                $('#block').css('border-color', 'green');
                counter = counter + 1;
            } else{
                $('#block').css('border-color', 'red');
            }
            
            
            console.log(counter);
            if (counter === 8){
                    this.uploadJSON('PUT', url, this.new_incident).then((data) => {
                    console.log(data);
                    alert("Incident added.");
                    this.reset();
                }).catch((error) => {
                    console.log(error);
                    if (error.status === 500){
                        $("#case_number").css('border-color', 'red');
                        alert("This incident already exists. Please try again");
                    }
                    else {
                        alert("404: Could not be added");
                    }
                })
            } else {
                alert("Please fill in all values.");
            }
        },

        reset() {
            this.getJSON('http://localhost:8005/incidents').then((data) => {
            this.incidents = data;
        }).catch((error) => {
            console.log('Error:', error);
        })
        },

        filterIncident(filterData) {
            let url = "http://localhost:8005/incidents?";
            let count = 0;
            //Incident Type
            let j;
            let incidentCount = 0;
            for(j = 0; j < filterData.incident_type.length; j++) {
                
                if(filterData.incident_type[j] == true) {
                    if(count > 0 && incidentCount == 0) {
                    url = url + "&";
                    }
                    if (incidentCount == 0) {
                        url = url + "code="  + j;
                    }
                    for(let i = ((j+1)*100); i < ((j+1)*100) + 100; i++) {
                        url = url + "," + i;
                    }
                    incidentCount++;
                    count++;
                }
                
                

            }
            //Start Date
            if(filterData.startDate != null) {
                if(count > 0) {
                    url = url + "&";
                }
                url = url + "start_date=" + filterData.startDate;
                count++;
            }
            //End Date
            if(filterData.endDate != null) {
                if(count > 0) {
                    url = url + "&";
                }
                url = url + "end_date=" + filterData.endDate;
                count++;
            }
            //Neighborhood Number
            let i;
            let neighborhoodCount = 0;
            for(i = 0; i < filterData.neighborhood_number.length; i++) {
                
                if(filterData.neighborhood_number[i] == true) {
                    if(count > 0 && neighborhoodCount == 0) {
                    url = url + "&";
                    }
                    if (neighborhoodCount == 0) {
                        url = url + "neighborhood="  + (i + 1);
                    }
                    if (neighborhoodCount > 0) {
                        url = url + ","  + (i + 1);
                    }
                    
                    neighborhoodCount++;
                    count++;
                }

            }
            //Limit
            if(filterData.limit != null) {
                if(count > 0) {
                    url = url + "&";
                }
                url = url + "limit=" + filterData.limit;
                count++;
            }

            console.log(url);
            this.getJSON(url)
            .then((data) => {
                this.incidents = data;
            })
        },

        removeIncident(i) {
            let url = "http://localhost:8005/remove-incident";
            this.uploadJSON('DELETE', url, this.incidents[i]).then((data) => {
                console.log(data);
                alert("Incident removed.");
                this.reset();
            }).catch((error) => {
                console.log(error);
                alert("This incident could not be deleted. Please try again");
            })
        },

        goLocation(query_address){
            console.log(query_address);
            //alert("This does not work, bold to you assume im that smart >:(");
            if (query_address == null){
                //empty
                alert("No search entry, please enter in a location to search");
            } else {
                let url= "http://nominatim.openstreetmap.org/search?street="+query_address+"&city=ST.+PAUL&state=MINNESOTA&format=json";

                console.log(url);
                
                this.getJSON(url)
                .then((data) => {
                    //console.log(data);
                    
                    console.log(data[0].lat);
                    
                    //this.leaflet.map.panTo(this.leaflet.map, data[0].lat, data[0].lon);
                    var greenIcon = L.icon({
                        iconUrl: '../images/green_marker.png',
                        shadowUrl: '../images/marker-shadow.png',

                        iconSize:     [30, 45], // size of the icon
                        shadowSize:   [50, 64], // size of the shadow
                        iconAnchor:   [5, 40], // point of the icon which will correspond to marker's location
                        shadowAnchor: [4, 62],  // the same for the shadow
                        popupAnchor:  [10, -35] // point from which the popup should open relative to the iconAnchor
                    });

                    this.leaflet.map.flyTo([data[0].lat, data[0].lon], 18);
                    var loc_marker = L.marker([data[0].lat, data[0].lon], {icon: greenIcon});
                    loc_marker.addTo(this.leaflet.map);
                    loc_marker.bindPopup("Location: " + query_address + "<br/>"+
                    "<button id=\"marker_delete\" class=\"cell small-3 button\" type=\"button\"> Delete Marker</button>"
                    +"<br/> <button id=\"marker_center\" class=\"cell small-3 button\" type=\"button\"> Back to Center</button>");
                    loc_marker.on("popupopen", () => {
                                document.querySelector("#marker_delete")
                                .addEventListener("click", e => {
                                    this.leaflet.map.removeLayer(loc_marker);
                                    //center lat: 44.955139,
                                    //center lng: -93.102222,
                                    this.leaflet.map.flyTo([44.955139, -93.102222], 12);
                                });
                                document.querySelector("#marker_center")
                                .addEventListener("click", e => {
                                    loc_marker.closePopup();
                                    this.leaflet.map.flyTo([44.955139, -93.102222], 12);
                                });
                            })
                    //loc_marker._icon.classList.add("huechange");
                    loc_marker.valueOf()._icon.style.color = 'green'
                }).catch((err) => {
                    console.log(err);
                    alert("Location not in St. Paul, Minnesota, please try again.");
                })
            }
        }, 

        createMarker(block, index){
            let address = block.split(" ");
            let search_address = [];
            if (address[0].includes("X")){
                let add_number = address[0].replace(/X/g, '0');
                console.log(add_number);
                search_address.push(add_number);
            } else {
                search_address.push(address[0]);
            }
            let i;
            for (i=1; i < address.length; i++){
                if (address[i].includes("AND")){
                    search_address.push("&");
                } else {
                    search_address.push(address[i]);
                }
            }
            console.log(search_address);
            
            //https://nominatim.openstreetmap.org/search?street=WARNER+RD+&+SIBLEY&city=ST.+PAUL&format=json
            let query_address = "";
            for (i=0; i < search_address.length; i++){
                query_address = query_address + search_address[i];
                if (i != search_address.length - 1) {
                    query_address = query_address + "+";
                }
            }
            console.log(query_address);
            let url = "http://nominatim.openstreetmap.org/search?street="+query_address+"&city=ST.+PAUL&format=json";

            this.getJSON(url)
            .then((data) => {
                console.log(data);
                for (i=0; i< data.length; i++){
                    console.log(data[i].lat);
                    console.log(data[i].lon);
                    /*
                    nw: {lat: 45.008206, lng: -93.217977},
                    se: {lat: 44.883658, lng: -92.993787}
                    */
                    if (data[i].lat > this.leaflet.bounds.se.lat && data[i].lat < this.leaflet.bounds.nw.lat){
                        if (data[i].lon < this.leaflet.bounds.se.lng && data[i].lon > this.leaflet.bounds.nw.lng){
                            console.log("Data is: ");
                            console.log(data[i]);
                            //create marker with latitude and longitude here
                            var marker = L.marker([data[i].lat, data[i].lon]).addTo(this.leaflet.map);
                            alert("Added chosen row as marker to the map");
                            //add that sexy sexy popup ohhh yeeeeehhh
                            marker.bindPopup("Date: " + this.incidents[index].date + "<br/>" + "Time: " + this.incidents[index].time + "<br/>" 
                            + "Incident: " + this.incidents[index].incident + "<br/>" + "<button id=\"marker_delete\" class=\"cell small-3 button\" type=\"button\"> Delete Marker</button>"
                            +"<button id=\"marker_center\" class=\"cell small-3 button\" type=\"button\"> Back to Center</button>");
                            marker._icon.classList.add("huechange");
                            this.leaflet.map.flyTo([data[i].lat, data[i].lon], 14);
                            marker.on("popupopen", () => {
                                document.querySelector("#marker_delete")
                                .addEventListener("click", e => {
                                    this.leaflet.map.removeLayer(marker);
                                    this.leaflet.map.flyTo([44.955139, -93.102222], 12);
                                });
                                document.querySelector("#marker_center")
                                .addEventListener("click", e => {
                                    marker.closePopup();
                                    this.leaflet.map.flyTo([44.955139, -93.102222], 12);
                                });
                            })
                            break;
                        }
                    }
                }
            })

            
        },

        center() {
            document.querySelector("#center")
            .addEventListener("click", e => {
                this.leaflet.map.flyTo([44.955139, -93.102222], 12);
            });
        }
    },
    mounted() {

        this.leaflet.map = L.map('leafletmap', {zoomAnimation: false}).setView([this.leaflet.center.lat, this.leaflet.center.lng], this.leaflet.zoom);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
            minZoom: 11,
            maxZoom: 18
        }).addTo(this.leaflet.map);
        this.leaflet.map.setMaxBounds([[44.883658, -93.217977], [45.008206, -92.993787]]);
        /*
        this.leaflet.map.on('zoom', function(ev) {
            console.log(ev.target.getZoom());
            console.log(ev.target.getBounds());
        });
        this.leaflet.map.on('click', function(ev) {
            console.log(ev.latlng); // ev is an event object (MouseEvent in this case)
        });
        
        console.log(this.leaflet.bounds.nw.lat);
        console.log(this.leaflet.bounds.nw.lng); */

        for(var i = 0; i < this.leaflet.neighborhood_markers.length; i++) {
            this.leaflet.neighborhood_markers[i].marker = L.marker([this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]])
                .bindPopup(this.leaflet.neighborhood_markers[i].name)
                .addTo(this.leaflet.map);
            
        }

        let district_boundary = new L.geoJson();
        district_boundary.addTo(this.leaflet.map);
        //console.log(district_boundary);

        this.getJSON('/data/StPaulDistrictCouncil.geojson').then((result) => {
            // St. Paul GeoJSON
            $(result.features).each((key, value) => {
                district_boundary.addData(value);
            });
        }).catch((error) => {
            console.log('Error:', error);
        });


        this.getJSON('http://localhost:8005/codes').then((data) => {
            this.codes = data;
        }).catch((error) => {
            console.log('Error:', error);
        })

        this.getJSON('http://localhost:8005/neighborhoods').then((data) => {            
            this.neighborhoods = data;
        }).catch((error) => {
            console.log('Error:', error);
        })

        this.getJSON('http://localhost:8005/incidents').then((data) => {
            this.incidents = data;
            //loop over incidents to count number of crimes with this.incidents.neighborhood_number, \
            //17 counters to ocunt for each neighborhood
            let i;
            let neighborhood_array = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
            for (i=0; i < this.incidents.length; i++){
                //check neighborhood_number
                neighborhood_array[this.incidents[i].neighborhood_number - 1] ++;
            }
            //console.log(neighborhood_array);
            for (i=0; i < 17; i++){
                this.leaflet.neighborhood_markers[i].marker.setPopupContent(this.leaflet.neighborhood_markers[i].name + "<br/>" + neighborhood_array[i]);
            }
            //also do this when implementing other UI features since this only loads for first stuff
        }).catch((error) => {
            console.log('Error:', error);
        })

    },
       
} 
</script>

<template>
    <div class="grid-container">
        <div class="grid-x grid-padding-x">
            <p :class="'cell small-4 ' + ((view === 'map') ? 'selected' : 'unselected')" @click="viewMap">Map</p>
            <p :class="'cell small-4 ' + ((view === 'new_incident') ? 'selected' : 'unselected')" @click="viewNewIncident">New Incident</p>
            <p :class="'cell small-4 ' + ((view === 'about') ? 'selected' : 'unselected')" @click="viewAbout">About</p>
        </div>
    </div>
    <div v-show="view === 'map'">
   
        <div class="grid-container">
            <div class="grid-x grid-padding-x"> 
                <div class="cell large-12">
                    <form>
                        <h1>Filters</h1>
                        <!-- Incident Type check boxes  class="cell large-12" -->
                        <div style="width: 100%;">
                            <div style="width: 50%; height: 35rem; float: left;">
                                <h4>Incident Type</h4>
                                <ul>
                                        <li><input type="checkbox" id="Homicide" name="Homicide" value="Homicide" v-model="filter.incident_type[0]">
                                            <label for="Homicide">Homicide</label><br></li>
                                        <li><input type="checkbox" id="Rape" name="Rape" value="Rape" v-model="filter.incident_type[1]">
                                            <label for="Rape">Rape</label><br></li>  
                                        <li><input type="checkbox" id="Robbery" name="Robbery" value="Robbery" v-model="filter.incident_type[2]">
                                            <label for="Robbery">Robbery</label><br></li>                                                                                            
                                        <li><input type="checkbox" id="Agg_Assault" name="Agg_Assault" value="Agg_Assault" v-model="filter.incident_type[3]">
                                            <label for="Agg_Assault">Agg. Assault</label><br></li>                         
                                        <li><input type="checkbox" id="Burglary" name="Burglary" value="Burglary" v-model="filter.incident_type[4]">
                                            <label for="Burglary">Burglary</label><br></li>       
                                        <li><input type="checkbox" id="Theft" name="Theft" value="Theft" v-model="filter.incident_type[5]">
                                            <label for="Theft">Theft</label><br></li>
                                        <li><input type="checkbox" id="Auto Theft" name="Auto Theft" value="Auto Theft" v-model="filter.incident_type[6]">
                                            <label for="Auto Theft">Auto Theft</label><br></li>
                                        <li><input type="checkbox" id="Simple Assault Dom" name="Simple Assault Dom" value="Simple Assault Dom" v-model="filter.incident_type[7]">
                                            <label for="Simple Assault Dom">Simple Assault Dom</label><br></li>
                                        <li><input type="checkbox" id="Arson" name="Arson" value="Arson" v-model="filter.incident_type[8]">
                                            <label for="Arson">Arson</label><br></li>
                                        <li><input type="checkbox" id="Criminal Damage" name="Criminal Damage" value="Criminal Damage" v-model="filter.incident_type[13]">
                                            <label for="Criminal Damage">Criminal Damage</label><br></li>
                                        <li><input type="checkbox" id="Narcotics" name="Narcotics" value="Narcotics" v-model="filter.incident_type[17]">
                                            <label for="Narcotics">Narcotics</label><br></li>
                                        <li><input type="checkbox" id="Discharge" name="Discharge" value="Discharge" v-model="filter.incident_type[25]">
                                            <label for="Discharge">Discharge</label><br></li>
                                        <li><input type="checkbox" id="Death-Investigation" name="Death-Investigation" value="Death-Investigation" v-model="filter.incident_type[30]">
                                            <label for="Death-Investigation">Death-Investigation</label><br></li>
                                        <li><input type="checkbox" id="Proactive Police Visit" name="Proactive Police Visit" value="Proactive Police Visit" v-model="filter.incident_type[98]">
                                            <label for="Proactive Police Visit">Proactive Police Visit</label><br></li>
                                </ul>
                            </div>
                        <!-- Neighborhood Name check boxes -->
                            <div style="margin-left: 50%; height: 37rem;">
                                <h4>Neighborhoods</h4>
                                <ul>
                                    <li v-for="(item, index) in neighborhoods">
                                        <input type="checkbox" id={{item.name}} name={{item.name}} value={{item.name}} v-model="filter.neighborhood_number[index]">
                                        <label for="{{item.name}}">{{item.name}}</label><br>
                                    </li>
                                </ul>
                            </div>
                        </div>

                        <div style="width: 100%;">
                        <!-- Date Range - start date and end date -->
                            <h4>Date Range</h4>
                            <p>Start Date: </p><input type="date" v-model="filter.startDate">
                            <p>End Date: </p><input type="date" v-model="filter.endDate">
                        </div>
                        <!-- Max Incidents limit -->
                        <div style="width: 100%;">
                            <h4>Max Incidents</h4>
                            <input type="text" v-model="filter.limit">

                                <!-- <span> Filter has: {{ filter }}</span> -->
                            <button id="update" class="cell small-1 button" type="button" @click="filterIncident(filter)">Update</button>
                            <button id="reset" class="cell small-1 button" type="button" @click="reset()">Reset</button>
                        </div>
                    </form>
                </div>
                <div id="leafletmap" class="cell auto"></div>
                <div class="cell small-12 medium-12 large-12">
                    <br/>
                    <button id="center" class="cell small-3 button" type="button" @click="center()"> Center Map </button>
                </div>
                <div class="cell small-12 medium-6 large-4"></div>
                <div class="cell small-12 medium-6 large-4">
                    <legend class="center"> Colors </legend>
                        <center>
                        <label for="violent" class="center">Violent Crimes:
                        <span style="width: 15px; height: 15px; margin:auto; display: inline-block; border: 1px solid gray; vertical-align: middle; border-radius: 2px; background: rgb(255, 204, 204) "></span>
                        </label>
                        <label for="property" class="center">Property Crimes:
                        <span style="width: 15px; height: 15px; margin:auto; display: inline-block; border: 1px solid gray; vertical-align: middle; border-radius: 2px; background: rgb(188, 245, 189) "></span>
                        </label>
                        <label for="other">Other Crimes:
                        <span style="width: 15px; height: 15px; margin:auto; display: inline-block; border: 1px solid gray; vertical-align: middle; border-radius: 2px; background: rgb(181, 221, 249) "></span>
                        </label>
                        </center>
                </div>
                <div class="cell small-12 medium-6 large-4"></div>
                <div class="cell small-12 medium-12 large-12">
                    <!-- KEY for colors -->
                    <form>
                        <p>Enter in location</p>
                        <input type="text" id="go" placeholder="Example: University of St. Thomas" v-model="searchbar.address"><br/>
                        <button id="go" class="cell small-1 button" type="button" @click="goLocation(searchbar.address)">Go</button>
                    </form>
                </div>
                <table class="center">
                    <thead>
                        <tr>
                            <th>#</th>
                            <th>Case Number</th>
                            <th>Date</th>
                            <th>Time</th>
                            <th>Incident</th>
                            <th>Incident in Detail</th>
                            <th>Police Grid</th>
                            <th>Neighborhood</th>
                            <th>Block</th>
                            <th>Delete?</th>
                            <th>Show on Map</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="(itemIncidents, index) in incidents" :class="
                            {'violent': 
                            itemIncidents.incident === 'Agg. Assault'||
                            itemIncidents.incident === 'Agg. Assault Dom' ||
                            itemIncidents.incident === 'Agg. Assault Dom.' ||
                            itemIncidents.incident === 'Arson' ||
                            itemIncidents.incident === 'Criminal Discharge' ||
                            itemIncidents.incident === 'Discharge' ||
                            getIncidentType(itemIncidents.code).includes('Weapons') ||
                            itemIncidents.incident === 'Homicide' ||
                            itemIncidents.incident === 'Rape' ||
                            itemIncidents.incident === 'Simple Assault Dom' ||
                            itemIncidents.incident === 'Simple Asasult Dom.' ||
                            itemIncidents.incident === 'Simple Assault Dom.' ||
                            getIncidentType(itemIncidents.code).includes('Assault') ||
                            getIncidentType(itemIncidents.code).includes('Murder')||
                            itemIncidents.incident === 'Robbery' ||
                            getIncidentType(itemIncidents.code).includes('Robbery') ||
                            getIncidentType(itemIncidents.code).includes('Rape'),

                            'property': 
                            itemIncidents.incident === 'Auto Theft' ||
                            getIncidentType(itemIncidents.code).includes('Theft') ||
                            itemIncidents.incident === 'Burglary' ||
                            getIncidentType(itemIncidents.code).includes('Burglary')||
                            itemIncidents.incident === 'Criminal Damage' ||
                            getIncidentType(itemIncidents.code).includes('Criminal Damage')||
                            getIncidentType(itemIncidents.code).includes('Graffiti')||
                            itemIncidents.incident === 'Graffiti' ||
                            itemIncidents.incident === 'Theft' ||
                            itemIncidents.incident === 'Vandalism',

                            'other': 
                            itemIncidents.incident === 'Community Engagement Event' ||
                            itemIncidents.incident === 'Community Event' ||
                            itemIncidents.incident === 'Narcotics' ||
                            getIncidentType(itemIncidents.code).includes('Narcotics') ||
                            itemIncidents.incident === 'Others' ||
                            itemIncidents.incident === 'Proactive Foot Patrol' ||
                            itemIncidents.incident === 'Proactive Police Visit' 
                            }">
                            
                            <td>{{ index + 1 }}</td>
                            <td>{{ itemIncidents.case_number }}</td>
                            <td>{{ itemIncidents.date }}</td>
                            <td>{{ itemIncidents.time }}</td>
                            <td>{{ itemIncidents.incident }}</td>
                            <td>{{ getIncidentType(itemIncidents.code) }}</td>
                            <td>{{ itemIncidents.police_grid }}</td>
                            <td>{{ neighborhoods[itemIncidents.neighborhood_number - 1].name }}</td>
                            <td>{{ itemIncidents.block }}</td>
                            <td>
                                <button id="lookup" class="cell small-3 button" type="button" @click="removeIncident(index)"> Delete </button>
                            </td>
                            <td>
                                <button id="lookup" class="cell small-3 button" type="button" @click="createMarker(itemIncidents.block, index)"> Select </button>
                            </td>
                            </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
     <div class="grid-container">
        <div class="grid-x grid-padding-x">
        
        </div>
    </div>
    <div v-if="view === 'new_incident'">
        <!-- Replace this with your actual form: can be done here or by making a new component
        getIncidentType(itemIncidents.code)
        neighborhoods[itemIncidents.neighborhood_number - 1].name
        -->
        <div class="grid-container">
            <div class="grid-x grid-padding-x">
                <h1 class="cell auto">New Incident Form</h1>
            </div>    
                <div class="cell small-12 large-12">
                    <form id="form" method="PUT" action="/new-incident" @submit.prevent="sendMessage">
                        <span>Case Number:</span><br/>
                        <input id="case_number" type="text" placeholder="Example: 12345678" v-model="new_incident.case_number" required/>
                        <label for="case_number"></label>
                        <br/>

                        <span>Date:</span><br/>
                        <input id="date" type="text" placeholder="Example: 2019-04-26" v-model="new_incident.date" required/>
                        <label for="date"></label>
                        <br/>
                        
                        <span>Time</span><br/>
                        <input id="time" type="text" placeholder="Example: 19:15:00" v-model="new_incident.time" required/>
                        <br/>
                        <label for="time"></label>

                        <span>Code</span><br/>
                        <input id="code" type="text" placeholder="Example: 600" v-model="new_incident.code" required/>
                        <br/>

                        <span>Incident</span><br/>
                        <input id="incident" type="text" placeholder="Example: Theft" v-model="new_incident.incident" required/>
                        <br/>

                        <span>Police Grid</span><br/>
                        <input id="police_grid" type="text" placeholder="Example: 49" v-model="new_incident.police_grid" required/>
                        <br/>

                        <span>Neighborhood Number</span><br/>
                        <input id="neighborhood_number" type="text" placeholder="Example: 1" v-model="new_incident.neighborhood_number" required/>
                        <br/>

                        <span>Block</span><br/>
                        <input id="block" type="text" placeholder="Example: 212 OLD HUDSON RD" v-model="new_incident.block" required/>
                        <br/>
                        <button id="lookup" class="cell small-3 button" type="button" @click="newIncident">Submit</button>
                    </form>
                </div>
            
        </div>
    </div>

    <div v-if="view === 'about'">
        <!-- Replace this with your actual about the project content: can be done here or by making a new component -->
        <div class="grid-container">
            <div class="grid-x grid-padding-x">
                <h1 class="cell auto">About the Project</h1>
                <div class="cell small-12 large-12">
                    <h2 class="cell auto">About Us</h2>
                </div>
                <div class="cell small-12 medium-12 large-8">
                    <h3> Alina Kanayinkal </h3>
                </div>
                <div class="cell small-12 medium-12 large-6">
                    <h5> 
                        I am a third year student at the University of St. Thomas.
                        I am majoring in Computer Science and minoring in Applied
                        Statistics. 
                    </h5>
                </div>
                <div class="cell small-12 medium-12 large-2"></div>
                <!-- add image -->
                <div class="cell small-12 medium-12 large-4">
                    <img src="../images/alina_prof_pic.jpg" alt="Alina Photo"/>
                </div>
                
                
                <div class="cell small-12 medium-12 large-8">
                    <h3> Ivan Jimenez </h3>
                </div>
                <div class="cell small-12 medium-12 large-8">
                    <h5> 
                        I am a fourth year student at the University of St. Thomas.
                        I am majoring in Applied Mathematics and minoring in Computer 
                        Science.
                    </h5>
                </div>
                <!-- add image -->
                <div class="cell small-12 medium-12 large-4">
                    <img src="../images/ivan_prof_pic.jpeg" alt="Ivan Photo"/>
                </div>

                <div class="cell small-12 medium-12 large-8">
                    <h3> Maddie Moeller </h3>
                </div>
                <div class="cell small-12 medium-12 large-8">
                    <h5> 
                        I am a third year student at the University of St. Thomas.
                        I am majoring in Computer Science and minoring in Mathematics.
                    </h5>
                </div>
                <!-- add image -->
                <div class="cell small-12 medium-12 large-4">
                    <img src="../images/maddie_prof_pic.jpg" alt="Maddie Photo"/>
                </div>

                <div class="cell small-12 large-12">
                    <h2 class="cell auto">Tools Used</h2>
                </div>
                <div class="cell small-12 medium-12 large-4">
                    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/95/Vue.js_Logo_2.svg/1184px-Vue.js_Logo_2.svg.png" alt="Vue Logo"/>
                </div>
                <div class="cell small-12 medium-12 large-8">
                    <h3> Vue </h3>
                    <p>"Vue is a JavaScript framework for building user interfaces. 
                    It builds on top of standard HTML, CSS, and JavaScript and provides a declarative and 
                    component-based programming model that helps you efficiently develop user interfaces, 
                    be they simple or complex."</p>
                    <a href="https://vuejs.org/">More Info</a>

                </div>

                <div class="cell small-12 medium-12 large-4">
                    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/1/13/Leaflet_logo.svg/800px-Leaflet_logo.svg.png.png" alt="Leaflet Logo"/>
                </div>
                <div class="cell small-12 medium-12 large-8">
                    <h3> Leaflet </h3>
                    <p>"Leaflet is the leading open-source JavaScript library for mobile-friendly interactive maps. 
                    Weighing just about 42 KB of JS, it has all the mapping features most developers ever need. 
                    Leaflet is designed with simplicity, performance and usability in mind. It works efficiently across 
                    all major desktop and mobile platforms, can be extended with lots of plugins, has a beautiful, easy to 
                    use and well-documented API and a simple, readable source code that is a joy to contribute to."</p>
                    <a href="https://leafletjs.com/">More Info</a>
                </div>

                <div class="cell small-12 medium-12 large-4">
                    <img src="https://raw.githubusercontent.com/juancarlospaco/nim-overpass/master/osm.jpg" alt="Nominatim Logo"/>
                </div>
                <div class="cell small-12 medium-12 large-8">
                    <h3> Nominatim </h3>
                    <p>"Nominatim uses OpenStreetMap data to find locations on Earth by name
                     and address (geocoding). It can also do the reverse, find an address 
                     for any location on the planet."</p>
                     <a href="https://nominatim.org/">More Info</a>
                </div>

                <div class="cell small-12 medium-12 large-4">
                    <img src="https://www.arcgis.com/sharing/rest/content/items/1e2bdd73bc384d70aea7a337ba0cd3ce/resources/stpaul-logo-horizontal-web-only.png?v=1671424003839" alt="St Paul Logo"/>
                </div>
                <div class="cell small-12 medium-12 large-8">
                    <h3> St. Paul Crime API </h3>
                    <p>"Incidents from Aug 14 2014 through the most recent available in the City of Saint Paul.
                     The data is released by the Saint Paul Police Department every 2 to 3 weeks and includes the 
                     following categories: Homicide, Rape, Robbery, Aggravated Assault, Burglary, Theft, Auto Theft, 
                     Arson, Domestic Assaults, Vandalism, Narcotics, and Firearm Discharges. 
                     Statistics displayed do not reflect official crime index totals, and may change after full investigation"</p>
                     <a href="https://information.stpaul.gov/datasets/stpaul::crime-incident-report/about">More Info</a>
                </div>
                
                
                <div class="cell small-12 large-12">
                <br>
                <br>
                    <h2 class="cell auto">6 Interesting Findings</h2>
                    <ol>
                        <li>Incidents were misspelled like 'Asasult' which threw off trying to color incident by types </li>
                        <li>Some of the incidents didn't have incident attached, they were only categorized by code</li>
                        <li>The Capitol River neighborhood has the most incidents at 107 with the number
                        of incidents generally geting smaller the further out of the city one gets</li>
                        <li>Generally more violent crimes take place very early in the morning</li>
                        <li>Proactive Police Visit is the most common incident type</li>
                        <li>There is only one Investigation of a Death recorded in the dataset</li>
                    </ol>
                </div>

                <div class="cell small-12 large-12">
                <br>
                <br>
                    <h2 class="cell auto">Video Demo</h2>
                </div>
                <iframe width="1425" height="620" src="https://www.youtube.com/embed/XzJs1ssa0cQ" title="stinky presentation" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
            </div>
        </div>
    </div>
</template>

<style>
#leafletmap {
    height: 500px;
}

.selected {
    background-color: rgb(10, 100, 126);
    color: white;
    border: solid 1px white;
    text-align: center;
    cursor: pointer;
}
.unselected {
    background-color: rgb(200, 200, 200);
    color: black;
    border: solid 1px white;
    text-align: center;
    cursor: pointer;

    
}

th, td {
    border: solid, 1px, black;
}

ul {
    list-style: none;
}

.violent{
    background-color: rgb(255, 204, 204);
}

.property {
    background-color: rgb(188, 245, 189);
}

.other {
    background-color: rgb(181, 221, 249);
}

.center {
  margin-left: auto;
  margin-right: auto;
}

</style>
