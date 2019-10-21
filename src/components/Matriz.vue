<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
  <div>
    <b-table
      striped hover
      id="my-table"
      thead-class="hidden_header"
      :items="items"
      :fields="fields">
      <template v-slot:cell()="row">
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
        matrizAtual: '', 
        listaDeMatrizes: []
      }
    },
    props: ['items','tamanho'],
    created (){
      let vm = this

      let tamanho = parseInt(vm._props.tamanho, 10)
      let valores = vm._props.items

      if (valores) {
        valores.forEach(i => {
          i.forEach(j => {
            this.valores[j.nome] = j.valor
          })
        })
      }
    },
    mounted () {
      this.$on('pegarValores', this.getValores)
      this.matrizAtual = this.$refs.matrizInicial
    },
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
