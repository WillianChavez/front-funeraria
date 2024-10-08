<template>
  <v-container class="pa-0 pa-sm-2 justify-center d-flex" fill-height fluid>
    <v-flex xs12 sm11 md10 lg9>
      <v-card class="pa-2 pa-sm-4" rounded="lg">
        <v-card-title primary-title class="d-flex justify-space-between blueGrayPrincipal--text">
          {{ !$route.query.id ? 'Crear' : 'Editar' }} Servicio
        </v-card-title>
        <v-card-text>

          <v-text-field v-model="form.nombre" label="Nombre" :error-messages="nombreErrors"
            @blur="$v.form.nombre.$touch()" required></v-text-field>

          <v-text-field v-model="form.precio_base" label="Precio Base ($)" :error-messages="precioBaseErrors"
            @blur="$v.form.precio_base.$touch()" required></v-text-field>

          <v-autocomplete v-model="form.id_tipo_servicio" :items="tipoServicios" item-text="nombre" item-value="id"
            label="Tipo de servicio" :error-messages="tipoServicioErrors" @blur="$v.form.id_tipo_servicio.$touch()"
            required></v-autocomplete>
          <v-text-field v-model="form.costo" label="Costo ($)" :error-messages="costoErrors"
            @blur="$v.form.costo.$touch()" required></v-text-field>

          <v-textarea v-model="form.descripcion" label="Descripción" :error-messages="descripcionErrors"
            @blur="$v.form.descripcion.$touch()" required></v-textarea>

        </v-card-text>

        <v-row>
          <v-col class="m-5" cols="12" md="12">
            <v-btn color="bluePrincipal" class="white--text ma-1" rounded @click="save(true)"
              :small="$vuetify.breakpoint.xs" :loading="loading_navigate">
              <v-icon left>mdi-content-save</v-icon>
              {{ !$route.query.id ? 'Crear' : 'Editar' }}
            </v-btn>
            <v-btn :to="{ name: 'servicios' }" rounded :small="$vuetify.breakpoint.xs">
              <v-icon left>mdi-arrow-left</v-icon>
              Regresar al listado
            </v-btn>
          </v-col>
        </v-row>
      </v-card>
    </v-flex>
  </v-container>
</template>
<script>
import { mapActions } from "vuex";
import AppSearchList from "../../components/AppSearchList";
import { required, numeric } from "vuelidate/lib/validators";

export default {
  name: "ServicioForm",
  components: { AppSearchList },
  data: () => ({
    form: {
      nombre: null,
      precio_base: null,
      id_tipo_servicio: null,
      costo: null,
      descripcion: null,
    },
    editMode: false,
    loading: false,
    loading_navigate: false,
    tipoServicios: [],
    numericRules: [
      v => /^\d+(\.\d{1,2})?$/.test(v) || 'Este campo debe ser numérico',
    ],
  }),

  computed: {
    nombreErrors() {
      const errors = []
      if (!this.$v.form.nombre.$dirty) return errors
      !this.$v.form.nombre.required && errors.push('Nombre es requerido')
      return errors
    },

    precioBaseErrors() {
      const errors = []
      if (!this.$v.form.precio_base.$dirty) return errors
      !this.$v.form.precio_base.required && errors.push('Precio Base es requerido')
      !this.$v.form.precio_base.numeric && errors.push('Precio Base debe ser numérico')
      return errors
    },

    tipoServicioErrors() {
      const errors = []
      if (!this.$v.form.id_tipo_servicio.$dirty) return errors
      !this.$v.form.id_tipo_servicio.required && errors.push('Tipo de servicio es requerido')
      return errors
    },

    costoErrors() {
      const errors = []
      if (!this.$v.form.costo.$dirty) return errors
      !this.$v.form.costo.required && errors.push('Costo es requerido')
      !this.$v.form.costo.numeric && errors.push('Costo debe ser numérico')

      return errors
    },

    descripcionErrors() {
      const errors = []
      if (!this.$v.form.descripcion.$dirty) return errors
      !this.$v.form.descripcion.required && errors.push('Descripción es requerido')
      return errors
    },

  },

  validations: {
    form: {
      nombre: { required },
      precio_base: { required, numeric },
      id_tipo_servicio: { required },
      costo: { required, numeric },
      descripcion: { required },
    },
  },


  methods: {
    ...mapActions("utils", ["getMenu"]),
    formClean() {
      this.$v.$reset()
      this.form = {
        nombre: null,
        precio_base: null,
        id_tipo_servicio: null,
        costo: null,
        descripcion: null,
      }
    },

    async save(redirect) {
      this.$v.$touch()
      if (this.$v.$invalid) return
      this.loading_navigate = true
      try {

        if (this.$route.query.id) {
          await this.services.servicio.updateServicio(this.$route.query.id, this.form)
        } else {
          await this.services.servicio.storeServicio(this.form)
        }

        this.formClean()
        if (redirect) this.$router.push({ name: "servicios" })
      } catch (error) {
        console.log(error)

      } finally {
        this.loading_navigate = false
      }
    },


    async getTiposServicios() {
      this.loading = true;
      const response = await this.services.tiposervicio.getAll()
      this.tipoServicios = response.data;
      this.loading = false;
    },

    async getServicio(id) {
      this.loading = true;
      const response = await this.services.servicio.getServicio(id)
      this.loading = false;
      return response.data
    },
  },

  

  async created() {
    const id = this.$route.query.id
    if (id) {
      this.editMode = true
      const servicio = await this.getServicio(id)
      this.form = {
        nombre: servicio.nombre,
        precio_base: servicio.precio_base,
        id_tipo_servicio: servicio.id_tipo_servicio,
        costo: servicio.costo,
        descripcion: servicio.descripcion,
      }

    } else {
      this.formClean()
      this.editMode = false
      await this.getTiposServicios()
    }
  },
};
</script>
