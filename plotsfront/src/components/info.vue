<template>
  <div class="info__container">
    <div class="info__btns">
      <DefaultBtn @click.prevent="addEvent">Add new plot</DefaultBtn>
      <DefaultBtn @click.prevent="saveEvent" :isDisabled="isDisabled">Save plot</DefaultBtn>
    </div>
    <div class="info__footnotes">
      <p>To create a plot, you should to add a minimum of 3 points.</p>
    </div>
  </div>
</template>

<script>
import DefaultBtn from './common/defaultBtn.vue';

export default {
name: 'InfoArea',
components: {
  DefaultBtn
},
props:{
  isDisabled:{
    type: Boolean,
    default: false,
  },
  addEvent:{
    required: true
  },
  saveEvent:{
    required: true
  }
}

}
</script>

<style lang="sass" scoped>
.info
  &__container
    background-color: #fff
    position: fixed
    top: 0
    left: 0
    width: 25%
    height: fit-content
    padding: 15px 10px
  &__btns
    display: flex
    justify-content: center
    align-items: center
    gap: 30px
    margin-bottom: 15px
  &__footnotes
    color: #000
    font-size: 18px
    p 
      margin: 0
</style>
