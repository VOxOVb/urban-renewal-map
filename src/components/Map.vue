<template>
    <main>
        <div id="mapContainer" ref="mapContainer" style="width:100%; height: 400px;"></div>
        <ul>
            <li v-for="(location, index) in locations" :key="index">
                {{ location.stop_name }} - 距離: {{ location.distance }} 公里
            </li>
        </ul>
    </main>
</template>

<script>
import { onMounted, ref } from "vue";
import L from "leaflet"
import "leaflet/dist/leaflet.css";

export default {
    name: "Map",
    setup() {
        const mapContainer = ref(null);
        const locations = ref([]);
        let map = null;
        let marker = null;
        let pins = [];
        let polygons = [];
        const initMap = () => {
            map = L.map(mapContainer.value, {
                zoomControl: true,
                zoom: 17,
            }).setView([24.97357326439524,121.44434094429018]);

            L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
                attribution:
                    '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
            }).addTo(map);

            marker = L.marker([24.97357326439524,121.44434094429018])
                .addTo(map)
                .bindPopup("目前位置")
                .openPopup();

            map.on("click", handleMapClick);
            fetchLocations(25.033, 121.5654);
        };

        const updateMarkers = () => {
            clearMarkers();
            locations.value.forEach(location => {
                const pin = L.marker([location.latitude, location.longitude])
                    .addTo(map)
                pins.push(pin);
            });
        };

        const clearMarkers = () => {
            pins.forEach(marker => map.removeLayer(marker));
            pins = [];
        };

        const updatePolygons = (newPolygons) => {
            polygons.forEach(polygon => map.removeLayer(polygon));
            polygons = [];

            newPolygons.forEach(coords => {
                const latLngs = coords.map(ring => ring.map(coord => [coord[1], coord[0]]));
                const polygon = L.polygon(latLngs, { color: "skyblue", fillOpacity: 0.8 }).addTo(map);
                polygons.push(polygon);
            });
        };


        const fetchLocations = async (lat, lng) => {
            console.log(lat,lng)
            try {
                const locationsResponse = await fetch(`https://enterprise.oakmega.ai/api/v1/server/xinbei/calc-distance`, {
                    method: "POST",
                    headers: {
                        "Content-Type": "application/json",
                    },
                    body: JSON.stringify({ lat, lng }),
                });

                if (!locationsResponse.ok) {
                    throw new Error("API 請求失敗");
                }

                const locationsData = await locationsResponse.json();
                if (locationsData.result) {
                    locations.value = locationsData.result
                        .sort((a, b) => a.distance - b.distance)
                    // updateMarkers();
                } else {
                    console.warn("API 回傳的資料中沒有 `result` 欄位");
                    locations.value = [];
                    clearMarkers();
                }
                if (polygons.length === 0) {
                    console.log('s')
                    const polygonResponse = await fetch(
                        "https://enterprise.oakmega.ai/api/v1/server/xinbei/geolocation-json?directory=tucheng.json",
                        { method: "GET" }
                    );

                    const polygonData = await polygonResponse.json();

                    if (polygonData.result.features) {
                        const rawPolygons = polygonData.result.features.map(feature => feature.geometry.coordinates);
                        updatePolygons(rawPolygons);
                    }
                }
            } catch (error) {
                console.error("API 請求錯誤:", error);
            }
        };

        const handleMapClick = async (e) => {
            const { lat, lng } = e.latlng;

            if (marker) {
                marker.setLatLng([lat, lng]);
            } else {
                marker = L.marker([lat, lng]).addTo(map).bindPopup("新位置").openPopup();
            }

            await fetchLocations(lat, lng);
        };

        onMounted(() => {
            initMap();
        });

        return {
            locations,
            mapContainer,
        };
    },
};
</script>
<style>
main {
    width: 100%;
    height: 50%;
    display: grid;
}
</style>