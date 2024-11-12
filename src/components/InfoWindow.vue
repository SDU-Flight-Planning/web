<template>
  <div
    class="info-window"
    :style="{ top: `${position.top}px`, left: `${position.left}px` }"
    @mousedown="startDrag"
  >
    <button class="close-btn" @click="closeWindow">×</button>
    <table class="info-table">
      <tbody>
        <tr>
          <th>纬度</th>
          <td>{{ formattedLat }}</td>
        </tr>
        <tr>
          <th>经度</th>
          <td>{{ formattedLng }}</td>
        </tr>
      </tbody>
    </table>
    <button class="delete-btn" @click="deleteMarker">删除标记</button>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from 'vue';

export default defineComponent({
  props: {
    lat: { type: Number, required: true },
    lng: { type: Number, required: true },
  },
  emits: ['deleteMarker', 'close'],
  setup(props, { emit }) {
    const position = ref({ top: 20, left: window.innerWidth - 270 });
    const isDragging = ref(false);
    const startPosition = ref({ x: 0, y: 0 });

    const formattedLat = computed(() => props.lat.toFixed(4));
    const formattedLng = computed(() => props.lng.toFixed(4));

    const startDrag = (event) => {
      isDragging.value = true;
      startPosition.value = { x: event.clientX - position.value.left, y: event.clientY - position.value.top };
      window.addEventListener('mousemove', drag);
      window.addEventListener('mouseup', stopDrag);
    };

    const drag = (event) => {
      if (isDragging.value) {
        position.value = {
          top: event.clientY - startPosition.value.y,
          left: event.clientX - startPosition.value.x,
        };
      }
    };

    const stopDrag = () => {
      isDragging.value = false;
      window.removeEventListener('mousemove', drag);
      window.removeEventListener('mouseup', stopDrag);
    };

    const deleteMarker = () => emit('deleteMarker');
    const closeWindow = () => emit('close');

    return { position, startDrag, deleteMarker, closeWindow, formattedLat, formattedLng };
  },
});
</script>

<style scoped>
.info-window {
  position: absolute;
  top: 20px;
  right: 20px;
  background: white;
  border: 1px solid #ccc;
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  width: 220px;
  cursor: move;
  user-select: none;
}

.close-btn {
  position: absolute;
  top: 8px;
  right: 8px;
  background: none;
  border: none;
  font-size: 18px;
  color: #666;
  cursor: pointer;
  padding: 0;
}

.close-btn:hover {
  color: #333;
}

.info-table {
  width: 100%;
  border-collapse: collapse;
  margin: 15px 0;
}

.info-table th {
  text-align: left;
  color: #555;
  padding-right: 10px;
  font-weight: normal;
}

.info-table td {
  text-align: right;
  color: #333;
}

.delete-btn {
  display: block;
  margin: 15px auto 0;
  padding: 8px 15px;
  background-color: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  font-weight: bold;
  cursor: pointer;
}

.delete-btn:hover {
  background-color: #d32f2f;
}
</style>
