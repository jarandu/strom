<script>

export let data

let output

const suggestions = {
    high: [
        "Vent med dusjen, strømmen er dyr!",
        "Ikke lad bilen nå, det er dyrt å bruke strøm."
    ],
    low: [
        "Ta deg en dusj, nå er det billig!",
        "Lad elbilen, nå er billig!"
    ],
    free:[
        "Nå er det nesten gratis! Men husk at du betaler nettleie også."
    ],
    normal: [
        "Strømprisen ligger på snittet for dagen.",
        "Helt ok pris."
    ]
}

output = suggestions[data][Math.floor(Math.random() * suggestions[data].length)]

</script>

<span>{output}</span>