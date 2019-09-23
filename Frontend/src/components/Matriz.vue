<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
  <div>
    <b-table
      striped hover
      thead-class="hidden_header"
      :items="items"
      :fields="fields">
      <template v-slot:cell()="row">
        <b-input></b-input>
      </template>
    </b-table>
  </div>
</template>
<script>
  export default {
    data () {
      return {
        fields: []
      }
    },
    props: ['items']
  }
</script>
<style>
  .hidden_header {
    display: none;
  }
</style>
