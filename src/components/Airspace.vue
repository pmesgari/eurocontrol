<template>
  <v-container fluid fill-height>
    <v-layout>
      <v-flex d-flex xs3 id="contour-tables">
          <contour-table v-on:getcoordinate="getCoordinates"></contour-table>
          <contour-version-table v-on:getcoordinate="getCoordinates"></contour-version-table>
      </v-flex>
      <v-flex d-flex xs6 style="height:650px">
          <div id="map"></div>
      </v-flex>
      <v-flex d-flex xs3>
        <v-flex>
          <contour-points-table v-on:requestDraw="drawAirspace" v-on:requestValidation="validateAirspace" :coordinates="coordinates"></contour-points-table>
        </v-flex>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
  /* eslint-disable */
  import axios from 'axios'
  import loadGoogleMapsAPI from 'load-google-maps-api'
  import ContourTable from './ContourTable'
  import ContourVersionTable from './ContourVersionTable'
  import ContourPointsTable from './ContourPointsTable'
  
  export default {
    name: 'airspace',
    components: {'contour-table': ContourTable, 'contour-points-table': ContourPointsTable, 'contour-version-table': ContourVersionTable},
    data () {
      return {
        contourId: '',

        coordinates: [],
        flightPlanCoordinates: [],

        googleMaps: null
      }
    },
    methods: {
      drawAirspace: function (args) {
        console.log('drawing')
        var map = new this.googleMaps.Map(document.getElementById('map'), {
          zoom: 8,
          center: {lat: 50.85, lng: 5.6833333},
          mapTypeId: 'terrain'
        })
        let filterArgs = args.map(a => {
          let tempObj = {}
          tempObj['lat'] = Number(a.lat)
          tempObj['lng'] = Number(a.lng)
          return tempObj
        })
        //   [{lat: -27.467, lng: 153.027}]
        this.flightPlanCoordinates = filterArgs
        
        var flightPath = new this.googleMaps.Polyline({
          path: this.flightPlanCoordinates,
          geodesic: true,
          strokeColor: '#FF0000',
          strokeOpacity: 1.0,
          strokeWeight: 2
        })

        flightPath.setMap(map)
      },
      validateAirspace: function (args) {
        console.log('validating airspace for: ' + this.contourId)
        console.log('Coordinates are: ')
        console.log(args)
        axios({
          method: 'post',
          url: 'http://localhost:8090/airspace/validate',
          auth: {
            username: 'EURO',
            password: 'EURO'
          },
          data: {
            contourId: this.contourId,
            contourPoints: ''
          }
        })
          .then(function (response) {
            console.log(response)
            
          }.bind(this))
          .catch(function (error) {
            console.log(error)
          })
      },
      getCoordinates: function (event) {
        this.contourId = event.srcElement.textContent
        console.log('getting coordinates for: ' + event.srcElement.textContent)
        axios({
          method: 'get',
          url: 'http://localhost:8090/airspace/points;contourId='+event.srcElement.textContent,
          auth: {
            username: 'EURO',
            password: 'EURO'
          }
        })
          .then(function (response) {
            console.log(response)
            this.coordinates = response.data
            
          }.bind(this))
          .catch(function (error) {
            console.log(error)
          })
      }
    },
    mounted () {
      loadGoogleMapsAPI({key: 'AIzaSyDZGJjCUrl_QAs7-syu-263iJOUv7PWDuE'}).then(function (response) {
        this.googleMaps = response
      }.bind(this)).catch((err) => {
        console.error(err)
      })
    }
  }
</script>

<style>
  #contour-tables {
    flex-direction: column;
  }
  td {
    cursor: pointer;
  }
  table.table tbody td, table.table tbody th {
    height: 28px !important;
    font-size: 12px;
    padding: 0 15px !important;
  }
</style>
