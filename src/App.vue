<template>
  <q-layout view="lHh Lpr lFf" :class="$q.dark.isActive ? 'bg-grey-10 text-white' : 'bg-grey-1 text-grey-9'">

    <!-- ENCABEZADO -->
    <q-header elevated :class="$q.dark.isActive ? 'bg-grey-9 text-white' : 'bg-red-9 text-white'">
      <q-toolbar class="q-py-sm">

        <!-- BOTÓN NUEVO EQUIPO A LA IZQUIERDA -->
        <q-btn
          :color="$q.dark.isActive ? 'grey-8' : 'white'"
          :text-color="$q.dark.isActive ? 'white' : 'red-9'"
          unelevated
          icon="add_circle"
          label="Nuevo equipo"
          class="text-weight-bold q-mr-md"
          @click="nuevoServicio"
        />

        <q-avatar color="white" text-color="red-9" class="shadow-2 q-mr-sm">
          🤖
        </q-avatar>

        <q-toolbar-title class="text-weight-bold">
          Reparaciones Don Efraín
          <div class="text-caption" :class="$q.dark.isActive ? 'text-grey-4' : 'text-red-2'">
            Control Pro de Celulares y Tablets
          </div>
        </q-toolbar-title>

        <!-- BOTÓN MODO OSCURO -->
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

          <div class="col-12 col-sm-4">
            <q-card class="dashboard-card" :class="$q.dark.isActive ? 'bg-grey-9 text-white border-left-red' : 'bg-white border-left-red'">
              <q-card-section>
                <div class="text-subtitle2" :class="$q.dark.isActive ? 'text-grey-4' : 'text-grey-7'">
                  Total Registrados
                </div>
                <div class="text-h4 text-red-7 text-weight-bolder">
                  {{ servicios.length }}
                </div>
              </q-card-section>
            </q-card>
          </div>

          <div class="col-12 col-sm-4">
            <q-card class="dashboard-card" :class="$q.dark.isActive ? 'bg-grey-9 text-white border-left-amber' : 'bg-white border-left-amber'">
              <q-card-section>
                <div class="text-subtitle2" :class="$q.dark.isActive ? 'text-grey-4' : 'text-grey-7'">
                  Pendientes en Taller
                </div>
                <div class="text-h4 text-amber-6 text-weight-bolder">
                  {{ contarPendientes() }}
                </div>
              </q-card-section>
            </q-card>
          </div>

          <div class="col-12 col-sm-4">
            <q-card class="dashboard-card" :class="$q.dark.isActive ? 'bg-grey-9 text-white border-left-green' : 'bg-white border-left-green'">
              <q-card-section>
                <div class="text-subtitle2" :class="$q.dark.isActive ? 'text-grey-4' : 'text-grey-7'">
                  Servicios Pagados
                </div>
                <div class="text-h4 text-green-5 text-weight-bolder">
                  {{ contarPagados() }}
                </div>
              </q-card-section>
            </q-card>
          </div>

        </div>

        <!-- BARRA DE BÚSQUEDA Y FILTROS -->
        <div class="row q-col-gutter-md q-mb-lg items-center">
          <div class="col-12 col-md-6">
            <q-input
              v-model="filtroTexto"
              outlined
              dense
              clearable
              placeholder="Buscar por cliente, equipo o IMEI..."
              :dark="$q.dark.isActive"
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
              flat
              class="q-pa-xs border-toggle"
              :options="[
                {label: 'Todos', value: 'todos'},
                {label: 'Pendientes', value: 'Pendiente'},
                {label: 'Pagados', value: 'Pagado'}
              ]"
            />
          </div>
        </div>

        <!-- CUANDO NO HAY SERVICIOS -->
        <q-card
          v-if="serviciosFiltrados.length === 0"
          class="q-pa-xl text-center shadow-0 rounded-borders"
          :class="$q.dark.isActive ? 'bg-grey-9 text-white' : 'bg-white text-grey-9'"
        >
          <q-icon
            name="phonelink_off"
            size="70px"
            color="red-3"
          />
          <div class="text-h6 q-mt-md">
            Sin resultados encontrados
          </div>
          <div class="text-grey-5 q-mb-md">
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
              $q.dark.isActive ? 'bg-grey-9 text-white' : 'bg-white text-grey-9',
              {
                'border-pending': servicio.estadoPago === 'Pendiente',
                'border-abono': servicio.estadoPago === 'Abono',
                'border-paid': servicio.estadoPago === 'Pagado'
              }
            ]"
          >
            <q-card-section>
              <div class="row items-center justify-between">
                
                <div class="row items-center">
                  <q-avatar
                    :color="$q.dark.isActive ? 'red-10' : 'red-1'"
                    :text-color="$q.dark.isActive ? 'red-2' : 'red-9'"
                    size="50px"
                  >
                    <q-icon name="smartphone" size="26px" />
                  </q-avatar>
                  <div class="q-ml-md">
                    <div class="text-h6 text-weight-bold">
                      {{ servicio.equipo }}
                    </div>
                    <div :class="$q.dark.isActive ? 'text-grey-4 text-subtitle2' : 'text-grey-7 text-subtitle2'">
                      Cliente: <span class="text-weight-bold">{{ servicio.cliente }}</span> 
                      <span v-if="servicio.imei" :class="$q.dark.isActive ? 'text-grey-5' : 'text-grey-5'"> | IMEI: {{ servicio.imei }}</span>
                    </div>
                  </div>
                </div>

                <!-- ESTADO PAGO BADGE -->
                <div>
                  <q-chip
                    dense
                    :color="servicio.estadoPago === 'Pagado' ? 'green-1' : servicio.estadoPago === 'Abono' ? 'orange-1' : 'red-1'"
                    :text-color="servicio.estadoPago === 'Pagado' ? 'green-9' : servicio.estadoPago === 'Abono' ? 'orange-9' : 'red-9'"
                    class="text-weight-bold q-px-md"
                  >
                    {{ servicio.estadoPago }}
                  </q-chip>
                </div>

              </div>
            </q-card-section>

            <q-separator inset />

            <!-- DETALLES -->
            <q-card-section>
              <div class="row q-col-gutter-md">
                
                <div class="col-12 col-sm-6 col-md-3">
                  <div class="info-label" :class="$q.dark.isActive ? 'text-grey-4' : ''">Reparación</div>
                  <div class="info-value text-weight-medium">
                    <q-icon name="build" color="red-7" size="xs" class="q-mr-xs" />
                    {{ servicio.reparacion }}
                  </div>
                </div>

                <div class="col-12 col-sm-6 col-md-3">
                  <div class="info-label" :class="$q.dark.isActive ? 'text-grey-4' : ''">Técnico Asignado</div>
                  <div class="info-value text-weight-medium">
                    <q-icon name="badge" color="red-7" size="xs" class="q-mr-xs" />
                    {{ servicio.tecnico }}
                  </div>
                </div>

                <div class="col-12 col-sm-6 col-md-3">
                  <div class="info-label" :class="$q.dark.isActive ? 'text-grey-4' : ''">Fecha de Ingreso</div>
                  <div class="info-value">
                    <q-icon name="event" color="red-7" size="xs" class="q-mr-xs" />
                    {{ servicio.fecha.replace('T', ' ') }}
                  </div>
                </div>

                <div class="col-12 col-sm-6 col-md-3">
                  <div class="info-label" :class="$q.dark.isActive ? 'text-grey-4' : ''">Costo Total</div>
                  <div class="info-value text-weight-bold text-red-7">
                    ${{ formatearPrecio(servicio.precio) }}
                  </div>
                </div>

              </div>

              <!-- ESTADO DEL EQUIPO Y OBSERVACIONES -->
              <div class="row q-mt-md items-center justify-between q-pa-sm rounded-borders" :class="$q.dark.isActive ? 'bg-grey-8' : 'bg-grey-1'">
                <div class="text-caption">
                  <strong>Estado actual:</strong> 
                  <span class="text-red-6 text-weight-bold q-ml-xs">{{ servicio.estadoEquipo }}</span>
                </div>
                <div v-if="servicio.abono > 0" class="text-caption">
                  <strong>Abono:</strong> <span class="text-green-5 text-weight-bold">${{ formatearPrecio(servicio.abono) }}</span>
                </div>
              </div>

              <!-- CALIFICACIÓN EN TARJETA -->
              <div v-if="servicio.calificacion > 0" class="q-mt-sm">
                <div class="info-label" :class="$q.dark.isActive ? 'text-grey-4' : ''">Calificación</div>
                <q-rating
                  :model-value="servicio.calificacion"
                  readonly
                  size="20px"
                  color="orange-8"
                  icon="star_border"
                  icon-selected="star"
                />
              </div>

              <div v-if="servicio.observaciones" class="q-mt-sm text-caption text-grey-5">
                <strong>Notas:</strong> {{ servicio.observaciones }}
              </div>
            </q-card-section>

            <q-separator />

            <!-- ACCIONES -->
            <q-card-actions align="right" class="q-px-md" :class="$q.dark.isActive ? 'bg-grey-9' : 'bg-grey-1'">
              <q-btn
                flat
                dense
                color="red-7"
                icon="print"
                label="Imprimir Comprobante"
                @click="imprimirComprobante(servicio)"
              />
              <q-btn
                flat
                dense
                color="red-7"
                icon="edit"
                label="Modificar"
                @click="cargarServicio(servicio)"
              />
              <q-btn
                flat
                dense
                color="grey-6"
                icon="delete_outline"
                label="Borrar"
                @click="eliminarServicio(servicio.id)"
              />
            </q-card-actions>

          </q-card>
        </div>

      </q-page>
    </q-page-container>

    <!-- MODAL DE REGISTRO / EDICIÓN -->
    <q-dialog v-model="mostrarModal" persistent>
      <q-card class="form-card" :class="$q.dark.isActive ? 'bg-grey-9 text-white' : 'bg-white text-grey-9'">

        <q-card-section class="bg-red-9 text-white row items-center justify-between">
          <div class="text-h6 text-weight-bold">
            {{ modoEdicion ? 'Actualizar Orden' : 'Nueva Orden de Servicio' }}
          </div>
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>

        <q-form @submit.prevent="guardarServicio">
          <q-card-section class="q-gutter-md">

            <q-input
              v-model="servicioActual.cliente"
              label="Nombre del Cliente *"
              outlined
              dense
              :dark="$q.dark.isActive"
              :rules="[val => !!val || 'Campo obligatorio']"
            />

            <div class="row q-col-gutter-sm">
              <div class="col-12 col-sm-7">
                <q-input
                  v-model="servicioActual.equipo"
                  label="Equipo (Marca y Modelo) *"
                  outlined
                  dense
                  :dark="$q.dark.isActive"
                  placeholder="Ej. iPhone 11 / Moto G54"
                  :rules="[val => !!val || 'Campo obligatorio']"
                />
              </div>
              <div class="col-12 col-sm-5">
                <q-input
                  v-model="servicioActual.imei"
                  label="IMEI / Serial (Opcional)"
                  outlined
                  dense
                  :dark="$q.dark.isActive"
                />
              </div>
            </div>

            <div class="row q-col-gutter-sm">
              <div class="col-12 col-sm-6">
                <q-select
                  v-model="servicioActual.reparacion"
                  label="Tipo de Servicio *"
                  outlined
                  dense
                  :dark="$q.dark.isActive"
                  :options="['Cambio de pantalla', 'Batería', 'Pin de carga', 'Software', 'Diagnóstico', 'Otro']"
                  :rules="[val => !!val || 'Seleccione una opción']"
                />
              </div>
              <div class="col-12 col-sm-6">
                <q-select
                  v-model="servicioActual.tecnico"
                  label="Técnico a cargo *"
                  outlined
                  dense
                  :dark="$q.dark.isActive"
                  :options="['Don Efraín', 'Julian', 'Ana']"
                  :rules="[val => !!val || 'Seleccione un técnico']"
                />
              </div>
            </div>

            <div class="row q-col-gutter-sm">
              <div class="col-12 col-sm-4">
                <q-input
                  v-model.number="servicioActual.precio"
                  label="Precio Total *"
                  type="number"
                  prefix="$"
                  outlined
                  dense
                  :dark="$q.dark.isActive"
                  :rules="[val => val >= 0 || 'Valor no válido']"
                />
              </div>
              <div class="col-12 col-sm-4">
                <q-input
                  v-model.number="servicioActual.abono"
                  label="Abono inicial"
                  type="number"
                  prefix="$"
                  outlined
                  dense
                  :dark="$q.dark.isActive"
                />
              </div>
              <div class="col-12 col-sm-4">
                <q-select
                  v-model="servicioActual.estadoPago"
                  label="Estado Pago *"
                  outlined
                  dense
                  :dark="$q.dark.isActive"
                  :options="['Pendiente', 'Abono', 'Pagado']"
                />
              </div>
            </div>

            <div class="row q-col-gutter-sm">
              <div class="col-12 col-sm-6">
                <q-select
                  v-model="servicioActual.estadoEquipo"
                  label="Estado del Equipo *"
                  outlined
                  dense
                  :dark="$q.dark.isActive"
                  :options="['Recibido', 'En reparación', 'Listo para entregar', 'Entregado']"
                />
              </div>
              <div class="col-12 col-sm-6">
                <q-input
                  v-model="servicioActual.fecha"
                  label="Fecha de Entrada *"
                  type="datetime-local"
                  outlined
                  dense
                  :dark="$q.dark.isActive"
                />
              </div>
            </div>

            <!-- CALIFICACIÓN -->
            <div class="q-mb-md">
              <div class="text-subtitle2 q-mb-sm">
                Calificación del cliente
              </div>
              <q-rating
                v-model="servicioActual.calificacion"
                size="32px"
                color="orange-8"
                icon="star_border"
                icon-selected="star"
              />
              <div class="text-caption text-grey-5">
                Califica el servicio al finalizar la entrega
              </div>
            </div>

            <!-- OBSERVACIONES -->
            <q-input
              v-model="servicioActual.observaciones"
              label="Observaciones y fallas reportadas"
              type="textarea"
              outlined
              dense
              rows="2"
              :dark="$q.dark.isActive"
            />

          </q-card-section>

          <q-card-actions align="right" class="q-pa-md" :class="$q.dark.isActive ? 'bg-grey-10' : 'bg-grey-2'">
            <q-btn flat label="Cancelar" color="grey-6" v-close-popup />
            <q-btn type="submit" color="red-9" label="Guardar Registro" unelevated />
          </q-card-actions>
        </q-form>

      </q-card>
    </q-dialog>

  </q-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const servicios = useLocalStorage('servicios-tecnicos-don-efrain-red', [])

const mostrarModal = ref(false)
const modoEdicion = ref(false)
const filtroTexto = ref('')
const filtroEstado = ref('todos')

const servicioActual = ref({
  id: null,
  cliente: '',
  equipo: '',
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

const serviciosFiltrados = computed(() => {
  return servicios.value.filter(s => {
    const textoMatch = 
      s.cliente.toLowerCase().includes(filtroTexto.value.toLowerCase()) ||
      s.equipo.toLowerCase().includes(filtroTexto.value.toLowerCase()) ||
      (s.imei && s.imei.toLowerCase().includes(filtroTexto.value.toLowerCase()))
    
    const estadoMatch = filtroEstado.value === 'todos' || s.estadoPago === filtroEstado.value
    
    return textoMatch && estadoMatch
  })
})

function limpiarFormulario() {
  servicioActual.value = {
    id: null,
    cliente: '',
    equipo: '',
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
  }
}

function nuevoServicio() {
  modoEdicion.value = false
  limpiarFormulario()
  const ahora = new Date()
  const anio = ahora.getFullYear()
  const mes = String(ahora.getMonth() + 1).padStart(2, '0')
  const dia = String(ahora.getDate()).padStart(2, '0')
  const hora = String(ahora.getHours()).padStart(2, '0')
  const minutos = String(ahora.getMinutes()).padStart(2, '0')
  servicioActual.value.fecha = `${anio}-${mes}-${dia}T${hora}:${minutos}`
  mostrarModal.value = true
}

function guardarServicio() {
  if (modoEdicion.value) {
    editarServicio()
  } else {
    agregarServicio()
  }
  mostrarModal.value = false
}

function agregarServicio() {
  servicios.value.push({
    id: Date.now(),
    ...servicioActual.value
  })
}

function cargarServicio(servicio) {
  modoEdicion.value = true
  servicioActual.value = { ...servicio }
  mostrarModal.value = true
}

function editarServicio() {
  const index = servicios.value.findIndex(s => s.id === servicioActual.value.id)
  if (index !== -1) {
    servicios.value[index] = { ...servicioActual.value }
  }
}

function eliminarServicio(id) {
  if (window.confirm('¿Desea eliminar permanentemente este registro?')) {
    servicios.value = servicios.value.filter(s => s.id !== id)
  }
}

function imprimirComprobante(s) {
  const ventanaImpresion = window.open('', '_blank')
  ventanaImpresion.document.write(`
    <html>
      <head>
        <title>Comprobante de Servicio - Don Efraín</title>
        <style>
          body { font-family: monospace; padding: 20px; font-size: 14px; color: #000; }
          .center { text-align: center; }
          .bold { font-weight: bold; }
          .line { border-bottom: 1px dashed #000; margin: 10px 0; }
        </style>
      </head>
      <body>
        <div class="center bold">REPARACIONES DON EFRAÍN</div>
        <div class="center">Control Pro de Celulares y Tablets</div>
        <div class="line"></div>
        <div><strong>Orden ID:</strong> ${s.id}</div>
        <div><strong>Fecha:</strong> ${s.fecha.replace('T', ' ')}</div>
        <div><strong>Cliente:</strong> ${s.cliente}</div>
        <div><strong>Equipo:</strong> ${s.equipo}</div>
        <div v-if="${s.imei}"><strong>IMEI:</strong> ${s.imei}</div>
        <div class="line"></div>
        <div><strong>Servicio:</strong> ${s.reparacion}</div>
        <div><strong>Técnico:</strong> ${s.tecnico}</div>
        <div><strong>Costo Total:</strong> $${formatearPrecio(s.precio)}</div>
        <div><strong>Abono:</strong> $${formatearPrecio(s.abono)}</div>
        <div><strong>Estado Pago:</strong> ${s.estadoPago}</div>
        <div><strong>Estado Equipo:</strong> ${s.estadoEquipo}</div>
        <div class="line"></div>
        <div><strong>Notas:</strong> ${s.observaciones || 'Ninguna'}</div>
        <div class="line"></div>
        <div class="center">¡Gracias por confiar en nosotros!</div>
        <script>
          window.print();
          window.close();
        <\/script>
      </body>
    </html>
  `)
  ventanaImpresion.document.close()
}

function contarPendientes() {
  return servicios.value.filter(s => s.estadoEquipo !== 'Entregado').length
}

function contarPagados() {
  return servicios.value.filter(s => s.estadoPago === 'Pagado').length
}

function formatearPrecio(precio) {
  if (!precio) return '0'
  return Number(precio).toLocaleString('es-CO')
}
</script>

<style>
.dashboard-card {
  border-radius: 12px;
  border-left: 5px solid #b71c1c;
}
.border-left-red { border-left-color: #b71c1c !important; }
.border-left-amber { border-left-color: #f57c00 !important; }
.border-left-green { border-left-color: #388e3c !important; }

.service-card {
  border-radius: 10px;
  border-left: 5px solid #9e9e9e;
  box-shadow: 0 1px 5px rgba(0,0,0,0.05);
  transition: transform 0.2s;
}
.service-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(183, 28, 28, 0.15);
}

.border-pending { border-left-color: #d32f2f !important; }
.border-abono { border-left-color: #f57c00 !important; }
.border-paid { border-left-color: #388e3c !important; }

.info-label {
  font-size: 11px;
  color: #757575;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}
.info-value {
  font-size: 14px;
}

.form-card {
  width: 600px;
  max-width: 90vw;
  border-radius: 12px;
}
</style>