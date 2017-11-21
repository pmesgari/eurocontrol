<template>
  <v-container fluid fill-height>
    <v-layout>
      <v-flex d-flex xs3 id="contour-tables">
          <!-- <contour-select v-on:getcoordinate="getCoordinates" v-on:clearCoordinates="clearCoordinates"></contour-select> -->
          <contour-table v-on:getcoordinate="getCoordinates" v-on:getversions="getVersions" v-on:clearversions="clearCoordinates"></contour-table>
          <contour-version-table v-on:getversionedcoordinate="getVersionedCoordinate" v-on:getcoordinate="getCoordinates" :airspaceVersions="airspaceVersions"></contour-version-table>
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
        mapColors: ['red', 'brown', 'yellow'],

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
        // group args based on their areaContourId
        let groupedArgs = {}
        this.selectedContourIds.forEach(sCid => {
          // debugger
          let extraKey = ''
          if (sCid.airspaceVersion) {
            extraKey = sCid.airspaceVersion.slice(0,5)
          }
          else {
            extraKey = 'noVersion'
          }
          groupedArgs[sCid.contourId + '_' + extraKey] = args.filter(a => {
            return a.airspaceVersion == sCid.airspaceVersion
          })
        })
        let filteredArgs = []
        Object.keys(groupedArgs).forEach(k => {
          let newObj = groupedArgs[k].map(r => {
            let tempObj = {}
            tempObj['lat'] = Number(r.lat)
            tempObj['lng'] = Number(r.lng)
            return tempObj
          })
          filteredArgs.push(newObj)
        })
        //   [{lat: -27.467, lng: 153.027}]
        let mapArgs = []
        filteredArgs.forEach(fArg => {
          let mapCoordinates = fArg
          let mapPath = new google.maps.Polyline({
            path: mapCoordinates,
            geodesic: true,
            strokeColor: this.mapColors[Math.floor(Math.random() * this.mapColors.length)],
            strokeOpacity: 1.0,
            strokeWeight: 2
          });
          mapArgs.push(mapPath)
        })
        // debugger
        mapArgs.forEach(ma => {
          ma.setMap(map)
        })
        // this.flightPlanCoordinates = filteredArgs
        // // debugger
        // var flightPlanCoordinates2 = [
        //   {lat: 37.772, lng: -122.214},
        //   {lat: 21.291, lng: -157.821},
        //   {lat: -18.142, lng: 178.431},
        //   {lat: -27.467, lng: 153.027}
        // ];
        // var flightPath2 = new google.maps.Polyline({
        //   path: flightPlanCoordinates2,
        //   geodesic: true,
        //   strokeColor: '#FF0000',
        //   strokeOpacity: 1.0,
        //   strokeWeight: 2
        // });
        // var flightPath = new this.googleMaps.Polyline({
        //   path: this.flightPlanCoordinates,
        //   geodesic: true,
        //   strokeColor: '#FF0000',
        //   strokeOpacity: 1.0,
        //   strokeWeight: 2
        // })

        // flightPath.setMap(map)
        // flightPath2.setMap(map)
      },
      validateAirspace: function (args) {
        // debugger
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
            // this.updateVersions()
          }.bind(this))
          .catch(function (error) {
            console.log(error)
          })
      },
      getVersions: function (args) {
        // debugger
        this.airspaceVersions = []
        this.selectedContourIds = args
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
      getVersionedCoordinate: function (args) {
        console.log('getting versioned')
        this.selectedContourIds = args
        this.coordinates = []
        let promiseArr = args.map(a => {
          let axiosObj = axios({
            method: 'get',
            url: 'http://localhost:8090/airspace/points/version;airspaceVersion=' + a.airspaceVersion,
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
                this.coordinates.push(record)
              })
            })
          }.bind(this))
      },
      clearCoordinates: function () {
        this.airspaceVersions = []
        this.selectedContourIds = []
        this.coordinates = []
        this.flightPlanCoordinates = []
        this.drawAirspace([])
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
