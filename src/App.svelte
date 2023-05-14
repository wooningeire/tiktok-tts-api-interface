<script lang="ts">
import VoiceCode from "./lib/VoiceCode.svelte";

let audio: HTMLAudioElement;

let nOutstandingRequests = 0;
let audioUri = "";
let errorMessage = "";

const setVoice = async (text: string, voice: string) => {
    nOutstandingRequests += 1;
    const req = await fetch("https://tiktok-tts.weilnet.workers.dev/api/generation", {
        headers: {
            "Content-Type": "application/json",
        },
        body: JSON.stringify({text, voice}),
        method: "POST",
    })
            .finally(() => {
                nOutstandingRequests -= 1;
            });

    const json = await req.json();
    if (!json.success) {
        audioUri = "";
        errorMessage = `“${json.error}”`;
        console.log(json);
    } else {
        audioUri = `data:application/octet-stream;base64,${json.data}`;
        errorMessage = "";
    }
};

let text: string = "";
let voice: string = "en_female_f08_salut_damour";

const MAX_TEXT_LENGTH = 300;

$: loading = nOutstandingRequests >  0;
$: inputInvalid = text.length > MAX_TEXT_LENGTH || text.length === 0;
</script>

<main>
    <audio controls
            bind:this={audio}
            on:loadeddata={() => audio.play()}
            src={audioUri}
            class:disabled={!audioUri}></audio>

    <h2>Voice text</h2>
    <textarea bind:value={text}></textarea>

    <div class:warning={inputInvalid}>{text.length} / {MAX_TEXT_LENGTH} chars</div>

    <h2>Voice code</h2>
    <div>
        <input bind:value={voice} />
        <select bind:value={voice} multiple>
            <optgroup label="singing">
                <VoiceCode code="en_female_f08_salut_damour" />
                <VoiceCode code="en_male_m03_lobby" />
                <VoiceCode code="en_female_f08_warmy_breeze" />
                <VoiceCode code="en_male_m03_sunshine_soon" />
                <VoiceCode code="en_female_ht_f08_glorious" />
                <VoiceCode code="en_male_sing_funny_it_goes_up" />
                <VoiceCode code="en_male_m2_xhxs_m03_silly" />
                <VoiceCode code="en_female_ht_f08_wonderful_world" />
            </optgroup>

            <optgroup label="english">
                <VoiceCode code="en_au_001" /> <!-- English AU - Female -->
                <VoiceCode code="en_au_002" /> <!-- English AU - Male -->
                <VoiceCode code="en_uk_001" /> <!-- English UK - Male 1 -->
                <VoiceCode code="en_uk_003" /> <!-- English UK - Male 2 -->
                <VoiceCode code="en_us_001" /> <!-- English US - Female (Int. 1) -->
                <VoiceCode code="en_us_002" /> <!-- English US - Female (Int. 2) -->
                <VoiceCode code="en_us_006" /> <!-- English US - Male 1 -->
                <VoiceCode code="en_us_007" /> <!-- English US - Male 2 -->
                <VoiceCode code="en_us_009" /> <!-- English US - Male 3 -->
                <VoiceCode code="en_us_010" /> <!-- English US - Male 4 -->
            </optgroup>

            <optgroup label="other english">
                <VoiceCode code="en_male_narration" />
                <VoiceCode code="en_male_funny" />
                <VoiceCode code="en_female_emotional" />
                <VoiceCode code="en_male_cody" />
            </optgroup>

            <optgroup label="europe">
                <VoiceCode code="fr_001" /> <!-- French - Male 1 -->
                <VoiceCode code="fr_002" /> <!-- French - Male 2 -->
                <VoiceCode code="de_001" /> <!-- German - Female -->
                <VoiceCode code="de_002" /> <!-- German - Male -->
                <VoiceCode code="es_002" /> <!-- Spanish - Male -->
            </optgroup>

            <optgroup label="america">
                <VoiceCode code="es_mx_002" /> <!-- Spanish MX - Male -->
                <VoiceCode code="br_001" /> <!-- Portuguese BR - Female 1 -->
                <VoiceCode code="br_003" /> <!-- Portuguese BR - Female 2 -->
                <VoiceCode code="br_004" /> <!-- Portuguese BR - Female 3 -->
                <VoiceCode code="br_005" /> <!-- Portuguese BR - Male -->
            </optgroup>

            <optgroup label="asia">
                <VoiceCode code="id_001" />
                <VoiceCode code="jp_001" />
                <VoiceCode code="jp_003" />
                <VoiceCode code="jp_005" />
                <VoiceCode code="jp_006" />
                <VoiceCode code="kr_002" />
                <VoiceCode code="kr_003" />
                <VoiceCode code="kr_004" />
            </optgroup>

            <optgroup label="disney">
                <VoiceCode code="en_us_ghostface" />
                <VoiceCode code="en_us_chewbacca" />
                <VoiceCode code="en_us_c3po" />
                <VoiceCode code="en_us_stitch" />
                <VoiceCode code="en_us_stormtrooper" />
                <VoiceCode code="en_us_rocket" />
                <VoiceCode code="en_female_madam_leota" />
                <VoiceCode code="en_male_ghosthost" />
                <VoiceCode code="en_male_pirate" />
            </optgroup>
        </select>
    </div>

    <button on:click={() => setVoice(text, voice)}
            class:loading={loading}
            class:warning={inputInvalid}>play</button>

    <div class="warning">{errorMessage}</div>
</main>

<style lang="scss">
main {
    width: 100%;
    max-width: 1280px;
    margin: 0 auto;
    padding: 2rem;

    display: flex;
    flex-flow: column;
    align-items: center;
    gap: 1em;

    text-align: center;

    > div {
        display: flex;
        flex-flow: column;
    }
}

textarea {
    width: 100%;
    height: 10em;
}

select {
    height: 16rem;
}

input,
textarea,
select,
optgroup,
option {
    font-family: inherit;
}

.warning {
    color: #d19;
}
button.warning:hover {
    border-color: #f381da;
}

@function solid($color) {
    @return linear-gradient($color, $color);
}


button {
    background-image: solid(#0000);

    transition: border-color 0.25s,
            background-color 0.25s ease-in-out;
}

button.loading {
    --SQRT_2: 1.4142135623730951;
    --bg-size: calc(2 * var(--stripe-size) * var(--SQRT_2));

    background-image: repeating-linear-gradient(135deg,
            #0000 0 var(--stripe-size),
            #9ea9f03f var(--stripe-size) calc(2 * var(--stripe-size)));

    background-size: var(--bg-size) var(--bg-size);
    animation: stripe-slide 1s infinite linear;

    --stripe-size: 12px;
    @keyframes stripe-slide {
        0% {
            background-position: 0 0;
        }
        100% {
            background-position: var(--bg-size) var(--bg-size);
        }
    }
}


.disabled {
    pointer-events: none;
    opacity: 0.25;
}

</style>