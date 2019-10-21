<template>
  <div id="app">
    <b-container>

      <b-row class="line-padin2">
        <b-col>
          <b-row class="justify-content-md-center">
            <h2>Metodo da Eliminação de Gauss</h2>
          </b-row>
          <b-row class="justify-content-md-center">
            <b-img fuild rounded="circle" class="img-gauss" src="../static/images/gauss.jpg"></b-img>
          </b-row>
        </b-col>
      </b-row>

      <b-row>
        <b-input-group prepend="Tamanho:" class="mt-3 inputCamp">
          <b-form-input v-model="tamanho" placeholder="Ex: 2" type="number"></b-form-input>
          <b-input-group-append>
            <b-button variant="outline-primary" @click="gerarMatrizVazia">Gerar Matriz</b-button>
          </b-input-group-append>
        </b-input-group>
      </b-row>

      <b-row>
        <b-col class="linha-padin">
          <h3 v-if="isResultado">{{tituloResultado}}</h3>
          <h5 v-else>Tamanho: {{ tamanho }} X {{ tamanho }}</h5>
        </b-col>
      </b-row>
      
      <b-row v-if="isResultado"  class="linha-padin">
        <b-col>
          <h5>Passo {{passo}}/{{totalDePassos}}</h5>
        </b-col>

        <b-row>
          <b-col>
            <div v-if="passo > 1">
              <b-button @click="anteriorPasso">Anterior</b-button>
            </div>
            <div v-else>
              <b-button disabled @click="anteriorPasso">Anterior</b-button>
            </div>  
          </b-col>

          <b-col>
            <div v-if="passo < totalDePassos">
              <b-button @click="proximoPasso">Proximo</b-button>
            </div>
            <div v-else>
              <b-button disabled @click="proximoPasso">Proximo</b-button>
            </div>
          </b-col>
        </b-row>

        <b-col>
          <b-button ref="btnLU" variant="outline-primary" @click="exibirMatrizLU">Exibir matriz L U</b-button>
        </b-col>
      </b-row>

      <b-row  class="linha-padin">
        <b-col ref="linhaMatriz">
          <matriz></matriz>
        </b-col>
        <b-col cols="1">
          
          <b-row>
            <b-button variant="outline-primary" @click="pegarDados">Calcular</b-button>
            <b-button v-if="isDev" variant="outline-primary" @click="testeresponse">Calcular-dev</b-button>
          </b-row>

        </b-col>
      </b-row>

      <b-row class="linha-padin">
        <b-col>
          <b-row>
            <b-col>
              <h5>{{tituloL}}</h5>
            </b-col>
          </b-row>
          <b-row>
            <b-col ref="linhaMatrizL">
              <matriz class="matriz"></matriz>
            </b-col>
          </b-row>
        </b-col>

        <b-col>
          <b-row>
            <b-col>
              <h5 >{{tituloU}}</h5>
            </b-col>
          </b-row>
          <b-row>
            <b-col ref="linhaMatrizU">
              <matriz></matriz>
            </b-col>
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
      isDev: true,
      isResultado: false,
      tituloResultado: '',
      passo: 0,
      totalDePassos: 0,
      todosOsPassos: [],
      exibirLU: false,
      tituloL: '',
      tituloU: '',
      casasDecimais: 3,
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
      this.isResultado = false
      this.tituloL = ''
      this.tituloU = ''
      let rowL = this.$refs.linhaMatrizL;
      let rowU = this.$refs.linhaMatrizU;
      if (rowL.hasChildNodes()) {
        rowL.removeChild(rowL.childNodes[0]);
      }
      if (rowU.hasChildNodes()) {
        rowU.removeChild(rowU.childNodes[0]);
      }
    },
    gerarMatrizComValores (valoresA, valoresB, tamanho) {
      let tamanhoMatriz = Number(tamanho)

      let valoresNovos = []
      for (let i = 1; i < tamanhoMatriz+1; i++) {
        let linha = [];
        for (let j = 1; j < tamanhoMatriz+1; j++) {
          let ref = i + '' + j
          let valor = valoresA[i-1][j-1]
          valor = Number.isInteger(valor) ? valor : valor.toPrecision(this.casasDecimais)
          linha.push({nome: ref, valor: valor})
        }
        if (valoresB) {
          let refB = i + '' + (tamanhoMatriz+1)
          let valorB = valoresB[i-1]
          valorB = Number.isInteger(valorB) ? valorB : valorB.toPrecision(this.casasDecimais)
          linha.push({nome: refB, valor: valorB})
        }
        valoresNovos.push(linha)
      }
      return valoresNovos
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
      this.exibirLU = true

      let matL = this.gerarMatrizComValores(this.matrizL, null, this.matrizL.length)
      let matU = this.gerarMatrizComValores(this.matrizU, null, this.matrizL.length)

      let matrizClass = Vue.extend(Matriz);
      let instanciaMatrizL = new matrizClass({
        propsData: {items: matL, tamanho: this.matrizL.length}
      });
      let instanciaMatrizU = new matrizClass({
        propsData: {items: matU, tamanho: this.matrizU.length}
      });

      instanciaMatrizL.$mount();
      instanciaMatrizU.$mount();

      let rowL = this.$refs.linhaMatrizL;
      let rowU = this.$refs.linhaMatrizU;

      if (rowL.childNodes.length > 0) {
        rowL.replaceChild(instanciaMatrizL.$el, rowL.firstChild)
      } else {
        rowL.appendChild(instanciaMatrizL.$el)
      }
      if (rowU.childNodes.length > 0) {
        rowU.replaceChild(instanciaMatrizU.$el, rowU.firstChild)
      } else {
        rowU.appendChild(instanciaMatrizU.$el)
      }

      this.tituloL = 'Matriz L'
      this.tituloU = 'Matriz U'
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
      let maxKey = 0
      for (let [k, v] of passos) {
        maxKey = k > maxKey ? k : maxKey
      }

      this.totalDePassos = maxKey
      this.passo = maxKey

      this.todosOsPassos = passos

      this.isResultado = true

      this.matrizL = matrizL
      this.matrizU = matrizU

      this.tituloResultado = 'Resultado'

      let matrizResultado = passos.get(maxKey)
      let matrizA = matrizResultado[0]
      let matrizB = matrizResultado[1]

      let m = this.gerarMatrizComValores(matrizA, matrizB, 3)
      this.instanciarMatriz(m, m.length)
    },
    proximoPasso () {
      if (this.passo < this.totalDePassos) {
        let passo = Number(this.passo) + 1
        this.trocaPasso(passo)
      }
    },
    anteriorPasso () {
      if (this.passo > 1) {
        let passo = Number(this.passo) - 1
        this.trocaPasso(passo)
      }
    },
    trocaPasso (passo) {
      let proximaMatriz = this.todosOsPassos.get(String(passo))
      let matrizA = proximaMatriz[0]
      let matrizB = proximaMatriz[1]
      let m = this.gerarMatrizComValores(matrizA, matrizB, matrizA.length)
      this.instanciarMatriz(m, m.length)
      this.passo = passo
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
.linha-padin {
  padding: 10px;
}
.line-padin2 {
  padding: 10px;
  padding-bottom: 30px;
}
.img-gauss {
  width: 15%;
  height: 15%;
}
</style>
