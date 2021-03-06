<template>
  <div>
    <!-- StatsBox{start} -->
    <md-card class="md-layout md-alignment-center">
      <md-content class="md-layout-item stat-box md-medium-size-33 md-small-size-50 md-xsmall-size-100" style="background: rgb(255, 131, 29);">
        <h2>National statistics</h2>
        Help Seeking Rate <span class="stat-val">23%</span><br>
        Young Australians Reached <span class="stat-val">2,988,390</span><br>
        Total Number of Schools <span class="stat-val">4,177</span><br>
      </md-content>
      <md-content class="md-layout-item stat-box md-medium-size-33 md-small-size-50 md-xsmall-size-100" style="background: rgb(0, 104, 165);">
        <h2>Current <b>batyr</b> statistics</h2>
        Help Seeking Rate <span class="stat-val">69%</span><br>
        Young Australians Reached <span class="stat-val">85,000+</span><br>
        Number of Schools Visited <span class="stat-val">232</span><br>
      </md-content>
      <md-content class="md-layout-item stat-box md-medium-size-33 md-small-size-50 md-xsmall-size-100" style="background: rgb(0, 165, 222);">
        <h2><b>batyr</b> 2022 target</h2>
          Help Seeking Rate <span class="stat-val">75%</span><br>
          Young Australians Reached <span class="stat-val">1 Million</span><br>
          Number of Schools Visited <span class="stat-val">1000+</span><br>
      </md-content>
    </md-card>
    <!-- StatsBox{end} -->
    <!-- SearchBox{start} -->
    <md-toolbar>
      <div class="md-toolbar-row">
        <md-autocomplete
          v-model="schoolInput"
          :md-options="batyrIndexNames"
        >
          <label>Search by school name</label>
          <template slot="md-autocomplete-item" slot-scope="{ item, term }">
            <md-highlight-text :md-term="term">{{ item }}</md-highlight-text>
          </template>
          <template slot="md-autocomplete-empty" slot-scope="{ term }">
            No schools matching "{{ term }}" were found. <a target="_blank" href="http://www.batyr.com.au/school-programs/#wpcf7-f152-p97-o1">Click here</a> to request a visit!
          </template>
        </md-autocomplete>
        <md-button
          class="md-raised searchButton"
          @click="findSchool()"
        >
          Find School
        </md-button>
      </div>
    </md-toolbar>
    <!-- SearchBox{end} -->
    <!-- GoogleMap{start} -->
    <md-card>
      <gmap-map
        v-if="googleMapsInitialized"
        :center='currCoord'
        :zoom='zoomLevel'
        map-type-id='terrain'
        class='gmap-map'
      >
        <gmap-marker
          v-for="school in schools"
          :icon="icon"
          :key="school.id"
          :position="getCoordinates(school)"
          :clickable="true"
          @click="showSchool(school)"
        >
        </gmap-marker>
        <!-- <gmap-info-window
          v-if="currSchool"
          :position="getCoordinates(currSchool)"
        >
          {{currSchool}}
        </gmap-info-window> -->
      </gmap-map>
    </md-card>
    <!-- GoogleMap{end} -->
    <!-- InfoBox{start} -->
    <md-card class="info-box">
      <div v-if="currSchool">
        <h1>{{currSchool.name}}</h1>
        <h3>{{currSchool.address}}</h3>
        <span v-if="currSchool.students_attended"><b>Students attended</b>: {{currSchool.students_attended}}<br /></span>
        <span v-if="currSchool.students_surveyed"><b>Students surveyed</b>: {{currSchool.students_surveyed}}<br /></span>
        <span v-if="currSchool.engagement"><b>Engagement</b>: {{currSchool.engagement}}%<br /></span>
        <span v-if="currSchool.help_seeking"><b>Help seeking</b>: {{currSchool.help_seeking}}%<br /></span>
      </div>
      <div v-if="!currSchool">
        <md-empty-state
          class="md-primary"
          md-label="Can't find your school?"
          md-description="batyr@school programs aim to remove the stigma around mental health and engage, educate and empower young people to reach out for support when they need it."
        >
          <md-button
            href="http://www.batyr.com.au/school-programs/#wpcf7-f152-p97-o1"
            target="_blank"
            class="md-raised md-primary"
          >
            Request a program at your school
          </md-button>
        </md-empty-state>
      </div>
    </md-card>
    <!-- InfoBox{end} -->
    <small>
      <br>* Slade, T., et al., The mental health of Australians 2: Report on the 2007 national survey of mental health and wellbeing. 2009, Canberra, Australia: Department of Health and Ageing.
      <br>“Service use was lowest among the youngest age groups with less than one quarter of people having used services for mental health problems in the previous 12 months (23.3% aged 16-34 years).”
      <br>** <a href=" http://www.abs.gov.au/AUSSTATS/abs@.nsf/mediareleasesbyReleaseDate/AC02F0705E320F58CA25817C00016A47?OpenDocument">2016 Census - a ‘selfie’ of young people in Australia</a>
      <br>** <a href="http://www.abs.gov.au/ausstats/abs@.nsf/mf/4221.0">4221.0 - Schools, Australia, 2017</a>
    </small>
  </div>
</template>

<script>
/* eslint-disable */
import { loaded } from 'vue2-google-maps';
import fecha from 'fecha';
import SchoolsData from '../assets/data.json';
import MapMarker from '../assets/mapmarker.svg';

export default {
  name: 'Home',
  data() {
    return {
      schoolInput: null,
      batyrIndexNames: SchoolsData.map(school => school.name),
      icon: MapMarker,
      googleMapsInitialized: false,
      zoomLevel: 11,
      schools: SchoolsData,
      currPlace: null,
      currSchool: null,
      currCoord: { lat: -33.865143, lng: 151.209900 },
    }
  },
  methods: {
    clearPlace() {
      this.currPlace = null;
      this.currSchool = null;
    },
    formatDate(date) {
      return fecha.format(new Date(date), 'MMMM Do, YYYY');
    },
    getCoordinates (school) {
      if (!school) return;
      return { lng: school.longitude, lat: school.latitude }
    },
    zoomIn() {
      this.zoomLevel = 10;
      setTimeout(() => {
        this.zoomLevel = 17;
      }, 10);
    },
    showSchool(school) {
      this.currCoord = this.getCoordinates(school);
      this.currSchool = school;
      this.zoomIn();
    },
    findSchool() {
      SchoolsData.forEach(school => {
        if (school.name === this.schoolInput) {
          this.showSchool(school);
        }
      })
    },
  },
  async mounted() {
   loaded.then(()=>{
     this.googleMapsInitialized = true;
   });
  },
};
</script>

<style scoped>
.home {
  width: 1024px;
}
/* Misc */
.stat-val {
  font-size: 1.4em;
}
/* Boxes */
.info-box {
  min-height: 250px;
  padding: 1em;
}
.stat-box {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  min-height: 250px;
  color: white;
}
/* Components */
.searchButton {
  height: 45px;
}
.gmap-map {
  height: 400px;
}
.md-card {
  min-height: 100px;
}
</style>
