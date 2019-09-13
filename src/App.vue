<template>
  <div id="app">
    <h1 class="text-center">Operacional {{connectionid}}</h1>
    <table class="table table-bordered">
      <thead>
        <tr>
          <th>Data</th>
          <th>Origem</th>
          <th>Destino</th>
          <th>Veiculo</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(viagem, index) in viagens" :key="index">
          <td>{{viagem.data}}</td>
          <td>
            <input type="text" class="form-control" v-model="viagem.origem" @blur="updateViagens($event, viagem.id)" />
          </td>
          <td>{{viagem.destino}}</td>
          <td>
            <select
              v-model="viagem.veiculo"
              class="form-control"
              @change="updateViagens($event, viagem.id)"
            >
              <option
                v-for="veiculo in veiculos"
                :key="veiculo.id"
                :value="veiculo.id"
              >{{ veiculo.placa }}</option>
            </select>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
<script>
import Viagem from "./components/Viagem.vue";
import io from "socket.io-client";
import axios from "axios";
import qs from "query-string";

export default {
  name: "app",
  components: {
    Viagem,
    axios,
    qs
  },
  data: function() {
    return {
      socket: io("notification-talentuminformatica-com-br.umbler.net"),
      application: "nambei",
      user: 1,
      connectionid: "",
      viagens: [
        {
          id: 1,
          data: "2019-09-17",
          origem: "teste origem",
          destino: "teste destino"
        },
        {
          id: 2,
          data: "2019-09-18",
          origem: "teste origem 2 ",
          destino: "teste destino 2 "
        }
      ],
      veiculos: [
        {
          id: 1,
          placa: "EAN-3464"
        },
        {
          id: 2,
          placa: "EAN-1010"
        },
        {
          id: 3,
          placa: "EAN-1111"
        }
      ]
    };
  },
  mounted() {
    let application = this.application;
    let user = this.user;

    console.log(this.socket.connectionid);

    this.socket.on("message", data => {
      console.log(data.viagens);
      this.viagens = JSON.parse(data.viagens);
    });

    this.socket.on("test", data => {
      alert("teste");
    });

    this.socket.on("connect", function(socket) {
      let reference = { application: application, register: user };
      this.emit("register", reference);
    });
  },
  watch: {
    socket: function() {
      alert("mudou");
    }
  },
  methods: {
    updateViagens: function(event, viagem) {
      var requisicao = {
        application: this.application,
        register: 1,
        alterando: "veiculo",
        me: this.socket.id,
        viagens: JSON.stringify(this.viagens)
      };

      //convertendo requisição para form data
      const formData = new FormData();
      Object.keys(requisicao).forEach(key =>
        formData.append(key, requisicao[key])
      );

      const config = {
        headers: {
          "Content-Type": "application/x-www-form-urlencoded"
        }
      };
      this.$http
        .post("http://notification-talentuminformatica-com-br.umbler.net/send", qs.stringify(requisicao), config)
        .then(response => {
          console.log("ok");
        });
    }
  }
};
</script>

<style>
@import "https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css";
</style>
