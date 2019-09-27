<template>
  <div id="app">
    <img src="./assets/logo.png">
    <HelloWorld/>

    <b-container>

      <b-row>
        <b-input-group prepend="Tamanho:" class="mt-3 inputCamp">
          <b-form-input v-model="tamanho" placeholder="Ex: 2" type="number"></b-form-input>
          <b-input-group-append>
            <b-button variant="outline-primary" @click="gerarMatriz">Gerar Matriz</b-button>
          </b-input-group-append>
        </b-input-group>
      </b-row>

      <b-row>
        <b-col>
          <div class="mt-2">Tamanho: {{ tamanho }} X {{ tamanho }}</div>
        </b-col>
      </b-row>

      <b-row>
        <b-col ref="linhaMatriz">
          <matriz></matriz>
        </b-col>
        <b-col cols="1">
          <b-button variant="outline-primary" @click="pegarDados">Calcular</b-button>
        </b-col>
      </b-row>


    </b-container>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld'
import Matriz from './components/Matriz'
import Vue from 'vue'

export default {
  name: 'App',
  components: {
    HelloWorld,
    Matriz
  },
  data () {
    return {
      tamanho: '3',
      itemsMatriz: [],
      instanciaMatriz: '',
    }
  },
  mounted () {
    // this.$on('valoresACalcular', this.calcular)
  },
  methods: {
    gerarMatriz () {
      let tamanhoMatriz = this.tamanho;

      for (let i = 1; i <= tamanhoMatriz; i++) {
        let linha = [];
        for (let j = 1; j <= tamanhoMatriz; j++) {
          let ref = i + '' + j
          linha.push({nome: ref, valor: 0})
        }
        this.itemsMatriz.push(linha)
      }
      //console.log(this.itemsMatriz);

      let matrizClass = Vue.extend(Matriz);
      let instanciaMatriz = new matrizClass({
        propsData: {items: this.itemsMatriz, tamanho: tamanhoMatriz }
      });
      instanciaMatriz.$mount();

      let rowTabs = this.$refs.linhaMatriz;

      if (rowTabs.childNodes.length > 0) {
        rowTabs.replaceChild(instanciaMatriz.$el, rowTabs.firstChild)
      } else {
        rowTabs.appendChild(instanciaMatriz.$el)
      }
      // console.log(instanciaMatriz)
      this.instanciaMatriz = instanciaMatriz
    },
    pegarDados () {
      this.instanciaMatriz.$emit('pegarValores')
      let mapatriz = this.instanciaMatriz.$data.mapatriz
      this.calcular(mapatriz)
    },
    calcular (mapatriz) {

      function itemMapToObj(value, key, object){
        object[key] = value
      }

      let obMapatriz = {}
      mapatriz.forEach((value, key) => itemMapToObj(value, key, obMapatriz))

      let strMapatriz = JSON.stringify(obMapatriz)

      this.axios.post('http://localhost:8000/gauss/', strMapatriz)
        .then((response) => {
          let dados = response.data;
          this.processarResposta(dados)
        })
        .catch(function (error) {
          console.log(error)
        })
    },
    processarResposta (dados) {
      let itens = dados.split("§§§")
      // console.log(itens)

      let matriz_x = itens[0]
      let strPassos = itens[1]

      let passos_iteracoes = strPassos.split("$$$")
      passos_iteracoes.shift()
      // console.log(passos_iteracoes)

      let passos = new Map()

      passos_iteracoes.forEach((strPasso) => {
        let arrPasso = strPasso.split("###")
        let numPasso = arrPasso[0]
        // console.log( numPasso )
        let strAB = arrPasso[1].split("&&&")

        let strA = strAB[0]
        let arrA = JSON.parse(strA)
        // console.log(arrA)

        let strB = strAB[1]
        let arrB = JSON.parse(strB)
        // console.log(arrB)

        passos.set(numPasso, [arrA, arrB])
      })

      console.log(passos)
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
