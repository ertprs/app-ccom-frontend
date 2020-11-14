<template>
  <div>
    <v-row justify="center">
      <v-col class="text-center text-h6 mx-3" cols="12" sm="12" md="12">
        <h3 class="white--text">Informar viagem com veículo vazio.</h3>
        <p class="description">
          Favor preencher o formulário abaixo com os dados da viagem.
        </p>
        <p class="description">
          Dúvidas contate o CCOM:
        </p>
        <p class="description -item">51 3462-3500</p>
        <p class="description -item">51 9 9186-3152</p>
        <p class="description -item">ccom.controle@modular.com.br</p>
      </v-col>
    </v-row>
    <v-row justify="center" class="viagem-card">
      <v-card class="mx-3" width="900">
        <v-row justify="center">
          <v-form
            @submit.prevent="save"
            ref="form"
            v-model="valid"
            lazy-validation
            class="ml-5 mr-5 mt-3"
          >
            <v-row justify="center" class="mx-auto">
              <v-col cols="12" sm="4" md="4" lg="5">
                <v-text-field
                  filled
                  label="* Placa do Cavalo"
                  v-model="dados.placa"
                  :rules="rules.placa"
                  v-mask="'XXX-#X##'"
                  required
                ></v-text-field>
              </v-col>
              <v-col cols="12" sm="4" md="4" lg="5">
                <v-text-field
                  filled
                  label="* Motorista"
                  v-model="dados.motorista"
                  required
                  hint="Nome do motorista"
                ></v-text-field>
              </v-col>
              <v-col cols="12" sm="4" md="4" lg="5">
                <v-select
                  filled
                  label="* Estado de origem"
                  v-model="state"
                  :items="states"
                  @change="loadOriginCity"
                ></v-select>
              </v-col>
              <v-col cols="12" sm="4" md="4" lg="5">
                <v-select
                  filled
                  label="* Cidade de origem"
                  v-model="city"
                  :items="originCities"
                ></v-select>
              </v-col>
              <v-col cols="12" sm="4" md="4" lg="5">
                <v-select
                  filled
                  label="* Estado de destino"
                  v-model="destinyState"
                  :items="states"
                  @change="loadDestinyCity"
                ></v-select>
              </v-col>
              <v-col cols="12" sm="4" md="4" lg="5">
                <v-select
                  filled
                  label="* Cidade de destino"
                  v-model="destinyCity"
                  :items="destinyCities"
                ></v-select>
              </v-col>
              <v-col cols="12" sm="4" md="4" lg="5">
                <v-text-field
                  filled
                  label="* Distância a percorrer vazio"
                  v-model="dados.distancia"
                  required
                ></v-text-field>
              </v-col>
              <v-col cols="12" sm="4" md="4" lg="5">
                <v-select
                  filled
                  label="* Autorizado por:"
                  v-model="dados.nome"
                  :items="frota"
                  required
                ></v-select>
              </v-col>
              <v-col cols="12" sm="4" md="4" lg="5">
                <v-select
                  filled
                  label="* Possui liberação de jornada?"
                  v-model="dados.jornada"
                  :items="jornada"
                  required
                ></v-select>
              </v-col>
              <v-col cols="12" md="12" lg="10">
                <v-textarea
                  filled
                  label="Observação"
                  v-model="dados.observacao"
                  value=""
                ></v-textarea>
              </v-col>
              <v-col cols="12" lg="10">
                <v-btn
                  type="submit"
                  color="#2c3e50"
                  class="white--text mr-4"
                  @click="validate"
                >
                  Enviar
                </v-btn>
                <v-btn color="#2c3e50" class="white--text mr-4" @click="reset">
                  Limpar
                </v-btn>
              </v-col>
              <v-col cols="12" lg="10">
                <small class="red--text ">* Campo obrigatório</small>
              </v-col>
            </v-row>
          </v-form>
        </v-row>
      </v-card>
    </v-row>
  </div>
</template>

<script>
import axios from "axios";

import api from "../../services/api";
import { capitalize, capitalizeOne, getRules } from "../../helpers";

export default {
  name: "AppViagem",
  data: () => ({
    valid: true,
    dados: {
      placa: "",
      motorista: "",
      distancia: "",
      nome: "",
      jornada: "",
      observacao: "",
    },
    state: "",
    destinyState: "",
    city: "",
    destinyCity: "",
    states: [],
    originCities: [],
    destinyCities: [],
    rules: getRules(),
    frota: [
      "Alexsandro Velho",
      "Everson Fernandes",
      "Frota Apoio",
      "Frota Controle",
    ],
    jornada: [
      "Liberado do Pernoite",
      "Liberado Velocidade",
      "Liberado Pernoite / Velocidade",
      "Sem Liberação",
    ],
  }),
  async created() {
    try {
      await axios
        .get("https://servicodados.ibge.gov.br/api/v1/localidades/estados")
        .then((res) => {
          const uf = res.data.map((uf) => uf.sigla);
          this.states = uf.sort();
        });
    } catch (error) {
      console.log(error);
      this.$swal({
        icon: "error",
        text: "Erro no servidor, verifique a conexão!",
        showConfirmButton: false,
        timer: 1800,
      });
    }
  },

  methods: {
    async loadOriginCity(event) {
      let state = event;
      await axios
        .get(
          `https://servicodados.ibge.gov.br/api/v1/localidades/estados/${state}/municipios`
        )
        .then((res) => {
          const cityName = res.data.map((city) => city.nome);
          this.originCities = cityName;
          this.destinyCities = cityName;
        });
    },
    async loadDestinyCity(event) {
      let state = event;
      await axios
        .get(
          `https://servicodados.ibge.gov.br/api/v1/localidades/estados/${state}/municipios`
        )
        .then((res) => {
          const cityName = res.data.map((city) => city.nome);
          this.destinyCities = cityName;
        });
    },
    validate() {
      this.$refs.form.validate();
    },
    reset() {
      this.$refs.form.reset();
    },
    async save() {
      try {
        const viagem = {
          placa: this.dados.placa
            ? this.dados.placa.toUpperCase()
            : this.dados.placa,
          motorista: this.dados.motorista
            ? capitalize(this.dados.motorista)
            : this.dados.motorista,
          estadoOrigem: this.state,
          cidadeOrigem: this.city,
          estadoDestino: this.destinyState,
          cidadeDestino: this.destinyCity,
          distancia: this.dados.distancia,
          nome: this.dados.nome,
          jornada: this.dados.jornada,
          observacao: this.dados.observacao
            ? capitalizeOne(this.dados.observacao.trim())
            : this.dados.observacao,
          status: "Aberto",
        };
        await api().post("/viagem-vazio", viagem);
        this.$swal({
          icon: "success",
          text: "Informação registrada com sucesso!",
          showConfirmButton: false,
          timer: 1800,
        });
        this.reset();
      } catch (error) {
        this.$swal({
          icon: "error",
          text: error,
          showConfirmButton: false,
          timer: 1800,
        });
      }
    },
  },
};
</script>

<style scoped>
.viagem-card {
  padding: 15px;
}

.description {
  color: #fff;
  font-size: 1rem;
  margin-top: 10px;
  margin-bottom: 0;
}

.-item {
  margin: 0;
}
</style>