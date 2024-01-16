<template>
  <div style="width:70%;height:70%">
    <MglMap
      ref="mapboxMap"
      :access-token="accessToken"
      :map-style.sync="mapStyle"
      :center="center"
      :zoom="zoom"
      style="width: 100%; height: 100vh"
    >
      <!-- <MglMarker :coordinates="markerPosition" :options="markerOptions" /> -->
      <MglMarker
        v-for="(feature, index) in geojson.features"
        :key="index"
        :coordinates="feature.geometry.coordinates"
      >
        <template slot="marker">
          <div :class="['marker', 'marker-' + index]">
            <img :src="'/images/img-' + (index + 1) + '.svg'" alt="" />
          </div>
        </template>
        <MglPopup>
          <div>
            <h3>{{ feature.properties.title }}</h3>
            <p>{{ feature.properties.description }}</p>
          </div>
        </MglPopup>
      </MglMarker>
      <div
        class="search-box"
        style="position: absolute; top: 10px; left: 10px; padding: 7px"
      >
        <!-- @blur="clearSuggestions" -->
        <input
          v-model="searchQuery"
          @input="handleSearch"
          placeholder="Search for a location"
          style="
            border: 1px solid gray;

            padding: 7px;
            border-radius: 5px;
            outline: none;
          "
        />
        <ul v-if="suggestions.length">
          <li
            v-for="(suggestion, index) in suggestions"
            :key="index"
            @click="selectSuggestion(suggestion)"
          >
            {{ suggestion.place_name }}
          </li>
        </ul>
      </div>
    </MglMap>
  </div>
</template>

<script>
import { MglMap, MglMarker, MglPopup } from "vue-mapbox";
import "mapbox-gl/dist/mapbox-gl.css";

export default {
  name: "MapBox",
  components: {
    MglMap,
    MglMarker,
    MglPopup,
  },
  data() {
    return {
      accessToken:
        "pk.eyJ1Ijoic2hvcnRkaXYiLCJhIjoiY2l3OGc5YmE5MDJzZjJ5bWhkdDZieGdzcSJ9.1z-swTWtcCHYI_RawDJCEw",
      mapStyle: "mapbox://styles/mapbox/streets-v12",
      center: [-96, 37.8],
      zoom: 5,
      markerPosition: [],
      geojson: {
        type: "FeatureCollection",
        features: [
          {
            type: "Feature",
            geometry: {
              type: "Point",
              coordinates: [-99.8628, 37.752], // Dodge City, Kansas
            },
            properties: {
              title: "Mapbox",
              description: "Dodge City, Kansas",
              iconUrl: "/profile.svg",
            },
          },
          {
            type: "Feature",
            geometry: {
              type: "Point",
              coordinates: [-94.5786, 39.0997], // Kansas City, Missouri
            },
            properties: {
              title: "Mapbox",
              description: "Kansas City, Missouri",
              iconUrl: "https://placekitten.com/g/60/60",
            },
          },
          {
            type: "Feature",
            geometry: {
              type: "Point",
              coordinates: [-100.8628, 37.752], // Dodge City, Kansas
            },
            properties: {
              title: "Mapbox",
              description: "Dodge City, Kansas",
              iconUrl: "/profile.svg",
            },
          },
          {
            type: "Feature",
            geometry: {
              type: "Point",
              coordinates: [-80.5786, 39.0997], // Kansas City, Missouri
            },
            properties: {
              title: "Mapbox",
              description: "Kansas City, Missouri",
              iconUrl: "https://placekitten.com/g/60/60",
            },
          },
          {
            type: "Feature",
            geometry: {
              type: "Point",
              coordinates: [-98.5786, 39.0997], // Kansas City, Missouri
            },
            properties: {
              title: "Johni Sinse",
              description: "special doctor for special patients",
              iconUrl: "https://placekitten.com/g/60/60",
            },
          },
        ],
      },
      suggestions: [],
      searchQuery: "",
      markerOptions: {
        offset: [0, -30], // Adjust the marker's offset as needed
        color: "#d9534f", // Marker color
        animation: true, // Enable marker animation
      },
    };
  },
  methods: {
    async handleSearch() {
      if (!this.searchQuery) {
        this.clearSuggestions();
        return;
      }

      try {
        // Use Mapbox Geocoding API to get suggestions based on searchQuery
        const response = await fetch(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${encodeURIComponent(
            this.searchQuery
          )}.json?access_token=${this.accessToken}`
        );

        const data = await response.json();

        if (data.features.length > 0) {
          this.suggestions = data.features;
        } else {
          this.clearSuggestions();
        }
      } catch (error) {
        console.error("Error fetching geocoding data:", error);
        // Handle error
        this.clearSuggestions();
      }
    },
    clearSuggestions() {
      this.suggestions = [];
    },
    selectSuggestion(suggestion) {
      console.log(this.$refs.mapboxMap);
      const coordinates = suggestion.geometry.coordinates;
      // this.center = [coordinates[0], coordinates[1]];
      this.markerPosition = [coordinates[0], coordinates[1]];
      // Update map center

      this.$refs.mapboxMap.map.easeTo({
        zoom: 0, 
        duration: 2000, 
      });

      setTimeout(() => {
        this.$refs.mapboxMap.map.easeTo({
          center: [coordinates[0], coordinates[1]],
          zoom: 12, 
          duration: 4000,
        });

        // Set marker position with animation
        this.markerPosition = [coordinates[0], coordinates[1]];
      }, 1000);
      this.searchQuery = suggestion.place_name; // Set the input field to the selected suggestion
      this.clearSuggestions();
    },
  },
};
</script>

<style scoped>
.marker {
  background-size: cover;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  cursor: pointer;
}
.marker img {
  width: 60px;
  height: 60px;
  border-radius: 50%;
}
/* .marker img.joni {
  width: 60px;
  height: 60px;
  border: 6px solid #fff;
} */
.marker-0 {
  /* background-image: url('https://pbs.twimg.com/profile_banners/29773272/1704299219/1080x360'); */
  /* Additional styling for testing */
  /* border: 2px solid red; */
}

.marker-1 {
  /* background-image: url('https://pbs.twimg.com/profile_banners/29773272/1704299219/1080x360'); */
  /* Additional styling for testing */
  /* border: 2px solid blue; */
}
.mapboxgl-popup {
  max-width: 200px;
}

.mapboxgl-popup-content {
  text-align: center;
  font-family: "Open Sans", sans-serif;
}
.search-box {
  position: absolute;
  top: 10px;
  left: 10px;
  padding: 7px;
}
.search-box ul {
  background: #fff;
  padding: 10px;
}
.search-box ul li {
  cursor: pointer;
  list-style-type: none;
}
</style>
