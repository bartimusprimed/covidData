<script>
  import { onMount } from "svelte";
  let data = [];
  let total_deaths = 0;
  let total_confirmed = 0;
  let total_active = 0;
  let total_recovered = 0;
  let total_fatality_rate = 0;
  let total_recovery_rate = 0;
  let url =
    "https://services9.arcgis.com/N9p5hsImWXAccRNI/arcgis/rest/services/Z7biAeD8PAkqgmWhxG2A/FeatureServer/1/query?f=json&where=Confirmed%20%3E%200&outFields=Country_Region,Confirmed,Deaths,Recovered,Active&orderByFields=Confirmed%20desc";

  function getTotalFatalities(item) {
    return (item.Deaths / item.Confirmed) * 100;
  }

  function getTotalRecoveries(item) {
    return (item.Recovered / item.Confirmed) * 100;
  }

  onMount(async function getData() {
    alert("Data is being loaded from: Johns Hopkins Arcgis Chart");
    const resp = await fetch(url);
    var combined = {};
    var temp = await resp.json();
    var records = temp["features"];
    records.map(item => {
      item = item["attributes"];
      var build_item = {
        Confirmed: item.Confirmed,
        Active: item.Active,
        Deaths: item.Deaths,
        Recovered: item.Recovered
      };
      if (item.Country_Region in combined) {
        combined[item.Country_Region].Confirmed += build_item.Confirmed;
        combined[item.Country_Region].Active += build_item.Active;
        combined[item.Country_Region].Deaths += build_item.Deaths;
        combined[item.Country_Region].Recovered += build_item.Recovered;
      } else {
        combined[item.Country_Region] = build_item;
      }
      total_deaths += item.Deaths;
      total_confirmed += item.Confirmed;
      total_active += item.Active;
      total_recovered += item.Recovered;
    });
    total_fatality_rate = (total_deaths / total_confirmed) * 100;
    total_recovery_rate = (total_recovered / total_confirmed) * 100;
    data = combined;
  });
</script>

<style>
  main {
    font-family: sans-serif;
    text-align: center;
  }
</style>

<main class="container is-padded">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@0.8.0/css/bulma.min.css">
    <script defer src="https://use.fontawesome.com/releases/v5.3.1/js/all.js"></script>
<div>

<h1 class="title">
Legend
</h1>
<table class="table is-bordered is-narrow is-hoverable">
<thead>
<tr>
<th class="has-background-danger">
Higher than Average Fatality
</th>
<th class="has-background-warning">
Lower than Average Recovery
</th>
<th class="has-background-success">
Higher than Average Recovery OR Lower than Average Fatality
</th>
<th class="has-background-link">
More Active than Recovered
</th>
<th class="has-background-primary">
More Recovered than Active
</th>
</tr>
</thead>
</table>
</div>
<div>
<h1 class="title">
Totals:
</h1>
<table class="table is-bordered is-narrow is-hoverable is-fullwidth">
<thead>
<tr>
<th>Confirmed</th>
<th>Active</th>
<th>Recovered</th>
<th>Deaths</th>
<th>Fatality Rate</th>
<th>Recovery Rate</th>
</tr>
</thead>
<tbody>
<tr>
<td>
{total_confirmed}
</td>
<td>
{total_active}
</td>
<td>
{total_recovered}
</td>
<td>
{total_deaths}
</td>
<td>
{total_fatality_rate.toFixed(2)}
</td>
<td>
{total_recovery_rate.toFixed(2)}
</td>
</tr>
</tbody>
</table>
</div>
<br>
<div>
  <h1 class="title">
  By Country:
  </h1>
  <table class="table is-bordered is-narrow is-hoverable is-fullwidth">
  <thead>
  <tr>
  <th>Country Name</th>
  <th>Confirmed</th>
  <th>Active</th>
  <th>Recovered</th>
  <th>Deaths</th>
  <th>Fatality Rate</th>
  <th>Recovery Rate</th>
  </tr>
  </thead>
  <tbody>
  {#each Object.keys(data) as item}
      <tr>
      <td>
      {String(JSON.stringify(item)).replace(/"/g, "")}
      </td>
      <td>
      {JSON.stringify(data[item]["Confirmed"])}
      </td>
      <td class="{
          data[item]["Recovered"] < data[item]["Active"] ? "has-background-link" : "" 
          }">
      {JSON.stringify(data[item]["Active"])}
      </td>
      <td class="{
          data[item]["Recovered"] > data[item]["Active"] ? "has-background-primary" : "" 
          }">
      {JSON.stringify(data[item]["Recovered"])}
      </td>
      <td>
      {JSON.stringify(data[item]["Deaths"])}
      </td>
      <td class="{
        getTotalFatalities(data[item]) > total_fatality_rate ? "is-danger" : "" 
        }{
          getTotalFatalities(data[item]) < total_fatality_rate ? "is-success" : "" 
          }">
      {getTotalFatalities(data[item]).toFixed(2)}
      </td>
      <td class="{
        getTotalRecoveries(data[item]) < total_recovery_rate ? "is-warning" : "" 
        }{
          getTotalRecoveries(data[item]) > total_recovery_rate ? "is-success" : "" 
          }">
      {getTotalRecoveries(data[item]).toFixed(2)}
      </td>
      </tr>
  {/each}
  </tbody>
    </table>
</div>
</main>