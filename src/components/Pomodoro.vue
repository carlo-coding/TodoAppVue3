<script setup>
import { ref, reactive } from 'vue';
import { NSpace, NButton, NLayout, NInput } from "naive-ui";
import DeleteIcon from "../assets/Delete.svg";
import EditIcon from "../assets/Edit.svg";
import * as yup from 'yup';
import { throwError } from 'naive-ui/lib/_utils';
import tuturu from "../audios/tuturu.mp3";
const createId = ()=> "id-"+Math.random().toString(16).slice(2);

const activitySchema = yup.object().shape({
  hours: yup.number("Ingresa las horas o 0").min(0, "Minimo 0 hrs").max(99, "Máximo 99 hrs").required("Horas requeridas"),
  minutes: yup.number("Ingresa minutos o 0").min(0, "Minimo 0 min").max(99, "Máximo 99 min").required("Minutos requeridos"),
  title: yup.string().required("El titulo es requerido"),
  checked: yup.boolean().required("checked requerido"),
  id: yup.string().required("Id requerido")
})

const initialValues = {
  hours: undefined,
  minutes: undefined,
  title: undefined,
  checked: false,
  seconds: 0,
  id: undefined,
}
const newActivity = reactive(initialValues);

const activities = ref([]);

const validateFields = (obj, not) => {
  return Object.values(obj).reduce((prev, curr) => {prev = Boolean(curr)});
}

const handleNewActivity = async ()=> {
  newActivity.id = createId();
  newActivity.seconds = newActivity.minutes*60 + newActivity.hours*60*60;
  try {
    await activitySchema.validate(newActivity);
    activities.value.push(reactive({...newActivity}));
    
    newActivity.title = "";
    newActivity.hours = 0;
    newActivity.minutes = 0;
  }catch (err) {
    alert(err.errors);
  }
}

function handleEdit({id, hours, minutes, title}) {
  newActivity.hours = hours;
  newActivity.minutes = minutes;
  newActivity.title = title;
  activities.value = activities.value.filter(act => act.id !== id);
}

const handleDelete = ({id}) => {
  activities.value = activities.value.filter(act => act.id !== id);
}

function * countDown(seconds) {
    while(true) {
      yield seconds--;
    }
}

async function asyncForEach(array, callback) {
  for (let index = 0; index < array.length; index++) {
    await callback(array[index], index, array);
  }
}

function createInterval(index) {
  const countGen = countDown(newActivity.seconds);
  let timeoutValue = (activities.value[index]?.seconds||0)*1000;
  setInterval(()=>{
      if (activities.value[index]?.seconds) {
        activities.value[index].seconds = countGen.next().value;
      }
  }, 1000);
  return new Promise((resolve,_)=> {
    setTimeout(()=> resolve(true), timeoutValue);
  })
}

async function handlePlay (){
    await asyncForEach(activities.value, async (_, index)=> {
      await createInterval(index);
      activities.value[index].checked = true;
      playSound();
    })
}

function handleReset() {
  asyncForEach(activities.value, async (act, index)=> {
      activities.value[index].checked = false;
      activities.value[index].seconds = act.minutes*60 + act.hours*60*60;
    })
}

function playSound() {
  const audio = new Audio(tuturu);
  audio.play()
}

function formatSeconds(sc) {
  let dateObj = new Date(sc * 1000);
  return `
    ${dateObj.getUTCHours().toString().padStart(2, "0")}:
    ${dateObj.getUTCMinutes().toString().padStart(2, "0")}:
    ${dateObj.getSeconds().toString().padStart(2, "0")}
  `
}


</script>

<template>
  <div class="container">
    <h1>Ingresa una actividad</h1>
    <n-space>
      <n-input v-model:value="newActivity.title" placeholder="Nombre de la actividad" color="#3B9FA0" />
      <n-input v-model:value="newActivity.hours" placeholder="Tiempo (Horas)" type="number" color="#3B9FA0" />
      <n-input v-model:value="newActivity.minutes" placeholder="Tiempo (min)" type="number" color="#3B9FA0" />
      <n-button @click="handleNewActivity" color="#3B9FA0">Añadir</n-button>
      <n-button @click="handlePlay" color="#49A03B">Play</n-button>
      <n-button @click="handleReset" color="#49A03B">Reset</n-button>
    </n-space>

    <div v-for="activity in activities" :key="activity.id" class="activity">
      <input type="checkbox" v-model="activity.checked"/> 
      <p :style="activity.checked&&`text-decoration:line-through`">{{activity.title}}</p>
      <span :class="activity.checked?`timer-done`:`timer`">
        {{formatSeconds(activity.checked?0:activity.seconds)}}
      </span> 
      <button @click="handleEdit(activity)"><img :src="EditIcon"/></button>
      <button @click="handleDelete(activity)"><img :src="DeleteIcon"/></button>
    </div>
  </div>
</template>

<style scoped>
a {
  color: #42b983;
}
.container {
  display: flex;
  flex-direction: column;
  padding: 0 100px;
}

.activity {
  display: flex;
  align-items: center;
  width: 100%;
  height: 50px;
  background: #e0e0e0;
  margin-top: 1em;
  border-radius: 10px;
  box-sizing: border-box;
}

.activity p {
  padding: 0 1em;
}

.activity button {
  background: transparent;
  border: none;
  width: 40px;
  height: 40px;
  margin: 0 1em;
}

.activity input[type="checkbox"] {
  margin-left: 10px;
  width: 20px;
  height: 20px;
}

.activity button img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.timer {
  margin-left: auto;
  margin-right: 20px;
  padding: 0.2em 0.5em;
  background: chartreuse;
  border-radius: 10px;
}

.timer-done {
  margin-left: auto;
  margin-right: 20px;
  padding: 0.2em 0.5em;
  background: rgb(255, 102, 0);
  border-radius: 10px;
}
</style>
