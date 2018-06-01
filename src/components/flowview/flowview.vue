<template lang="pug">

div.content
  navigation(v-bind:flow = 'currentFlow')

  // Trigger
  div.card
    header.card-header.has-background-light
      p.card-header-title IF
      a.card-header-icon
        a.button.is-primary.is-outlined(v-if = '!currentFlow.trigger') Add card
    div.card-content
      div.content
        triggercard(v-if = 'currentFlow.trigger' :card = 'currentFlow.trigger')

  // Conditions
  div.card(v-if = 'currentFlow.conditions')
    header.card-header.has-background-light
      p.card-header-title AND
      a.card-header-icon
        a.button.is-primary.is-outlined Add card
        //a.button.is-primary.is-outlined Add or
    div.card-content
      div.content
        conditioncard(v-for='card in currentFlow.conditions' :key = 'idForCard(card)' :card = 'card')
    footer.card-footer
      a.card-footer-item Add or

  // Actions
  div.card(v-if = 'currentFlow.actions')
    header.card-header.has-background-light
      p.card-header-title THEN
      a.card-header-icon
        a.button.is-primary.is-outlined(v-on:click="openModal()") Add card
        //a.button.is-primary.is-outlined Add else
    div.card-content
      div.content
        actioncard(v-if = 'currentFlow.actions' v-for='card in currentFlow.actions' :key = 'idForCard(card)' :card = 'card')
    footer.card-footer
      a.card-footer-item Add else

  div.modal(v-if="this.editModal" :class="(this.editModal ? 'is-active' : '')")
    div.modal-background
    div.modal-card
      header.modal-card-head
        p.modal-card-title Add action
        button.delete(v-on:click="closeModal()")
      section.modal-card-body
        div(v-for='cardGroup in cards' :key='cardGroup.uriObj.uri')
          div.columns(v-for='cardElement in cardGroup.cards.action' :key='cardElement.id')
            div.column.is-narrow.flow-icon-container
              img.flow-icon(:src="$homey._baseUrl + cardGroup.uriObj.icon")
            div.column.flow-title-container
              div.columns.is-paddingless
                div.column.has-text-left.is-marginless
                  span.flow-title {{ cardElement.title.en }}
      footer.modal-card-foot
        button.button.is-success Save changes
        button.button(v-on:click="closeModal()") Cancel


</template>

<script>
import navigation from './navigation';
import triggercard from './triggercard';
import conditioncard from './conditioncard';
import actioncard from './actioncard';

function reduceFlowType(arr) {
  return arr.reduce((acc, item) => {
    acc[item.id] = item;
    return acc;
  }, {});
}

export default {
  name: 'FlowView',
  props: ['flow'],
  components: {
    triggercard,
    conditioncard,
    actioncard,
    navigation
  },
  data() {
    return {
      editModal: false,
      currentFlow: {},
      cards: {}
    };
  },
  mounted() {
    if (!this.$route.params.flow) return;
    this.getData(this.$route.params.flow);
  },
  methods: {
    openModal() {
      this.editModal = true;
    },
    closeModal() {
      this.editModal = false;
    },
    idForCard(card) {
      if (card.uri === 'homey:manager:flow') {
        if (card.args && card.args.flow && card.args.flow.id) {
          return card.args.flow.id;
        }
      }
      return card.id;
    },
    getData(flow) {
      Promise.all([
        this.getCards(),
        this.getFlow(flow)
      ]).then(([ cards, flow ]) => {
        this.cards = cards;
        this.cardsByUri = cards.reduce((acc, card) => {
          acc[card.uri] = {
            triggers   : reduceFlowType(card.cards.trigger   || []),
            conditions : reduceFlowType(card.cards.condition || []),
            actions    : reduceFlowType(card.cards.action    || []),
          };
          return acc;
        }, {});
        this.annotateFlow(flow);
      }).catch(e => {
        console.error('Unable to load flow/cards', e);
      });
    },
    getFlow(flow) {
      return this.$homey.flow.getFlow({ id: flow });
    },
    getCards() {
      return this.$homey.flow.getCards();
    },
    annotateFlow(flow) {
      flow.trigger.base = this.cardsByUri[flow.trigger.uri].triggers[flow.trigger.id];

      flow.conditions.forEach(condition => {
        condition.base = this.cardsByUri[condition.uri].conditions[condition.id];
      });

      flow.actions.forEach(action => {
        action.base = this.cardsByUri[action.uri].actions[action.id];
      });

      this.currentFlow = flow;
    }
  },
  beforeRouteUpdate(to, from, next) {
    this.getFlow(to.params.flow).then(flow => this.annotateFlow(flow));
    next();
  }
};
</script>

<style scoped lang="stylus">

.card-group
  font-style bold
  font-size 35px
  text-align center
  width 80%
  margin auto

.flow-icon-container
  display flex
  justify-content center
  align-items center
  img
    height 55px
    width 55px
    -webkit-mask-size contain
    -webkit-mask-position top left
    -webkit-mask-repeat no-repeat

.flow-title
  font-weight bold

</style>
