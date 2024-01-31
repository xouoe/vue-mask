<template>
  <div class="row g-0 position-relative">
    <div class="col-sm-3">
      <div v-if="cityname.length" class="toolbox">
        <div class="sticky-top bg-white">
          <div class="form-group d-flex mb-2">
            <label for="cityname" class="cityname text-center d-flex align-items-center m-2">縣市</label>
            <select class="form-select" id="cityname" v-model="select.city" @change="select.area = ''">
              <option value="" disabled>請選擇縣市</option>
              <option :value="city.CityName" v-for="city in cityname" :key="city" >{{ city.CityName }}</option>
            </select>
          </div>
          <div class="form-group d-flex">
            <label for="areaname" class="areaname text-center d-flex align-items-center m-2">地區</label>
            <select class="form-select" id="areaname" v-model="select.area">
              <option value="" disabled>請選擇地區</option>
              <option :value="a.AreaName" v-for="a in cityname.find((city) => city.CityName === select.city)?.AreaList" :key="a.AreaName">
                {{ a.AreaName }}
              </option>
            </select>
          </div>
        </div>
        <ul class="list-group">
          <li class="list-group-item pharmacy" v-for="p in pharmacies" :key="p">
            <div class="d-flex justify-content-between">
              <h3>{{ p.properties.name }}</h3>
              <i class="fa-solid fa-magnifying-glass mt-1 glass" @click.prevent="penTo(p)"></i>
            </div>
            <p>地址：<a :href="`https://www.google.com.tw/maps/place/${p.properties.address}`" 
              target="_blank" rel="noreferrer noopenner" class="text-decoration-none">{{ p.properties.address }}</a>
            </p>
            <p>電話：{{ p.properties.phone }}</p>
            <div class="mb-3">
              <span class="me-2 p-2 rounded-pill" :class="p.properties.mask_adult === 0 ? 'grey' : 'adult'">成人口罩：{{ p.properties.mask_adult }}</span>
              <span class="p-2 rounded-pill" :class="p.properties.mask_child === 0 ? 'grey' : 'child'">兒童口罩：{{ p.properties.mask_child }}</span>
            </div>
            <small>最後更新時間：{{ p.properties.updated }}</small>
          </li>
        </ul>
      </div>
    </div>
    <div class="col-sm-9">
      <div id="map"></div>
    </div>
  </div>
</template>

<script>
import L from 'leaflet'
import cityname from './assets/CityName.json'
let openStreetMap = {}

export default {
  data() {
    return {
      datas:[],
      cityname,
      select:{
        city:'高雄市',
        area:'大寮區'
      }
    }
  },
  computed:{
    pharmacies(){
      return this.datas.filter((data) => {
        if (!this.select.area) {
          return data.properties?.county === this.select.city
        }else {
          return data.properties?.town === this.select.area
        }
      })
    }
  },
  watch:{
    pharmacies(){
      // console.log(value)
      this.updateMap()
    },
  },
  methods:{
    updateMap(){
      openStreetMap.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          openStreetMap.removeLayer(layer)
        }
      })
      this.pharmacies.forEach((pharmacy) => {
        L.marker([
          pharmacy.geometry.coordinates[1],pharmacy.geometry.coordinates[0]
        ]).addTo(openStreetMap).bindPopup(`<p><strong style="font-size: 20px;">${pharmacy.properties.name}</strong></p>
          <strong style="font-size: 16px; color: #d45345;">口罩剩餘：成人 - ${pharmacy.properties.mask_adult ? `${pharmacy.properties.mask_adult} 個` : '未取得資料'} / 兒童 - ${pharmacy.properties.mask_child ? `${pharmacy.properties.mask_child} 個` : '未取得資料'}</strong><br>
          地址: ${pharmacy.properties.address}<br>
          電話: ${pharmacy.properties.phone}<br>
          <small>最後更新時間: ${pharmacy.properties.updated}</small>`)
      });
      this.pharmacies.forEach((pharmacy) => {
        openStreetMap.panTo(new L.LatLng(pharmacy.geometry.coordinates[1], pharmacy.geometry.coordinates[0]))
        L.marker([pharmacy.geometry.coordinates[1], pharmacy.geometry.coordinates[0]], {
        }).addTo(openStreetMap).bindPopup(`<p><strong style="font-size: 20px;">${pharmacy.properties.name}</strong></p>
          <strong style="font-size: 16px; color: #d45345;">口罩剩餘：成人 - ${pharmacy.properties.mask_adult ? `${pharmacy.properties.mask_adult} 個` : '未取得資料'} / 兒童 - ${pharmacy.properties.mask_child ? `${pharmacy.properties.mask_child} 個` : '未取得資料'}</strong><br>
          地址: ${pharmacy.properties.address}<br>
          電話: ${pharmacy.properties.phone}<br>
          <small>最後更新時間: ${pharmacy.properties.updated}</small>`)
      });
    },
    penTo(pharmacy){
      openStreetMap.panTo(new L.LatLng(pharmacy.geometry.coordinates[1], pharmacy.geometry.coordinates[0]))
      L.marker([pharmacy.geometry.coordinates[1], pharmacy.geometry.coordinates[0]], {
        }).addTo(openStreetMap).bindPopup(`<p><strong style="font-size: 20px;">${pharmacy.properties.name}</strong></p>
        <strong style="font-size: 16px; color: #d45345;">口罩剩餘：成人 - ${pharmacy.properties.mask_adult ? `${pharmacy.properties.mask_adult} 個` : '未取得資料'} / 兒童 - ${pharmacy.properties.mask_child ? `${pharmacy.properties.mask_child} 個` : '未取得資料'}</strong><br>
        地址: ${pharmacy.properties.address}<br>
        電話: ${pharmacy.properties.phone}<br>
        <small>最後更新時間: ${pharmacy.properties.updated}</small>`).openPopup()
    }
  },
  mounted(){
    const api = 'https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json'
    this.$http.get(api).then(res => {
      this.datas = res.data.features
      // console.log(res)
    })
    openStreetMap = L.map('map', {
      center: [22.6346467, 120.3963962],
      zoom: 18
    })
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 20,
    }).addTo(openStreetMap)
  }
}
</script>

<style lang="scss">
@import './assets/all.scss';

.cityname{
  width: 60px;
}
.areaname{
  width: 60px;
}
#map{
  position: relative;
  height: 100vh;
}
.toolbox{
  overflow-y: auto;
  height: 100vh;
}
.pharmacy{
  height: 200px;
}
.adult{
  background-color: bisque;
}
.child{
  background-color: coral;
}
.glass{
  cursor: pointer;
}
.grey{
  background-color: #ccc;
}
</style>