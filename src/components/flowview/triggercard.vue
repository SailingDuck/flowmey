<template lang="pug">

div
  flowcard.flow-trigger-card(:card = 'card')

</template>

<script>
import flowcard from './flowcard';

export default {
  name: 'triggercard',
  props: [ 'card'],
  components: {
    flowcard
  }
};

</script>

<style scope lang='stylus'>
@import '~styles/variables.styl'

.flow-trigger-card
  border-color color-trigger !important

</style>
