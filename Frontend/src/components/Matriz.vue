<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
  <div>
    <b-table
      striped hover
      thead-class="hidden_header"
      :items="items"
      :fields="fields">
      <template v-slot:cell()="row">
        <h5>{{row.value.nome}}</h5>
        <b-input type="number" v-model="valores[row.value.nome]"></b-input>
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
        stringJsonMatriz: '',
        mapatriz: '',
      }
    },
    beforeMount (){
      let vm = this
      let tamanho = parseInt(vm._props.tamanho, 10)
      for (let i = 1; i <= tamanho; i++) {
        for (let j = 1; j <= tamanho; j++) {
          let refStr = i + "" + j
          let ref = parseInt(refStr, 10)
          this.valores[ref] = 0
        }
      }

    },
    mounted () {
      this.$on('pegarValores', this.getValores)
    },
    props: ['items','tamanho'],
    methods: {
      getValores () {
        let mapaMatriz = new Map()
        this.valores.forEach((value, index) => {
          if (value !== undefined) mapaMatriz.set(index, value)
        })
        this.mapatriz = mapaMatriz
      }
    }
  }
</script>
<style>
  .hidden_header {
    display: none;
  }
</style>
