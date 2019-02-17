<template>

    <div class="container">
        <br><br>

        <!-- <div class="images">
            <div class="scene">
                <div class="overlay" :style=" 'background-color: ' + sceneOverlayColour + '; opacity: ' + sceneOverlayOpacity + ';' "></div>
            </div>
            <div class="photo" :style=" 'filter: blur(' + blurAmount + 'px)' ">
                <div class="overlay" :style=" 'background-color: ' + photoOverlayColour + '; opacity: ' + photoOverlayOpacity + ';' "></div>
            </div>
        </div> -->

        <h2>
            {{ ev }}
            <span v-if="ev != 0">{{ ev > 0 ? ' stops over-exposed' : 'stops under-exposed' }}</span>
            <span v-if="ev == 0"> - correct exposure</span>
        </h2>

        <hr>

        <div class="controls">

            <h4>Scene brightness:</h4>

            <div class="form-group">
                <label for="light">{{ stops.light[input.light] }}lv - {{ lvDescription }}</label>
                <input v-model="input.light" type="range"  class="custom-range" id="light" name="light" min="0" :max="stops.light.length - 1">
            </div>

            <hr>

            <h4>Camera settings:</h4>

            <div class="card mb-3">
                <div class="card-body">
                    <div class="form-group">
                        <label for="aperture">Aperture f/{{ stops.aperture[input.aperture] }}</label>
                        <input v-model="input.aperture" type="range"  class="custom-range" id="aperture" name="aperture" min="0" :max="stops.aperture.length - 1">
                    </div>
                    <span>
                        ← More light let in<br>
                        ← Shallower depth of field
                    </span>
                    <span class="float-right">
                        Less light let in → <br>
                        Deeper depth of field →
                    </span>
                </div>
            </div>

            <div class="card mb-3">
                <div class="card-body">
                    <div class="form-group">
                        <label for="shutter">Shutter speed 1/{{ stops.shutter[input.shutter] }}s</label>
                        <input v-model="input.shutter" type="range"  class="custom-range" id="shutter" name="shutter" min="0" :max="stops.shutter.length - 1">
                    </div>
                    <span>
                        ← Light falls on sensor for longer<br>
                        ← Increased motion / handshake blur
                    </span>
                    <span class="float-right">
                        Light falls on sensor for shorter →<br>
                        Freezes fast-moving subjects →
                    </span>
                </div>
            </div>

            <div class="card mb-3">
                <div class="card-body">
                    <div class="form-group">
                        <label for="iso">ISO {{ stops.iso[input.iso] }}</label>
                        <input v-model="input.iso" type="range"  class="custom-range" id="iso" name="iso" min="0" :max="stops.iso.length - 1">
                    </div>
                    <span>
                        ← Sensor less sensitive to light<br>
                        ← Less noise / graininess
                    </span>
                    <span class="float-right">
                        Sensor more sensitive to light → <br>
                        Increased noise / graininess →
                    </span>
                </div>
            </div>

        </div>


    </div>

</template>

<script>
    export default {
        data() {
            return {
                stops: {
                    aperture: [1, 1.4, 2, 2.8, 4, 5.6, 8, 11, 16, 22, 32],
                    shutter: [1, 2, 4, 8, 15, 30, 60, 125, 250, 500, 1000, 2000, 4000, 8000],
                    light: [-5, -4, -3, -2, -1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16 ,17],
                    iso: [25, 50, 100, 200, 400, 800, 1600],
                },
                baseline: {
                    aperture: 0,
                    shutter: 0,
                    light: 15,
                    iso: 2,
                },
                input: {
                    aperture: 0,
                    shutter: 0,
                    light: 15,
                    iso: 2,
                }
            }
        },
        computed: {
            ev() {
                return this.stops.light[this.input.light] - this.deviation
            },
            deviation() {
                const aperture = this.input.aperture - this.baseline.aperture
                const shutter = this.input.shutter - this.baseline.shutter
                const iso = this.baseline.iso - this.input.iso
                const total = aperture + shutter + iso
                return total
            },
            lvDescription() {
                switch (this.stops.light[this.input.light]) {
                    case -8:
                        return "The Milky Way"
                    case -5:
                        return "Scene lit by the full moon"
                    case 1:
                        return "Dark scenes outdoors at night"
                    case 2:
                        return "Typical night street scenes"
                    case 3:
                        return "Brightly lit night street scenes"
                    case 7:
                        return "Typical indoors; light outdoors about 10 minutes after sunset"
                    case 10:
                        return "Dark, dreary overcast day in Boston, London or Paris"
                    case 12:
                        return "California bright overcast"
                    case 13:
                        return "Typical shadow cast in a daylight scene; cloudy bright days"
                    case 14:
                        return "Typical light level for side-lit daylight shots in good afternoon light"
                    case 15:
                        return "Gray card in full sunlight; typical exposure for ugly front-lit noon daylight photos"
                    case 16:
                        return "Light gray object or skin in full sunlight"
                    case 17:
                        return "White object in direct sunlight"
                    case 18:
                        return "Bright reflection off a sunlit object, including reflections off the sea"

                }
            },
            blurAmount() {
                return this.getBlurFromAperture()
            },
            sceneOverlayColour() {
                return this.input.light >= 11 ? 'white' : 'black'
            },
            sceneOverlayOpacity() {
                let light = this.input.light >= 11 ? 22 - this.input.light : this.input.light
                return 1 - light / 11
            },
            photoOverlayColour() {
                return this.exposureAmount >= 50 ? 'white' : 'black'
            },
            photoOverlayOpacity() {
                let normalised = this.ev + 21
                let opacity = normalised >= 21 ? 42 - normalised : normalised
                opacity = 1 - opacity / 21
                return opacity > 1 ? 1 : opacity
            }
        },
        methods: {
            getBlurFromAperture() {
                const blur = (1 / this.input.aperture) * 10
                return blur
            }
        }
    }
</script>
