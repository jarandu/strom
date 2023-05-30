<script>

import { onMount } from "svelte"
import Hours from "./Hours.svelte"
import Tell from "./Tell.svelte"
import Loading from "./Loading.svelte";

let obj = {
    hours: [],
    low: 0,
    high: 0,
    span: 0,
    avg: 0,
    // std: 0,
    // dev: 0
}
let data = {
    yesterday: obj,
    today: obj,
    tomorrow: obj
}
$: currentRegion = getRegion()
$: thisHour = { price: 0, level: "" }

function getNow(data) {
    let now = new Date()
    let hour = data.hours[now.getHours()]
    return {
        price: Math.round(hour.price),
        level: hour.level
    }
}

function getLevel(price, x) {
    let level = parseFloat(((price - x.low) / x.span).toFixed(2))
    let human = 
        level > 0.75 ? "high" :
        level < 0.25 ? "low" :
        level < 0 ? "free" :
        "normal"
    return human
}

function populateDays(d) {
    let tempData = { yesterday: [], today: [], tomorrow: [] }
    d.forEach((day, index) => {
        let when = index < 24 ? 'yesterday' :
            index < 48 ? 'today' :
            'tomorrow'
        tempData[when].push(day)
    })
    for (let day in tempData) {
        if (tempData[day].length != 0) {
            data[day] = analyseData(tempData[day])
        }
    }
    data = data
    thisHour = getNow(data.today)
    console.log(data)
}

function analyseData(d) {
    let x = {}
    
    // Get hours
    let hours = d.map(h => parseFloat((h.value / 10).toFixed(2)))

    // Get high/low
    x.high = Math.max(...hours)
    x.low = Math.min(...hours)
    x.span = parseFloat((x.high - x.low).toFixed(2))
    x.avg = parseFloat((hours.reduce((b, a) => b + a, 0) / 24).toFixed(2))
    // x.std = parseFloat((Math.sqrt(hours.map(h => Math.pow(h - x.avg, 2)).reduce((a, b) => a + b) / 24)).toFixed(2))
    // x.dev = x.std / x.avg

    // Update hours
    x.hours = hours.map((value, index) => {
        return {
            hour: index,
            price: value,
            level: getLevel(value, x)
        } 
    })

    return x
}

function getRegion() {
    let cookie = document.cookie.split("; ").find((row) => row.startsWith("Region="))?.split("=")[1] || ""
    return cookie
}

function setRegion(region) {
    document.cookie = `Region=${region}; SameSite=None; Secure`
    currentRegion = region
    getData()
}

function getData() {
    let now = new Date()
    now.setDate(now.getDate() - 2)
    let date = now.toISOString().substring(0,10)
    fetch(`https://services.api.no/api/acies/v1/custom/EntsoElectricityPrice?greaterThan=startTime:${date}T22:00:00.000Z&equal=deliveryArea:${currentRegion}&fields=(startTime,value)`)
    .then(r => r.json())
    .then(d => {
        populateDays(d)
    })
    .catch(e => console.log(e))
}

/* function getNextLow(hours) {
    let now = new Date()
    now.getHours()
    let lows = hours.filter(h => h.level == "low" || h.level == "free")
    console.log(lows);
} */

onMount(async () => {
    let currentRegion = await getRegion()
    if (currentRegion != "") getData()
})

</script>
<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="true">
	<link href="https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
</svelte:head>

<main>


<div class="track">
    <svg class="el" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 293 432">
        <polygon points="63 432 293 187 168 166 226 0 0 241 124 266 63 432"/>
    </svg>
    <h2>Strømprisen</h2>
{#if currentRegion === ""}
    <div class="choose-region">
        <div class="">Velg region:</div>
        <div class="pill" title="NO1" on:click={() => { setRegion("NO1") }} on:keypress={() => { setRegion("NO1") }}>Øst</div>
        <div class="pill" title="NO2" on:click={() => { setRegion("NO2") }} on:keypress={() => { setRegion("NO2") }}>Sør</div>
        <div class="pill" title="NO3" on:click={() => { setRegion("NO3") }} on:keypress={() => { setRegion("NO3") }}>Midt</div>
        <div class="pill" title="NO4" on:click={() => { setRegion("NO4") }} on:keypress={() => { setRegion("NO4") }}>Nord</div>
        <div class="pill" title="NO5" on:click={() => { setRegion("NO5") }} on:keypress={() => { setRegion("NO5") }}>Vest</div>
    </div>
{:else}
    {#if data.today.hours.length == 0 }
    <Loading />
    {:else}
    <div>
        Nå: <span class="price { thisHour.level == "normal" ? thisHour.level : thisHour.level + " pill"}"><strong>{thisHour.price}</strong>&nbsp;øre/kWh</span> 
        <Tell data={thisHour.level} />
    </div>
    <div>Idag: <strong>{Math.round(data.today.avg)}</strong> øre/kWh, { data.today.avg < data.yesterday.avg ? "lavere enn" : "høyere enn" } i går. 
        {#if data.tomorrow.hours.length != 0 }
        I morgen blir strømmen { data.today.avg > data.tomorrow.avg ? "billigere" : "dyrere" }.
        {/if}
    </div>    
    <div class="locale pill" on:click={() => { setRegion("") }} on:keypress={() => { setRegion("") }}>
        Bytt region&nbsp;
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 420 420">
            <path stroke-width="26" d="M209,15a195,195 0 1,0 2,0z"/>
            <path stroke-width="18" d="m210,15v390m195-195H15M59,90a260,260 0 0,0 302,0 m0,240 a260,260 0 0,0-302,0M195,20a250,250 0 0,0 0,382 m30,0 a250,250 0 0,0 0-382"/>
        </svg>
    </div>
    {/if}
{/if}
</div>

</main>

<style>
.track {
    height: 100%;
    width: 100%;
    font-family: "Open Sans", sans-serif;
    font-size: 13px;
    overflow-x: scroll;
    scrollbar-width: none;
    touch-action: pan-x;
    white-space: nowrap;
    line-height: 1;
    display: flex;
    align-items: center;
    gap: 5px;
}
.track::-webkit-scrollbar {
    display: none;
}
h2 {
    font-size: inherit;
    text-transform: uppercase;
}
.choose-region {
    display: flex;
    align-items: center;
    gap: 5px;
}
.choose-region .pill {
    cursor: pointer;
}
.low, .free {
    background-color: var(--green);
}
.high {
    background-color: var(--red);
}
svg.el {
    height: 15px;
    fill: var(--yellow);
}
.locale {
    background: var(--lightyellowgray);
    cursor: pointer;
}
.locale svg {
    width: 14px;
    fill: white;
    stroke: black;
}
</style>
