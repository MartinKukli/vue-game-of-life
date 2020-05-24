<template>
  <p
    v-if="renderCell"
    v-on:click="changeState"
    v-bind:id="state[0]"
    v-bind:class="[w3Style, state[1] ? 'w3-green' : 'w3-light-green']"
  ></p>
</template>

<script>
export default {
  name: "Cell",
  props: {
    state: Boolean | Array
  },
  data: function() {
    return {
      // cell name string, render else dont
      renderCell: this.state ? true : false,
      w3Style: "w3-border w3-border-black cell"
    };
  },
  methods: {
    changeState(event) {
      const targetId = event.target.id; // get cell id
      const newState = this.state[1] ? false : true; // change cell state to !state
      this.$emit("changeState", [targetId, newState]); // emit change
    }
  }
};
</script>

<style>
.cell {
  height: 3.5em;
  margin: 0px;
  width: 3.5em;
}
</style>