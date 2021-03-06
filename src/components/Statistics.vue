<template>
  <div class="statistics mt-4">
    <h2>Total Cases Worldwide</h2>
    <p>Last updated: {{ formatDate(totalStatisticsLastUpdated) }}</p>
    <div class="container chart-container">
      <BarChart v-if="loaded1" :chart-data="casesChart" :options="options" />
    </div>
    <br />
    <br />
    <br />
    <br />
    <br />
    <h2>Top 5 Infected Countries</h2>
    <p>Last updated: {{ formatDate(topFiveLastUpdated) }}</p>
    <div class="container chart-container">
      <BarChart v-if="loaded2" :chart-data="casesByCountryChart" :options="options" />
    </div>
  </div>
</template>

<script>
import BarChart from "./BarChart.vue";
import dayjs from "dayjs";

export default {
  name: "Statistics",
  components: {
    BarChart
  },
  data() {
    return {
      loaded1: false,
      loaded2: false,
      // Totals
      totalCases: 0,
      totalDeaths: 0,
      totalRecovered: 0,
      // Last Updates
      totalStatisticsLastUpdated: "",
      topFiveLastUpdated: "",
      // Cases By Country
      casesByCountryStats: [],
      // Confirmed
      countryOneCases: 0,
      countryTwoCases: 0,
      countryThreeCases: 0,
      countryFourCases: 0,
      countryFiveCases: 0,
      // Recovered
      countryOneRecovered: 0,
      countryTwoRecovered: 0,
      countryThreeRecovered: 0,
      countryFourRecovered: 0,
      countryFiveRecovered: 0,
      // Deaths
      countryOneDeaths: 0,
      countryTwoDeaths: 0,
      countryThreeDeaths: 0,
      countryFourDeaths: 0,
      countryFiveDeaths: 0,
      // Names
      countryOneName: "",
      countryTwoName: "",
      countryThreeName: "",
      countryFourName: "",
      countryFiveName: "",
      // ApiKey
      apiKey: process.env.VUE_APP_API_KEY
    };
  },

  async mounted() {
    // Fetches world statistics
    fetch(
      "https://coronavirus-monitor.p.rapidapi.com/coronavirus/worldstat.php",
      {
        method: "GET",
        headers: {
          "x-rapidapi-host": "coronavirus-monitor.p.rapidapi.com",
          "x-rapidapi-key": this.apiKey
        }
      }
    )
      .then(response => {
        return response.json();
      })
      .then(data => {
        this.totalCases = parseInt(data.total_cases.replace(/,/g, "")) + "";
        this.totalDeaths = parseInt(data.total_deaths.replace(/,/g, "")) + "";
        this.totalRecovered =
          parseInt(data.total_recovered.replace(/,/g, "")) + "";
        this.totalStatisticsLastUpdated = data.statistic_taken_at;

        this.loaded1 = true;
      })
      .catch(err => {
        console.log(err);
      });

    // Fetches cases by country
    fetch(
      "https://coronavirus-monitor.p.rapidapi.com/coronavirus/cases_by_country.php",
      {
        method: "GET",
        headers: {
          "x-rapidapi-host": "coronavirus-monitor.p.rapidapi.com",
          "x-rapidapi-key": this.apiKey
        }
      }
    )
      .then(response => {
        return response.json();
      })
      .then(data => {
        this.casesByCountryStats = data.countries_stat;

        // Converts the # of cases into an int so that we can sort numerically
        this.convertJsonStringToInt();

        // Sort by number of cases numerically and store it in the array
        this.casesByCountryStats.sort(this.sortData("cases", true, parseInt));

        // Top 5 Cases by Country
        this.countryOneCases =
          parseInt(this.casesByCountryStats[0].cases.replace(/,/g, "")) + "";
        this.countryTwoCases =
          parseInt(this.casesByCountryStats[1].cases.replace(/,/g, "")) + "";
        this.countryThreeCases =
          parseInt(this.casesByCountryStats[2].cases.replace(/,/g, "")) + "";
        this.countryFourCases =
          parseInt(this.casesByCountryStats[3].cases.replace(/,/g, "")) + "";
        this.countryFiveCases =
          parseInt(this.casesByCountryStats[4].cases.replace(/,/g, "")) + "";

        this.countryOneName = this.casesByCountryStats[0].country_name;
        this.countryTwoName = this.casesByCountryStats[1].country_name;
        this.countryThreeName = this.casesByCountryStats[2].country_name;
        this.countryFourName = this.casesByCountryStats[3].country_name;
        this.countryFiveName = this.casesByCountryStats[4].country_name;

        this.countryOneRecovered =
          parseInt(
            this.casesByCountryStats[0].total_recovered.replace(/,/g, "")
          ) + "";
        this.countryTwoRecovered =
          parseInt(
            this.casesByCountryStats[1].total_recovered.replace(/,/g, "")
          ) + "";
        this.countryThreeRecovered =
          parseInt(
            this.casesByCountryStats[2].total_recovered.replace(/,/g, "")
          ) + "";
        this.countryFourRecovered =
          parseInt(
            this.casesByCountryStats[3].total_recovered.replace(/,/g, "")
          ) + "";
        this.countryFiveRecovered =
          parseInt(
            this.casesByCountryStats[4].total_recovered.replace(/,/g, "")
          ) + "";

        this.countryOneDeaths =
          parseInt(this.casesByCountryStats[0].deaths.replace(/,/g, "")) + "";
        this.countryTwoDeaths =
          parseInt(this.casesByCountryStats[1].deaths.replace(/,/g, "")) + "";
        this.countryThreeDeaths =
          parseInt(this.casesByCountryStats[2].deaths.replace(/,/g, "")) + "";
        this.countryFourDeaths =
          parseInt(this.casesByCountryStats[3].deaths.replace(/,/g, "")) + "";
        this.countryFiveDeaths =
          parseInt(this.casesByCountryStats[4].deaths.replace(/,/g, "")) + "";

        this.topFiveLastUpdated = data.statistic_taken_at;

        this.loaded2 = true;
      })
      .catch(err => {
        console.log(err);
      });
  },
  methods: {
    formatDate(date) {
      return dayjs(date)
        .toDate()
        .toLocaleDateString("en-US", { timezone: "America/Los_Angeles " });
    },
    convertJsonStringToInt() {
      this.casesByCountryStats.forEach(function(item) {
        item.cases = item.cases.replace(/,/g, "");
      });
    },
    sortData(field, reverse, primer) {
      const key = primer
        ? function(x) {
            return primer(x[field]);
          }
        : function(x) {
            return x[field];
          };

      reverse = !reverse ? 1 : -1;

      return function(a, b) {
        return (a = key(a)), (b = key(b)), reverse * ((a > b) - (b > a));
      };
    }
  },

  computed: {
    casesChart() {
      return {
        labels: ["Total Cases"],
        datasets: [
          {
            label: "Confirmed",
            backgroundColor: "rgba(52,152,221, 0.6)",
            data: [this.totalCases]
          },
          {
            label: "Recovered",
            backgroundColor: "rgba(46,204,119, 0.6)",
            data: [this.totalRecovered]
          },
          {
            label: "Deaths",
            backgroundColor: "rgb(255,48,46, 0.6)",
            data: [this.totalDeaths]
          }
        ]
      };
    },
    casesByCountryChart() {
      return {
        labels: [
          this.countryOneName,
          this.countryTwoName,
          this.countryThreeName,
          this.countryFourName,
          this.countryFiveName
        ],
        datasets: [
          {
            label: "Confirmed",
            backgroundColor: "rgba(52,152,221, 0.6)",
            data: [
              this.countryOneCases,
              this.countryTwoCases,
              this.countryThreeCases,
              this.countryFourCases,
              this.countryFiveCases
            ]
          },
          {
            label: "Recovered",
            backgroundColor: "rgba(46,204,119, 0.6)",
            data: [
              this.countryOneRecovered,
              this.countryTwoRecovered,
              this.countryThreeRecovered,
              this.countryFourRecovered,
              this.countryFiveRecovered
            ]
          },
          {
            label: "Deaths",
            backgroundColor: "rgb(255,48,46, 0.6)",
            data: [
              this.countryOneDeaths,
              this.countryTwoDeaths,
              this.countryThreeDeaths,
              this.countryFourDeaths,
              this.countryFiveDeaths
            ]
          }
        ]
      };
    },
    options() {
      return {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          yAxes: [
            {
              type: this.scale,
              ticks: {
                precision: 0,
                fontColor: "#829AB1"
              }
            }
          ],
          xAxes: [
            {
              ticks: {
                fontColor: "#829AB1"
              }
            }
          ]
        }
      };
    }
  }
};
</script>

<style>
.chart-container {
  background: #fff;
}
</style>