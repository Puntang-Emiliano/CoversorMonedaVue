<template>
  <div>
    <v-layout :wrap="true">
      <v-flex xs12>
        <v-card>
          <v-date-picker
            v-model="fecha"
            full-width
            locale="es-ar"
            class="mt-4"
            :min="minimo"
            :max="maximo"
            @change="getDolar(fecha)"
          ></v-date-picker>
        </v-card>
        <v-card color="error" dark>
          <v-card-text class="display-1 text-center"
            >Valor del Dolar {{ venta }} -- {{ fecha }}
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>
  </div>
</template>

<script>
import axios from "axios";
import { mapMutations } from "vuex";

export default {
  data() {
    return {
      fecha: new Date().toISOString().substr(0, 10),
      minimo: "2010",
      maximo: new Date().toISOString().substr(0, 10),
      venta: null,
    };
  },
  methods: {
    ...mapMutations(["showLoading", "hideLoading"]),

    async getDolar(dia) {
      let url = "";
      if (dia == new Date().toISOString().substr(0, 10)) {
        url = "https://dolarapi.com/v1/dolares/blue";
      } else {
        dia = dia.split("-").join("/");
        url = `https://api.argentinadatos.com/v1/cotizaciones/dolares/blue/${dia}`;
      }
      try {
        this.showLoading({ title: "Cargando....", color: "secondary" });

        const response = await axios.get(`${url}`);
        this.venta = await response.data.venta;
      } catch (error) {
        console.error(error);
      } finally {
        this.hideLoading();
      }
    },
  },
  created() {
    this.getDolar("2024/09/23");
  },
};
</script>
