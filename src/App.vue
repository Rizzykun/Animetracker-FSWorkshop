<script setup async> //Script template style
import HelloWorld from './components/HelloWorld.vue'
import PocketBase from 'pocketbase'
import { ref , onMounted } from 'vue'
const pb = new PocketBase('http://127.0.0.1:8090/');
const counter = "counter: "
const records = ref([]);
const todoInput = ref("");


const isTrue = ref(true);
let solatAPI = "https://www.e-solat.gov.my/index.php?r=esolatApi/takwimsolat&period=today&zone=SGR01"
let prayerTime = ref({})

let name = ref("")
let index = ref("");
let number = ref(0);
let capitalCities = ref(["New York", "Los Angeles", "Chicago", "Houston", "Kuala Lumpur", "Tokyo"]);  

function increment() {
  number.value++;
  console.log(number)
}
function startInterval() {
  setInterval(increment,1000);
}
function editCity() {
  capitalCities.value[index.value++] = name.value;
  console.log(capitalCities)
}
function getPrayerTime() {
  fetch(solatAPI).then(result => result.json()).then(data => {prayerTime.value = data.prayerTime[0]
  console.log(prayerTime.value)})
}
function createTodo() {
  pb.collection('todos').create({
    Item: todoInput.value,
    isDone: false
  }).then((data) => {
    records.value.push(data);
    console.log(records.value)
  })
}



onMounted(() => {
  getPrayerTime()
  pb.collection('todos').getFullList({
}).then((data) => {
    records.value = data;
    console.log(records.value)
})})
</script>

<template>
  <input type="text" v-model="todoInput" placeholder="Enter the data"/>
  <button @click="createTodo">ENTER</button>
  <p>Todo:</p>
  <ol>
    <li v-for="record in records" :key="record">{{ record.Item }}</li>
  </ol>
  <h1>{{ counter }}</h1>
  <h2>{{ number }}</h2>
  <button @click="startInterval" >Increment</button>

  <h1>Enter data</h1>
  <input type="text" v-model="index" placeholder="Array no" size = "4"/>
  <input type="text" v-model="name" placeholder="Enter the data"/>
  <button @click="editCity">ENTER</button>
  <h2>Previous input is {{ name }}</h2>
  <ol>
    <li v-for="city in capitalCities" :key="city">{{ city }}</li>
  </ol>
  <h1 v-if="isTrue">England is my city</h1>
  <button @click="isTrue = !isTrue">Hide</button>
  <button @click="getPrayerTime">Get Prayer Time</button>

  <div class="border rounded-sm bg-slate -300 m-2 shadow-md p-2 bg-blue-200">
  <table class="w-full">
    <caption>Prayer Time</caption>
    <thead>
      <tr>
        <th>Prayer</th>
        <th>Fajr</th>
        <th>Dhuhr</th>
        <th>Asr</th>
        <th>Maghrib</th>
        <th>Isha</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Time</td>
        <td>{{ prayerTime.fajr }}</td>
        <td>{{ prayerTime.dhuhr }}</td>
        <td>{{ prayerTime.asr }}</td>
        <td>{{ prayerTime.maghrib }}</td>
        <td>{{ prayerTime.isha }}</td>
      </tr>
    </tbody>
  </table>
</div>
</template>


<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
