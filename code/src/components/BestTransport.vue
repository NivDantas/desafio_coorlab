<template>
  <div class="title">
    <b-navbar toggleable="lg" type="dark" variant="info">
      <b-navbar-brand class="ml-2"> </b-navbar-brand>
    </b-navbar>
    <div class="dim" v-if="exibirAviso"></div>
    <div class="grid">
      <div class="navfrete">
        <img class="logo" src="../assets/logo.png" />
        <h1>{{ appName }}</h1>
      </div>

      <div class="formulario">
        <div class="form-group">
          <h2>
            <img class="maplogo" src="../assets/maplogo.svg" /> Insira o destino
            e o peso
          </h2>
          <label for="city"><b>Cidade de destino:</b></label>
          <!-- Seleção da cidade com o uso de v-model e v-for. -->
          <select id="city" class="form-control" v-model="cidadeSelecionada">
            <option class="first-option" disabled selected value="">
              Selecione o destino
            </option>
            <option v-for="city in cidades" :key="city">{{ city }}</option>
          </select>
        </div>
        <div class="form-group">
          <!-- Uso de v-model para peso. -->
          <label for="peso"><b>Peso do frete:</b></label>
          <input
            type="number"
            id="peso"
            class="form-control"
            placeholder="Peso da carga em kg"
            v-model="pesoTempoReal"
          />
        </div>
        <button class="btn btn-primary" @click="filtrarValorFrete">
          Analisar
        </button>
      </div>
      <div class="resultados">
        <transition name="semDados">
          <div class="semDados" v-if="!freteRapido && !freteBarato">
            <h2>Nenhum dado selecionado.</h2>
          </div>
        </transition>
        <!-- Divisão com v-if para peso, e tipo de frete. -->
        <transition>
          <div class="valorFrete" v-if="freteBarato && peso <= 100">
            <img src="../assets/freteBarato.svg" />
            <p><b>Frete com menor valor</b></p>
            <p>Transportadora: {{ freteBarato.name }}</p>
            <p>Tempo de entrega: {{ freteBarato.lead_time }} horas</p>
            <p>
              Custo total: <b>R${{ formatarCustoTotalEconomica() }}</b>
            </p>
          </div>
        </transition>

        <transition>
          <div class="valorFrete" v-if="freteBarato && peso > 100">
            <img src="../assets/freteBarato.svg" />
            <p><b>Frete com menor valor</b></p>
            <p>Transportadora: {{ freteBarato.name }}</p>
            <p>Tempo de entrega: {{ freteBarato.lead_time }} horas</p>
            <p>
              Custo total: <b>R$ {{ formatarCustoTotalEconomica() }}</b>
            </p>
          </div>
        </transition>
        <transition>
          <div class="valorFrete" v-if="freteRapido && peso > 100">
            <img src="../assets/freteRapido.svg" />
            <p><b>Frete mais rápido</b></p>
            <p>Transportadora: {{ freteRapido.name }}</p>
            <p>Tempo de entrega: {{ freteRapido.lead_time }} horas</p>
            <p>
              Custo total: <b>R${{ formatarCustoTotalEconomica() }}</b>
            </p>
          </div>
        </transition>
        <transition>
          <div class="valorFrete" v-if="freteRapido && peso <= 100">
            <img src="../assets/freteRapido.svg" />
            <p><b>Frete mais rápido</b></p>
            <p>Transportadora: {{ freteRapido.name }}</p>
            <p>Tempo de entrega: {{ freteRapido.lead_time }} horas</p>
            <p>
              Custo total: <b>R$ {{ formatarCustoTotalRapida() }}</b>
            </p>
          </div>
        </transition>
        <transition>
          <!-- Botão para limpar os resultados. -->
          <button
            @click="limparFretes"
            class="limpar"
            v-if="freteRapido && freteBarato && peso"
          >
            Limpar
          </button>
        </transition>
        <transition>
          <!--  Aviso para caso dados não tenham sido inseridos pelo usuário. -->
          <div v-if="exibirAviso" class="aviso">
            <img src="../assets/warning.svg" />
            <h3>Insira os valores para realizar a análise.</h3>
            <button @click="fecharAviso">Fechar</button>
          </div>
        </transition>
      </div>
    </div>
  </div>
</template>

<script>
import { BNavbar, BNavbarBrand } from "bootstrap-vue";

export default {
  components: {
    BNavbar,
    BNavbarBrand,
  },
  data() {
    const appName = "";

    return {
      appName,
      pesoTempoReal: null,
      peso: 0,
      exibirAviso: false,
      cidadeSelecionada: "",
      cidades: [],
      freteBarato: null,
      freteRapido: null,
      apiData: [],
    };
  },
  created() {
    this.appName = "Melhor Frete";
    const apiUrl = "http://localhost:3000/transport";

    // Fetch nos dados da API
    fetch(apiUrl)
      .then((response) => response.json())
      .then((data) => {
        // Armazenando o dataset em uma variável para utilização nos methods.
        this.apiData = data;

        // Retornando as cidades únicas para o objeto cidades e formatando os dados para tipo Float/Inteiro.
        let cidadesUnicas = {};
        for (let i = 0; i < data.length; i++) {
          let obj = data[i];
          cidadesUnicas[obj.city] = true;
          this.apiData[i].cost_transport_light = parseFloat(
            this.apiData[i].cost_transport_light.replace(/[^0-9.]/g, "")
          );
          this.apiData[i].cost_transport_heavy = parseFloat(
            this.apiData[i].cost_transport_heavy.replace(/[^0-9.]/g, "")
          );
          this.apiData[i].lead_time = parseInt(this.apiData[i].lead_time);
        }
        // Pegando cidades únicas para valor do formulário.
        this.cidades = Object.keys(cidadesUnicas);
      })
      .catch((error) => {
        console.error("Erro ao obter dados do frete:", error);
      });
  },
  watch: {
    // Ver as mudanças do peso em tempo real.
    pesoTempoReal(newValue) {
      this.pesoTempoReal = parseFloat(newValue);
    },
  },

  methods: {
    // Implemente aqui os metodos utilizados na pagina
    filtrarValorFrete() {
      // Filtrando para pegar somente a cidade selecionada.
      const freteFiltrado = this.apiData.filter(
        (frete) => frete.city === this.cidadeSelecionada
      );
      console.log(freteFiltrado);
      // Definição do peso baseado no input.
      this.peso = this.pesoTempoReal;
      if (!this.cidadeSelecionada || !this.peso) {
        this.exibirAviso = true;
        return -1;
      }
      // Função sort para menor preço, dividindo em pesos.
      if (this.peso <= 100) {
        freteFiltrado.sort(
          (a, b) => a.cost_transport_light - b.cost_transport_light
        );
        this.freteBarato = freteFiltrado[0];
      } else if (this.peso > 100) {
        freteFiltrado.sort(
          (a, b) => a.cost_transport_heavy - b.cost_transport_heavy
        );
        this.freteBarato = freteFiltrado[0];
      }
      // Função sort para menor tempo.
      freteFiltrado.sort((a, b) => a.lead_time - b.lead_time);
      this.freteRapido = freteFiltrado[0];
    },
    // Método para formatar o custo total economico.
    formatarCustoTotalEconomica() {
      let custoTotal;
      if (this.peso <= 100) {
        custoTotal = this.freteBarato.cost_transport_light * this.peso;

        return custoTotal.toFixed(2);
      } else if (this.peso > 100) {
        custoTotal = this.freteBarato.cost_transport_heavy * this.peso;

        return custoTotal.toFixed(2);
      }
      // Método para formatar o custo total frete rápido.
    },
    formatarCustoTotalRapida() {
      let custoTotal;
      if (this.peso <= 100) {
        custoTotal = this.freteRapido.cost_transport_light * this.peso;

        return custoTotal.toFixed(2);
      } else if (this.peso > 100) {
        custoTotal = this.freteRapido.cost_transport_heavy * this.peso;

        return custoTotal.toFixed(2);
      }
    },
    // Método para fechar o aviso.
    fecharAviso() {
      this.exibirAviso = false;
    },
    // Método para limpar os resultados, junto com os valores inseridos pelo usuário no formulário.
    limparFretes() {
      this.freteRapido = null;
      this.freteBarato = null;
      this.pesoTempoReal = null;
      this.cidadeSelecionada = "";
    },
  },
};
</script>

<style scoped>
/* Estilos aplicados no componente, em ordem de inserção no HTML. */
html {
  background-color: #f2f2f2;
}

h1 {
  font-size: 32px;
}

h2 {
  font-size: 28px;
  margin-bottom: 30px;
  color: #363636;
}

h3 {
  font-size: 24px;
}

label {
  margin-bottom: 10px;
  color: #505059;
}

select {
  background-image: linear-gradient(45deg, transparent 50%, gray 50%),
    linear-gradient(135deg, gray 50%, transparent 50%),
    linear-gradient(to right, #ccc, #ccc);
  background-position: calc(100% - 20px) calc(1em + 2px),
    calc(100% - 15px) calc(1em + 2px), calc(100% - 2.5em) 0.5em;
  background-size: 5px 5px, 5px 5px, 1px 1.5em;
  background-repeat: no-repeat;
}

select:focus {
  background-image: linear-gradient(45deg, green 50%, transparent 50%),
    linear-gradient(135deg, transparent 50%, green 50%),
    linear-gradient(to right, #ccc, #ccc);
  background-position: calc(100% - 15px) 1em, calc(100% - 20px) 1em,
    calc(100% - 2.5em) 0.5em;
  background-size: 5px 5px, 5px 5px, 1px 1.5em;
  background-repeat: no-repeat;
  border-color: green;
  outline: 0;
}

.title .navbar {
  background-color: #596c8a !important;
}

.title .navbar-brand {
  margin-left: 20px;
}
.title {
  height: 100vh;
}
.navbar {
  height: 5vh;
}

.grid {
  height: 80vh;
  margin: 40px auto;
  width: 1500px;
  box-shadow: rgba(100, 100, 111, 0.2) 0px 7px 29px 0px;
  display: grid;
  grid-template-rows: 1fr 14fr;
  grid-template-columns: 1fr 2fr;
  align-items: center;
}

.navfrete {
  display: flex;
  align-items: center;
  grid-column: span 2;
  font-weight: bolder;
  color: white;
  width: 100%;
  height: 6vh;
  background-color: #7195bf;
  border-radius: 10px 10px 0px 0px;
  align-self: start;
}

.navfrete h1 {
  height: 30px;
  color: black;
}

.navfrete h1,
.logo {
  margin-left: 40px;
}
.logo {
  width: 50px;
  height: 50px;
}
.maplogo {
  width: 40px;
  height: 40px;
}
.formulario,
.resultado {
  margin: 40px 20px;
}

.formulario {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  align-self: flex-end;
  height: 90%;
  width: 500px;
  gap: 40px;
  background: #f2f2f2;
  border-radius: 10px;
}

.formulario input,
.formulario select {
  width: 400px;
}

.formulario button {
  width: 200px;
  background-color: #7195bf;
  color: #363636;
  font-weight: bolder;
  border: none;
}

.resultados {
  display: flex;
  flex-wrap: wrap;
  flex-direction: column;
  gap: 60px;
  justify-self: center;
}

.valorFrete {
  display: grid;
  border-radius: 10px;
  height: 150px;
  width: 800px;
  grid-template-columns: 1fr 4fr;
  grid-template-rows: auto;
  background-color: #5e6c89;
  background-image: linear-gradient(
    to right,
    rgba(0, 0, 0, 0) 20%,
    #e2e2e2 20%
  );
}

.valorFrete img {
  grid-column: 1;
  grid-row: span 4;
  justify-self: center;
  align-self: center;
}

.valorFrete p {
  justify-self: center;
  align-self: center;
  grid-column: 2;
  height: 20px;
}

.valorFrete b {
  font-size: 20px;
}

.aviso {
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  background-color: #fff;
  border-radius: 10px;
  gap: 30px;
  flex-direction: column;
  width: 750px;
  height: 375px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  box-shadow: rgba(100, 100, 111, 0.2) 0px 7px 29px 0px;
  z-index: 5;
}

.aviso button {
  width: 200px;
  height: 30px;
  background-color: #7195bf;
  color: #363636;
  font-weight: bolder;
  border: none;
  border-radius: 5px;
}

.first-option {
  color: gray;
}

.v-enter-active {
  transition: opacity 3s;
}

.v-leave-active {
  transition: opacity 0.5s;
}

.v-enter,
.v-leave-to {
  opacity: 0;
}

.semDados-enter {
  opacity: 0;
}

.semDados-enter-active {
  transition: opacity 7s;
}

.dim {
  height: 100%;
  width: 100%;
  position: fixed;
  left: 0;
  top: 0;
  z-index: 1 !important;
  background-color: rgba(0, 0, 0, 0.2);
}

.limpar {
  width: 200px;
  height: 30px;
  background-color: #626c88;
  color: #363636;
  font-weight: bolder;
  border: none;
  border-radius: 5px;
  align-self: flex-end;
}
</style>
