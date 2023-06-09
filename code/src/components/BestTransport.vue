<template>
  <div class="title">
    <b-navbar toggleable="lg" type="dark" variant="info">
      <b-navbar-brand class="ml-2">
        <b>{{ appName }}</b>
      </b-navbar-brand>
    </b-navbar>

    <div class="form-group">
      <label for="peso">Peso do frete:</label>
      <input type="number" id="peso" class="form-control" v-model="pesoFrete">
    </div>

    <div class="form-group">
      <label for="city">Cidade de destino:</label>
      <select id="city" class="form-control" v-model="cidadeSelecionada">
        <option v-for="city in cidades" :key="city"> {{ city }}</option>
      </select>
    </div>
    <button class="btn btn-primary" @click="methodFoo">Analisar</button>
  </div>

</template>

<script>
import {
  BNavbar,
  BNavbarBrand,
} from 'bootstrap-vue'

export default {
  components: {
    BNavbar,
    BNavbarBrand,
  },
  data() {
    const appName = ''

    return {
      appName,
      pesoFrete: 0,
      cidadeSelecionada: '',
      cidades: [],
      freteBarato: null,
      freteRapido: null
    }
  },
  created() {
    this.appName = 'Melhor Frete'
    const apiUrl = 'http://localhost:3000/transport'

    // Fetch nos dados da API
    fetch (apiUrl)
    .then (response => response.json())
    .then(data =>{

        // Retornando as cidades únicas para o objeto cidades.
        let cidadesUnicas = {};
        for (let i = 0; i < data.length; i++){
          let obj = data[i];
          cidadesUnicas[obj.city] = true;
        }

        this.cidades = Object.keys(cidadesUnicas);
        console.log(this.cidades)
    })
    .catch(error =>{
        console.error('Erro ao obter dados do frete:', error);
      })
  },
  methods: {
    // Implemente aqui os metodos utilizados na pagina
    methodFoo() {
    },
  },
}
</script>

<style scoped>
.title .navbar {
  background-color: #00aca6 !important;
}

.title .navbar-brand {
  margin-left: 20px;
}
</style>
