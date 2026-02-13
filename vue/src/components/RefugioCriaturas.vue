<script setup>
import { ref, computed } from "vue";

const criaturas = ref([]);

const formulario = ref({
  nombre: "",
  edad: null,
  especie: "Yeti",
  peligro: "Bajo",
  cuarentena: false,
  bitacora: "",
});

const indiceEditando = ref(null);

const UMBRAL_RIESGO = 50;

const tituloFormulario = computed(() => {
  if (indiceEditando.value === null) return "Registrando nueva criatura";
  const nombre = formulario.value.nombre?.trim() || "criatura";
  return `Actualizando a ${nombre}`;
});

const nombreValido = computed(() => {
  return (formulario.value.nombre || "").trim().length >= 3;
});

/* Funcion: suma de edades de criaturas con peligro "Alto" */

const indiceRiesgoTotal = computed(() => {
  return criaturas.value.reduce((acc, c) => {
    const esPeligrosa = c.peligro === "Alto";
    const edad = Number.isFinite(c.edad) ? c.edad : 0;
    return esPeligrosa ? acc + edad : acc;
  }, 0);
});

const superaUmbral = computed(() => indiceRiesgoTotal.value > UMBRAL_RIESGO);

function limpiarFormulario() {
  formulario.value = {
    nombre: "",
    edad: null,
    especie: "Yeti",
    peligro: "Bajo",
    cuarentena: false,
    bitacora: "",
  };
  indiceEditando.value = null;
}

function guardarCriatura() {
  const nombre = (formulario.value.nombre || "").trim();

  if (nombre.length < 3) return;

  const payload = {
    id: indiceEditando.value === null ? crypto.randomUUID() : criaturas.value[indiceEditando.value].id,
    nombre,
    edad: Number.isFinite(formulario.value.edad) ? formulario.value.edad : 0,
    especie: formulario.value.especie,
    peligro: formulario.value.peligro,
    cuarentena: !!formulario.value.cuarentena,
    bitacora: formulario.value.bitacora || "",
  };

  if (indiceEditando.value === null) {
    criaturas.value.push(payload);
  } else {
    criaturas.value[indiceEditando.value] = payload;
  }

  limpiarFormulario();
}

function eliminarCriatura(index) {
  criaturas.value.splice(index, 1);

  if (indiceEditando.value === index) limpiarFormulario();

  if (indiceEditando.value !== null && index < indiceEditando.value) {
    indiceEditando.value -= 1;
  }
}

function editarCriatura(index) {
  const c = criaturas.value[index];
  indiceEditando.value = index;

  formulario.value = {
    nombre: c.nombre,
    edad: c.edad,
    especie: c.especie,
    peligro: c.peligro,
    cuarentena: c.cuarentena,
    bitacora: c.bitacora,
  };
}

function reporteCuarentena() {
  const enCuarentena = criaturas.value.filter((c) => c.cuarentena);
  const nombres = enCuarentena.map((c) => c.nombre);

  console.log("Reporte de Cuarentena");
  if (nombres.length === 0) {
    console.log("No hay criaturas en cuarentena.");
    return;
  }
  console.log("Criaturas en cuarentena:", nombres.join(", "));
}
</script>

<template>
  <div class="container py-4">
    <header class="mb-4">
      <h1 class="mb-1">Organización Mundial de Criaturas Extraordinarias</h1>
      <p class="text-muted mb-0">Registro y monitoreo de criaturas rescatadas</p>
    </header>

    <!-- Alertas por condición -->
    <div v-if="superaUmbral" class="alert alert-danger">
      Warning: El <strong>Índice de Riesgo Total</strong> es
      <strong>{{ indiceRiesgoTotal }}</strong>, supera el umbral ({{ UMBRAL_RIESGO }}).
    </div>

    <div class="row g-4">
      <!-- Formulario -->
      <div class="col-12 col-lg-5">
        <div class="card shadow-sm">
          <div class="card-body">
            <h2 class="h5 mb-3">{{ tituloFormulario }}</h2>

            <div class="mb-3">
              <label class="form-label">Nombre de la criatura</label>
              <input
                v-model.trim="formulario.nombre"
                type="text"
                class="form-control"
                placeholder="Ej: Chupacabras"
              />
              <small v-if="!nombreValido" class="text-danger">
                El nombre debe tener al menos 3 caracteres.
              </small>
            </div>

            <div class="mb-3">
              <label class="form-label">Edad estimada</label>
              <input
                v-model.number="formulario.edad"
                type="number"
                min="0"
                class="form-control"
                placeholder="Ej: 12"
              />
            </div>

            <div class="mb-3">
              <label class="form-label">Especie</label>
              <select v-model="formulario.especie" class="form-select">
                <option>Yeti</option>
                <option>Bigfoot</option>
                <option>Chupacabras</option>
                <option>Dragón</option>
              </select>
            </div>

            <div class="mb-3">
              <label class="form-label d-block">Nivel de Peligro</label>

              <div class="form-check form-check-inline">
                <input
                  v-model="formulario.peligro"
                  class="form-check-input"
                  type="radio"
                  value="Bajo"
                  id="peligroBajo"
                />
                <label class="form-check-label" for="peligroBajo">Bajo</label>
              </div>

              <div class="form-check form-check-inline">
                <input
                  v-model="formulario.peligro"
                  class="form-check-input"
                  type="radio"
                  value="Medio"
                  id="peligroMedio"
                />
                <label class="form-check-label" for="peligroMedio">Medio</label>
              </div>

              <div class="form-check form-check-inline">
                <input
                  v-model="formulario.peligro"
                  class="form-check-input"
                  type="radio"
                  value="Alto"
                  id="peligroAlto"
                />
                <label class="form-check-label" for="peligroAlto">Alto</label>
              </div>
            </div>

            <div class="mb-3 form-check">
              <input
                v-model="formulario.cuarentena"
                type="checkbox"
                class="form-check-input"
                id="cuarentena"
              />
              <label class="form-check-label" for="cuarentena">En Cuarentena</label>
            </div>

            <div class="mb-3">
              <label class="form-label">Bitácora de comportamiento</label>
              <textarea
                v-model.lazy="formulario.bitacora"
                class="form-control"
                rows="3"
                placeholder="Notas de comportamiento..."
              ></textarea>
            </div>

            <div class="d-flex gap-2">
              <button
                class="btn btn-primary"
                @click="guardarCriatura"
                :disabled="!nombreValido"
              >
                {{ indiceEditando === null ? "Guardar" : "Actualizar" }}
              </button>

              <button class="btn btn-outline-secondary" @click="limpiarFormulario">
                Limpiar
              </button>

              <button class="btn btn-outline-dark ms-auto" @click="reporteCuarentena">
                Reporte Cuarentena (ver en la consola)
              </button>
            </div>

            <hr class="my-3" />

            <div class="d-flex justify-content-between align-items-center">
              <span class="text-muted">Índice de Riesgo Total</span>
              <strong>{{ indiceRiesgoTotal }}</strong>
            </div>
          </div>
        </div>
      </div>

      <!-- Tabla + estado vacío -->
      <div class="col-12 col-lg-7">
        <div class="card shadow-sm">
          <div class="card-body">
            <h2 class="h5 mb-3">Panel de Control</h2>

            <div v-if="criaturas.length === 0" class="alert alert-info mb-0">
              El refugio está vacío. Registra a tu primera criatura.
            </div>

            <div v-else class="table-responsive">
              <table class="table align-middle">
                <thead>
                  <tr>
                    <th>Nombre</th>
                    <th>Edad</th>
                    <th>Especie</th>
                    <th>Peligro</th>
                    <th>Cuarentena</th>
                    <th class="text-end">Acciones</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(c, index) in criaturas" :key="c.id">
                    <td>
                      <strong>{{ c.nombre }}</strong>
                      <div class="text-muted small" v-if="c.bitacora">
                        {{ c.bitacora }}
                      </div>
                    </td>
                    <td>{{ c.edad }}</td>
                    <td>{{ c.especie }}</td>
                    <td>
                      <span
                        class="badge"
                        :class="{
                          'text-bg-success': c.peligro === 'Bajo',
                          'text-bg-warning': c.peligro === 'Medio',
                          'text-bg-danger': c.peligro === 'Alto',
                        }"
                      >
                        {{ c.peligro }}
                      </span>
                    </td>
                    <td>
                      <span v-if="c.cuarentena" class="badge text-bg-dark">Sí</span>
                      <span v-else class="text-muted">No</span>
                    </td>
                    <td class="text-end">
                      <div class="btn-group">
                        <button class="btn btn-sm btn-outline-primary" @click="editarCriatura(index)">
                          Editar
                        </button>
                        <button class="btn btn-sm btn-outline-danger" @click="eliminarCriatura(index)">
                          Eliminar
                        </button>
                      </div>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>

          </div>
        </div>
      </div>
    </div>
  </div>
</template>
