<template>
  <q-page padding class="column q-gutter-y-lg text-white">
    <spin-wheel-renderer :items="options.filter(i => i.weight != 0)" ref="wheel" :key="updateWheel"/>
    <div class="row full-width q-col-gutter-x-md justify-center q-pr-md">
      <q-input
        v-for="prize in prizes.filter(i => i.value !== 'nothing')"
        dark
        class="col-sm-12 col-xs-12 col-md-3 col-lg-3 col-xl-3"
        :key="prize.value"
        :label="prize.label"
        :model-value="prize.weight"
        type="number"
        @update:model-value="updatePrizeAmount({ value: $event, prize })"
      />
    </div>
  </q-page>
</template>

<script>
import { LocalStorage } from 'quasar'
import { computed, defineComponent, reactive, ref } from "vue";
import { colors } from 'quasar'
import SpinWheelRenderer from "src/components/SpinWheelRenderer.vue";

export default defineComponent({
  name: "PageIndex",
  components: { SpinWheelRenderer },
  setup () {
    const wheel = ref(null)
    const updateWheel = ref(0)
    const savedItems = LocalStorage.getItem('wheelValue')

    const generateRandomColor = () => {
      return '#'+(0x1000000+Math.random()*0xffffff).toString(16).substr(1,6)
    }

    const prizeColors = reactive({
      coffee: generateRandomColor(),
      soda: generateRandomColor(),
      poke: generateRandomColor(),
      workout: generateRandomColor(),
      shave: generateRandomColor(),
      clean: generateRandomColor(),
      nothing: generateRandomColor()
    })
    const prizes = ref(savedItems || [
      { label: 'Coffee', value: 'coffee', weight: 0 },
      { label: 'Soda', value: 'soda', weight: 0 },
      { label: 'Poke', value: 'poke', weight: 0 },
      { label: 'Work Out', value: 'workout', weight: 0 },
      { label: 'Shave', value: 'shave', weight: 0 },
      { label: 'Clean Room', value: 'clean', weight: 0 },
      { label: 'Nothing', value: 'nothing', weight: 9 }
    ])
    const options = computed(() => {
      return prizes.value.map(prize => {
        return {
          ...prize,
          weight: parseInt(prize.weight),
          backgroundColor: prizeColors[prize.value],
          labelColor: colors.brightness(prizeColors[prize.value]) < 128 ? 'white' : 'black',
        }
      })
    })

    const summedPrizeValues = computed(() => {
      const numbers = prizes.value.filter(i => i.value !== 'nothing').map(i => i.weight)
      return numbers.reduce((r,c) => r + parseFloat(c), 0)
    })

    const updatePrizeAmount = ({value, prize}) => {
      const foundPrize = prizes.value.find(i => i.value === prize.value)
      const foundPrizeNothing = prizes.value.find(i => i.value === 'nothing')
      foundPrize.weight = value
      foundPrizeNothing.weight = 10 - summedPrizeValues.value
      LocalStorage.set('wheelValue', prizes.value)
      updateWheel.value += 1
    }

    return {
      updateWheel,
      wheel,
      options,
      prizes,
      summedPrizeValues,
      updatePrizeAmount,
      savedItems
    }
  }
});
</script>
