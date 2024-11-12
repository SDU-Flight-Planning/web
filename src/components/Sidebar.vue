<template>
  <div class="sidebar">
    <button @click="switchMap('SatelliteMap')">卫星图层</button>
    <button @click="switchMap('StandardMap')">标准图层</button>
    <button @click="switchMap('BuildingsMap')">3D 楼块图层</button>
    <button @click="clearAllMarkers">清除所有标记</button>
    <button @click="generatePath">路径生成</button>
    <button @click="clearPath">路径删除</button>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
  emits: ['switchMap', 'clearAllMarkers', 'generatePath', 'clearPath'],
  setup(_, { emit }) {
    const switchMap = (mapType) => emit('switchMap', mapType);
    const clearAllMarkers = () => emit('clearAllMarkers');
    const generatePath = () => emit('generatePath');
    const clearPath = () => emit('clearPath');
    return { switchMap, clearAllMarkers, generatePath, clearPath };
  },
});
</script>

<style scoped>
.sidebar {
  width: 20vw;
  height: 100vh;
  background-color: #333;
  color: #fff;
  display: flex;
  flex-direction: column;
  padding: 20px;
}
button {
  margin: 10px 0;
  padding: 10px;
  background-color: #555;
  color: white;
  border: none;
  cursor: pointer;
}
button:hover {
  background-color: #777;
}
</style>
