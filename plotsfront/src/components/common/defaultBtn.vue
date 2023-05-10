<template>
  <Button class="btn" :disabled="isDisabled">
    <slot></slot>
  </Button>
</template>

<script>

export default {
name: 'DefaultBtn',
props:{
  isDisabled:{
    type: Boolean,
    default: false,
  }
}
}
</script>

<style lang="sass" scoped>
.btn
  padding: 10px 20px
  background-color: #132BFF
  color: white
  border: none
  cursor: pointer
  font-size: 16px
  max-width: 150px
  width: 100%

.btn:disabled
  background-color: #4453DC
  cursor: default

.btn:hover
  background-color: #00109F

</style>
