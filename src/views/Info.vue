<template>
  <div>
    <mdb-navbar expand="large" dark class="" style="background: #ff9800;">
      <!-- Navbar brand -->
      <mdb-navbar-brand href="#">
        Music Stats
      </mdb-navbar-brand>
      <mdb-navbar-toggler>
        <mdb-navbar-nav>
         <router-link to="/"><mdb-nav-item>Home</mdb-nav-item></router-link>
        </mdb-navbar-nav>
      </mdb-navbar-toggler>
    </mdb-navbar>
    <TrackHeader v-bind:artist="artist" v-bind:trackhead="trackhead" />
    <div style="margin-top: 2rem"></div>
    <div class="pull-left align-left" style="width: 100%">
        <router-link to="/">
      <mdb-btn

        size="lg"
        style="float: left; margin-left: 6%; background: #ff9800"
        class=" -align-left pull-left"
        lg="2"
      >
        Back
      </mdb-btn>
        </router-link>
    </div>
    <div style="margin-top: 7rem"></div>

    <TrackData v-bind:trackhead="trackhead" v-bind:artist="artist" />
    <div style="margin-top: 5rem"></div>
    <mdb-container>
      <mdb-row>
        <mdb-col >
          <mdb-jumbotron class="hoverable">
            <mdb-row >
              <h4 class="mb-3 p-0" style="width: 100%">
                <strong class="font-weight-bold" style="color: #ff9800">Artist Top Tracks</strong>
              </h4>
              <p class="mb-3 p-0" style="width: 100%">
                This is the popularity of artists top 10 tracks in the selected country, select a different country to collect information for that country.
              </p>
              <select
                class="browser-default custom-select"
                v-on:change="selectCountry($event)"
              >
                <option disabled value="">Please select a Country</option>
                <option value="AR">Argentina</option>
                <option value="AU">Australia</option>
                <option value="AT">Austria</option>
                <option value="BE">Belgium</option>
                <option value="BG">Bulgaria</option>
                <option value="BR">Brazil</option>
                <option value="CA">Canada</option>
                <option value="CH">Switzerland</option>
                <option value="CL">Chile</option>
                <option value="CY">Cyprus</option>
                <option value="DE">Germany</option>
                <option value="ES">Spain</option>
                <option value="FI">Finland</option>
                <option value="FR">France</option>
                <option value="GR">Greece</option>
                <option value="ID">Indonesia</option>
                <option value="IE">Ireland</option>
                <option value="MX">Mexico</option>
                <option value="MY">Malaysia</option>
                <option value="NL">Netherlands</option>
                <option value="PL">Poland</option>
                <option value="SG">Singapore</option>
                <option value="US" selected>United States</option>
                <option value="UY">Uruguay</option>
              </select>
            </mdb-row>
            <mdb-row>
              <mdb-container>
                <div id="chart">
                  <apexchart
                    type="bar"
                    height="450"
                    :options="chartOptions"
                    :series="series"
                  />
                </div>
              </mdb-container>
            </mdb-row>
          </mdb-jumbotron>
        </mdb-col>
      </mdb-row>
    </mdb-container>
    <div style="margin-top: 5rem"></div>
    <SoundFeatures v-bind:trackurl="trackurl"/>
    <h4 class="mb-3 p-0" style="width: 100%">
      <strong class="font-weight-bold">All data is from Spotify</strong>
    </h4>
  </div>
</template>

<script>
import TrackHeader from "../components/TrackHeader.vue";
import TrackData from "../components/TrackData.vue";
import SoundFeatures from "../components/SoundFeatures.vue";
import { mapState, mapMutations } from "vuex";
import {
  mdbJumbotron,
  mdbRow,
  mdbContainer,
  mdbCol,
  mdbBtn,
  mdbNavbar,
  mdbNavbarBrand,
  mdbNavbarToggler,
  mdbNavItem,
  mdbNavbarNav
} from "mdbvue";
import axios from "axios";
import VueApexCharts from "vue-apexcharts";

export default {
  name: "Info",
  components: {
    TrackHeader,
    TrackData,
    SoundFeatures,
    mdbContainer,
    mdbCol,
    mdbJumbotron,
    mdbRow,
    mdbBtn,
    mdbNavbarNav,
    mdbNavItem,
    mdbNavbarToggler,
    mdbNavbarBrand,
    mdbNavbar,
    apexchart: VueApexCharts
  },
  props: ["catData"],
  computed: {
    ...mapState([
      "trackId",
      "artistId",
      "bearerId",
      "countryData",
      "chartCategories"
    ])
  },
  data() {
    return {
      series: [
        {
          data: []
        }
      ],
      chartOptions: {
        plotOptions: {
          bar: {
            vertical: true

          }
        },
        dataLabels: {
          enabled: false
        },
        xaxis: {
          categories: this.$store.state.chartCategories
        },
        colors: ['#ff9800']
      },
      trackhead: [],
        trackurl: "",
      artist: [],
      bearerToken: "",
      countryDataProp: [],
      chartData: [],
      trackName: []
    };
  },
  created() {
    this.trackResults(this.trackId);
    this.setData();
  },

  methods: {
    ...mapMutations(["CHANGE_DATA", "CHART_CAT"]),
    async trackResults(id) {
      //get bearer tokken

      const url = "api";
      const response = await axios.post(url);
      const token = response.data;
      this.bearerToken = token;

      const session_url = `api/track/${id}/${token}`;

      const trackResult = await axios.get(session_url);
      this.trackhead = trackResult;

      this.trackurl = trackResult.data.external_urls.spotify;

      //get artist data
      const artistId = trackResult.data.artists[0].id;
      const artist_url = `api/artist/${artistId}/${token}`;

      const artistResult = await axios.get(artist_url);
      this.artist = artistResult;
      //get charts data
      //get artist country data
      const countryCode = "US";
      const countryData_url = `api/toptracks/${artistId}/
      ${this.bearerToken}/${countryCode}`;

      const countryResult = await axios.get(countryData_url);
      this.countryDataProp = countryResult;
      this.countryData.data.tracks.forEach(track => {
        this.chartData.push(track.popularity);
        this.trackName.push(track.name);
      });
    },

    async selectCountry(event) {
      //get artist country data
      const artistId = this.trackhead.data.artists[0].id;
      const artist_url = `api/toptracks/${artistId}/
      ${this.bearerToken}/${event.target.value}`;

      const countryResult = await axios.get(artist_url);
      this.countryDataProp = countryResult;
      this.CHANGE_DATA(this.countryDataProp);
      this.updateData();
    },

    updateData: function() {
      this.chartData = [];
      this.countryData.data.tracks.forEach(track => {
        this.chartData.push(track.popularity);
      });
      this.series[0].data = this.chartData;
      this.chartOptions.xaxis.categories = this.trackName;
    },

    setData: function() {
      this.series[0].data = this.chartData;
    }
  }
};
</script>

<style scoped></style>
