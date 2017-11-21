<template>
  <v-container fluid fill-height>
    <v-layout>
      <v-flex d-flex xs3 id="contour-tables">
          <!-- <contour-select v-on:getcoordinate="getCoordinates" v-on:clearCoordinates="clearCoordinates"></contour-select> -->
          <contour-table v-on:getcoordinate="getCoordinates" v-on:clearcoordinates="clearCoordinates"></contour-table>
          <contour-version-table v-on:getcoordinate="getCoordinates" :airspaceVersions="airspaceVersions"></contour-version-table>
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
    components: {
      'contour-table': ContourTable, 
      'contour-points-table': ContourPointsTable, 
      'contour-version-table': ContourVersionTable},
    data () {
      return {
        contourId: '',

        coordinates: [],
        airspaceVersions: [],

        flightPlanCoordinates: [],
        selectedContourIds: [],
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
        console.log('validating airspace for: ' + this.selectedContourIds)
        console.log('Coordinates are: ')
        console.log(args)
        axios({
          method: 'post',
          url: 'http://localhost:8090/airspace/validate',
          auth: {
            username: 'EURO',
            password: 'EURO'
          },
          data: args
        })
          .then(function (response) {
            console.log(response)
            this.updateVersions()
          }.bind(this))
          .catch(function (error) {
            console.log(error)
          })
      },
      updateVersions: function () {
        let promiseArr = this.selectedContourIds.map(sCid => {
          let axiosObj = axios({
            method: 'get',
            url: 'http://localhost:8090/airspace/version;contourId=' + sCid.contourId,
            auth: {
              username: 'EURO',
              password: 'EURO'
            }
          })
          return axiosObj
        })
        axios.all(promiseArr)
          .then(function (response) {
            let tempObj = []
            response.forEach(r => {
              tempObj.push(r.data)
            })
            tempObj.forEach(tObj => {
              tObj.forEach(record => {
                this.airspaceVersions.push(record)
              })
            })
          }.bind(this))
      },
      getCoordinates: function (args) {
        this.selectedContourIds = args
        this.coordinates = []
        let promiseArr = args.map(a => {
          let axiosObj = axios({
            method: 'get',
            url: 'http://localhost:8090/airspace/points;contourId=' + a.contourId,
            auth: {
              username: 'EURO',
              password: 'EURO'
            }
          })
          return axiosObj
        })
        axios.all(promiseArr)
          .then(function (response) {
            this.updateVersions()
            let tempObj = []
            response.forEach(r => {
              tempObj.push(r.data)
            })
            tempObj.forEach(tObj => {
              tObj.forEach(record => {
                this.coordinates.push(record)
              })
            })
          }.bind(this))
      },
      clearCoordinates: function () {
        this.airspaceVersions = []
        this.selectedContourIds = []
        this.coordinates = []
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
