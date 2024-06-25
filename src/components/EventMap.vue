<template>
    <div id="map"></div>
    <div class="point-marker"></div>
  </template>
  
  <script>
  import mapboxgl from 'mapbox-gl';
  import event_venues from '@/assets/比赛场馆.geojson'; // 导入GeoJSON文件
  import event_route from "@/assets/奥运会比赛路线.geojson"; // 导入GeoJSON文件
  
  
  
  
  export default {
    name: 'RouteMap',
    data() {
      return {
        map: null, // 地图实例
        hoveredFeature: null, // 当前悬停特征信息
        clickedFeature: null, // 当前点击特征信息
        fireMarkerPosition: null// 火炬标记的位置
      };
    },
    mounted() {
      mapboxgl.accessToken = 'pk.eyJ1IjoibHdjeCIsImEiOiJjbHd3N3V6NzcwdHNwMm1yMTZocmFpcW9tIn0.02n3EfWEQ6aL9PL1RDAgLA';
  
      this.map = new mapboxgl.Map({
        container: 'map',
        style: 'mapbox://styles/lwcx/clxfz1vei004001pu5ivh503z', // 根据你的需要选择地图样式
        center: [2.3522, 48.8566], // 设置地图中心点的经纬度为巴黎的中心点
        zoom: 5, // 设置地图的缩放级别
      });
  
      // 设置地图范围为法国
      const bounds = [
      [1.824199, 48.615573], // Southwest coordinates (巴黎的经纬度)
      [2.60992, 49.02145]   // Northeast coordinates (巴黎的经纬度)
      ];
      this.map.fitBounds(bounds);
  
      this.map.addControl(new mapboxgl.NavigationControl());
  
      this.map.on('load', () => {
        this.initializeMap();
      });
  
      this.map.on('mousemove', this.handleMapHover);
      this.map.on('click', this.handleMapClick);
    },
    methods: {
      initializeMap() {
      this.map.addLayer({
        id: 'event-routes',
        type: 'line',
        source: {
          type: 'geojson',
          data: event_route,
        },
        paint: {
          'line-color': [
            'match',
            ['get', 'type'], // 根据路线的唯一标识符选择颜色字段
            1, '#FF5733',    // 路线1的颜色 - 橙色
            2, '#33FF57',    // 路线2的颜色 - 绿色
            3, '#FFD700',    // 路线3的颜色 - 金黄色
            4, '#00FFFF',    // 路线4的颜色 - 青色
            5, '#FF6347',    // 路线5的颜色 - 珊瑚红
            6, '#FFA500',    // 路线6的颜色 - 橙色
            7, '#FF1493',    // 路线7的颜色 - 深粉红
            8, '#ADFF2F',    // 路线8的颜色 - 青柠绿
            9, '#FF69B4',    // 路线9的颜色 - 热情的粉红
            10, '#32CD32',   // 路线10的颜色 - 酸橙绿
            '#808080' // 默认颜色（灰色），用于所有未匹配到的路线
              ],
          'line-width': 3,
        },
      });
      this.map.addLayer({
    id: 'event-venues',
    type: 'circle', // 修改为 circle 类型
    source: {
      type: 'geojson',
      data: event_venues,
    },
    paint: {
      'circle-radius': 6, // 圆的半径，根据需求调整
      'circle-color': '#b63150', // 圆的填充颜色
      'circle-stroke-color': '#b63150', // 圆的边框颜色
      'circle-stroke-width': 2, // 圆的边框宽度
      'circle-opacity': 1, // 圆的透明度
    },
  });
  
      // 假设 event_venues 是一个 GeoJSON 数据源，包含一组点
    event_venues.features.forEach(feature => {
      // 创建一个自定义的 div 元素作为 Marker
      var markerEl = document.createElement('div');
      markerEl.className = 'point-marker'; // 设置样式类名，需要在 CSS 中定义该类名的样式
  
      // 创建一个 Marker 并设置位置，然后添加到地图中
      new mapboxgl.Marker(markerEl)
        .setLngLat(feature.geometry.coordinates) // 设置 Marker 的位置，从 GeoJSON 数据中获取经纬度
        .addTo(this.map); // 添加 Marker 到地图中
  
    // 可以在这里为每个 Marker 添加事件监听器或其他逻辑
  });
  
  
      },
  
  
      
      showFeatureInfo(feature) {
        // 悬停特征时更新悬停的特征信息，但仅当没有点击的特征时才更新
        if (!this.clickedFeature) {
          this.hoveredFeature = feature;
        }
      },
      hideFeatureInfo() {
        // 离开时清除悬停的特征信息，但仅当没有点击的特征时才清除
        if (!this.clickedFeature) {
          this.hoveredFeature = null;
        }
      },
      handleFeatureClick(feature) {
        // 点击特征时更新点击的特征信息
        this.clickedFeature = feature;
        // 同时更新显示的信息内容为点击的特征
        this.showInfoContent = feature;
        // 点击特征时飞到对应的点
        this.flyToPoint(feature.geometry.coordinates);
      },
      handleMapHover(e) {
        const features = this.map.queryRenderedFeatures(e.point, {
          layers: ['fire-geojson-layer']
        });
        if (features.length) {
          this.hoveredFeature = features[0];
        } else {
          this.hoveredFeature = null;
        }
      },
      handleMapClick(e) {
        const features = this.map.queryRenderedFeatures(e.point, {
          layers: ['fire-geojson-layer']
        });
        if (features.length) {
          this.clickedFeature = features[0];
          this.flyToPoint(features[0].geometry.coordinates);
        }
      },
      handleFeatureHover(feature) {
        // 悬停特征时更新显示的信息内容，但仅当没有点击的特征时才更新
        if (!this.clickedFeature) {
          this.showInfoContent = feature;
        }
      },
      handleFeatureLeave() {
        // 离开时清除悬停的特征信息，但仅当没有点击的特征时才清除
        if (!this.clickedFeature) {
          this.showInfoContent = null;
        }
      }
    }
  };
  </script>
  
  <style>
  #map {
    width: 100%;
    height: 100%;
  }
  
  .mapboxgl-ctrl-group {
    display: none;
  }
  
  .mapboxgl-ctrl-attrib-button{
    display: none;
  }
  
  .mapboxgl-ctrl-attrib-inner{
    display: none;
  }
  
  .hover-popup {
    position: relative;
    pointer-events: none;
    background-color: white;
  }
  
  
  
  .mapboxgl-popup {
    max-width: 200px;
  }
  
  .mapboxgl-popup-content {
    text-align: center;
    font-family: 'Open Sans', sans-serif;
  }
  /* 火炬标记样式 */
  /* 火炬标记样式 */
  .point-marker {
    width: 60px;
    height: 60px;
    background-image: url('@/assets/venueslogo1.png'); /* 根据你的火炬图标路径调整 */
    background-size: cover;
    border-radius: 50%;
    position: absolute;
    top: 0; /* 根据设计调整 */
    left: 0; /* 根据设计调整 */
    transform: translate(-50%, -50%);
    pointer-events: none; /* 确保标记不会干扰地图交互 */
    z-index: 1; /* 确保它位于地图之上 */
  }
  
  
  
  </style>
  