<template>
  <q-layout
    view="lHh Lpr lFf"
    :class="$q.dark.isActive
      ? 'bg-grey-10 text-white'
      : 'bg-grey-1 text-grey-9'"
  >

    <!-- ENCABEZADO -->
    <q-header
      elevated
      :class="$q.dark.isActive
        ? 'bg-grey-9 text-white'
        : 'bg-red-9 text-white'"
    >
      <q-toolbar class="q-py-sm">

        <q-btn
          :color="$q.dark.isActive ? 'grey-8' : 'white'"
          :text-color="$q.dark.isActive ? 'white' : 'red-9'"
          unelevated
          icon="add_circle"
          label="Nuevo equipo"
          class="text-weight-bold q-mr-md"
          @click="nuevoServicio"
        />

        <q-avatar
          color="white"
          text-color="red-9"
          class="shadow-2 q-mr-sm"
        >
          🤖
        </q-avatar>

        <q-toolbar-title class="text-weight-bold text-h6">
          Reparaciones Don Efraín

          <div
            class="text-subtitle2"
            :class="$q.dark.isActive ? 'text-grey-4' : 'text-red-2'"
          >
            Control Pro de Celulares y Tablets
          </div>
        </q-toolbar-title>

        <!-- MODO OSCURO -->
        <q-btn
          flat
          round
          dense
          :icon="$q.dark.isActive ? 'light_mode' : 'dark_mode'"
          class="q-mr-sm"
          @click="$q.dark.toggle()"
        />

      </q-toolbar>
    </q-header>


    <!-- CONTENIDO -->
    <q-page-container>
      <q-page class="q-pa-lg">

        <!-- RESUMEN -->
        <div class="row q-col-gutter-md q-mb-xl">

          <!-- TOTAL -->
          <div class="col-12 col-sm-4">
            <q-card
              class="dashboard-card"
              :class="$q.dark.isActive
                ? 'bg-grey-9 text-white border-left-red'
                : 'bg-white border-left-red'"
            >
              <q-card-section>

                <div class="summary-label">
                  Total Registrados
                </div>

                <div class="summary-number text-red-7">
                  {{ servicios.length }}
                </div>

              </q-card-section>
            </q-card>
          </div>


          <!-- PENDIENTES -->
          <div class="col-12 col-sm-4">
            <q-card
              class="dashboard-card"
              :class="$q.dark.isActive
                ? 'bg-grey-9 text-white border-left-amber'
                : 'bg-white border-left-amber'"
            >
              <q-card-section>

                <div class="summary-label">
                  Pendientes en Taller
                </div>

                <div class="summary-number text-amber-6">
                  {{ contarPendientes() }}
                </div>

              </q-card-section>
            </q-card>
          </div>


          <!-- PAGADOS -->
          <div class="col-12 col-sm-4">
            <q-card
              class="dashboard-card"
              :class="$q.dark.isActive
                ? 'bg-grey-9 text-white border-left-green'
                : 'bg-white border-left-green'"
            >
              <q-card-section>

                <div class="summary-label">
                  Servicios Pagados
                </div>

                <div class="summary-number text-green-5">
                  {{ contarPagados() }}
                </div>

              </q-card-section>
            </q-card>
          </div>

        </div>


        <!-- BUSQUEDA Y FILTROS -->
        <div class="row q-col-gutter-md q-mb-lg items-center">

          <div class="col-12 col-md-6">

            <q-input
              v-model="filtroTexto"
              outlined
              clearable
              placeholder="Buscar por cliente, marca, modelo o IMEI..."
              :dark="$q.dark.isActive"
              input-class="text-body1"
            >

              <template v-slot:prepend>
                <q-icon name="search" />
              </template>

            </q-input>

          </div>


          <div class="col-12 col-md-6 text-right">

            <q-btn-toggle
              v-model="filtroEstado"
              toggle-color="red-9"
              unelevated
              class="border-toggle"
              :options="[
                { label: 'Todos', value: 'todos' },
                { label: 'Pendientes', value: 'Pendiente' },
                { label: 'Abonos', value: 'Abono' },
                { label: 'Pagados', value: 'Pagado' }
              ]"
            />

          </div>

        </div>


        <!-- SIN RESULTADOS -->
        <q-card
          v-if="serviciosFiltrados.length === 0"
          class="q-pa-xl text-center shadow-0 rounded-borders"
          :class="$q.dark.isActive
            ? 'bg-grey-9 text-white'
            : 'bg-white text-grey-9'"
        >

          <q-icon
            name="phonelink_off"
            size="70px"
            color="red-3"
          />

          <div class="text-h6 q-mt-md">
            Sin resultados encontrados
          </div>

          <div class="text-body1 text-grey-5">
            No hay órdenes que coincidan con tu búsqueda o filtros.
          </div>

        </q-card>


        <!-- LISTA DE SERVICIOS -->
        <div
          v-for="servicio in serviciosFiltrados"
          :key="servicio.id"
          class="q-mb-md"
        >

          <q-card
            class="service-card"
            :class="[
              $q.dark.isActive
                ? 'bg-grey-9 text-white'
                : 'bg-white text-grey-9',

              {
                'border-pending':
                  servicio.estadoPago === 'Pendiente',

                'border-abono':
                  servicio.estadoPago === 'Abono',

                'border-paid':
                  servicio.estadoPago === 'Pagado'
              }
            ]"
          >

            <!-- CABECERA -->
            <q-card-section>

              <div class="row items-center justify-between">

                <div class="row items-center">

                  <q-avatar
                    :color="$q.dark.isActive
                      ? 'red-10'
                      : 'red-1'"
                    :text-color="$q.dark.isActive
                      ? 'red-2'
                      : 'red-9'"
                    size="55px"
                  >

                    <q-icon
                      name="smartphone"
                      size="30px"
                    />

                  </q-avatar>


                  <div class="q-ml-md">

                    <div class="text-h6 text-weight-bold">
                      {{ servicio.marca }}
                      {{ servicio.modelo }}
                    </div>

                    <div
                      class="text-body1"
                      :class="$q.dark.isActive
                        ? 'text-grey-4'
                        : 'text-grey-7'"
                    >

                      Cliente:

                      <span class="text-weight-bold">
                        {{ servicio.cliente }}
                      </span>

                      <span
                        v-if="servicio.imei"
                        class="q-ml-sm"
                      >
                        | IMEI: {{ servicio.imei }}
                      </span>

                    </div>

                  </div>

                </div>


                <!-- ESTADO DE PAGO -->
                <q-chip
                  dense
                  :color="
                    servicio.estadoPago === 'Pagado'
                      ? 'green-1'
                      : servicio.estadoPago === 'Abono'
                        ? 'orange-1'
                        : 'red-1'
                  "
                  :text-color="
                    servicio.estadoPago === 'Pagado'
                      ? 'green-9'
                      : servicio.estadoPago === 'Abono'
                        ? 'orange-9'
                        : 'red-9'
                  "
                  class="text-weight-bold text-body1"
                >
                  {{ servicio.estadoPago }}
                </q-chip>

              </div>

            </q-card-section>


            <q-separator inset />


            <!-- DETALLES -->
            <q-card-section>

              <div class="row q-col-gutter-md">

                <!-- REPARACION -->
                <div class="col-12 col-sm-6 col-md-3">

                  <div class="info-label">
                    Reparación
                  </div>

                  <div class="info-value text-weight-medium">

                    <q-icon
                      name="build"
                      color="red-7"
                      size="sm"
                      class="q-mr-xs"
                    />

                    {{ servicio.reparacion }}

                  </div>

                </div>


                <!-- TECNICO -->
                <div class="col-12 col-sm-6 col-md-3">

                  <div class="info-label">
                    Técnico Asignado
                  </div>

                  <div class="info-value text-weight-medium">

                    <q-icon
                      name="badge"
                      color="red-7"
                      size="sm"
                      class="q-mr-xs"
                    />

                    {{ servicio.tecnico }}

                  </div>

                </div>


                <!-- FECHA -->
                <div class="col-12 col-sm-6 col-md-3">

                  <div class="info-label">
                    Fecha de Ingreso
                  </div>

                  <div class="info-value">

                    <q-icon
                      name="event"
                      color="red-7"
                      size="sm"
                      class="q-mr-xs"
                    />

                    {{ formatearFecha(servicio.fecha) }}

                  </div>

                </div>


                <!-- PRECIO -->
                <div class="col-12 col-sm-6 col-md-3">

                  <div class="info-label">
                    Costo Total
                  </div>

                  <div class="info-value text-weight-bold text-red-7">

                    ${{ formatearPrecio(servicio.precio) }}

                  </div>

                </div>

              </div>


              <!-- ESTADO Y ABONO -->
              <div
                class="row q-mt-md items-center justify-between q-pa-sm rounded-borders"
                :class="$q.dark.isActive
                  ? 'bg-grey-8'
                  : 'bg-grey-1'"
              >

                <div class="text-body1">

                  <strong>
                    Estado actual:
                  </strong>

                  <q-chip
                    dense
                    color="blue-1"
                    text-color="blue-9"
                    class="text-weight-bold"
                  >
                    {{ servicio.estadoEquipo }}
                  </q-chip>

                </div>


                <!-- MOSTRAR ABONO -->
                <div
                  v-if="servicio.estadoPago === 'Abono'"
                  class="text-body1"
                >

                  <strong>
                    Abono:
                  </strong>

                  <span class="text-green-6 text-weight-bold">
                    ${{ formatearPrecio(servicio.abono) }}
                  </span>

                </div>

              </div>


              <!-- OBSERVACIONES -->
              <div
                v-if="servicio.observaciones"
                class="q-mt-md text-body1"
              >

                <strong>
                  Notas:
                </strong>

                {{ servicio.observaciones }}

              </div>


              <!-- CALIFICACION -->
              <div
                v-if="servicio.estadoEquipo === 'Entregado'"
                class="q-mt-md"
              >

                <div class="info-label">
                  Calificación del cliente
                </div>


                <!-- YA CALIFICADO -->
                <div v-if="servicio.calificacion > 0">

                  <q-rating
                    :model-value="servicio.calificacion"
                    readonly
                    size="28px"
                    color="orange-8"
                    icon="star_border"
                    icon-selected="star"
                  />

                </div>


                <!-- SIN CALIFICAR -->
                <q-btn
                  v-else
                  flat
                  color="orange-8"
                  icon="star"
                  label="Calificar servicio"
                  @click="abrirCalificacion(servicio)"
                />

              </div>

            </q-card-section>


            <q-separator />


            <!-- ACCIONES -->
            <q-card-actions
              align="right"
              class="q-px-md"
              :class="$q.dark.isActive
                ? 'bg-grey-9'
                : 'bg-grey-1'"
            >

              <!-- ENTREGAR -->
              <q-btn
                v-if="servicio.estadoEquipo !== 'Entregado'"
                flat
                dense
                color="green-7"
                icon="check_circle"
                label="Entregar"
                @click="entregarServicio(servicio)"
              />


              <!-- MODIFICAR -->
              <!--
                IMPORTANTE:
                Ahora también aparece cuando está entregado,
                porque necesitamos poder cambiar el estado de pago.
              -->
              <q-btn
                flat
                dense
                color="red-7"
                icon="edit"
                label="Modificar"
                @click="cargarServicio(servicio)"
              />


              <!-- BORRAR -->
              <q-btn
                v-if="servicio.estadoEquipo !== 'Entregado'"
                flat
                dense
                color="grey-7"
                icon="delete_outline"
                label="Borrar"
                @click="confirmarEliminar(servicio)"
              />

            </q-card-actions>

          </q-card>

        </div>

      </q-page>
    </q-page-container>


    <!-- ============================== -->
    <!-- MODAL NUEVO / EDICION -->
    <!-- ============================== -->

    <q-dialog
      v-model="mostrarModal"
      persistent
    >

      <q-card
        class="form-card"
        :class="$q.dark.isActive
          ? 'bg-grey-9 text-white'
          : 'bg-white text-grey-9'"
      >

        <!-- TITULO -->
        <q-card-section
          class="bg-red-9 text-white row items-center justify-between"
        >

          <div class="text-h6 text-weight-bold">

            {{
              modoEdicion
                ? 'Actualizar Orden'
                : 'Nueva Orden de Servicio'
            }}

          </div>

          <q-btn
            icon="close"
            flat
            round
            dense
            v-close-popup
          />

        </q-card-section>


        <q-form
          @submit.prevent="guardarServicio"
        >

          <q-card-section class="q-gutter-md">

            <!-- CLIENTE -->
            <q-input
              v-model.trim="servicioActual.cliente"
              label="Nombre del Cliente *"
              outlined
              :dark="$q.dark.isActive"
              :rules="[
                val =>
                  !!val ||
                  'El nombre del cliente es obligatorio'
              ]"
              input-class="text-body1"
              :readonly="servicioBloqueado"
            />


            <!-- MARCA Y MODELO -->
            <div class="row q-col-gutter-sm">

              <!-- MARCA -->
              <div class="col-12 col-sm-6">

                <q-select
                  v-model="servicioActual.marca"
                  label="Marca *"
                  outlined
                  :dark="$q.dark.isActive"
                  :options="marcas"
                  :rules="[
                    val =>
                      !!val ||
                      'Seleccione una marca'
                  ]"
                  input-class="text-body1"
                  :readonly="servicioBloqueado"
                />

              </div>


              <!-- MODELO -->
              <div class="col-12 col-sm-6">

                <q-input
                  v-model.trim="servicioActual.modelo"
                  label="Modelo *"
                  outlined
                  :dark="$q.dark.isActive"
                  placeholder="Ej. iPhone 11"
                  :rules="[
                    val =>
                      !!val ||
                      'El modelo es obligatorio'
                  ]"
                  input-class="text-body1"
                  :readonly="servicioBloqueado"
                />

              </div>

            </div>


            <!-- IMEI -->
            <q-input
              v-model.trim="servicioActual.imei"
              label="IMEI / Serial (Opcional)"
              outlined
              :dark="$q.dark.isActive"
              input-class="text-body1"
              :readonly="servicioBloqueado"
            />


            <!-- REPARACION Y TECNICO -->
            <div class="row q-col-gutter-sm">

              <!-- REPARACION -->
              <div class="col-12 col-sm-6">

                <q-select
                  v-model="servicioActual.reparacion"
                  label="Tipo de Servicio *"
                  outlined
                  :dark="$q.dark.isActive"
                  :options="tiposReparacion"
                  :rules="[
                    val =>
                      !!val ||
                      'Seleccione el tipo de servicio'
                  ]"
                  input-class="text-body1"
                  :readonly="servicioBloqueado"
                />

              </div>


              <!-- TECNICO -->
              <div class="col-12 col-sm-6">

                <q-select
                  v-model="servicioActual.tecnico"
                  label="Técnico a cargo *"
                  outlined
                  :dark="$q.dark.isActive"
                  :options="tecnicos"
                  :rules="[
                    val =>
                      !!val ||
                      'Seleccione un técnico'
                  ]"
                  input-class="text-body1"
                  :readonly="servicioBloqueado"
                />

              </div>

            </div>


            <!-- PRECIO, ABONO Y ESTADO -->
            <div class="row q-col-gutter-sm">

              <!-- PRECIO -->
              <div class="col-12 col-sm-4">

                <q-input
                  v-model.number="servicioActual.precio"
                  label="Precio Total *"
                  type="number"
                  prefix="$"
                  outlined
                  :dark="$q.dark.isActive"
                  :rules="[
                    val =>
                      val !== null &&
                      val !== '' &&
                      Number(val) >= 0
                      ||
                      'Ingrese un precio válido'
                  ]"
                  input-class="text-body1"
                  :readonly="servicioBloqueado"
                />

              </div>


              <!-- ABONO -->
              <div class="col-12 col-sm-4">

                <q-input
                  v-if="
                    servicioActual.estadoPago === 'Abono'
                  "
                  v-model.number="servicioActual.abono"
                  label="Valor del Abono *"
                  type="number"
                  prefix="$"
                  outlined
                  :dark="$q.dark.isActive"
                  :rules="[

                    val =>
                      val !== null &&
                      val !== '' &&
                      Number(val) > 0
                      ||
                      'Ingrese el valor del abono',

                    val =>
                      Number(val) <=
                      Number(servicioActual.precio)
                      ||
                      'El abono no puede superar el precio total'

                  ]"
                  input-class="text-body1"
                />


                <q-input
                  v-else
                  :model-value="0"
                  label="Abono"
                  prefix="$"
                  outlined
                  readonly
                  :dark="$q.dark.isActive"
                  input-class="text-body1"
                />

              </div>


              <!-- ESTADO DE PAGO -->
              <div class="col-12 col-sm-4">

                <q-select
                  v-model="servicioActual.estadoPago"
                  label="Estado de Pago *"
                  outlined
                  :dark="$q.dark.isActive"
                  :options="estadosPago"
                  :rules="[
                    val =>
                      !!val ||
                      'Seleccione el estado de pago'
                  ]"
                  input-class="text-body1"
                />

              </div>

            </div>


            <!-- ESTADO DEL EQUIPO -->
            <q-select
              v-model="servicioActual.estadoEquipo"
              label="Estado del Equipo"
              outlined
              readonly
              :dark="$q.dark.isActive"
              :options="estadosEquipo"
              input-class="text-body1"
              hint="El estado del equipo se controla desde el botón Entregar"
            />


            <!-- FECHA AUTOMATICA -->
            <q-input
              v-model="servicioActual.fecha"
              label="Fecha de Entrada"
              outlined
              readonly
              :dark="$q.dark.isActive"
              input-class="text-body1"
              hint="La fecha se genera automáticamente"
            />


            <!-- OBSERVACIONES -->
            <q-input
              v-model.trim="servicioActual.observaciones"
              label="Observaciones y fallas reportadas"
              type="textarea"
              outlined
              :dark="$q.dark.isActive"
              rows="3"
              input-class="text-body1"
              :readonly="servicioBloqueado"
            />

          </q-card-section>


          <!-- AVISO PARA SERVICIO ENTREGADO -->
          <q-card-section
            v-if="servicioBloqueado"
            class="q-px-md q-py-sm"
          >

            <q-banner
              rounded
              class="bg-orange-1 text-orange-10"
            >

              <template v-slot:avatar>
                <q-icon
                  name="info"
                  color="orange-8"
                />
              </template>

              Este equipo ya fue entregado.
              Solo puedes modificar el
              <strong>Estado de Pago</strong>
              y el valor del abono.

            </q-banner>

          </q-card-section>


          <!-- BOTONES -->
          <q-card-actions
            align="right"
            class="q-pa-md"
            :class="$q.dark.isActive
              ? 'bg-grey-10'
              : 'bg-grey-2'"
          >

            <q-btn
              flat
              label="Cancelar"
              color="grey-7"
              v-close-popup
            />

            <q-btn
              type="submit"
              color="red-9"
              :label="
                servicioBloqueado
                  ? 'Actualizar Pago'
                  : 'Guardar Registro'
              "
              unelevated
            />

          </q-card-actions>

        </q-form>

      </q-card>

    </q-dialog>


    <!-- ============================== -->
    <!-- DIALOG ELIMINAR -->
    <!-- ============================== -->

    <q-dialog
      v-model="mostrarConfirmacionEliminar"
    >

      <q-card
        :class="$q.dark.isActive
          ? 'bg-grey-9 text-white'
          : 'bg-white text-grey-9'"
      >

        <q-card-section>

          <div class="text-h6">
            Confirmar eliminación
          </div>

        </q-card-section>


        <q-card-section class="text-body1">

          ¿Está seguro de que desea eliminar este registro?

          <br>

          <strong>
            {{ servicioAEliminar?.marca }}
            {{ servicioAEliminar?.modelo }}
          </strong>

        </q-card-section>


        <q-card-actions align="right">

          <q-btn
            flat
            label="Cancelar"
            color="grey-7"
            v-close-popup
          />

          <q-btn
            unelevated
            label="Eliminar"
            color="red-9"
            @click="eliminarConfirmado"
          />

        </q-card-actions>

      </q-card>

    </q-dialog>


    <!-- ============================== -->
    <!-- DIALOG CALIFICACION -->
    <!-- ============================== -->

    <q-dialog
      v-model="mostrarCalificacion"
    >

      <q-card
        class="rating-card"
        :class="$q.dark.isActive
          ? 'bg-grey-9 text-white'
          : 'bg-white text-grey-9'"
      >

        <q-card-section
          class="bg-orange-8 text-white"
        >

          <div class="text-h6 text-weight-bold">
            Calificar servicio
          </div>

          <div class="text-body1">
            Gracias por utilizar nuestro servicio.
          </div>

        </q-card-section>


        <q-card-section class="text-center">

          <div class="text-body1 q-mb-md">
            ¿Cómo califica la atención recibida?
          </div>


          <q-rating
            v-model="calificacionTemporal"
            size="45px"
            color="orange-8"
            icon="star_border"
            icon-selected="star"
          />


          <div class="text-body1 q-mt-md">
            {{ textoCalificacion }}
          </div>

        </q-card-section>


        <q-card-actions align="right">

          <q-btn
            flat
            label="Cancelar"
            color="grey-7"
            v-close-popup
          />

          <q-btn
            unelevated
            color="orange-8"
            label="Guardar Calificación"
            :disable="calificacionTemporal === 0"
            @click="guardarCalificacion"
          />

        </q-card-actions>

      </q-card>

    </q-dialog>

  </q-layout>
</template>


<script setup>

import { ref, computed } from 'vue'
import { useLocalStorage } from '@vueuse/core'


/* ==========================================
   LOCAL STORAGE
========================================== */

const servicios = useLocalStorage(
  'servicios-tecnicos-don-efrain-red',
  []
)


/* ==========================================
   VARIABLES
========================================== */

const mostrarModal = ref(false)

const modoEdicion = ref(false)

const mostrarConfirmacionEliminar = ref(false)

const servicioAEliminar = ref(null)

const mostrarCalificacion = ref(false)

const servicioACalificar = ref(null)

const calificacionTemporal = ref(0)

const filtroTexto = ref('')

const filtroEstado = ref('todos')


/* ==========================================
   OPCIONES
========================================== */

const marcas = [
  'Apple',
  'Samsung',
  'Motorola',
  'Xiaomi',
  'Huawei',
  'Honor',
  'Oppo',
  'Realme',
  'Tecno',
  'Infinix',
  'Nokia',
  'ZTE',
  'Otra'
]


const tiposReparacion = [
  'Cambio de pantalla',
  'Batería',
  'Pin de carga',
  'Software',
  'Diagnóstico',
  'Otro'
]


const tecnicos = [
  'Don Efraín',
  'Julian',
  'Ana'
]


const estadosPago = [
  'Pendiente',
  'Abono',
  'Pagado'
]


const estadosEquipo = [
  'Recibido',
  'En reparación',
  'Listo para entregar',
  'Entregado'
]


/* ==========================================
   SERVICIO INICIAL
========================================== */

const servicioInicial = () => ({

  id: null,

  cliente: '',

  marca: '',

  modelo: '',

  imei: '',

  reparacion: '',

  tecnico: '',

  fecha: '',

  precio: 0,

  abono: 0,

  estadoPago: 'Pendiente',

  estadoEquipo: 'Recibido',

  calificacion: 0,

  observaciones: ''

})


const servicioActual = ref(
  servicioInicial()
)


/* ==========================================
   SERVICIO BLOQUEADO
========================================== */

/*
 * Si estamos editando un servicio que ya fue
 * entregado, bloqueamos todos los campos excepto
 * Estado de Pago y Abono.
 *
 * Esto permite cobrar posteriormente un equipo
 * que fue entregado con pago pendiente.
 */

const servicioBloqueado = computed(() => {

  return (
    modoEdicion.value &&
    servicioActual.value.estadoEquipo === 'Entregado'
  )

})


/* ==========================================
   FILTROS
========================================== */

const serviciosFiltrados = computed(() => {

  const texto =
    filtroTexto.value
      .trim()
      .toLowerCase()


  return servicios.value.filter(s => {

    const cliente =
      String(s.cliente || '')
        .toLowerCase()

    const marca =
      String(s.marca || '')
        .toLowerCase()

    const modelo =
      String(s.modelo || '')
        .toLowerCase()

    const imei =
      String(s.imei || '')
        .toLowerCase()


    const textoMatch =
      cliente.includes(texto) ||
      marca.includes(texto) ||
      modelo.includes(texto) ||
      imei.includes(texto)


    const estadoMatch =
      filtroEstado.value === 'todos' ||
      s.estadoPago === filtroEstado.value


    return textoMatch && estadoMatch

  })

})


/* ==========================================
   LIMPIAR FORMULARIO
========================================== */

function limpiarFormulario() {

  servicioActual.value =
    servicioInicial()

}


/* ==========================================
   FECHA AUTOMATICA
========================================== */

function obtenerFechaActual() {

  const ahora = new Date()

  const anio =
    ahora.getFullYear()

  const mes =
    String(
      ahora.getMonth() + 1
    ).padStart(2, '0')

  const dia =
    String(
      ahora.getDate()
    ).padStart(2, '0')

  const hora =
    String(
      ahora.getHours()
    ).padStart(2, '0')

  const minutos =
    String(
      ahora.getMinutes()
    ).padStart(2, '0')


  return `${anio}-${mes}-${dia}T${hora}:${minutos}`

}


/* ==========================================
   NUEVO SERVICIO
========================================== */

function nuevoServicio() {

  modoEdicion.value = false

  limpiarFormulario()

  servicioActual.value.fecha =
    obtenerFechaActual()

  servicioActual.value.estadoEquipo =
    'Recibido'

  mostrarModal.value = true

}


/* ==========================================
   GUARDAR
========================================== */

function guardarServicio() {

  /*
   * Cuando el equipo ya fue entregado,
   * solamente permitimos modificar el pago.
   */

  if (servicioBloqueado.value) {

    actualizarPagoServicio()

    mostrarModal.value = false

    return

  }


  /*
   * Validaciones de campos obligatorios
   */

  if (
    !servicioActual.value.cliente ||
    !servicioActual.value.marca ||
    !servicioActual.value.modelo ||
    !servicioActual.value.reparacion ||
    !servicioActual.value.tecnico
  ) {

    return

  }


  /*
   * Validación del precio
   */

  if (
    servicioActual.value.precio === '' ||
    servicioActual.value.precio === null ||
    Number(servicioActual.value.precio) < 0
  ) {

    return

  }


  /*
   * Si el estado es ABONO,
   * debe existir un valor de abono.
   */

  if (
    servicioActual.value.estadoPago === 'Abono'
  ) {

    if (
      !servicioActual.value.abono ||
      Number(servicioActual.value.abono) <= 0
    ) {

      return

    }


    if (
      Number(servicioActual.value.abono) >
      Number(servicioActual.value.precio)
    ) {

      return

    }

  }
  else {

    /*
     * Si no es abono,
     * el valor del abono queda en cero.
     */

    servicioActual.value.abono = 0

  }


  /*
   * Guardar o editar
   */

  if (modoEdicion.value) {

    editarServicio()

  }
  else {

    agregarServicio()

  }


  mostrarModal.value = false

}


/* ==========================================
   ACTUALIZAR SOLO EL PAGO
========================================== */

function actualizarPagoServicio() {

  const index =
    servicios.value.findIndex(
      s =>
        s.id ===
        servicioActual.value.id
    )


  if (index === -1) {

    return

  }


  /*
   * Validar estado de pago
   */

  if (
    !servicioActual.value.estadoPago
  ) {

    return

  }


  /*
   * Si cambia a ABONO,
   * validar el valor.
   */

  if (
    servicioActual.value.estadoPago === 'Abono'
  ) {

    if (
      !servicioActual.value.abono ||
      Number(servicioActual.value.abono) <= 0
    ) {

      return

    }


    if (
      Number(servicioActual.value.abono) >
      Number(servicioActual.value.precio)
    ) {

      return

    }

  }
  else {

    /*
     * Pendiente o Pagado:
     * no deben conservar un abono.
     */

    servicioActual.value.abono = 0

  }


  /*
   * Conservamos TODOS los datos originales
   * y solamente cambiamos pago y abono.
   */

  servicios.value[index] = {

    ...servicios.value[index],

    estadoPago:
      servicioActual.value.estadoPago,

    abono:
      Number(servicioActual.value.abono || 0)

  }

}


/* ==========================================
   AGREGAR SERVICIO
========================================== */

function agregarServicio() {

  /*
   * Se genera el ID ANTES de guardar.
   */

  const nuevoId =
    Date.now()


  servicios.value.push({

    ...servicioActual.value,

    id: nuevoId,

    estadoEquipo: 'Recibido',

    calificacion: 0

  })

}


/* ==========================================
   CARGAR PARA EDITAR
========================================== */

function cargarServicio(servicio) {

  /*
   * Ahora SI permitimos abrir un servicio
   * entregado.
   *
   * Si está entregado, los campos quedan
   * bloqueados excepto el pago.
   */

  modoEdicion.value = true

  servicioActual.value = {
    ...servicio
  }


  mostrarModal.value = true

}


/* ==========================================
   EDITAR
========================================== */

function editarServicio() {

  const index =
    servicios.value.findIndex(
      s =>
        s.id ===
        servicioActual.value.id
    )


  if (index === -1) {

    return

  }


  /*
   * Si está entregado, solamente actualizar
   * estado de pago y abono.
   */

  if (
    servicios.value[index].estadoEquipo ===
    'Entregado'
  ) {

    actualizarPagoServicio()

    return

  }


  /*
   * Servicio todavía no entregado:
   * se puede modificar normalmente.
   */

  servicios.value[index] = {

    ...servicioActual.value

  }

}


/* ==========================================
   CONFIRMAR ELIMINACION
========================================== */

function confirmarEliminar(servicio) {

  /*
   * Los entregados no se pueden eliminar.
   */

  if (
    servicio.estadoEquipo === 'Entregado'
  ) {

    return

  }


  servicioAEliminar.value =
    servicio

  mostrarConfirmacionEliminar.value =
    true

}


/* ==========================================
   ELIMINAR
========================================== */

function eliminarConfirmado() {

  if (
    !servicioAEliminar.value
  ) {

    return

  }


  /*
   * Seguridad adicional
   */

  if (
    servicioAEliminar.value.estadoEquipo ===
    'Entregado'
  ) {

    mostrarConfirmacionEliminar.value =
      false

    servicioAEliminar.value =
      null

    return

  }


  servicios.value =
    servicios.value.filter(
      s =>
        s.id !==
        servicioAEliminar.value.id
    )


  mostrarConfirmacionEliminar.value =
    false

  servicioAEliminar.value =
    null

}


/* ==========================================
   ENTREGAR SERVICIO
========================================== */

function entregarServicio(servicio) {

  const index =
    servicios.value.findIndex(
      s =>
        s.id === servicio.id
    )


  if (index === -1) {

    return

  }


  /*
   * Si ya está entregado,
   * no hacemos nada.
   */

  if (
    servicios.value[index].estadoEquipo ===
    'Entregado'
  ) {

    return

  }


  servicios.value[index] = {

    ...servicios.value[index],

    estadoEquipo: 'Entregado'

  }


  servicio.estadoEquipo =
    'Entregado'

}


/* ==========================================
   CALIFICACION
========================================== */

function abrirCalificacion(servicio) {

  /*
   * Solo se puede calificar
   * cuando el equipo fue entregado.
   */

  if (
    servicio.estadoEquipo !==
    'Entregado'
  ) {

    return

  }


  servicioACalificar.value =
    servicio

  calificacionTemporal.value =
    servicio.calificacion || 0

  mostrarCalificacion.value =
    true

}


/* ==========================================
   TEXTO CALIFICACION
========================================== */

const textoCalificacion =
  computed(() => {

    switch (
      calificacionTemporal.value
    ) {

      case 1:
        return 'Muy malo'

      case 2:
        return 'Malo'

      case 3:
        return 'Regular'

      case 4:
        return 'Bueno'

      case 5:
        return 'Excelente'

      default:
        return 'Seleccione una calificación'

    }

  })


/* ==========================================
   GUARDAR CALIFICACION
========================================== */

function guardarCalificacion() {

  if (
    !servicioACalificar.value
  ) {

    return

  }


  if (
    calificacionTemporal.value === 0
  ) {

    return

  }


  const index =
    servicios.value.findIndex(
      s =>
        s.id ===
        servicioACalificar.value.id
    )


  if (index === -1) {

    return

  }


  /*
   * Solo puede calificarse
   * un servicio entregado.
   */

  if (
    servicios.value[index].estadoEquipo !==
    'Entregado'
  ) {

    return

  }


  servicios.value[index] = {

    ...servicios.value[index],

    calificacion:
      calificacionTemporal.value

  }


  mostrarCalificacion.value =
    false

  servicioACalificar.value =
    null

  calificacionTemporal.value =
    0

}


/* ==========================================
   CONTADORES
========================================== */

function contarPendientes() {

  return servicios.value.filter(
    s =>
      s.estadoEquipo !==
      'Entregado'
  ).length

}


function contarPagados() {

  return servicios.value.filter(
    s =>
      s.estadoPago ===
      'Pagado'
  ).length

}


/* ==========================================
   FORMATEAR PRECIO
========================================== */

function formatearPrecio(precio) {

  if (
    precio === null ||
    precio === undefined ||
    precio === ''
  ) {

    return '0'

  }


  return Number(precio)
    .toLocaleString('es-CO')

}


/* ==========================================
   FORMATEAR FECHA
========================================== */

function formatearFecha(fecha) {

  if (!fecha) {

    return ''

  }


  return String(fecha)
    .replace('T', ' ')

}

</script>


<style>

/* ==========================================
   TARJETAS DE RESUMEN
========================================== */

.dashboard-card {

  border-radius: 12px;

  border-left:
    5px solid #b71c1c;

}


.border-left-red {

  border-left-color:
    #b71c1c !important;

}


.border-left-amber {

  border-left-color:
    #f57c00 !important;

}


.border-left-green {

  border-left-color:
    #388e3c !important;

}


/* ==========================================
   TEXTOS DEL RESUMEN
========================================== */

.summary-label {

  font-size: 16px;

  font-weight: 600;

  color: #757575;

}


.summary-number {

  font-size: 34px;

  font-weight: 800;

  margin-top: 5px;

}


/* ==========================================
   TARJETAS DE SERVICIO
========================================== */

.service-card {

  border-radius: 10px;

  border-left:
    5px solid #9e9e9e;

  box-shadow:
    0 1px 5px
    rgba(0, 0, 0, 0.05);

  transition:
    transform 0.2s;

}


.service-card:hover {

  transform:
    translateY(-2px);

  box-shadow:
    0 4px 12px
    rgba(183, 28, 28, 0.15);

}


/* ==========================================
   COLORES DE ESTADO
========================================== */

.border-pending {

  border-left-color:
    #d32f2f !important;

}


.border-abono {

  border-left-color:
    #f57c00 !important;

}


.border-paid {

  border-left-color:
    #388e3c !important;

}


/* ==========================================
   INFORMACION
========================================== */

.info-label {

  font-size: 13px;

  font-weight: 700;

  color: #757575;

  text-transform:
    uppercase;

  letter-spacing:
    0.5px;

  margin-bottom: 4px;

}


.info-value {

  font-size: 16px;

}


/* ==========================================
   FORMULARIO
========================================== */

.form-card {

  width: 650px;

  max-width: 92vw;

  border-radius: 12px;

}


/* ==========================================
   CALIFICACION
========================================== */

.rating-card {

  width: 450px;

  max-width: 90vw;

  border-radius: 12px;

}


/* ==========================================
   FILTROS
========================================== */

.border-toggle {

  border:
    1px solid #ddd;

  border-radius: 8px;

}


/* ==========================================
   RESPONSIVE
========================================== */

@media (max-width: 600px) {

  .summary-number {

    font-size: 30px;

  }


  .info-value {

    font-size: 16px;

  }

}

</style>
