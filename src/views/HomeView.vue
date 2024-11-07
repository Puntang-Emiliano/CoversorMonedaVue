<template>
  <v-app>
    <v-main>
      <v-container>
        <!-- Selector de fecha -->
        <v-card class="mt-5 pa-4" outlined>
          <v-row>
            <v-col cols="12">
              <v-date-picker
                v-model="fecha"
                full-width
                locale="es-ar"
                class="mt-4"
                :min="minimo"
                :max="maximo"
                @change="getDolar(fecha)"
              ></v-date-picker>
            </v-col>
          </v-row>
        </v-card>

        <!-- Valor del Dólar -->
        <v-card class="mt-5 pa-4" outlined>
          <v-row>
            <v-col cols="12">
              <v-card color="primary" dark class="text-center pa-4">
                <v-card-text class="display-1">
                  Valor del Dólar: {{ venta }}
                </v-card-text>
                <div class="text-subtitle-1">Fecha: {{ fecha }}</div>
              </v-card>
            </v-col>
          </v-row>
        </v-card>

        <!-- Selección de monedas y conversión -->
        <v-card class="mt-5 pa-4" outlined>
          <v-row>
            <!-- Moneda de origen -->
            <v-col cols="12" md="6">
              <v-card-title class="mb-3">Moneda de Origen</v-card-title>
              <v-select
                v-model="monedaOrigen"
                :items="monedas"
                label="Selecciona la moneda de origen"
                item-text="nombre"
                item-value="moneda"
                @change="actualizarValorOrigen"
                outlined
              ></v-select>
              <v-text-field
                :value="valorVentaOrigen"
                label="Valor de venta de moneda origen"
                readonly
                outlined
                class="mt-2"
              ></v-text-field>
              <v-text-field
                v-model="cantidadOrigen"
                label="Cantidad a convertir"
                type="number"
                outlined
                class="mt-2"
              ></v-text-field>
            </v-col>

            <!-- Moneda de destino -->
            <v-col cols="12" md="6">
              <v-card-title class="mb-3">Moneda de Destino</v-card-title>
              <v-select
                v-model="monedaDestino"
                :items="monedas"
                label="Selecciona la moneda de destino"
                item-text="nombre"
                item-value="moneda"
                @change="actualizarValorDestino"
                outlined
              ></v-select>
              <v-text-field
                :value="valorVentaDestino"
                label="Valor de venta de moneda destino"
                readonly
                outlined
                class="mt-2"
              ></v-text-field>
              <v-text-field
                :value="calcularConversionTotal()"
                label="Resultado de la conversión"
                readonly
                outlined
                class="mt-2"
              ></v-text-field>
            </v-col>
          </v-row>
        </v-card>

        <!-- Tabla con todas las cotizaciones -->
        <v-card class="mt-5" outlined>
          <v-data-table
            :headers="headers"
            :items="cotizaciones"
            item-key="moneda"
            class="elevation-1"
          >
            <template v-slot:top>
              <v-toolbar flat color="primary" dark>
                <v-toolbar-title>Cotizaciones Actuales</v-toolbar-title>
                <v-spacer></v-spacer>
                <v-btn @click="getAllCotizaciones" color="secondary">
                  Actualizar Cotizaciones
                </v-btn>
              </v-toolbar>
            </template>
            <template v-slot:item="{ item }">
              <tr>
                <td>{{ item.nombre }}</td>
                <td>{{ item.casa }}</td>
                <td>{{ item.compra }}</td>
                <td>{{ item.venta }}</td>
                <td>
                  {{
                    new Date(item.fechaActualizacion).toLocaleString("es-AR")
                  }}
                </td>
              </tr>
            </template>
          </v-data-table>
        </v-card>

        <!-- Dialogo de carga -->
        <v-dialog v-model="loading.state" hide-overlay persistent width="300">
          <v-card :color="loading.color" dark>
            <v-card-text class="pt-5">
              {{ loading.title }}
              <v-progress-linear
                indeterminate
                color="white"
                class="mb-0"
              ></v-progress-linear>
            </v-card-text>
          </v-card>
        </v-dialog>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import axios from "axios";
import { mapState, mapMutations } from "vuex";

export default {
  data() {
    return {
      fecha: new Date().toISOString().substr(0, 10),
      minimo: "2010-01-01", // Fecha mínima ajustada
      maximo: new Date().toISOString().substr(0, 10),
      venta: null,
      cotizaciones: [], // Arreglo para almacenar todas las cotizaciones
      monedaOrigen: null,
      monedaDestino: null,
      cantidadOrigen: 1, // Cantidad ingresada para convertir
      valorVentaOrigen: null,
      valorVentaDestino: null,
      headers: [
        { text: "Moneda", align: "start", value: "nombre" },
        { text: "Casa", value: "casa" },
        { text: "Compra", value: "compra" },
        { text: "Venta", value: "venta" },
        { text: "Fecha de Actualización", value: "fechaActualizacion" },
      ],
    };
  },
  computed: {
    ...mapState(["loading"]),
    monedas() {
      return this.cotizaciones.map((c) => ({
        moneda: c.moneda,
        nombre: c.nombre,
      }));
    },
  },
  methods: {
    ...mapMutations(["showLoading", "hideLoading"]),

    async getDolar(dia) {
      let url = "";
      if (dia === new Date().toISOString().substr(0, 10)) {
        url = "https://dolarapi.com/v1/dolares/blue";
      } else {
        dia = dia.split("-").join("/"); // Formatear la fecha para la API
        url = `https://api.argentinadatos.com/v1/cotizaciones/dolares/blue/${dia}`;
      }
      try {
        this.showLoading({ title: "Cargando...", color: "secondary" });
        const response = await axios.get(url);
        this.venta = response.data.venta;
      } catch (error) {
        console.error(error);
      } finally {
        this.hideLoading();
      }
    },

    async getAllCotizaciones() {
      try {
        this.showLoading({
          title: "Cargando cotizaciones...",
          color: "primary",
        });

        const responseCotizaciones = await axios.get(
          "https://dolarapi.com/v1/cotizaciones"
        );
        const responseDolares = await axios.get(
          "https://dolarapi.com/v1/dolares"
        );

        this.cotizaciones = [
          ...responseCotizaciones.data,
          ...responseDolares.data,
        ];
      } catch (error) {
        console.error(error);
      } finally {
        this.hideLoading();
      }
    },

    actualizarValorOrigen() {
      const origen = this.cotizaciones.find(
        (c) => c.moneda === this.monedaOrigen
      );
      this.valorVentaOrigen = origen ? origen.venta : null;
    },

    actualizarValorDestino() {
      const destino = this.cotizaciones.find(
        (c) => c.moneda === this.monedaDestino
      );
      this.valorVentaDestino = destino ? destino.venta : null;
    },

    calcularConversionTotal() {
      if (!this.valorVentaOrigen || !this.valorVentaDestino) return null;
      return (
        (this.cantidadOrigen * this.valorVentaOrigen) /
        this.valorVentaDestino
      ).toFixed(4);
    },
  },
  created() {
    this.getAllCotizaciones(); // Cargar todas las cotizaciones al iniciar
  },
};
</script>
