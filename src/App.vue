<template>
  <div id="app">
    <div class="container text-center">
        <h1>Which runway(s) will Heathrow be using?</h1>
        <h6>Heathrow should currently be taking off from the {{ current.departure.runway }}ern runway heading towards {{ current.departure.direction }} and landing on the {{ current.arrival.runway }}ern runway over {{ current.arrival.direction }}</h6>
        <hr>
    </div>
    <div class="container">
        <div class="row">
            <div v-for="day in ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']" class="col"><h3>{{ day }}</h3></div>
        </div>
        <div class="row" v-for="week in weeks">
            <div v-for="index in week.offset_days" class="col"></div>
            <div v-for="forecast in week.forecasts" class="col">
                <h5 :style="(forecast.easterly ? 'color:green' : '')">{{ forecast.date }}</h5>
                <p>Departures: {{ forecast.runways.departures[0] }}/{{ forecast.runways.departures[1] }}</p>
                <p>Arrivals: {{ forecast.runways.arrivals[0] }}/{{ forecast.runways.arrivals[1] }}</p>
            </div>
            <div v-for="index in week.offset_days_post" class="col"></div>
        </div>
    </div>
    <div class="container">
        <hr>
        <div class="row">
            <div class="col-lg-4">
                <h3>Runway Alternation</h3>
                <p>At 3pm each day, the runway used for departures and landings alternates. This means that the runway used before 3pm for arrivals will become the runway used for departures. This alternates each week, so one week departures may be on the southern runway before 3pm with the next week having departures on the southern runway after 3pm. Runway alternation does not take place on easterly operations.</p>
            </div>
            <div class="col-lg-4">
                <h3>Wind Direction</h3>
                <p>Aircraft must take off and land into the wind. If the wind is coming from the east, Heathrow switches to what is known as easterly operations and aircraft take off to the east (i.e. over London). Alternation does not take place on easterly operations as the taxiway infrastructure is insufficient. The infrastructure was not developed as a result of the Cranford Agreement, which whilst it has since expired, prevented aircraft from taking off over Cranford (i.e. 09L departures). Heathrow operates on a 'westerly preference'. This means that even if the wind is coming from the east, if the winds are light (less than 5kts, generally), Heathrow will continue to operate on westerly operations.</p>
            </div>
            <div class="col-lg-4">
                <h3>More Info</h3>
                Live updates: <a href="https://twitter.com/HeathrowNoise">@HeathrowNoise</a><br>
                More information: <a href="https://www.heathrow.com/company/local-community/noise/operations">heathrow.com</a>
            </div>
        </div>
    </div>
  </div>
</template>
<script>
import 'bootstrap/dist/css/bootstrap.min.css'
export default {
  name: 'App',
  components: {
  },
  data: function() {
      return {
        weeks: [{forecasts:[],offset_days:0,offset_days_post:0}],
        current: {departure: {}, arrival: {}}
     }
  },
  mounted() {
        Date.prototype.getWeekNumber = function(){
            let d = new Date(Date.UTC(this.getFullYear(), this.getMonth(), this.getDate()));
            let dayNum = d.getUTCDay() || 7;
            d.setUTCDate(d.getUTCDate() + 4 - dayNum);
            let yearStart = new Date(Date.UTC(d.getUTCFullYear(),0,1));
            return Math.ceil((((d - yearStart) / 86400000) + 1)/7)
        };

    fetch("https://api.weatherbit.io/v2.0/forecast/daily?&city=London&country=UK&key=2e562a75b5934c92b46a0146ba921871").then(response => response.json())
    .then(data => {
        let months = ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"]

        let offsetMap = [6, 0, 1, 2, 3, 4, 5];
        this.weeks[0].offset_days = offsetMap[new Date(data.data[0].datetime).getDay()]

        let singleRunwayOperations = true;

        let weekOffset = new Date().getWeekNumber();
        

        for(let forecast of data.data) {
            let date = new Date(forecast.datetime);
            let isEvenWeek = (date.getWeekNumber() % 2 == 0);
            let easterly = forecast.wind_dir > 22.5 && forecast.wind_dir < 157.5 && forecast.wind_gust_spd >= 3;
            let runways = {};
            if(singleRunwayOperations) {
                if(isEvenWeek && !easterly) runways = {departures: ["27R", "27L"], arrivals: ["27L", "27R"]};
                if(isEvenWeek && easterly) runways = {departures: ["09R", "09R"], arrivals: ["09L", "09L"]};
                if(!isEvenWeek && !easterly) runways = {departures: ["27L", "27R"], arrivals: ["27R", "27L"]};
                if(!isEvenWeek && easterly) runways = {departures: ["09R", "09R"], arrivals: ["09L", "09L"]};
            }

            if(this.current.departure.direction == null) {
                let isAfter3 = new Date().toLocaleString("en-GB", {hour: "2-digit", hour12: false, timeZone: "Europe/London"}) >= 15 ? 1 : 0;

                this.current.departure.direction = easterly ? "London" : "Windsor";
                this.current.arrival.direction = easterly ? "Windsor" : "London";

                this.current.arrival.runway = (runways.arrivals[isAfter3] == "09L" || runways.arrivals[isAfter3] == "27R") ? "north" : "south";
                this.current.departure.runway = (runways.departures[isAfter3] == "09L" || runways.departures[isAfter3] == "27R") ? "north" : "south";
            }
            console.log("offset: " + (date.getWeekNumber()-weekOffset));
            if(this.weeks[date.getWeekNumber()-weekOffset] == null) {
                this.weeks[date.getWeekNumber()-weekOffset] = {forecasts:[], offset_days: 0, offset_days_post: 0}
            }
            this.weeks[date.getWeekNumber()-weekOffset].forecasts.push({date: date.getDate() + "/" + months[date.getMonth()], easterly: easterly, runways: runways})
        }
        this.weeks[this.weeks.length-1].offset_days_post = 7 - this.weeks[this.weeks.length-1].forecasts.length
    });
  }
}
</script>