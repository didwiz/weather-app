<template>
  <div class="p-2 text-white mb-8">
    <div class="places-input text-gray-800 ">
      <input
        type="search"
        v-model="location.name"
        id="location"
        class="w-full p-2 rounded-sm"
        placeholder="Start typing an address"
      />
    </div>
    <div
      class="weather-container font-sans w-128 max-w-lg overflow-hidden bg-gray-900 shadow-lg mt-5 rounded-lg"
    >
      <div class="current-weather flex items-center justify-between px-6 py-8">
        <div class="flex items-center">
          <div>
            <div class="text-6xl font-semibold">{{ currentTemp.actual }}°C</div>
            <div>Feels like {{ currentTemp.feels }}°C</div>
          </div>
          <div class="mx-5">
            <div class="font-semibold">{{ currentTemp.summary }}</div>
            <div>{{ location.name }}</div>
          </div>
        </div>

        <div>
          <canvas
            ref="iconCurrent"
            id="iconCurrent"
            width="96"
            height="96"
          ></canvas>
        </div>
      </div>

      <div class="future-weather text-sm bg-gray-800 px-6 py-8 overflow-hidden">
        <div
          v-for="(day, index) in daily"
          :key="day.time"
          :class="{ 'mt-8': index > 0 }"
          v-if="index < 5"
          class="flex items-center"
        >
          <div class="w-1/6 text-lg text-gray-200 uppercase">
            {{ day.time | day }}
          </div>
          <div class="w-4/6 px-4 flex items-center">
            <div>
              <canvas
                :id="`icon${index + 1}`"
                :data-icon="toKebabCase(day.icon)"
                width="24"
                height="24"
              ></canvas>
            </div>
            <div class="ml-3">{{ day.summary }}</div>
          </div>
          <div class="w-1/6 text-right">
            <div>{{ Math.round(day.temperatureHigh) }}°C</div>
            <div>{{ Math.round(day.temperatureHigh) }}°C</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  mounted() {
    this.fetchData();
    let placesAutoComplete = places({
      container: document.querySelector("#location")
    }).configure({
      type: "city",
      aroundLatLngViaIP: false
    });
    placesAutoComplete.on("change", ({ suggestion: e }) => {
      console.log(e);
      this.location.name = e.name;
      this.location.lat = e.latlng.lat;
      this.location.lng = e.latlng.lng;
      this.fetchData();
    });
    placesAutoComplete.on("clear", e => {
      this.location.name = "";
      this.location.lat = 0;
      this.location.lng = 0;
    });
  },
  filters: {
    day(val) {
      const newDate = new Date(val * 1000);
      const days = ["sun", "mon", "tue", "wed", "thur", "fri", "sat"];
      return days[newDate.getDay()];
    }
  },
  data() {
    return {
      currentTemp: {
        actual: 0,
        feels: "",
        summary: "",
        icon: ""
      },
      location: {
        name: "Toronto, Canada",
        lat: 43.6532,
        lng: -79.38323
      },
      daily: []
    };
  },
  methods: {
    fetchData() {
      let skycons = new Skycons({ color: "white" });
      axios
        .get("/api/weather", {
          params: {
            lat: this.location.lat,
            lng: this.location.lng
          }
        })
        .then(({ data: { currently: ct, daily } }) => {
          this.currentTemp.actual = Math.round(ct.temperature);
          this.currentTemp.feels = Math.round(ct.apparentTemperature);
          this.currentTemp.summary = ct.summary;
          this.currentTemp.icon = this.toKebabCase(ct.icon);

          this.daily = daily.data;

          skycons.add("iconCurrent", this.currentTemp.icon);
          skycons.play();

          this.$nextTick(() => {
            skycons.add(
              "icon1",
              document.getElementById("icon1").getAttribute("data-icon")
            );
            skycons.add(
              "icon2",
              document.getElementById("icon2").getAttribute("data-icon")
            );
            skycons.add(
              "icon3",
              document.getElementById("icon3").getAttribute("data-icon")
            );
            skycons.add(
              "icon4",
              document.getElementById("icon4").getAttribute("data-icon")
            );
            skycons.add(
              "icon5",
              document.getElementById("icon5").getAttribute("data-icon")
            );
            skycons.play();
          });
        });
    },
    toKebabCase(string) {
      return string.split(" ").join("-");
    }
  }
};
</script>
