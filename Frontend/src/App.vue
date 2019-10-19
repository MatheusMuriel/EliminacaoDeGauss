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
            <b-button variant="outline-primary" @click="testeresponse">Calcular-dev</b-button>
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
    testeresponse () {
      let respontaRequestFixa = '$$$1###[[1.0, 2.0, 3.0], [0.0, -3.0, -4.0], [3.0, 2.0, 1.0]]&&&[9.0, -11.0, 6.0]$$$2###[[1.0, 2.0, 3.0], [0.0, -3.0, -4.0], [0.0, -4.0, -8.0]]&&&[9.0, -11.0, -21.0]$$$3###[[1.0, 2.0, 3.0], [0.0, -3.0, -4.0], [0.0, 0.0, -2.666666666666667]]&&&[9.0, -11.0, -6.333333333333334]§§§[0.875, 0.5, 2.375]§§§[[1, 0, 0], [2, 1, 0], [3, 1, 1]]§§§[[1.0, 2.0, 3.0], [0.0, -3.0, -4.0], [0.0, 0.0, -2.666666666666667]]'

      this.processarResposta(respontaRequestFixa)
    },
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
    exibirMatrizLU () {
      // TODO
    },
    processarResposta (dados) {
      // str_passos + "§§§" + matriz_x + "§§§" + matriz_l + "§§§" + matriz_u
      let itens = dados.split("§§§")

      console.table(itens)

      let strPassos = itens[0]
      let strMatrizX = itens[1]
      let strMatrizL = itens[2]
      let strMatrizU = itens[3]
      
      let passosIteracoes = strPassos.split("$$$")
          passosIteracoes.shift()

      let passos = new Map()
      passosIteracoes.forEach((strPasso) => {
        let arrPasso = strPasso.split("###")
        let numPasso = arrPasso[0]
        let strAB = arrPasso[1].split("&&&")

        let strA = strAB[0]
        let arrA = JSON.parse(strA)

        let strB = strAB[1]
        let arrB = JSON.parse(strB)

        passos.set(numPasso, [arrA, arrB])
      })
      // O resultado é o ultimo item de passos

      let matrizX = JSON.parse(strMatrizX)

      let matrizL = JSON.parse(strMatrizL)
      let matrizU = JSON.parse(strMatrizU)


      console.log('Passos ')
      console.log(passos)

      console.log('Matriz X', matrizX)

      console.log('Matriz L', matrizL)
      console.log('Matriz U', matrizU)
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
