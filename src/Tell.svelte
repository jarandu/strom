<script>

export let data

let output

const suggestions = {
    high: [
        "Vent med dusjen, strømmen er dyr!",
        "Ikke lad bilen nå, det er dyrt å bruke strøm."
    ],
    low: [
        "Ta deg en dusj, nå er det billig!!"
    ],
    normal: [
        "Strømprisen ligger på snittet for dagen.",
        "Helk ok pris."
    ]
}

output = suggestions[data][Math.floor(Math.random() * suggestions[data].length)]

</script>

<span>{output}</span>