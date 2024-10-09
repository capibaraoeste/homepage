<template>
  <div class="app-container">
    <h1>แนะนำเพลง</h1>
    <ul>
      <li v-for="(song, index) in songList" :key="index">
        <a :href="song.link" target="_blank">{{ song.title }} - {{ song.artist }}</a>
      </li>
    </ul>
    <button @click="goToHome">กลับไปยังหน้าเริ่มต้น</button>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { doc, setDoc, getFirestore, updateDoc, arrayUnion } from 'firebase/firestore';
import { getApp } from 'firebase/app';

const songList = ref([]);
const route = useRoute();
const router = useRouter();
const db = getFirestore(getApp());

onMounted(() => {
  init();
});

async function init() {
  const params = new URLSearchParams(window.location.search);
  const type = params.get('type'); // อ่านประเภทเพลง
  const score = parseInt(route.query.score);
  const documentId = router.currentRoute.value.query.id;

  console.log(`Type: ${type}, Score: ${score}`);

  // แนะนำเพลงตามประเภทและคะแนน
  if (type === 'thai') {
    if (score > 3) {
      songList.value = [
        { searchQuery: "เพลงรัก" }, // คำค้นหา
        { searchQuery: "เพลงเศร้า" },
        { searchQuery: "เพลงที่ฟังสบาย" }
      ];
    } else {
      songList.value = [
        { searchQuery: "เพลงคิดถึง" },
        { searchQuery: "เพลงอกหัก" },
        { searchQuery: "เพลงช้า" }
      ];
    }
  } else if (type === 'international') {
    if (score > 3) {
      songList.value = [
        { searchQuery: "เพลงป๊อป" },
        { searchQuery: "เพลงแดนซ์" },
        { searchQuery: "เพลงยอดนิยม" }
      ];
    } else {
      songList.value = [
        { searchQuery: "เพลงเศร้า" },
        { searchQuery: "เพลงรัก" },
        { searchQuery: "เพลงช้า" }
      ];
    }
  }

  if (documentId) {
    // อัปเดต Firestore โดยเพิ่มเพลงลงใน songRecommendations
    const docRef = doc(db, 'tables', documentId);
    await updateDoc(docRef, {
      songRecommendations: arrayUnion(...songList.value) // เพิ่มเพลงทั้งหมดใน songList
    });
  } else {
    console.error('Document ID is undefined');
  }
}




function goToHome() {
  router.push('/');
}
</script>


<style scoped>
.app-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f0f0f0;
}

h1 {
  margin-bottom: 20px;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  margin: 10px 0;
  padding: 10px;
  background-color: lightgray;
  border-radius: 5px;
  width: 300px;
  text-align: center;
}

a {
  text-decoration: none;
  color: black;
}

a:hover {
  color: blue;
}

button {
  margin-top: 20px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  background-color: lightblue;
  border: none;
  border-radius: 5px;
}
</style>
