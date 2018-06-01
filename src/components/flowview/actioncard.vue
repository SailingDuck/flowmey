<template lang="pug">

// TODO: THEN/ElSE grouping card.group contains then, else to specify it's target

div
  flowcard.flow-action-card(:card = 'card')

</template>

<script>
import flowcard from './flowcard';

export default {
  name: 'actioncard',
  props: ['card'],
  components: {
    flowcard
  }
};

</script>

<style scope lang='stylus'>
@import '~styles/variables.styl'

.flow-action-card
  border-color color-action !important

</style>
