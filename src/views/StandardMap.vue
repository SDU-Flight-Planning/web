<template>
  <div id="mapContainer" class="map-container"></div>
  <InfoWindow
    v-if="infoWindowData"
    :lat="infoWindowData.lat"
    :lng="infoWindowData.lng"
    @deleteMarker="deleteMarker"
    @close="closeInfoWindow"
  />
  <InfoWindow
    v-if="selectedPathPoint"
    :lat="selectedPathPoint.lat"
    :lng="selectedPathPoint.lng"
    @close="closePathPointInfoWindow"
  />
</template>

<script lang="ts">
import { defineComponent, onMounted, ref, watch, PropType } from 'vue';
import InfoWindow from '../components/InfoWindow.vue';

export default defineComponent({
  components: { InfoWindow },
  props: {
    markers: {
      type: Array as PropType<{ id: number; lat: number; lng: number }[]>,
      required: true,
    },
    polygonPath: {
      type: Array as PropType<[number, number][]>,
      required: true,
    },
    pathPoints: {
      type: Array as PropType<{ id: number; lat: number; lng: number }[]>,
      required: true,
    },
  },
  emits: ['addMarker', 'updateMarkerPosition', 'deleteMarker', 'updatePolygonPath', 'updateMarkers', 'updatePathPoint'],
  setup(props, { emit }) {
    let map;
    let polygon;
    let pathLine;
    const markerInstances = ref([]);
    const pathPointInstances = ref([]);
    const infoWindowData = ref(null);
    const selectedPathPoint = ref(null);
    const distanceLabels = ref([]); // Store distance labels

    onMounted(() => {
      const satelliteLayer = new AMap.TileLayer();
      map = new AMap.Map('mapContainer', {
        center: [117.140109, 36.667382],
        zoom: 15,
        layers: [satelliteLayer],
      });

      props.markers.forEach(addMarkerToMap);
      drawPolygon(props.polygonPath);
      drawPathLine(props.pathPoints);

      map.on('click', (e) => {
        const markerData = { lat: e.lnglat.getLat(), lng: e.lnglat.getLng() };
        emit('addMarker', markerData);
      });

      window.addEventListener('keydown', handleKeydown);
    });

    const addMarkerToMap = (markerData) => {
      const marker = new AMap.Marker({
        position: [markerData.lng, markerData.lat],
        map: map,
        draggable: true,
        content: '<div style="width:10px;height:10px;background:red;border-radius:50%;"></div>',
        offset: new AMap.Pixel(-5, -5),
      });

      markerInstances.value.push({ marker, id: markerData.id });

      marker.on('dragging', (e) => {
        emit('updateMarkerPosition', markerData.id, e.lnglat.getLat(), e.lnglat.getLng());
      });

      marker.on('click', () => {
        infoWindowData.value = { id: markerData.id, lat: markerData.lat, lng: markerData.lng };
      });
    };

    const drawPolygon = (path) => {
      if (polygon) map.remove(polygon);
      clearDistances();

      if (path.length < 3) return;
      polygon = new AMap.Polygon({
        path,
        strokeColor: '#ffffff',
        strokeWeight: 2,
        fillColor: '#d3a4f9',
        fillOpacity: 0.5,
      });
      map.add(polygon);
      displayDistances(path);
    };

    const displayDistances = (path) => {
      clearDistances();
      for (let i = 0; i < path.length; i++) {
        const start = path[i];
        const end = path[(i + 1) % path.length];
        const distance = AMap.GeometryUtil.distance(start, end);
        const midPoint = [(start[0] + end[0]) / 2, (start[1] + end[1]) / 2];
        const text = new AMap.Text({
          text: `${distance.toFixed(2)}米`,
          anchor: 'center',
          style: { color: '#000', fontSize: '15px', backgroundColor: 'transparent', border: '0' },
          position: midPoint,
        });
        map.add(text);
        distanceLabels.value.push(text);
      }
    };

    const clearDistances = () => {
      distanceLabels.value.forEach(label => map.remove(label));
      distanceLabels.value = [];
    };

    const drawPathLine = (points) => {
      pathPointInstances.value.forEach(({ marker }) => map.remove(marker));
      pathPointInstances.value = [];
      if (pathLine) map.remove(pathLine);

      const path = points.map((point) => [point.lng, point.lat]);
      pathLine = new AMap.Polyline({
        path,
        strokeColor: '#ffdd00',
        strokeWeight: 3,
      });
      map.add(pathLine);

      points.forEach((point) => {
        const pathPointMarker = new AMap.Marker({
          position: [point.lng, point.lat],
          map: map,
          draggable: true,
          content: '<div style="width:10px;height:10px;background:green;border-radius:50%;"></div>',
          offset: new AMap.Pixel(-5, -5),
        });

        pathPointInstances.value.push({ marker: pathPointMarker, id: point.id });

        pathPointMarker.on('dragend', (e) => {
          emit('updatePathPoint', point.id, e.lnglat.getLat(), e.lnglat.getLng());
        });

        pathPointMarker.on('click', () => {
          selectedPathPoint.value = { id: point.id, lat: point.lat, lng: point.lng };
        });
      });
    };

    const deleteMarker = () => {
      const markerId = infoWindowData.value.id;
      const markerIndex = markerInstances.value.findIndex(m => m.id === markerId);
      if (markerIndex !== -1) {
        const { marker } = markerInstances.value[markerIndex];
        map.remove(marker);
        markerInstances.value.splice(markerIndex, 1);
        emit('deleteMarker', markerId);
        const updatedMarkers = props.markers.filter(m => m.id !== markerId);
        emit('updateMarkers', updatedMarkers);
        const newPolygonPath = computeConvexHull(updatedMarkers.map(m => ({ lat: m.lat, lng: m.lng })));
        emit('updatePolygonPath', newPolygonPath);
        drawPolygon(newPolygonPath);
        infoWindowData.value = null;
      }
    };

    const closeInfoWindow = () => {
      infoWindowData.value = null;
    };

    const closePathPointInfoWindow = () => {
      selectedPathPoint.value = null;
    };

    const handleKeydown = (event) => {
      const step = 0.0001;
      if (selectedPathPoint.value) {
        let { lat, lng, id } = selectedPathPoint.value;

        switch (event.key.toLowerCase()) {
          case 'w':
            lat += step;
            break;
          case 's':
            lat -= step;
            break;
          case 'a':
            lng -= step;
            break;
          case 'd':
            lng += step;
            break;
          default:
            return;
        }

        emit('updatePathPoint', id, lat, lng);
        selectedPathPoint.value.lat = lat;
        selectedPathPoint.value.lng = lng;
        pathPointInstances.value.find(p => p.id === id).marker.setPosition([lng, lat]);
        drawPathLine(props.pathPoints);
      }
    };

    watch(
      () => props.polygonPath,
      (newPath) => {
        drawPolygon(newPath);
      },
      { deep: true }
    );

    watch(
      () => props.markers,
      (newMarkers) => {
        markerInstances.value.forEach(({ marker }) => map.remove(marker));
        markerInstances.value = [];
        newMarkers.forEach(addMarkerToMap);
        drawPolygon(props.polygonPath);
      },
      { deep: true }
    );

    watch(
      () => props.pathPoints,
      (newPoints) => {
        drawPathLine(newPoints);
      },
      { deep: true }
    );

    function computeConvexHull(points) {
      if (points.length < 3) return points.map(p => [p.lng, p.lat]);
      points = points.slice().sort((a, b) => a.lng - b.lng || a.lat - b.lat);

      const cross = (o, a, b) => (a.lng - o.lng) * (b.lat - o.lat) - (a.lat - o.lat) * (b.lng - o.lng);
      const lower = [];
      for (const point of points) {
        while (lower.length >= 2 && cross(lower[lower.length - 2], lower[lower.length - 1], point) <= 0) {
          lower.pop();
        }
        lower.push(point);
      }

      const upper = [];
      for (const point of points.reverse()) {
        while (upper.length >= 2 && cross(upper[upper.length - 2], upper[upper.length - 1], point) <= 0) {
          upper.pop();
        }
        upper.push(point);
      }

      lower.pop();
      upper.pop();
      return [...lower, ...upper].map(p => [p.lng, p.lat]);
    }

    return { infoWindowData, selectedPathPoint, deleteMarker, closeInfoWindow, closePathPointInfoWindow };
  },
});
</script>

<style scoped>
.map-container {
  width: 100%;
  height: 100%;
}
</style>
