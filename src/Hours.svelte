<script>
import { fly } from "svelte/transition"

export let data

let width = 120,
    height = 18,
    padding = 3,
    points = ""

// Build polyline points
data.hours.forEach(hour => {
    points += 
        ((hour.hour * ((width - padding * 2) / 23)) + padding).toFixed(0)
        + "," + 
        ((height - padding * 2) - (hour.price - data.low) / (data.span / (height - padding * 2)) + padding).toFixed(1)
        + " "
})

let hour = "", 
    value = ""
function highlight(i) {
    if (i === undefined) {
        hour = value = ""
    }
    else {
        hour = "Kl " + parseInt(i).toLocaleString('nb-NO', { minimumIntegerDigits: 2 }) + "-" + parseInt(i+1).toLocaleString('nb-NO', { minimumIntegerDigits: 2 }) + ": "
        value = "<strong>" + parseInt(data.hours[i].price) + "</strong> øre"
    }
}

</script>

<div class="ticker-el-expanded">
    <div class="ticker-el-hours">
        <svg viewBox="0 0 {width} {height}" on:mouseleave={() => { highlight() }}>
            <rect class="quartile-marker" x="{width / 4}" width="1" height="100%" />
            <rect class="quartile-marker" x="{width / 4 * 2}" width="1" height="100%" />
            <rect class="quartile-marker" x="{width / 4 * 3}" width="1" height="100%" />
            {#each data.hours as hour}
            <rect class="hour-bar { hour.level == -1 ? "low" : hour.level == 1 ? "high" : "" }" width={width / 24} height="100%" x={width / 24 * hour.hour} 
                on:mouseover={() => { highlight(hour.hour) }} 
                on:focus={() => { highlight(hour.hour) }} />
            {/each}
            <polyline points={points}></polyline>
        </svg>
    </div>
    {#if hour != ""}
    <span class="border">{@html hour}{@html value}</span>
    {/if}
</div>

<style>
.ticker-el-expanded {
    display: inline-flex;
    align-items: center;
    height: 100%;
}
.ticker-el-hours {
    margin-right: 5px;
}
.border {
    margin-right: 8px;
    padding-right: 8px;
    border-right: 1px solid #ccc;
}
svg {
    height: 20px;
    display: block;
    width: auto;
}
polyline {
    stroke: black;
    stroke-width: 1.5px;
    stroke-linecap: round;
    stroke-linejoin: round;
    fill: none;
}
rect.quartile-marker {
    fill: #ccc;
    display: none;
}
rect.hour-bar {
    fill: transparent;
    cursor: pointer;
    outline: none;
}
rect.hour-bar.high {
    fill: var(--red);
}
rect.hour-bar.low {
    fill: var(--green);
}
rect.hour-bar:hover {
    fill: #fcda51;
    opacity: .5;
}
</style>