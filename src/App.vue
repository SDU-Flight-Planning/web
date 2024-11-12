<template>
  <div class="app-container">
    <Sidebar @switchMap="switchMap" @clearAllMarkers="clearAllMarkers" @generatePath="generatePath" @clearPath="clearPath" />
    <component
      :is="currentMapComponent"
      class="map-container"
      :markers="markers"
      :polygonPath="polygonPath"
      :pathPoints="pathPoints"
      @addMarker="addMarker"
      @updateMarkerPosition="updateMarkerPosition"
      @deleteMarker="removeMarker"
      @updatePolygonPath="updatePolygonPath"
      @updateMarkers="setMarkers"
      @updatePathPoint="updatePathPoint"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, reactive, computed } from 'vue';
import Sidebar from './components/Sidebar.vue';
import SatelliteMap from './views/SatelliteMap.vue';
import StandardMap from './views/StandardMap.vue';
import BuildingsMap from './views/BuildingsMap.vue';

export default defineComponent({
  components: {
    Sidebar,
    SatelliteMap,
    StandardMap,
    BuildingsMap,
  },
  setup() {
    const currentMapComponent = ref('SatelliteMap');
    const markers = reactive([]);
    const pathPoints = reactive([]);
    let markerIdCounter = 0;
    let pathPointIdCounter = 0;

    const addMarker = (marker) => {
      markers.push({ id: markerIdCounter++, ...marker });
      updatePolygonPath();
    };

    const updateMarkerPosition = (id, lat, lng) => {
      const marker = markers.find((m) => m.id === id);
      if (marker) {
        marker.lat = lat;
        marker.lng = lng;
        updatePolygonPath();
      }
    };

    const removeMarker = (id) => {
      const index = markers.findIndex((m) => m.id === id);
      if (index !== -1) {
        markers.splice(index, 1);
        updatePolygonPath();
      }
    };

    const setMarkers = (updatedMarkers) => {
      markers.splice(0, markers.length, ...updatedMarkers);
      updatePolygonPath();
    };

    const polygonPath = computed(() => {
      return computeConvexHull(markers.map((m) => ({ lat: m.lat, lng: m.lng })));
    });

    const updatePolygonPath = () => {
      const newPath = computeConvexHull(markers.map((m) => ({ lat: m.lat, lng: m.lng })));
      polygonPath.value = newPath;
    };

    const switchMap = (mapType) => {
      currentMapComponent.value = mapType;
    };

    const clearAllMarkers = () => {
      markers.splice(0, markers.length);
      polygonPath.value = [];
      pathPoints.splice(0, pathPoints.length);
    };

    const generatePath = () => {
      if (polygonPath.value.length < 3) return;
      pathPoints.splice(0, pathPoints.length, ...generatePathPoints(polygonPath.value));
    };

    const clearPath = () => {
      pathPoints.splice(0, pathPoints.length);
    };

    const updatePathPoint = (id, lat, lng) => {
      const point = pathPoints.find((p) => p.id === id);
      if (point) {
        point.lat = lat;
        point.lng = lng;
      }
    };

    const generatePathPoints = (polygon) => {
      const points = [];
      const bounds = polygon.reduce(
        (acc, point) => ({
          minLat: Math.min(acc.minLat, point[1]),
          maxLat: Math.max(acc.maxLat, point[1]),
          minLng: Math.min(acc.minLng, point[0]),
          maxLng: Math.max(acc.maxLng, point[0]),
        }),
        { minLat: Infinity, maxLat: -Infinity, minLng: Infinity, maxLng: -Infinity }
      );
      const interval = 0.0005;
      let direction = true;

      for (let lat = bounds.minLat; lat <= bounds.maxLat; lat += interval) {
        const line = [];
        for (let lng = direction ? bounds.minLng : bounds.maxLng; direction ? lng <= bounds.maxLng : lng >= bounds.minLng; lng += direction ? interval : -interval) {
          if (isPointInPolygon([lng, lat], polygon)) {
            if (line.length === 0) {
              line.push({ id: pathPointIdCounter++, lat, lng });
            }
          }
        }
        if (line.length) points.push(...line);
        direction = !direction;
      }
      return points;
    };

    const isPointInPolygon = (point, polygon) => {
      let c = false;
      for (let i = 0, j = polygon.length - 1; i < polygon.length; j = i++) {
        const pi = polygon[i], pj = polygon[j];
        if ((pi[1] > point[1]) !== (pj[1] > point[1]) && point[0] < ((pj[0] - pi[0]) * (point[1] - pi[1])) / (pj[1] - pi[1]) + pi[0]) {
          c = !c;
        }
      }
      return c;
    };

    const computeConvexHull = (points) => {
      if (points.length < 3) return points.map((p) => [p.lng, p.lat]);
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
      return [...lower, ...upper].map((p) => [p.lng, p.lat]);
    };

    return {
      currentMapComponent,
      markers,
      addMarker,
      updateMarkerPosition,
      removeMarker,
      setMarkers,
      polygonPath,
      switchMap,
      clearAllMarkers,
      generatePath,
      pathPoints,
      updatePathPoint,
      clearPath,
    };
  },
});
</script>

<style>
.app-container {
  display: flex;
  width: 100vw;
  height: 100vh;
}
.map-container {
  flex: 1;
  height: 100vh;
}
</style>
