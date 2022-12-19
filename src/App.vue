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
            for (i in this.new_incident){
                for (j in this.new_incident[i]){
                    //console.log(this.new_incident[i][j]);
                    if (this.new_incident[i][j] != null){
                        counter++;
                    }
                }
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
                }

            }
            //Limit
            if(filterData.limit != null) {
                if(count > 0) {
                    url = url + "&";
                }
                url = url + "limit=" + filterData.limit;
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

        crimeFinder(incident){
            //console.log(incident);
            if (incident.toLowerCase() === 'theft' || 'burglary'){
                this.class = "violent";
                //console.log(this.class);
                return this.class;

            } else if (incident.toLowerCase() === 'agg. assault') {
                this.class = "property";
                //console.log(this.class);
                return this.class;

            }
        }
    },
    mounted() {

        this.leaflet.map = L.map('leafletmap').setView([this.leaflet.center.lat, this.leaflet.center.lng], this.leaflet.zoom);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
            minZoom: 11,
            maxZoom: 18
        }).addTo(this.leaflet.map);
        this.leaflet.map.setMaxBounds([[44.883658, -93.217977], [45.008206, -92.993787]]);


         for(var i = 0; i < this.leaflet.neighborhood_markers.length; i++) {
            this.leaflet.neighborhood_markers[i].marker = L.marker([this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]])
                .bindPopup(this.leaflet.neighborhood_markers[i].name)
                .addTo(this.leaflet.map);
            
        }


        let district_boundary = new L.geoJson();
        district_boundary.addTo(this.leaflet.map);

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
            console.log(neighborhood_array);
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
                    <!-- Incident Type check boxes -->
                    <h4>Incident Type</h4>
                     <ul>
                            <li><input type="checkbox" id="Homicide" name="Homicide" value="Homicide" v-model="filter.incident_type[0]">
                                <label for="Homicide">Homicide</label><br></li>
                            <li><input type="checkbox" id="Murder" name="Murder" value="Murder" v-model="filter.incident_type[1]">
                                <label for="Murder">Murder</label><br></li> 
                            <li><input type="checkbox" id="Rape" name="Rape" value="Rape" v-model="filter.incident_type[2]">
                                <label for="Rape">Rape</label><br></li>  
                            <li><input type="checkbox" id="Robbery" name="Robbery" value="Robbery" v-model="filter.incident_type[3]">
                                <label for="Robbery">Robbery</label><br></li>                                                                                            
                            <li><input type="checkbox" id="Agg_Assault" name="Agg_Assault" value="Agg_Assault" v-model="filter.incident_type[4]">
                                <label for="Agg_Assault">Agg. Assault</label><br></li>                         
                            <li><input type="checkbox" id="Burglary" name="Burglary" value="Burglary" v-model="filter.incident_type[5]">
                                <label for="Burglary">Burglary</label><br></li>       
                            <li><input type="checkbox" id="Theft" name="Theft" value="Theft" v-model="filter.incident_type[6]">
                                <label for="Theft">Theft</label><br></li>
                            <li><input type="checkbox" id="Auto Theft" name="Auto Theft" value="Auto Theft" v-model="filter.incident_type[7]">
                                <label for="Auto Theft">Auto Theft</label><br></li>
                            <li><input type="checkbox" id="Simple Assault Dom" name="Simple Assault Dom" value="Simple Assault Dom" v-model="filter.incident_type[8]">
                                <label for="Simple Assault Dom">Simple Assault Dom</label><br></li>
                            <li><input type="checkbox" id="Arson" name="Arson" value="Arson" v-model="filter.incident_type[9]">
                                <label for="Arson">Arson</label><br></li>
                            <li><input type="checkbox" id="Criminal Damage" name="Criminal Damage" value="Criminal Damage" v-model="filter.incident_type[10]">
                                <label for="Criminal Damage">Criminal Damage</label><br></li>
                            <li><input type="checkbox" id="Narcotics" name="Narcotics" value="Narcotics" v-model="filter.incident_type[11]">
                                <label for="Narcotics">Narcotics</label><br></li>
                            <li><input type="checkbox" id="Discharge" name="Discharge" value="Discharge" v-model="filter.incident_type[12]">
                                <label for="Discharge">Discharge</label><br></li>
                            <li><input type="checkbox" id="Death-Investigation" name="Death-Investigation" value="Death-Investigation" v-model="filter.incident_type[13]">
                                <label for="Death-Investigation">Death-Investigation</label><br></li>
                            <li><input type="checkbox" id="Proactive Police Visit" name="Proactive Police Visit" value="Proactive Police Visit" v-model="filter.incident_type[14]">
                                <label for="Proactive Police Visit">Proactive Police Visit</label><br></li>
                    </ul>

                    <!-- Neighborhood Name check boxes -->
                    <h4>Neighborhoods</h4>
                        <ul>
                            <li v-for="(item, index) in neighborhoods">
                                <input type="checkbox" id={{item.name}} name={{item.name}} value={{item.name}} v-model="filter.neighborhood_number[index]">
                                <label for="{{item.name}}">{{item.name}}</label><br>
                        </li>
                        </ul>
                        

                    <!-- Date Range - start date and end date -->
                    <h4>Date Range</h4>
                    <p>Start Date: </p><input type="date" v-model="filter.startDate">
                    <p>End Date: </p><input type="date" v-model="filter.endDate">

                    <!-- Max Incidents limit -->
                    <h4>Max Incidents</h4>
                    <input type="text" v-model="filter.limit">

                        <span> Filter has: {{ filter }}</span>
                    <button id="update" class="cell small-1 button" type="button" @click="filterIncident(filter)">Update</button>
                    <button id="reset" class="cell small-1 button" type="button" @click="reset()">Reset</button>
                </form>
                </div>
                <div id="leafletmap" class="cell auto"></div>
                <table>
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
        </tr>
        </thead>
        <!--
            Crimes:
                Violent = 
                Property = Vandalism, Theft, Auto Theft, 
                Other = Narcotics, Proactive Police Visit, 
                Total = 23 values

                {'violent': 
                 itemIncidents.incident === 'Theft'||
                 itemIncidents.incident === 'Burglary' ||
                 itemIncidents.incident === 'Auto Theft',
                 'property': itemIncidents.incident == 'Agg. Assault'
                 //'other': itemIncidents.incident == ''
                 
                }

                {'violent': 
                 itemIncidents.code < 300,
                 'property': itemIncidents.code > 299 && itemIncidents.code < 500
                 //'other': itemIncidents.incident == ''
                }
        -->
        <tbody>
            <tr v-for="(itemIncidents, index) in incidents" :class="
                {'violent': 
                 itemIncidents.incident === 'Theft'||
                 itemIncidents.incident === 'Burglary' ||
                 itemIncidents.incident === 'Auto Theft',
                 'property': itemIncidents.incident == 'Agg. Assault'
                 //'other': itemIncidents.incident == '' .name
                 
                }
                ">
                <td>{{ index + 1 }}</td>
                <td>{{ itemIncidents.case_number }}</td>
                <td>{{ itemIncidents.date }}</td>
                <td>{{ itemIncidents.time }}</td>
                <td >{{ itemIncidents.incident }}</td>
                <td>{{ getIncidentType(itemIncidents.code) }}</td>
                <td>{{ itemIncidents.police_grid }}</td>
                <td>{{ neighborhoods[itemIncidents.neighborhood_number - 1] }}</td>
                <td>{{ itemIncidents.block }}</td>
                <td>
                    <button id="lookup" class="cell small-3 button" type="button" @click="removeIncident(index)"> Delete </button>
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
                        <span> New incident has: {{ new_incident }}</span>

                        <button id="lookup" class="cell small-3 button" type="button" @click="newIncident">Submit</button>

                    <!--
                        <table>
                            <tr>
                                <td>Case Number:</td> 
                                <td><input id="case_number" type="text" placeholder="case number" name="case_number" required></td>
                            </tr>
                            <tr>
                                <td>Date:</td>
                                <td><input id="date" type="date" placeholder="" name="date" required></td>
                            </tr>
                            <tr>
                                <td>Time:</td>
                                <td><input id="time" type="time" placeholder="" name="time" required></td>
                            </tr>
                            <tr>
                                <td>Code:</td>
                                <td><input id="code" type="text" placeholder="code" name="code" required></td>
                            </tr>
                            <tr>
                                <td>Incident:</td>
                                <td><input id="incident" type="text" placeholder="incident" name="incident" required></td>
                            </tr>
                            <tr>
                                <td>Police Grid:</td>
                                <td><input id="police_grid" type="text" placeholder="police grid" name="police_grid" required></td>
                            </tr>
                            <tr>
                                <td>Neighborhood:</td>
                                    <select>
                                        <option v-for="(item, index) in neighborhoods"> {{item.name}} </option>
                                    </select>
                            </tr>
                            <!- <tr v-for="(item, index) in neighborhoods">
                                <td>Neighborhood:</td>
                                <td>
                                    <select>
                                        <option>{{item.name}}</option>
                                    </select>
                                </td>
                                <!- <td id="neighbothood_number">{{item.id}}</td> --
                            </tr> --
                            <tr>
                                <td>Block:</td>
                                <td><input id="block" type="text" placeholder="block" name="block" required></td>
                            </tr>
                            
                        </table>
                        <button id="submit" type="button" value="Submit" onclick="uploadJSON()">Submit</button> --
                        
                        <button id="submit" type="submit" value="Submit" onclick="uploadJSON(PUT, http://localhost:8005/new-incident, data)">Submit</button> -->
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
                    <img src="images/alina_prof_pic.png" alt="Alina Photo"/>
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
                    <img src="images/my_profile_picture.png" alt="Ivan Photo"/>
                </div>

                <div class="cell small-12 medium-12 large-8">
                    <h3> Maddie Moeller </h3>
                </div>
                <div class="cell small-12 medium-12 large-8">
                    <h5> 
                        example text 
                    </h5>
                </div>
                <!-- add image -->
                <div class="cell small-12 medium-12 large-4">
                    <img src="images/maddie_prof_pic.jpg" alt="Maddie Photo"/>
                </div>

                <div class="cell small-12 large-12">
                    <h2 class="cell auto">Tools Used</h2>
                </div>
                <div class="cell small-12 medium-12 large-4">
                    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/95/Vue.js_Logo_2.svg/1184px-Vue.js_Logo_2.svg.png" alt="Vue Logo"/>
                </div>
                <div class="cell small-12 medium-12 large-8">
                    <h3> Vue </h3>
                </div>

                <div class="cell small-12 medium-12 large-4">
                    <img src="images/my_profile_picture.png" alt="Leaflet Logo"/>
                </div>
                <div class="cell small-12 medium-12 large-8">
                    <h3> Leaflet </h3>
                </div>

                <div class="cell small-12 medium-12 large-4">
                    <img src="images/my_profile_picture.png" alt="Nominatim Logo"/>
                </div>
                <div class="cell small-12 medium-12 large-8">
                    <h3> Nominatim </h3>
                </div>

                <div class="cell small-12 medium-12 large-4">
                    <img src="images/my_profile_picture.png" alt="Nominatim Logo"/>
                </div>
                <div class="cell small-12 medium-12 large-8">
                    <h3> St. Paul Crime API </h3>
                </div>

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
    background-color: rgb(162, 206, 235);
}

.property {
    background-color: rgb(142, 204, 143);
}

.other {
    background-color: rgb(227, 145, 145);
}

</style>
