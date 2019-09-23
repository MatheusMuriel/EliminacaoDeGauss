<template>
  <div>
    <div class="lineInput">



      <b-container>



      </b-container>
    </div>
  </div>

</template>
<script>
  export default {
    name: 'InputSize',
    data () {
      return {
        tamanho: '3'
      }
    },
    methods: {
      gerarMatriz () {
        this.$emit('gerarMatriz', {tamanho: this.tamanho})
      }
    }
  }
</script>
<style>
.inputCamp {
  width: 35%;
}
</style>
