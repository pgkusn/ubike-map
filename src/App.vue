<template>
    <div id="app">
        <div class="row no-gutters">
            <!-- 選擇區域 -->
            <div class="toolbox col-sm-3 p-2 bg-white">
                <div class="form-group d-flex">
                <label for="city" class="col-form-label mr-2 text-right">縣市</label>
                <div class="flex-fill">
                    <select id="city" class="form-control" v-model="select.city">
                        <option v-for="city in cityName" :key="city.name">{{city.name}}</option>
                    </select>
                </div>
                </div>
                <div class="form-group d-flex">
                <label for="dist" class="col-form-label mr-2 text-right">地區</label>
                <div class="flex-fill">
                    <select id="dist" class="form-control" v-model="select.dist">
                        <option v-for="dist in districts" :key="dist.name">{{dist.name}}</option>
                    </select>
                </div>
                </div>
            </div>

            <!-- 顯示地圖和 UBike 站點 -->
            <div class="col-sm-9">
                <div id="map"></div>
            </div>
        </div>
    </div>
</template>

<script>
import cityName from './assets/cityName.json';
import L from 'leaflet';

export default {
    name: 'App',
    data() {
        return {
            cityName,
            select: {
                city: '台北市',
                dist: '中正區',
            },
            ubikes: [],
            OSMmap: {},
        }
    },
    computed: {
        districts() {
            return this.cityName.find(city => city.name === this.select.city).districts;
        },
        currentUbikes() {
            return this.ubikes.filter(bike => this.select.dist === bike.sarea);
        },
    },
    watch: {
        currentUbikes() {
            this.updateMap();
        }
    },
    methods: {
        updateMap() {
            // remove marker
            this.OSMmap.eachLayer(layer => {
                if (layer instanceof L.Marker) {
                    this.OSMmap.removeLayer(layer);
                }
            });

            // add marker & popup
            this.currentUbikes.forEach(bike => {
                L.marker([bike.lat, bike.lng])
                    .bindPopup(`<p><strong style="font-size: 20px;">${bike.sna}</strong></p>
                        <strong style="font-size: 16px; color: #d45345;">可租借車輛剩餘：${bike.sbi} 台</strong><br>
                        可停空位剩餘: ${bike.bemp}<br>
                        <small>最後更新時間: ${bike.mday}</small>`)
                    .addTo(this.OSMmap);
            });

            // move view
            this.cityName[0].districts.forEach(dist => {
                if (dist.name === this.select.dist) {
                    this.OSMmap.flyTo([dist.latitude, dist.longitude]);
                }
            });
        }
    },
    created() {
        const api = 'https://tcgbusfs.blob.core.windows.net/blobyoubike/YouBikeTP.json';
        this.axios.get(api).then(res => {
            this.ubikes = Object.keys(res.data.retVal).map(key => res.data.retVal[key]);
        })
    },
    mounted() {
        // 初始化地圖
        this.OSMmap = L.map('map', {
            center: [25.041956, 121.508791],
            zoom: 18
        });

        // 將 OSM 圖資放置於地圖上
        // 圖資網址來源：https://leafletjs.com/reference-1.6.0.html#tilelayer-url-template 中的 Usage example
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
            maxZoom: 18,
        }).addTo(this.OSMmap);
    },
}
</script>

<style lang="scss">
@import 'bootstrap/scss/bootstrap';
#map {
    position: relative;
    height: 100vh;
}
</style>
