<template>
  <asyncPageWithOptions></asyncPageWithOptions>
  <!-- <SensorComponent></SensorComponent> -->

  <!-- <div v-for="(record, index) in info.sensorAverage" :key=index>
    <p>{{record.sensorValue}} {{record.sensorTime}}</p>
  </div> -->
</template>
<script>
// Components Import
import SensorComponent from "./components/view/SensorComponent";
import ErrorComponent from "./components/view/ErrorComponent.vue";
import LoadingComponent from "./components/view/LoadingComponent.vue";
// Packages Import
// const axios = require("axios");
import { defineAsyncComponent } from "vue";
// Style Import
import "./main.css";

// Async component with options
const asyncPageWithOptions = defineAsyncComponent({
  loader: () => import("./components/view/SensorComponent.vue"),
  delay: 200,
  timeout: 3000,
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent,
});

var script = {
  name: "App",
  components: {
    SensorComponent,
    asyncPageWithOptions,
  },
  data() {
    return {
      info: "",
    };
  },
  async created() {
    // fetch the data when the view is created and the data is
    // already being observed
    // this.fetchData();
  },
  watch: {
    // call again the method if the route changes
    // $route: "fetchData",
  },
  methods: {
    // async fetchData() {
    //   axios
    //     .get(
    //       "https://dev.dasstec.ro/api/v2/get-data/constanta/sensor22?admin=alexbarbu"
    //     )
    //     .then((response) => {
    //       this.info = response.data[0];
    //       // console.log(response.data[0])
    //     });
    // },
  },
  mounted() {},
};

export default script;
</script>

<style>
</style>
