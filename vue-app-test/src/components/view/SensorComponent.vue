<template>
  <div id="sensor-component">
    <p>Sensor Component</p>

    <div
      v-for="(sensorid, index) in sensorData"
      :key="index"
      class="sensor-chart"
    >
      <canvas
        :sensor="sensorid"
        :xlabel="sensorDataX[index]"
        :ylabel="sensorDataY[index]"
        :id="sensorid + '-chart'"
        width="400"
        height="400"
      >
      </canvas>
    </div>
  </div>
</template>

<script>
// Components Import
// Packages Import
const axios = require("axios");
const Chart = require("chart.js");
// Style Import
import "../css/SensorComponent.css";
// JS Import
import "../js/SensorComponent.js";

var script = {
  name: "SensorComponent",
  data() {
    return {
      sensorDataX: [],
      sensorDataY: [],
      sensorData: [],
    };
  },
  async created() {
    // fetch the data when the view is created and the data is
    // already being observed
    // this.fetchData();
    this.fetchData2();
  },
  watch: {
    // call again the method if the route changes
    $route: "fetchData2",
  },
  methods: {
    async fetchData() {
      axios
        .get(
          "https://dev.dasstec.ro/api/v2/get-data/sensorId/ilfov?admin=noriel"
        )
        .then((response) => {
          this.sensorData = response.data[0];

          for (var index in response.data[0].sensorIdList) {
            var sensorid = response.data[0].sensorIdList[index];

            axios
              .get(
                "https://dev.dasstec.ro/api/v2/get-data/ilfov/" +
                  sensorid +
                  "?admin=noriel"
              )
              .then((reponse) => {
                // this.sensorData.push(reponse.data[0]);
                var data = reponse.data[0];
                var sensorDataX_aux = [];
                var sensorDataY_aux = [];
                for (var index in data.sensorAverage) {
                  var value = data.sensorAverage[index].sensorValue;
                  var time = data.sensorAverage[index].sensorTime;
                  sensorDataX_aux.push(time);
                  sensorDataY_aux.push(value);
                }
                this.sensorDataY.push(sensorDataY_aux);
                this.sensorDataX.push(sensorDataX_aux);

                // console.log("1")

                return { data, sensorDataY_aux, sensorDataX_aux };

                // return ()
                // console.log("=====")
                // console.log(reponse.data[0].sensorQueried)
                // console.log(reponse.data[0].sensorAverage)
              })
              .then((result) => {
                // console.log("2", result)
                // console.log(this.sensorData.sensorIdList,sensorid)
                // console.log("Test")
                this.createChart(
                  result.data.sensorQueried,
                  result.sensorDataX_aux,
                  result.sensorDataY_aux
                );
              });
          }
        });
    },
    async fetchData2() {
      // const url = "https://dev.dasstec.ro/api/v2/get-data/constanta/sensor22?admin=alexbarbu"
      var url = "https://dev.dasstec.ro/api/vue/influx?query=show series";
      const { data } = await axios.get(url);
      // var json = JSON.parse(data.key)
      // var sensors = []
      data.forEach((item) => {
        var array = item.key.split(",");
        this.sensorData.push(array[5].split("=")[1]);
      });
      // console.log(sensors)

      // Date
      var today = new Date(); // this is -1h romanian timezone
      // cannot query for today date starting at 00:00 because influx tz is -1h than romanian tz
      // set today 00:00 as yesterday 23:00
      today.setDate(today.getDate() - 1);
      var dd = String(today.getDate()).padStart(2, "0");
      var mm = String(today.getMonth() + 1).padStart(2, "0"); //January is 0!
      var yyyy = today.getFullYear();
      today = yyyy + "-" + mm + "-" + dd + "T23:00:00Z";
      dd = (parseInt(dd) + 1).toLocaleString("en-US", {
        minimumIntegerDigits: 2,
        useGrouping: false,
      });
      var todayEnd = "'" + yyyy + "-" + mm + "-" + dd + "T23:00:00Z" + "'";
      // End date

      this.sensorData.forEach(async (sensor) => {
        var sensorid = sensor;
        var whereQuery =
          `where sensorId='` +
          sensorid +
          `' and time>='` +
          today +
          `' and time<=` +
          todayEnd +
          ``;
        var url =
          "https://dev.dasstec.ro/api/vue/influx?query=select mean(value) as value from sensors " +
          whereQuery +
          " GROUP BY time(5m) ORDER BY time DESC";
        // console.log(url);
        axios
          .get(url)
          .then((response) => {
            // console.log(response.config.url);
            // console.log(response.data);
            var xAux = [];
            var yAux = [];
            response.data.forEach((item) => {
              xAux.push(item.time);
              yAux.push(item.value);
            });
            return [response.config.url, xAux, yAux];
          })
          .then((data) => {
            var sensorid = data[0].split("sensorId='")[1].split("' and")[0];
            this.createChart(sensorid, data[1], data[2]);
          });
      });
    },
    createChart(chartId, chartDataX, chartDataY) {
      // console.log("3", chartId, chartDataX.length, chartDataY.length)
      const ctx = document.getElementById(chartId + "-chart");
      // console.log(ctx);
      var options = {
        animation: false,
        responsive: true,
        maintainAspectRatio: false,
        drawBorder: false,
        // onResize: console.log("chart resize"),
        legend: {
          labels: {
            fontColor: "white",
          },
        },
        scales: {
          xAxes: [
            {
              type: "time",
              time: {
                unit: "minute",
              },
              distribution: "series",
              gridLines: {
                color: "rgba(0, 0, 0, 0)",
              },
              ticks: {
                fontColor: "white",
              },
            },
          ],
          yAxes: [
            {
              ticks: {
                min: 15,
                // beginAtZero: false,
                fontColor: "white",
              },
              gridLines: {
                color: "#415f7d",
                zeroLineColor: "#415f7d",
              },
            },
          ],
        },
      };
      const myChart = new Chart(ctx, {
        type: "line",
        data: {
          labels: chartDataX,
          datasets: [
            {
              label: chartId,
              data: chartDataY,
              // new design
              backgroundColor: "rgba(51, 153, 255, 0.2)",
              borderColor: "rgba(51, 153, 255, 1)",
              pointBorderColor: "#343a40",
              pointBackgroundColor: "rgba(51, 153, 255, 1)",
              pointHoverBackgroundColor: "#ffc107",
              pointRadius: 1.5,
              pointHoverRadius: 7,
              pointBorderWidth: 1,
              borderWidth: 1,
              lineTension: 0.2,
            },
          ],
        },
        options,
      });
      myChart;
    },
  },
  // mounted() {
  //   console.log(this.sensorData)
  //   // this.createChart(this.sensorDat + "-chart", sensorDataX_aux, sensorDataY_aux);
  // },
};

export default script;
</script>

<style>
</style>
