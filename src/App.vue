<template>
  <div id="app">
    <b-container>

      <b-row>
        <b-input-group prepend="Tamanho:" class="mt-3 inputCamp">
          <b-form-input v-model="tamanho" placeholder="Ex: 2" type="number"></b-form-input>
          <b-input-group-append>
            <b-button variant="outline-primary" @click="gerarMatrizVazia">Gerar Matriz</b-button>
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
            <b-button v-if="isDev" variant="outline-primary" @click="testeresponse">Calcular-dev</b-button>
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
  created () {
    let isDev = process.env.NODE_ENV === 'development'

    let endereco = isDev ? 'http://localhost:8000' : 'https://matheusmuriel.pythonanywhere.com'

    this.isDev = isDev
    this.urlApi = endereco
  },
  data () {
    return {
      tamanho: '3',
      instanciaMatriz: '',
      urlApi: '',
      isDev: true
    }
  },
  methods: {
    testeresponse () {
      let respontaRequestFixa = '$$$1###[[1.0, 2.0, 3.0], [0.0, -3.0, -4.0], [3.0, 2.0, 1.0]]&&&[9.0, -11.0, 6.0]$$$2###[[1.0, 2.0, 3.0], [0.0, -3.0, -4.0], [0.0, -4.0, -8.0]]&&&[9.0, -11.0, -21.0]$$$3###[[1.0, 2.0, 3.0], [0.0, -3.0, -4.0], [0.0, 0.0, -2.666666666666667]]&&&[9.0, -11.0, -6.333333333333334]§§§[0.875, 0.5, 2.375]§§§[[1, 0, 0], [2, 1, 0], [3, 1, 1]]§§§[[1.0, 2.0, 3.0], [0.0, -3.0, -4.0], [0.0, 0.0, -2.666666666666667]]'

      this.processarResposta(respontaRequestFixa)
    },
    gerarMatrizVazia () {
      let tamanhoMatriz = Number(this.tamanho)

      let valores = []
      for (let i = 1; i <= tamanhoMatriz; i++) {
        let linha = [];
        for (let j = 1; j <= tamanhoMatriz+1; j++) {
          let ref = i + '' + j
          linha.push({nome: ref, valor: 0})
        }
        valores.push(linha)
      }

      this.instanciarMatriz(valores, tamanhoMatriz)
    },
    gerarMatrizComValores (valoresA, valoresB, tamanho) {
      let tamanhoMatriz = Number(tamanho)

      let valoresNovos = []
      for (let i = 1; i < tamanhoMatriz+1; i++) {
        let linha = [];
        for (let j = 1; j < tamanhoMatriz+1; j++) {
          let ref = i + '' + j
          let valor = valoresA[i-1][j-1]
          linha.push({nome: ref, valor: valor})
        }
        let refB = i + '' + (tamanhoMatriz+1)
        console.log(refB)
        let valorB = valoresB[i-1]
        console.log(valorB)
        linha.push({nome: refB, valor: valorB})
        
        valoresNovos.push(linha)
      }
      console.log(valoresNovos)
      this.instanciarMatriz(valoresNovos, tamanhoMatriz)
    },
    instanciarMatriz (valores, tamanho) {
      let matrizClass = Vue.extend(Matriz);
      let instanciaMatriz = new matrizClass({
        propsData: {items: valores, tamanho: tamanho}
      });
      instanciaMatriz.$mount();

      let rowTabs = this.$refs.linhaMatriz;

      if (rowTabs.childNodes.length > 0) {
        rowTabs.replaceChild(instanciaMatriz.$el, rowTabs.firstChild)
      } else {
        rowTabs.appendChild(instanciaMatriz.$el)
      }
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

      console.log(strMapatriz)

      this.axios.post(this.urlApi + '/gauss/', strMapatriz)
        .then((response) => {
          let dados = response.data;
          this.processarResposta(dados)
        })
        .catch(function (error) {
          console.log(error)
        })
    },
    exibirMatrizLU () {
      let botaoExibir = this.$refs.btnLU;

      botaoExibir.setAttribute('style', 'display: auto')

    },
    processarResposta (dados) {
      // str_passos + "§§§" + matriz_x + "§§§" + matriz_l + "§§§" + matriz_u
      let itens = dados.split("§§§")

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

      this.exibirResposta(passos, matrizX, matrizL, matrizU)
    },
    exibirResposta (passos, matrizX, matrizL, matrizU) {
      /**
      console.table(passos)
      console.table(matrizX)
      console.table(matrizL)
      console.table(matrizU)
      */

      let maxKey = 0

      for (let [k, v] of passos) {
        maxKey = k > maxKey ? k : maxKey
      }

      let matrizResultado = passos.get(maxKey)

      let matrizA = matrizResultado[0]
      let matrizB = matrizResultado[1]

      this.gerarMatrizComValores(matrizA, matrizB, 3)
      this.exibirMatrizLU()
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
