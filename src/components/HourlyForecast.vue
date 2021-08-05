<template>
    <table class="table">
        <tr>
            <th>Hour</th>
            <th v-for="hour in ['00-03', '03-06', '06-09', '09-12', '12-15', '15-18', '18-21', '21-00']">{{ hour }}</th>
        </tr>
        <tr v-for="forecast in forecasts">
            <th>{{ forecast.day.toDateString() }}</th>
            <td v-for="x in forecast.offset" />
            <td v-for="hour in forecast.forecast">
                Arrivals: {{ hour.runway.arrival }}<br>
                Departures: {{ hour.runway.departure }}<br>
                {{ hour.wind_direction }}&deg; at {{ Math.round(hour.wind_speed) }}kts
            </td>
        </tr>
    </table>
</template>
<script>
export default {
    name: "HourlyForecast",
    props: {
        forecasts: {
            type: Array
        }
    }
}
</script>
