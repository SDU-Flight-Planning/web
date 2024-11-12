<template>
  <div id="mapContainer" class="map-container"></div>
</template>

<script lang="ts">
import { defineComponent, onMounted, watch, PropType } from 'vue';

export default defineComponent({
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
  setup(props) {
    let map;
    let polygon;
    let pathLine;
    const markerInstances = [];
    const distanceLabels = []; // 存储距离标签
    const pathPointMarkers = []; // 存储路径转折点标记

    onMounted(() => {
      const satelliteLayer = new AMap.TileLayer.Satellite();
      const buildingsLayer = new AMap.Buildings({
        zooms: [15, 20],
        heightFactor: 2,
        zIndex: 50,
      });

      map = new AMap.Map('mapContainer', {
        center: [117.140109, 36.667382],
        zoom: 15,
        viewMode: '3D',
        pitch: 60,
        rotation: -35,
        layers: [satelliteLayer, buildingsLayer],
      });

      // 初始化标记点、多边形和路径
      props.markers.forEach(markerData => addMarkerToMap(markerData));
      drawPolygon(props.polygonPath);
      drawPathLine(props.pathPoints);

      // 添加键盘事件控制视角
      window.addEventListener('keydown', handleKeydown);
    });

    const handleKeydown = (event) => {
      const step = 5;
      switch (event.key.toLowerCase()) {
        case 'q':
          map.setRotation(map.getRotation() - step);
          break;
        case 'e':
          map.setRotation(map.getRotation() + step);
          break;
        case 'w':
          map.setPitch(Math.min(map.getPitch() + step, 83)); // 限制最大俯仰角
          break;
        case 's':
          map.setPitch(Math.max(map.getPitch() - step, 0)); // 限制最小俯仰角
          break;
      }
    };

    const addMarkerToMap = (markerData) => {
      const marker = new AMap.Marker({
        position: [markerData.lng, markerData.lat],
        map: map,
        draggable: false,
        content: '<div style="width:10px;height:10px;background:red;border-radius:50%;"></div>',
        offset: new AMap.Pixel(-5, -5),
      });

      markerInstances.push(marker);
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
        zIndex: 5,
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
        distanceLabels.push(text);
      }
    };

    const clearDistances = () => {
      distanceLabels.forEach(label => map.remove(label));
      distanceLabels.length = 0;
    };

    const drawPathLine = (points) => {
      pathPointMarkers.forEach(marker => map.remove(marker));
      pathPointMarkers.length = 0;
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
          draggable: false,
          content: '<div style="width:10px;height:10px;background:green;border-radius:50%;"></div>',
          offset: new AMap.Pixel(-5, -5),
        });

        pathPointMarkers.push(pathPointMarker);
      });
    };

    const clearMarkersAndPolygon = () => {
      markerInstances.forEach(marker => map.remove(marker));
      markerInstances.length = 0;

      if (polygon) {
        map.remove(polygon);
        polygon = null;
      }

      clearDistances();
      pathPointMarkers.forEach(marker => map.remove(marker));
      pathPointMarkers.length = 0;
      if (pathLine) {
        map.remove(pathLine);
        pathLine = null;
      }
    };

    watch(
      () => props.markers,
      (newMarkers) => {
        clearMarkersAndPolygon();
        newMarkers.forEach(addMarkerToMap);
        drawPolygon(props.polygonPath);
        drawPathLine(props.pathPoints);
      },
      { deep: true }
    );

    watch(
      () => props.polygonPath,
      (newPath) => {
        drawPolygon(newPath);
      },
      { deep: true }
    );

    watch(
      () => props.pathPoints,
      (newPathPoints) => {
        drawPathLine(newPathPoints);
      },
      { deep: true }
    );

    return {};
  },
});
</script>

<style scoped>
.map-container {
  width: 100%;
  height: 100%;
}
</style>
