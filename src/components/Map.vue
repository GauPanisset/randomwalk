<template>
  <div id="map-container">
    <div id="map"></div>
    <div class="text-xs-center">
      <v-btn color="info" v-on:click="drawZone">Display Zone</v-btn>
      <v-btn color="success" v-on:click="move">Move</v-btn>
      <v-btn color="error" v-on:click="clear">Clear</v-btn>
    </div>
  </div>
</template>

<script>
    import axios from 'axios';
    import L from 'leaflet';

    const Api = "http://localhost:5000";

    export default {
      name: "Map",
      props: ["nbFrames", "persistence"],
      data() {
        return {
          tileLayers: null,
          map: null,
          playersLayer: null,
          zoneLayer: null,
        }
      },
      methods: {
        displayMap(mapDisplay) {
          this.tileLayers[mapDisplay ? 0 : 1].remove(this.map);
          this.tileLayers[mapDisplay ? 1 : 0].addTo(this.map);
        },
        move() {
          axios.get(Api + '/moveplayers/'+this.nbFrames).catch(err => {console.log(err)});
          axios.get(Api + '/playerspositions')
            .then(response => {
              this.updatePath(response.data, this.playersLayer);
            })
            .catch(err => {
              console.log(err);
            });
        },
        updatePath(players, layer) {
          layer.clearLayers();
          players.forEach(player => {
            if (this.persistence) {
              let path = [];
              player.positions.forEach(point => {
                path.push(this.map.unproject(L.point(point[0] * Math.pow(2, this.map.getZoom() - 2), point[1] * Math.pow(2, this.map.getZoom() - 2))));
              });
              const color = 'rgb(' + player.color[0] + ',' + player.color[1] + ',' + player.color[2] + ')';
              L.polyline(path, {'color': color}).addTo(layer);
            } else {
              const color = 'rgb(' + player.color[0] + ',' + player.color[1] + ',' + player.color[2] + ')';
              const point = player.positions[player.positions.length - 1];
              L.circle(this.map.unproject(L.point(point[0] * Math.pow(2, this.map.getZoom() - 2), point[1] * Math.pow(2, this.map.getZoom() - 2))),
                {radius: 0.5, fill: true, opacity: 1, color: color})
                .addTo(this.playersLayer);
            }
          });
        },
        clear() {
          axios.get(Api + '/resetzone').catch(err => {console.log(err)});
          axios.get(Api + '/resetplayers').catch(err => {console.log(err)});
          this.playersLayer.clearLayers();
          this.zoneLayer.clearLayers();
        },
        drawZone() {
          this.zoneLayer.clearLayers();
          axios.get(Api + '/zoneposition')
            .then(response => {
              const point = response.data.center;
              L.circle(this.map.unproject(L.point(point[0] * Math.pow(2, this.map.getZoom() - 2), point[1] * Math.pow(2, this.map.getZoom() - 2))),
                {radius: 100 * response.data.radius/response.data.first, fill: false, opacity: 1, color: 'red'})
                .addTo(this.zoneLayer);
            })
            .catch(err => {
              console.log(err);
            });
        }
      },
      mounted() {

        let map = L.map('map', {
          dragging: false,
          crs: L.CRS.Simple,
        }).setView([-100, 150], 2);

        this.map = map;

        //https://oyster.ignimgs.com/ignmedia/wikimaps/fortnite/season-8/{z}/{x}-{y}.jpg
        this.tileLayers = [L.tileLayer('', {
          minZoom: 1,
          maxZoom: 6,
          attribution: 'fortnite-map',
          tms: false,
          noWrap: true,
        }), L.tileLayer('https://oyster.ignimgs.com/ignmedia/wikimaps/fortnite/season-8/{z}/{x}-{y}.jpg', {
          minZoom: 1,
          maxZoom: 6,
          attribution: 'fortnite-map',
          tms: false,
          noWrap: true,
        })];

        this.tileLayers[0].addTo(map);

        this.playersLayer = L.layerGroup();
        this.map.addLayer(this.playersLayer);

        this.zoneLayer = L.layerGroup();
        this.map.addLayer(this.zoneLayer);

        let that = this;
        map.on('click', function (e) {
          let point = that.map.project(e.latlng);
          console.log(L.point(point.x / Math.pow(2, that.map.getZoom() - 2), point.y / Math.pow(2, that.map.getZoom() - 2)));
        });
      }
    }

</script>

<style scoped>

  #map {
    margin: auto;
    width: 70vw;
    height: 80vh;
  }

  .leaflet-container.circle-cursor-enabled {
    cursor:crosshair;
  }

</style>
