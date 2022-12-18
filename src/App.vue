<script>
import $ from 'jquery'

export default {
    data() {
        return {
            view: 'map',
            codes: [],
            neighborhoods: [], 
            incidents: [],
            new_incident:{
                case_number: null,
                date: null,
                time: null,
                code: "",
                incident: null,
                police_grid: null,
                neighborhood_number: "",
                block: null
            },
            filter: {
                case_number: null,
                date: null,
                time: null,
                code: "",
                incident: null,
                police_grid: null,
                neighborhood_number: "",
                block: null
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
             while (code != this.codes[i].code) {
                i++;
            }
            return this.codes[i].type;
        },

        newIncident(event){
            console.log(event);
            let url = "http://localhost:8005/new-incident";
            this.uploadJSON('PUT', url, this.new_incident).then((data) => {
                console.log(data);
            }).catch((error) => {
                console.log(error);
                alert("This incident already exists. Please try again");
            })
        },

        filterIncident(event) {
            let url = "http://localhost:8000/incidents?";
            this.uploadJSON('GET', url, this.filter).then((data) => {
                console.log(data);
            }).catch((error) => {
                console.log(error);
            })
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
        </tr>
        </thead>
        <tbody>
            <tr v-for="(itemIncidents, index) in incidents" :class="(index % 2 === 0) ? 'even' : 'odd'">
                <td>{{ index + 1 }}</td>
                <td>{{ itemIncidents.case_number }}</td>
                <td>{{ itemIncidents.date }}</td>
                <td>{{ itemIncidents.time }}</td>
                <td>{{ itemIncidents.incident }}</td>
                <td>{{ getIncidentType(itemIncidents.code) }}</td>
                <td>{{ itemIncidents.police_grid }}</td>
                <td>{{ neighborhoods[itemIncidents.neighborhood_number - 1].name }}</td>
                <td>{{ itemIncidents.block }}</td>
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

                        <select>
                            <option v-for="(item, index) in neighborhoods"> {{item.name}} </option>
                        </select>

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
                <div class="cell small-12 medium-12 large-6">
                    <h3> Alina Kanayinkal </h3>
                </div>
                <!-- add image -->
                <div class="cell small-12 medium-12 large-4">
                    <img src="images/my_profile_picture.png" alt="Alina Photo"/>
                </div>
                
                <div class="cell small-12 medium-12 large-6">
                    <h3> Ivan Jimenez </h3>
                </div>
                <!-- add image -->
                <div class="cell small-12 medium-12 large-4">
                    <img src="images/my_profile_picture.png" alt="Ivan Photo"/>
                </div>

                <div class="cell small-12 medium-12 large-8">
                    <h3> Maddie Moeller </h3>
                </div>
                <!-- add image -->
                <div class="cell small-12 medium-12 large-4">
                    <img src="images/my_profile_picture.png" alt="Maddie Photo"/>
                </div>

                <div class="cell small-12 large-12">
                    <h2 class="cell auto">Tools Used</h2>
                </div>
                <div class="cell small-12 medium-12 large-6">
                    <h3> Vue </h3>
                </div>
                <div class="cell small-12 medium-12 large-6">
                    <h3> Leaflet </h3>
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
.even {
    width: 40rem;
    margin: 0;
    background-color: rgb(162, 206, 235);
}
.odd {
    width: 40rem;
    margin: 0;
    background-color: rgb(241, 241, 241);
}

th, td {
    border: solid, 1px, black;
}


</style>
