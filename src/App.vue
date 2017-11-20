<template>
  <div id="app">
    <navbar v-bind:report="report" v-bind:showToc="showToc"></navbar>
    <div class="container">
      <div id="mapid"></div>
      <button  type="button" class="btn btn-secondary btn-lg btn-block" v-on:click="createReport(geometry)">
        <span v-show="!loading">HENT RAPPORT</span>
        <i class="fa fa-refresh fa-spin" style="font-size:24px" v-show="loading"></i>
      </button>
      <a href="#" id="pdf" v-on:click="pdf" v-show="showToc">
        <i class="fa fa-file-pdf-o fa-2x bottom-right"></i>
      </a>
        <div id="report">
          <div class="jumbotron" v-bind:id="item.title" v-for="item in data" v-show="item.title !== 'TotalKom' && item.title !== 'TotalNiveau'">
            <h1>{{item.title.split('_').join(' ')}}</h1>
            <table class="table table-striped table-sm">
              <thead>
                <tr>
                  <td><strong>VARIABEL</strong></td>
                  <td><strong>VÆRDI</strong></td>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in item.data">  
                  <td v-if="item.variable === ''">Samlet</td>
                  <td v-else>{{item.variable}}</td>                  
                  <td>
                    <div v-for="(value,key) in item.values">
                      <div v-if="key === 'AntalNiveau' && value !== ''">
                        {{key}}: {{value}}
                      </div>
                      <div v-else-if="key === 'AntalKom' && value !== ''">
                        {{key}}: {{value}}
                      </div>                      
                      <div class="progress" v-else-if="key === 'PctNiveau' && value !== ''">
                        <div class="progress-bar bg-success" role="progressbar" v-bind:style="{width: value + '%'}" v-bind:aria-valuenow="value" aria-valuemin="0" aria-valuemax="100">Niveau: {{value}} %</div>
                      </div>
                      <div class="progress" v-else-if="key === 'PctKom' &&  value !== ''">
                        <div class="progress-bar bg-warning" role="progressbar" v-bind:style="{width: value + '%'}" v-bind:aria-valuenow="value" aria-valuemin="0" aria-valuemax="100">Kommune: {{value}} %</div>
                      </div>
                      <div v-else-if="key === 'TotalKom'">
                        {{key}}: {{value}}
                      </div>
                      <div v-else-if="key === 'TotalNiveau'">
                        {{key}}: {{value}}
                      </div>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
    </div>
  </div> 
</template>

<script>
import axios from 'axios'
import L from 'leaflet'
import Draw from 'leaflet-draw'
import Navbar from './components/Navbar.vue'
import jsPDF from 'jspdf'

export default {
  name: 'app',
  components: {
    'navbar': Navbar
  },
  data () {
    return {
      report: {},
      data: {},
      geometry: {},
      map: {},
      loading: false,
      showToc: false
    }
  },
  methods: {
    getData: function (geom) {
      this.loading = true
      let self = this;
      axios.post('http://bal0-lois:8080/api/datapakkeresultat?procedureID=3&procedureParametre=%7B%22UdStraekID%22%3A%221%22%2C%22LS2RapportID%22%3A%223%22%7D', geom)
        .then(function (response) {
          self.loading = false;
          self.showToc = true;
          self.report = response.data.Result;
          //console.log(self.report)
          self.data = self.beutifyResponse(self.report);
        })
        .catch(function (error) {
          self.loading = false;
          alert('Kunne ikke få fat i data');
          console.info(error);
        });
    },
    beutifyResponse: function(data) {

        let tables = [];
        
        data.forEach(function(element) {
            let title = element.SheetName;
            let table = element.Table[0];
            let dataArr = [];
            let excist = [];


            for (var key in table) {
                let variable = key.split('_').slice(1, key.split('_').length - 1).join(' ');
                let valueKey = key.split('_')[0] + key.split('_')[key.split('_').length-1];

                let datamodel = {
                    variable: '',
                    values: {
                        AntalNiveau:'',
                        AntalKom: '',
                        PctNiveau: '',
                        PctKom: ''
                    }
                }
                
                //check if variable already exicts
                if (excist.indexOf(variable) === -1) {
                    excist.push(variable);
                    datamodel.variable = variable;
                    datamodel.values[valueKey] = table[key];
                    dataArr.push(datamodel);
                } else {
                    dataArr.forEach(function(item){
                        for (var k in item) {
                            if (item[k] == variable) {
                                item.values[valueKey] = table[key];
                                break;
                            }
                        }
                    })
                }

            }

            let obj = {
                title: title,
                data: dataArr
            }
            tables.push(obj);
        }); 

      return tables;
    },
    createReport: function (geom) {
      //Check if polygon is drawn
      if (Object.keys(this.geometry).length === 0 && this.geometry.constructor === Object) {
        alert('Du mangler at tegne et område på kortet')
      } else {
        //remove existing report
        this.report = {};
        //request report with drawn 
        this.getData(geom);
      }
    },
    pdf: function() {
      var doc = new jsPDF();

     // We'll make our own renderer to skip this editor
      var specialElementHandlers = {
        '#editor': function(element, renderer){
          return true;
        },
        '.controls': function(element, renderer){
          return true;
        }
      };

      // All units are in the set measurement for the document
      // This can be changed to "pt" (points), "mm" (Default), "cm", "in"
      doc.fromHTML($('#report').get(0), 15, 15, {
        'width': 170, 
        'elementHandlers': specialElementHandlers
      });

      doc.save('lois_raport.pdf')

    }
  },
  mounted: function () {
    //wait for document ready
    this.$nextTick(function () {

      // baselayers
      const Esri_WorldImagery = L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', {
        attribution: 'Tiles &copy; Esri'
      });
      const Stamen_TonerLite = L.tileLayer('https://stamen-tiles-{s}.a.ssl.fastly.net/toner-lite/{z}/{x}/{y}.{ext}', {
        attribution: 'Map tiles by <a href="http://stamen.com">Stamen Design</a>, <a href="http://creativecommons.org/licenses/by/3.0">CC BY 3.0</a> &mdash; Map data &copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
        subdomains: 'abcd',
        minZoom: 0,
        maxZoom: 20,
        ext: 'png'
      });
      
      // create a map in the "map" div, set the view to a given place and zoom
      this.map = L.map('mapid', {
        center: [55.731267,12.363396],
        zoom: 13,
        layers: [Stamen_TonerLite]
      });

      const baseMaps = {
          "Gråskala": Stamen_TonerLite,
          "Luftfoto": Esri_WorldImagery
      };

      L.control.layers(baseMaps).addTo(this.map);

      // Initialize the FeatureGroup to store editable layers
      let drawnItems = new L.FeatureGroup();
      this.map.addLayer(drawnItems);

      // Initialize the draw control and pass it the FeatureGroup of editable layers
      let drawControl = new L.Control.Draw({
          draw: {
              polygon: true,
              marker: false,
              polyline: false,
              rectangle: false,
              circle: false,
              circlemarker: false
          },
          edit: {
              featureGroup: drawnItems
          }
      });
      this.map.addControl(drawControl);

      let self = this
      this.map.on('draw:created', function(e) {

        let type = e.layerType,
            layer = e.layer,
            crs = {
                "type": "name", 
                "properties": {
                    "name": "urn:ogc:def:crs:OGC:1.3:CRS84"
                }
            };

        // get geojson and store in model
        let shape = layer.toGeoJSON()
        self.geometry = shape;
        // add crs def to geojson
        self.geometry.crs = crs;
        drawnItems.removeLayer(layer);
        drawnItems.addLayer(layer);
      });

      this.map.on('draw:edited', function(e) {
      });

      this.map.on('draw:deleted', function() {
          // Update db to save latest changes.
          self.geometry = {};
      });

    })
  }
}
</script>

<style>
  @import "../node_modules/leaflet/dist/leaflet.css";
  @import "../node_modules/leaflet-draw/dist/leaflet.draw.css";
  @import "../node_modules/font-awesome/css/font-awesome.css";

  html, body, .btn {
    font-family: 'Quicksand', sans-serif;
  }


  #mapid {
    margin-top: 55px;
    height: 500px;
  }

  #report {
    padding-top: 30px;
  }

  .table td, .table th {
    vertical-align: middle;
  }

  .jumbotron {
    background: #f7f7f7;
  }

  .progress-bar {
    white-space: nowrap;
    height: 1.5rem;
    line-height: 1.5rem;
    vertical-align: middle;
    font-size: 0.85rem;
    color: black;
  }

  .bottom-right {
    position: fixed;
    bottom: 20px;
    right: 20px;
    color: gray;
  }
</style>
