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
      itemsMatriz: []
    }
  },
  methods: {
    gerarMatriz() {
      let tamanhoMatriz = this.tamanho;

      console.log('Tamanho da Matrix: ', tamanhoMatriz);
      let linha = {};

      for (let i = 0; i < tamanhoMatriz; i++) {
        linha["X" + i] = "0"
      }
      console.log(linha);

      for (let i = 0; i < tamanhoMatriz; i++) {
        this.itemsMatriz[i] = linha
      }
      console.log(this.itemsMatriz);


      let matrizClass = Vue.extend(Matriz)
      let instanciaMatriz = new matrizClass({
        propsData: {items: this.itemsMatriz}
      })
      instanciaMatriz.$mount()

      let rowTabs = this.$refs.linhaMatriz

      if (rowTabs.childNodes.length > 0) {
        rowTabs.replaceChild(instanciaMatriz.$el, rowTabs.firstChild)
      } else {
        rowTabs.appendChild(instanciaMatriz.$el)
      }

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
