<template>
  <div id="app">
    <navbar v-bind:data="data" v-bind:showToc="showToc"></navbar>
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
        <div class="jumbotron" v-bind:id="item.tablename" v-for="item in data" v-show="item.tablename !== 'TotalKom' && item.tablename !== 'TotalNiveau'">
          <h1>{{item.title}}</h1>
          <h5>{{item.desciption}}</h5>
          <p id="date">{{new Date(item.data[0].values.BeregnDatoKom).getDate() +'/'+ (Number(new Date(item.data[0].values.BeregnDatoKom).getMonth()) + 1) + '-' + new Date(item.data[0].values.BeregnDatoKom).getFullYear()}}</p>
          <div class="col-md-12">
            <table class="table table-striped table-sm table-hover">
              <thead>
                <tr class="row">
                  <td class="col-md-4"><strong>VARIABEL</strong></td>
                  <td class="col-md-8"><strong>VÆRDIER</strong></td>
                </tr>
              </thead>
              <tbody>
                <tr class="row" v-for="item in item.data">  
                  <td class="col-md-4" v-if="item.variable === ''">Total</td>
                  <td class="col-md-4" id="align" v-else>{{item.variable}}</td>                  
                  <td class="col-md-8">
                    <div v-for="(value,key) in item.values">
                      <div v-if="key === 'AntalNiveau' && value !== ''">
                        <div>Niveau: {{value}}</div>
                      </div>
                      <div v-else-if="key === 'AntalKom' && value !== ''">
                        <div>Kommune: {{value}}</div>
                      </div>                      
                      <div class="progress" v-else-if="key === 'PctNiveau' && value !== ''">
                        <div class="progress-bar bg-success" role="progressbar" v-bind:style="{width: value + '%'}" v-bind:aria-valuenow="value" aria-valuemin="0" aria-valuemax="100">Niveau: {{value}} %</div>
                      </div>
                      <div class="progress" v-else-if="key === 'PctKom' &&  value !== ''">
                        <div class="progress-bar bg-warning" role="progressbar" v-bind:style="{width: value + '%'}" v-bind:aria-valuenow="value" aria-valuemin="0" aria-valuemax="100">Kommune: {{value}} %</div>
                      </div>
                      <div v-else-if="key === 'TotalKom'">
                        <div>Kommune: {{value}}</div> 
                      </div>
                      <div v-else-if="key === 'TotalNiveau'">
                        <div>Niveau: {{value}}</div>
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
      axios.post('http://bal0-lois:8080/api/datapakkeresultat?procedureID=3&geoNavn=&procedureParametre={"UdStraekID":"1","LS2RapportID":"3"}&type=json', geom)
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
          let title = element.Meta.LangtNavn;
          let desciption = element.Meta.Beskrivelse;
          let tablename = element.Meta.SheetName;
          let table = element.Table[0];
          let dataArr = [];
          let excist = [];

          //iteratring through columnnames and adding key/value to datamodel
          for (var key in table) {
              let variable, valueKey;
              if (key.split(' ')[key.split(' ').length-1] == 'Niveau' ) {
                  // remove prefix and suffix (antal/pct niveau)
                  variable = key.split(' ').slice(1, key.split(' ').length - 1).join(' ');
                  // set value key dependent on antal/pct niveau
                  valueKey = key.split(' ')[0] + key.split(' ')[key.split(' ').length-1];
              } else {
                  variable = key.split(' ').slice(1, key.split(' ').length).join(' ');
                  valueKey = key.split(' ')[0] + 'Kom';
              }
                  
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
                  // setting valuekeys to value
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

          // collect the beautified data and push to array
          let obj = {
              title: title,
              tablename: tablename,
              desciption: desciption,
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
        'width': 190, 
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

      // CRS to append to geom obj
      let crs = {
          "type": "name", 
          "properties": {
              "name": "urn:ogc:def:crs:OGC:1.3:CRS84"
          }
      };

      this.map.on('draw:created', function(e) {

        let layer = e.layer
        
        //remove existing drawn objects
        drawnItems.clearLayers(layer)

        // get geojson and store in model and add crs
        self.geometry = layer.toGeoJSON();
        self.geometry.crs = crs;

        drawnItems.addLayer(layer);
      });

      this.map.on('draw:edited', function(e) {
        let layers = e.layers;
        layers.eachLayer(function (layer) {
          self.geometry = layer.toGeoJSON();
          self.geometry.crs = crs;
        });
      });

      this.map.on('draw:deleted', function() {
          // Update db to save latest changes.
          self.geometry = {};
          self.data = {};
          self.showToc = false;
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

  .table{
    margin-top: 3rem;
  }

  /* horrible css to virtically center aling text in td */
  #align {
    padding-top: 40px;
  }

  .jumbotron {
    background: #f7f7f7;
    position: relative;
  }

  #date {
    position: absolute;
    top: 24px;
    right: 24px;
    color:darkgrey;
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
