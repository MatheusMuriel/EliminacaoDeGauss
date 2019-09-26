<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
  <div>
    <b-table
      striped hover
      thead-class="hidden_header"
      :items="items"
      :fields="fields">
      <template v-slot:cell()="row">
        <h5>{{row.value.nome}}</h5>
        <b-input type="number" v-model="valores[Number(row.value.nome)]" model="0" @input="updateModel({$event})"></b-input>
      </template>
    </b-table>
  </div>
</template>
<script>
  export default {
    data () {
      return {
        fields: [],
        valores: [],
      }
    },
    props: ['items','tamanho'],
    methods: {
      updateModel (e) {
        console.log(e,this.valores)
      }
    }
  }
</script>
<style>
  .hidden_header {
    display: none;
  }
</style>
