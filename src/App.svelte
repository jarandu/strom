<script>

import { onMount } from "svelte"
import Hours from "./Hours.svelte"
import Tell from "./Tell.svelte"

let data = {
    hours: [],
    low: 0,
    high: 0,
    span: 0,
    avg: 0,
    std: 0,
    dev: 0
}
$: currentRegion = getRegion()

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
        "normal"
    return human
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
    x.std = parseFloat((Math.sqrt(hours.map(h => Math.pow(h - x.avg, 2)).reduce((a, b) => a + b) / 24)).toFixed(2))
    x.dev = x.std / x.avg

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
    let cookie = document.cookie.split("; ").find((row) => row.startsWith("Region="))?.split("=")[1]
    return cookie
}

function setRegion(region) {
    document.cookie = `Region=${region}; SameSite=None; Secure`
    currentRegion = region
}

function populate() {
    fetch("https://services.api.no/api/acies/v1/custom/EntsoElectricityPrice?greaterThan=startTime:2023-05-03T22:00:00.000Z&equal=deliveryArea:NO1&fields=(startTime,value)")
    .then(r => r.json())
    .then(d => {
        data = analyseData(d)
        console.log(data)
    })
    .catch(e => console.log(e))
}

onMount(async () => {
    let currentRegion = await getRegion()
    console.log("Region: " + currentRegion)
    populate()
})

</script>
<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="true">
	<link href="https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
</svelte:head>

<main>

{#each data.hours as hour}
<div class="{ hour.level }">
    Kl {parseInt(hour.hour).toLocaleString('nb-NO', { minimumIntegerDigits: 2 })}-{parseInt(hour.hour+1).toLocaleString('nb-NO', { minimumIntegerDigits: 2 })}: <strong>{hour.price}</strong> øre
</div>
{/each}


{#if data.hours.length == 0 }
<div class="loading">
    <div style="animation-delay: 0.2s;"></div>
    <div style="animation-delay: 0.4s;"></div>
    <div style="animation-delay: 0.6s;"></div>
</div>
{:else}
<div class="track">
    <svg class="el" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 293 432">
        <polygon points="63 432 293 187 168 166 226 0 0 241 124 266 63 432"/>
    </svg>
    <h2>Strømprisen</h2>
    {#if currentRegion == ""}
    <div class="choose-region">
        <div class="">Velg region:</div>
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <div class="pill" title="NO1" on:click={() => { setRegion("NO1") }}>Øst</div>
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <div class="pill" title="NO2" on:click={() => { setRegion("NO2") }}>Sør</div>
    </div>
    {:else}
    <div>
        Nå: <span class="price {getNow(data).level}{ getNow(data).level !== "normal" ? " pill" : ""}"><strong>{getNow(data).price}</strong>&nbsp;øre/kWh</span>. <Tell data={getNow(data).level} />
    </div>
    <div>Idag: <strong>{Math.round(data.avg)}</strong> øre/kWh, litt lavere enn <a href="">i går</a>. <a href="">I morgen</a> blir strømmen dyrere.</div>
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <div class="locale pill" on:click={() => { setRegion("") }}>
        Bytt region&nbsp;
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 420 420">
            <path stroke-width="26" d="M209,15a195,195 0 1,0 2,0z"/>
            <path stroke-width="18" d="m210,15v390m195-195H15M59,90a260,260 0 0,0 302,0 m0,240 a260,260 0 0,0-302,0M195,20a250,250 0 0,0 0,382 m30,0 a250,250 0 0,0 0-382"/>
        </svg>
    </div>
    {/if}
</div>
{/if}

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
.loading {
    width: 20px;
    display: flex;
    gap: 2px;
    justify-content: center;
    align-items: center;
}
.loading > div {
    width: 3px;
    height: 3px;
    animation: loading 1.5s infinite ease-in-out;
    transform: scale(1);
    transform-origin: center;
    clip-path: circle();
    background: black;
    opacity: 0;
}
@keyframes loading {
    0% {
        transform: scale(1);
        opacity: 0;
    } 
    30% {
        transform: scale(1.4);
        opacity: 1;
    } 
    40% {
        transform: scale(1.1);
    }
    60% {
        transform: scale(1);
        opacity: 0;
    }
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
.low {
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
