<template>
  <div id="app">
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
          
          <b-row>
            <b-button variant="outline-primary" @click="pegarDados">Calcular</b-button>
          </b-row>

          <b-row>
            <b-button style="display: none" ref="btnLU" variant="outline-primary" @click="exibirMatrizLU">L U</b-button>
          </b-row>

        </b-col>
      </b-row>


    </b-container>
  </div>
</template>

<script>
import Matriz from './components/Matriz'
import Vue from 'vue'

export default {
  name: 'App',
  components: {
    Matriz
  },
  data () {
    return {
      tamanho: '3',
      itemsMatriz: [],
      instanciaMatriz: ''
    }
  },
  mounted () {
    // this.$on('valoresACalcular', this.calcular)
  },
  methods: {
    gerarMatriz () {
      let tamanhoMatriz = Number(this.tamanho)

      for (let i = 1; i <= tamanhoMatriz; i++) {
        let linha = [];
        for (let j = 1; j <= tamanhoMatriz+1; j++) {
          let ref = i + '' + j
          linha.push({nome: ref, valor: 0})
        }
        this.itemsMatriz.push(linha)
      }
      // console.log(this.itemsMatriz);

      let matrizClass = Vue.extend(Matriz);
      let instanciaMatriz = new matrizClass({
        propsData: {items: this.itemsMatriz, tamanho: tamanhoMatriz}
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

      this.gerartabNav()
    },
    gerartabNav () {

      let navMatrizClass = Vue.extend(NavMatriz);
      let instanciaNavMatriz = new navMatrizClass({
        propsData: {listaItens: this.itemsMatriz}
      });
      instanciaNavMatriz.$mount();

      let rowNav = this.$refs.nav;

      rowNav.appendChild(instanciaNavMatriz.$el)
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

      console.log(strMapatriz)

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

      let matriz_x = JSON.parse(itens[0])
      let strPassos = itens[1]

      let passos_iteracoes = strPassos.split("$$$")
      passos_iteracoes.shift()

      let passos = new Map()

      passos_iteracoes.forEach((strPasso) => {
        let arrPasso = strPasso.split("###")
        let numPasso = arrPasso[0]
        let strAB = arrPasso[1].split("&&&")

        let strA = strAB[0]
        let arrA = JSON.parse(strA)

        let strB = strAB[1]
        let arrB = JSON.parse(strB)

        passos.set(numPasso, [arrA, arrB])
      })
      console.log(matriz_x, passos)
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
