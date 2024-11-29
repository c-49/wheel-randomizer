<template>
  <div class="text-white column justify-center items-center">
    <!-- prize -->
    <div class="text-white full-width text-center" v-if="prizeGiven">
      <div class="text-h5" :class="{
        'text-red': prizeGiven.value === 'nothing',
        'text-green': prizeGiven.value !== 'nothing'
      }">
        {{ prizeGiven.value === 'nothing' ? 'Whoops! Better luck next time!' : 'Congrats!' }}
      </div>
      <div class="text-h6 text-green" v-if="prizeGiven.value !== 'nothing'">
        You win {{ prizeGiven.value }}!
      </div>
    </div>
    <!-- wheel -->
    <div class="wheel fit" ref="wheelWrapper" style="max-width:1080px;max-height:720px;"/>
    <!-- btn -->
    <div>
      <q-btn no-caps label="Spin!" @click="spinWheel" color="primary" size="lg"/>
    </div>
  </div>
</template>

<script>
import { onMounted, reactive, ref } from 'vue'
import { Wheel } from 'spin-wheel/dist/spin-wheel-esm.js'
const smile = require('../assets/smiley-1.svg')
export default {
    name: 'SpinWheelRenderer',
    props: {
        items: Array
    },
    setup(props, {emit}) {
      const prizeGiven = ref(null)
      const wheelWrapper = ref(null)
      const wheel = ref(null)
      const options = reactive({
        name: 'Randomizer',
        radius: 0.89,
        pointerAngle: 90,
        itemLabelRadius: 0.92,
        itemLabelRadiusMax: 0.37,
        itemLabelRotation: 0,
        itemLabelColors: ['#000'],
        itemLabelBaselineOffset: -0.06,
        itemLabelFont: 'Roboto',
        itemBackgroundColors: ['#fbf8c4', '#e4f1aa', '#c0d26e', '#ff7d7d'],
        rotationSpeedMax: 700,
        rotationResistance: -110,
        lineWidth: 0,
        overlayImage: smile,
        items: props.items
      })
      const loadWheel = () => {
        wheel.value = new Wheel(wheelWrapper.value)
        wheel.value.init({
          ...options,
          onRest: e => onWheelEnd(e),
          onSpin: e => prizeGiven.value = null
        })
      }
    const onWheelEnd = (e) => {
        prizeGiven.value = wheel.value.items[e.currentIndex]
    }
    onMounted(() => {
      loadWheel()
    })
    return {
      wheelWrapper,
      prizeGiven,
      spinWheel: () => wheel.value.spin((Math.floor(Math.random() * 20) + 1) * 100)
    }
  }
}
</script>
