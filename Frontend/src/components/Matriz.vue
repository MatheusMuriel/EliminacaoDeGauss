<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
  <div>
    <!--
    <b-pagination
      v-model="matrizAtual"
      :total-rows="totalDeMatrizes.length"
      :per-page="1"
      aria-controls="my-table"
    ></b-pagination>
    -->

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
    beforeMount (){
      let vm = this
      let tamanho = parseInt(vm._props.tamanho, 10)
      for (let i = 1; i <= tamanho; i++) {
        for (let j = 1; j <= tamanho+1; j++) {
          let refStr = i + "" + j
          let ref = parseInt(refStr, 10)
          this.valores[ref] = 1
        }
      }
      // this.listaDeMatrizes.push(vm._props.items)
      // console.log("aaaaa", this.listaDeMatrizes)

    },
    mounted () {
      this.$on('pegarValores', this.getValores)
      this.matrizAtual = this.$refs.matrizInicial
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
