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
            :class="$q.dark.isActive
              ? 'text-grey-4'
              : 'text-red-2'"
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

                    {{ servicio.fecha }}

                  </div>

                </div>


                <!-- HORA -->
                <div class="col-12 col-sm-6 col-md-3">

                  <div class="info-label">
                    Hora de Ingreso
                  </div>

                  <div class="info-value">

                    <q-icon
                      name="schedule"
                      color="red-7"
                      size="sm"
                      class="q-mr-xs"
                    />

                    {{ servicio.hora }}

                  </div>

                </div>


                <!-- PRECIO -->
                <div class="col-12 col-sm-6 col-md-3">

                  <div class="info-label">
                    Costo Total
                  </div>

                  <div class="info-value text-weight-bold text-red-7">

                    $ {{ formatearPrecio(servicio.precio) }} COP

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
                    :color="
                      servicio.estadoEquipo === 'Entregado'
                        ? 'green-1'
                        : 'blue-1'
                    "
                    :text-color="
                      servicio.estadoEquipo === 'Entregado'
                        ? 'green-9'
                        : 'blue-9'
                    "
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
                    $ {{ formatearPrecio(servicio.abono) }} COP
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
                v-if="servicio.estadoEquipo === 'Recibido'"
                flat
                dense
                color="green-7"
                icon="check_circle"
                label="Entregar"
                @click="entregarServicio(servicio)"
              />


              <!-- MODIFICAR -->
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


              <!-- MARCA PERSONALIZADA -->
              <div
                v-if="servicioActual.marca === 'Otra'"
                class="col-12 col-sm-6"
              >

                <q-input
                  v-model.trim="servicioActual.marcaOtra"
                  label="Especifique la marca *"
                  outlined
                  :dark="$q.dark.isActive"
                  placeholder="Ej. Vivo"
                  :rules="[
                    val =>
                      servicioActual.marca !== 'Otra' ||
                      !!val ||
                      'Escriba la marca'
                  ]"
                  input-class="text-body1"
                  :readonly="servicioBloqueado"
                />

              </div>


              <!-- MODELO -->
              <div
                :class="
                  servicioActual.marca === 'Otra'
                    ? 'col-12'
                    : 'col-12 col-sm-6'
                "
              >

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


              <!-- SERVICIO PERSONALIZADO -->
              <div
                v-if="servicioActual.reparacion === 'Otra'"
                class="col-12 col-sm-6"
              >

                <q-input
                  v-model.trim="servicioActual.reparacionOtra"
                  label="Especifique el servicio *"
                  outlined
                  :dark="$q.dark.isActive"
                  placeholder="Ej. Cambio de cámara"
                  :rules="[
                    val =>
                      servicioActual.reparacion !== 'Otra' ||
                      !!val ||
                      'Escriba el tipo de servicio'
                  ]"
                  input-class="text-body1"
                  :readonly="servicioBloqueado"
                />

              </div>


              <!-- TECNICO -->
              <div
                :class="
                  servicioActual.reparacion === 'Otra'
                    ? 'col-12'
                    : 'col-12 col-sm-6'
                "
              >

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


            <!-- ========================================== -->
            <!-- PRECIO, ABONO Y ESTADO -->
            <!-- ========================================== -->

            <div class="row q-col-gutter-sm">

              <!-- PRECIO TOTAL -->
              <div class="col-12 col-sm-4">

                <q-input
                  :model-value="precioFormateado"
                  @update:model-value="actualizarPrecio"
                  label="Precio Total *"
                  prefix="$"
                  suffix="COP"
                  outlined
                  :dark="$q.dark.isActive"
                  placeholder="Ej. 150.000"
                  :rules="[
                    val =>
                      val !== null &&
                      val !== '' &&
                      Number(String(val).replace(/\./g, '')) >= 0 ||
                      'Ingrese un precio válido'
                  ]"
                  input-class="text-body1"
                  :readonly="servicioBloqueado"
                  inputmode="numeric"
                />

              </div>


              <!-- ABONO -->
              <div class="col-12 col-sm-4">

                <q-input
                  v-if="servicioActual.estadoPago === 'Abono'"
                  :model-value="abonoFormateado"
                  @update:model-value="actualizarAbono"
                  label="Valor del Abono *"
                  prefix="$"
                  suffix="COP"
                  outlined
                  :dark="$q.dark.isActive"
                  placeholder="Ej. 50.000"
                  :rules="[

                    val =>
                      val !== null &&
                      val !== '' &&
                      Number(String(val).replace(/\./g, '')) > 0 ||
                      'Ingrese el valor del abono',

                    val =>
                      Number(String(val).replace(/\./g, '')) <=
                      Number(servicioActual.precio) ||
                      'El abono no puede superar el precio total'

                  ]"
                  input-class="text-body1"
                  inputmode="numeric"
                />


                <q-input
                  v-else
                  :model-value="'0'"
                  label="Abono"
                  prefix="$"
                  suffix="COP"
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


            <!-- ESTADO DEL SERVICIO -->
            <q-input
              v-model="servicioActual.estadoEquipo"
              label="Estado del Servicio"
              outlined
              readonly
              disable
              :dark="$q.dark.isActive"
              input-class="text-body1"
              hint="El estado cambia automáticamente"
            />


            <!-- FECHA Y HORA -->
            <div class="row q-col-gutter-sm">

              <!-- FECHA -->
              <div class="col-12 col-sm-6">

                <q-input
                  v-model="servicioActual.fecha"
                  label="Fecha de Entrada"
                  outlined
                  readonly
                  :dark="$q.dark.isActive"
                  input-class="text-body1"
                  hint="Formato: AAAA-MM-DD"
                >

                  <template v-slot:prepend>
                    <q-icon name="event" />
                  </template>

                </q-input>

              </div>


              <!-- HORA -->
              <div class="col-12 col-sm-6">

                <q-input
                  v-model="servicioActual.hora"
                  label="Hora de Entrada"
                  outlined
                  readonly
                  :dark="$q.dark.isActive"
                  input-class="text-body1"
                  hint="Formato: HH:mm"
                >

                  <template v-slot:prepend>
                    <q-icon name="schedule" />
                  </template>

                </q-input>

              </div>

            </div>


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


          <!-- AVISO SERVICIO ENTREGADO -->
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
              Los datos principales del servicio
              permanecen bloqueados.

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
                modoEdicion
                  ? 'Actualizar'
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

import { ref, computed, watch } from 'vue'
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
  'Otra'
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


/* ==========================================
   ESTADOS DEL SERVICIO
========================================== */

const estadosEquipo = [
  'Recibido',
  'Entregado'
]


/* ==========================================
   SERVICIO INICIAL
========================================== */

const servicioInicial = () => ({

  id: null,

  cliente: '',

  marca: '',

  marcaOtra: '',

  modelo: '',

  imei: '',

  reparacion: '',

  reparacionOtra: '',

  tecnico: '',

  fecha: '',

  hora: '',

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

const servicioBloqueado = computed(() => {

  return (
    modoEdicion.value &&
    servicioActual.value.estadoEquipo === 'Entregado'
  )

})


/* ==========================================
   MOSTRAR MARCA PERSONALIZADA
========================================== */

watch(
  () => servicioActual.value.marca,
  (nuevaMarca) => {

    if (nuevaMarca !== 'Otra') {

      servicioActual.value.marcaOtra = ''

    }

  }
)


/* ==========================================
   MOSTRAR SERVICIO PERSONALIZADO
========================================== */

watch(
  () => servicioActual.value.reparacion,
  (nuevoServicio) => {

    if (nuevoServicio !== 'Otra') {

      servicioActual.value.reparacionOtra = ''

    }

  }
)


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


  return `${anio}-${mes}-${dia}`

}


/* ==========================================
   HORA AUTOMATICA
========================================== */

function obtenerHoraActual() {

  const ahora = new Date()

  const horas =
    String(
      ahora.getHours()
    ).padStart(2, '0')

  const minutos =
    String(
      ahora.getMinutes()
    ).padStart(2, '0')


  return `${horas}:${minutos}`

}


/* ==========================================
   NUEVO SERVICIO
========================================== */

function nuevoServicio() {

  modoEdicion.value = false

  limpiarFormulario()

  servicioActual.value.fecha =
    obtenerFechaActual()

  servicioActual.value.hora =
    obtenerHoraActual()

  servicioActual.value.estadoEquipo =
    'Recibido'

  mostrarModal.value = true

}


/* ==========================================
   PRECIO FORMATEADO
========================================== */

const precioFormateado = computed(() => {

  return formatearPrecio(
    servicioActual.value.precio
  )

})


/* ==========================================
   ACTUALIZAR PRECIO
========================================== */

function actualizarPrecio(valor) {

  const limpio =
    String(valor || '')
      .replace(/\./g, '')
      .replace(/\D/g, '')

  servicioActual.value.precio =
    limpio === ''
      ? 0
      : Number(limpio)

}


/* ==========================================
   ABONO FORMATEADO
========================================== */

const abonoFormateado = computed(() => {

  return formatearPrecio(
    servicioActual.value.abono
  )

})


/* ==========================================
   ACTUALIZAR ABONO
========================================== */

function actualizarAbono(valor) {

  const limpio =
    String(valor || '')
      .replace(/\./g, '')
      .replace(/\D/g, '')

  servicioActual.value.abono =
    limpio === ''
      ? 0
      : Number(limpio)

}


/* ==========================================
   GUARDAR
========================================== */

function guardarServicio() {

  if (servicioBloqueado.value) {

    actualizarServicioEntregado()

    mostrarModal.value = false

    return

  }


  /* VALIDAR CAMPOS */

  if (
    !servicioActual.value.cliente ||
    !servicioActual.value.marca ||
    !servicioActual.value.modelo ||
    !servicioActual.value.reparacion ||
    !servicioActual.value.tecnico
  ) {

    return

  }


  /* VALIDAR MARCA PERSONALIZADA */

  if (
    servicioActual.value.marca === 'Otra' &&
    !servicioActual.value.marcaOtra?.trim()
  ) {

    return

  }


  /* VALIDAR SERVICIO PERSONALIZADO */

  if (
    servicioActual.value.reparacion === 'Otra' &&
    !servicioActual.value.reparacionOtra?.trim()
  ) {

    return

  }


  /* CONVERTIR MARCA PERSONALIZADA */

  if (
    servicioActual.value.marca === 'Otra'
  ) {

    servicioActual.value.marca =
      servicioActual.value.marcaOtra.trim()

  }


  /* CONVERTIR SERVICIO PERSONALIZADO */

  if (
    servicioActual.value.reparacion === 'Otra'
  ) {

    servicioActual.value.reparacion =
      servicioActual.value.reparacionOtra.trim()

  }


  /* VALIDAR PRECIO */

  if (
    servicioActual.value.precio === '' ||
    servicioActual.value.precio === null ||
    Number(servicioActual.value.precio) < 0
  ) {

    return

  }


  /* CONVERTIR PRECIO A NUMERO */

  servicioActual.value.precio =
    Number(
      servicioActual.value.precio || 0
    )


  /* SEGURIDAD ESTADO */

  if (!modoEdicion.value) {

    servicioActual.value.estadoEquipo =
      'Recibido'

  }


  /* VALIDAR ESTADO PAGO */

  if (
    !servicioActual.value.estadoPago
  ) {

    return

  }


  /* VALIDAR ABONO */

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


    servicioActual.value.abono =
      Number(servicioActual.value.abono)

  }
  else {

    servicioActual.value.abono = 0

  }


  /* GUARDAR O EDITAR */

  if (modoEdicion.value) {

    editarServicio()

  }
  else {

    agregarServicio()

  }


  mostrarModal.value = false

}


/* ==========================================
   ACTUALIZAR SERVICIO ENTREGADO
========================================== */

function actualizarServicioEntregado() {

  const index =
    servicios.value.findIndex(
      s =>
        s.id ===
        servicioActual.value.id
    )


  if (index === -1) {

    return

  }


  if (
    servicios.value[index].estadoEquipo !==
    'Entregado'
  ) {

    return

  }


  /* VALIDAR ESTADO DE PAGO */

  if (
    !servicioActual.value.estadoPago
  ) {

    return

  }


  /* VALIDAR ABONO */

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

    servicioActual.value.abono = 0

  }


  servicios.value[index] = {

    ...servicios.value[index],

    estadoEquipo: 'Entregado',

    estadoPago:
      servicioActual.value.estadoPago,

    abono:
      Number(
        servicioActual.value.abono || 0
      )

  }

}


/* ==========================================
   AGREGAR SERVICIO
========================================== */

function agregarServicio() {

  const nuevoId =
    Date.now()


  servicios.value.push({

    ...servicioActual.value,

    id: nuevoId,

    precio:
      Number(
        servicioActual.value.precio || 0
      ),

    abono:
      Number(
        servicioActual.value.abono || 0
      ),

    estadoEquipo: 'Recibido',

    calificacion: 0

  })

}


/* ==========================================
   CARGAR PARA EDITAR
========================================== */

function cargarServicio(servicio) {

  modoEdicion.value = true

  servicioActual.value = {

    ...servicio,

    precio:
      Number(
        servicio.precio || 0
      ),

    abono:
      Number(
        servicio.abono || 0
      ),

    marcaOtra:
      servicio.marcaOtra || '',

    reparacionOtra:
      servicio.reparacionOtra || ''

  }


  /* COMPATIBILIDAD CON MARCAS PERSONALIZADAS ANTIGUAS */

  if (
    servicioActual.value.marca &&
    !marcas.includes(servicioActual.value.marca)
  ) {

    servicioActual.value.marcaOtra =
      servicioActual.value.marca

    servicioActual.value.marca =
      'Otra'

  }


  /* COMPATIBILIDAD CON SERVICIOS PERSONALIZADOS ANTIGUOS */

  if (
    servicioActual.value.reparacion &&
    !tiposReparacion.includes(
      servicioActual.value.reparacion
    )
  ) {

    servicioActual.value.reparacionOtra =
      servicioActual.value.reparacion

    servicioActual.value.reparacion =
      'Otra'

  }


  /* COMPATIBILIDAD FECHA */

  if (
    !servicioActual.value.fecha
  ) {

    servicioActual.value.fecha =
      obtenerFechaActual()

  }


  /* COMPATIBILIDAD FECHA ANTIGUA */

  if (
    !servicioActual.value.hora
  ) {

    const fechaAntigua =
      String(
        servicioActual.value.fecha || ''
      )


    if (
      fechaAntigua.includes('T')
    ) {

      const partes =
        fechaAntigua.split('T')


      servicioActual.value.fecha =
        partes[0]


      servicioActual.value.hora =
        partes[1]
          ? partes[1].substring(0, 5)
          : obtenerHoraActual()

    }
    else {

      servicioActual.value.hora =
        obtenerHoraActual()

    }

  }


  /* SEGURIDAD ESTADO */

  if (
    servicioActual.value.estadoEquipo !==
      'Recibido' &&
    servicioActual.value.estadoEquipo !==
      'Entregado'
  ) {

    servicioActual.value.estadoEquipo =
      'Recibido'

  }


  /* COMPATIBILIDAD CALIFICACION */

  if (
    servicioActual.value.calificacion ===
    undefined
  ) {

    servicioActual.value.calificacion = 0

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


  /* ENTREGADO */

  if (
    servicios.value[index].estadoEquipo ===
    'Entregado'
  ) {

    actualizarServicioEntregado()

    return

  }


  /* RECIBIDO */

  servicios.value[index] = {

    ...servicioActual.value,

    precio:
      Number(
        servicioActual.value.precio || 0
      ),

    estadoEquipo:
      'Recibido',

    abono:
      servicioActual.value.estadoPago === 'Abono'
        ? Number(
            servicioActual.value.abono || 0
          )
        : 0

  }

}


/* ==========================================
   CONFIRMAR ELIMINACION
========================================== */

function confirmarEliminar(servicio) {

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


  if (
    servicios.value[index].estadoEquipo !==
    'Recibido'
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
      s.estadoEquipo === 'Recibido'
  ).length

}


function contarPagados() {

  return servicios.value.filter(
    s =>
      s.estadoPago === 'Pagado'
  ).length

}


/* ==========================================
   FORMATEAR PRECIO
   PESOS COLOMBIANOS
========================================== */

function formatearPrecio(precio) {

  if (
    precio === null ||
    precio === undefined ||
    precio === ''
  ) {

    return '0'

  }


  const numero =
    Number(precio)


  if (
    Number.isNaN(numero)
  ) {

    return '0'

  }


  return numero.toLocaleString(
    'es-CO',
    {
      minimumFractionDigits: 0,
      maximumFractionDigits: 0
    }
  )

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
   COLORES DE ESTADO DE PAGO
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
