<template>
  <div style="height: 100%;">
    <div id="map"></div>
    <button @click="openAddMarkerDialog">Добавить метку</button>
    <div v-if="showDialog" class="dialog">
      <h3>Добавить новую метку</h3>
      <label>Описание:</label>
      <input v-model="newMarker.title" placeholder="Введите описание" />
      <label>Тип метки:</label>
      <select v-model="newMarker.type">
        <option value="red">Красный маркер</option>
        <option value="blue">Синий маркер</option>
      </select>
      <button @click="addNewMarker">Добавить</button>
      <button @click="closeAddMarkerDialog">Отмена</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import L from 'leaflet';
import moment from 'moment';
import 'leaflet/dist/leaflet.css';

class MapInfoPoints{
  score: number=0;
  date: Date=new Date();
  points: MarkPoint[]=[];

}
class MarkPoint{
  id: string='';
  lat: number=0;
  lng: number=0;
  title: string='';
  type: number=0;
  oldMinutes: number=0;
  comments: MarkComment[]=[]
  date: Date= new Date();

}
class MarkComment{
  id: string='';
  date: Date=new Date();
  text: string='';
}

const map = ref<L.Map | null>(null);
const showDialog = ref(false);
const newMarker = ref<MarkPoint>({
  id: '',
  lat: 0,
  lng: 0,
  title: '',
  type: 0,
  oldMinutes: 0,
  comments: [],
  date: new Date(),
});

const markersData = ref<MapInfoPoints>({
  score: 0,
  date: new Date(),
  points: [],


});
const markerIcons = {
  red: new L.Icon({
    iconUrl: 'https://example.com/red-icon.png',
    iconSize: [25, 41],
    iconAnchor: [12, 41],
  }),
  blue: new L.Icon({
    iconUrl: 'https://example.com/blue-icon.png',
    iconSize: [25, 41],
    iconAnchor: [12, 41],
  }),
};

const loadMarkersFromJSON = async () => {
  try {
    //const response = await fetch('./markers.json');
    const response = await fetch('https://api.jsonbin.io/v3/b/67251516acd3cb34a8a0c115');
    
    const data = await response.json();
    markersData.value = data.record;
    addMarkersToMap();
  } catch (error) {
    console.error('Ошибка загрузки данных:', error);
  }
};

const initMap = () => {
  if(!map) return;
  let m  = L.map('map').setView([50, 36.26], 13);
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
    attribution: '© OpenStreetMap contributors',
  }).addTo(m);
  map.value = m
};

const addMarkersToMap = () => {
  markersData.value.points.forEach((markerData:MarkPoint) => {
    //const icon = markerIcons[markerData.type] || markerIcons.red;
    const markerStyle = {
      color: 'red',
      fillColor: '#f03',
      fillOpacity: 0.5,
      radius: 200,
    };

    if (markerData.oldMinutes > 30) {
      markerStyle.radius = 50 + 150 * (30 / markerData.oldMinutes);
    }

    switch (markerData.type) {
      case 0:
        markerStyle.color = 'yellow';
        markerStyle.fillColor = '#3F3';
        break;
      case 1:
        markerStyle.color = 'red';
        markerStyle.fillColor = '#f03';
        break;
      case -1:
        markerStyle.color = 'green';
        markerStyle.fillColor = '#3F3';
        break;
    }

    const marker = L.circle([markerData.lat, markerData.lng], markerStyle).addTo(<any>map.value);

    let time = '';
    if (markerData.date) {
      time = moment(markerData.date).format('HH:mm');
    }
    let comments = markerData.comments.sort((a,b)=>moment(b.date).unix()-moment(a.date).unix()); //sort by date desc
    const popupContent = `
  ${time} <strong>${markerData.title}</strong><br>
      ${comments
        .map((comment) => `<p><strong>${moment(comment.date).format('HH:mm')}:</strong> ${comment.text}</p>`)
        .join('')}
    `;
    marker.bindPopup(popupContent);
  });
};

const openAddMarkerDialog = () => {
  showDialog.value = true;
};

const closeAddMarkerDialog = () => {
  showDialog.value = false;
};

const addNewMarker = () => {
  /*
  const icon = markerIcons[newMarker.value.type] || markerIcons.red;
  const marker = L.marker([newMarker.value.lat, newMarker.value.lng], { icon }).addTo(map.value);
  marker.bindPopup(`<strong>${newMarker.value.description}</strong>`);

  markersData.value.push({ ...newMarker.value });
  showDialog.value = false;
  newMarker.value = {
    lat: 50,
    lng: 36.26,
    description: '',
    oldMinutes:0,
    type: 0,
  };
  */
};

onMounted(() => {
  initMap();
  loadMarkersFromJSON();
  setInterval(()=>loadMarkersFromJSON(), 2*60_000);
});
</script>

<style>
#map {
  width: 100vw;
  height: 100%;
}
.dialog {
  position: absolute;
  top: 10%;
  left: 10%;
  background-color: white;
  padding: 20px;
  border: 1px solid black;
  border-radius: 8px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.2);
}
</style>
