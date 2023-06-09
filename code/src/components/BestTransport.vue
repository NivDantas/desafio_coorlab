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

    <!-- Divisão com v-if para peso, e tipo de frete. -->
    <div v-if="freteBarato && pesoAnt <= 100">
    <h3>Frete mais econômico:</h3>
    <p>Transportadora: {{freteBarato.name}}</p>
    <p>Custo total: R${{ formatarCustoTotalEconomica() }}</p>
    <p>Tempo de entrega: {{ freteBarato.lead_time}} horas</p>
    </div>

    <div v-if="freteBarato && pesoAnt > 100">
      <h3>Frete mais econômico:</h3>
      <p>Transportadora: {{freteBarato.name}}</p>
      <p>Custo total: R$ {{ formatarCustoTotalEconomica()}}</p>
      <p>Tempo de entrega: {{freteBarato.lead_time}} horas</p>
    </div>

    <div v-if="freteRapido && pesoAnt > 100">
      <h3>Frete mais rápido:</h3>
      <p>Transportadora: {{freteRapido.name}}</p>
      <p>Custo total: R$ {{ formatarCustoTotalRapida()}}</p>
      <p>Tempo de entrega: {{freteRapido.lead_time}} horas</p>
    </div>

    <div v-if="freteRapido && pesoAnt <= 100">
      <h3>Frete mais rápido:</h3>
      <p>Transportadora: {{freteRapido.name}}</p>
      <p>Custo total: R$ {{ formatarCustoTotalRapida()}}</p>
      <p>Tempo de entrega: {{freteRapido.lead_time}} horas</p>
    </div>
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
      pesoAnt: 0,
      cidadeSelecionada: '',
      cidades: [],
      freteBarato: null,
      freteRapido: null,
      apiData: [],
    }
  },
  created() {
    this.appName = 'Melhor Frete'
    const apiUrl = 'http://localhost:3000/transport'
    console.log('Oii')

    // Fetch nos dados da API
    fetch (apiUrl)
    .then (response => response.json())
    .then(data =>{

        // Armazenando o dataset em uma variável para utilização nos methods.
        this.apiData = data;

        // Retornando as cidades únicas para o objeto cidades e formatando os dados para tipo Float/Inteiro.
        let cidadesUnicas = {};
        for (let i = 0; i < data.length; i++){
          let obj = data[i];
          cidadesUnicas[obj.city] = true;
          this.apiData[i].cost_transport_light = parseFloat(this.apiData[i].cost_transport_light.replace(/[^0-9.]/g, ''))
          this.apiData[i].cost_transport_heavy = parseFloat(this.apiData[i].cost_transport_heavy.replace(/[^0-9.]/g, ''))
          this.apiData[i].lead_time = parseInt(this.apiData[i].lead_time)
        }

        this.cidades = Object.keys(cidadesUnicas);
        
})
    .catch(error =>{
        console.error('Erro ao obter dados do frete:', error);
      })
  },

  watch: {
  pesoFrete(newValue) {
    this.pesoFrete = parseFloat(newValue);
  }
  },
  methods: {
    // Implemente aqui os metodos utilizados na pagina
    methodFoo() {
      // Filtrando para pegar somente a cidade selecionada.
      const freteFiltrado = this.apiData.filter(frete => frete.city === this.cidadeSelecionada)
      console.log(freteFiltrado)
      // Definição do peso baseado no input.
      this.pesoAnt = this.pesoFrete
      // Função sort para menor preço, dividindo em pesos.
      if (this.pesoAnt <= 100){
        freteFiltrado.sort((a, b) => a.cost_transport_light - b.cost_transport_light)
        this.freteBarato = freteFiltrado[0];

      }
      else if (this.pesoAnt > 100){
        freteFiltrado.sort((a, b) => a.cost_transport_heavy - b.cost_transport_heavy)
        this.freteBarato = freteFiltrado[0];
      }
      // Função sort para menor tempo.
      freteFiltrado.sort((a, b) => a.lead_time - b.lead_time)
      this.freteRapido = freteFiltrado[0];

      console.log(this.freteRapido);
    },
    // Método para formatar o custo total economico.
    formatarCustoTotalEconomica() {
      let custoTotal;
      if (this.pesoAnt <= 100){
      custoTotal = this.freteBarato.cost_transport_light * this.pesoAnt;

      return custoTotal.toFixed(2);
      }
      else if (this.pesoAnt > 100){
      custoTotal = this.freteBarato.cost_transport_heavy * this.pesoAnt
        
      return custoTotal.toFixed(2);
      }
    // Método para formatar o custo total frete rápido.
    },
    formatarCustoTotalRapida() {
      let custoTotal;
      if (this.pesoAnt <= 100){
      custoTotal = this.freteRapido.cost_transport_light * this.pesoAnt;

      return custoTotal.toFixed(2);
      }
      else if (this.pesoAnt > 100){
      custoTotal = this.freteRapido.cost_transport_heavy * this.pesoAnt
        
      return custoTotal.toFixed(2);
      }

    }
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
