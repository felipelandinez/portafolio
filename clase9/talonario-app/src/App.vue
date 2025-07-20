<template>
  <div id="app" class="bg-light min-vh-100">
    <header :class="['header-gradient', 'py-4', 'text-center', 'mb-4', 'shadow-sm']" :style="headerGradientStyle">
      <h1 class="mb-0 display-5 fw-bold">Talonario</h1>
    </header>
    <main class="container-fluid">
      <div class="row justify-content-between align-items-start g-4">
        <div class="col-12 col-md-4 d-flex flex-column align-items-center">
          <div v-for="(card, idx) in cards" :key="idx" class="card card-gradient shadow-lg mb-4 w-100 animate__animated animate__fadeIn main-card" style="min-height: 340px; max-width: 500px; min-width: 320px;">
            <div class="card-body">
              <h5 class="card-title mb-3 text-success fw-bold text-center">Datos ingresados</h5>
              <ul class="list-group list-group-flush mb-4">
                <li class="list-group-item py-3"><strong>Premio de loteria</strong> {{ card.dato1 }}</li>
                <li class="list-group-item py-3"><strong>Valor de boleta</strong> {{ card.dato2 }}</li>
                <li class="list-group-item py-3"><strong>Loteria</strong> {{ card.dato3 }}</li>
                <li class="list-group-item py-3"><strong>Fecha de juego</strong> {{ card.dato4 }}</li>
              </ul>
              <div class="d-flex justify-content-center">
                <button :style="buttonMainStyle" class="btn btn-main px-4" @click="editCard(idx)">Editar</button>
              </div>
            </div>
          </div>
        </div>
        <div class="col-12 col-md-4 d-flex flex-column align-items-center justify-content-start">
          <div v-if="boletasConfirmadas === '99' || boletasConfirmadas === '999'" class="d-flex flex-column align-items-center my-4 boletas-buttons-container">
            <div v-for="(fila, rowIdx) in boletasFilas" :key="'row-' + rowIdx" class="d-flex justify-content-center mb-2 gap-2">
              <button
                v-for="num in fila"
                :key="'btn-' + num"
                type="button"
                :class="['btn',
                  getBoletaEstado(num) === 'Pagada' ? 'btn-paid' :
                  getBoletaEstado(num) === 'Apartada' ? 'btn-warning' :
                  getBoletaEstado(num) === 'Ganadora' ? 'btn-winner' :
                  'btn-outline-primary',
                  'boleta-btn', 'flex-grow-1']"
                style="width: 48px;"
                @click="mostrarBoleta(num)"
              >
                {{ num }}
              </button>
            </div>
          </div>
        </div>
        <div class="col-12 col-md-4 d-flex flex-column align-items-center">
          <div class="card card-gradient shadow-lg mb-4 w-100 animate__animated animate__fadeIn side-card" style="min-height: 340px; max-width: 500px; min-width: 320px;">
            <div class="card-body text-center d-flex flex-column justify-content-center h-100">
              <h5 class="card-title mb-4 text-success fw-bold">Acciones</h5>
              <div class="d-flex flex-column gap-4 mt-4">
                <button :style="buttonMainStyle" class="btn btn-main w-100 py-3 fs-5" @click="abrirTablaBoletas">Listar boletas</button>
                <button :style="buttonMainStyle" class="btn btn-main w-100 py-3 fs-5" @click="showPersonalizarModal = true">Personalizar</button>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div v-if="showModal" class="modal fade show d-block" tabindex="-1" style="background:rgba(0,0,0,0.4);">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header modal-header-gradient d-flex justify-content-between align-items-center" :style="headerGradientStyle">
              <h2 class="modal-title fs-5">CONFIGURA TU TALONARIO</h2>
              <button type="button" class="btn-close btn-close-white" aria-label="Cerrar" @click="showModal = false"></button>
            </div>
            <form @submit.prevent="handleSubmit">
              <div class="modal-body">
                <div v-if="errorMessage && errorType === 'modal'" class="alert alert-danger animate__animated animate__fadeIn mb-3 text-center fw-bold" style="font-size:1.1rem;">
                  {{ errorMessage }}
                </div>
                <div class="mb-3">
                  <label for="dato1" class="form-label">Igresa el premio de la rifa</label>
                  <input id="dato1" v-model="dato1" type="text" class="form-control" />
                </div>
                <div class="mb-3">
                  <label for="dato2" class="form-label">Ingresa el valor de la voleta</label>
                  <input id="dato2" v-model="dato2" type="text" class="form-control" />
                </div>
                <div class="mb-3">
                  <label for="dato3" class="form-label">Elija su loteria</label>
                  <select id="dato3" v-model="dato3" class="form-select">
                    <option value="Opción 1">Cruz Roja</option>
                    <option value="Opción 2">La Perla</option>
                    <option value="Opción 3">Santander</option>
                    <option value="Opción 4">Baloto</option>
                  </select>
                </div>
                <div class="mb-3">
                  <label for="dato4" class="form-label">Cantidad de boletas</label>
                  <select id="dato4" v-model="dato4" class="form-select">
                    <option value="99">0/99</option>
                    <option value="999">0/999</option>
                  </select>
                </div>
                <div class="mb-3">
                  <label for="dato5" class="form-label"></label>
                  <input id="dato5" v-model="dato5" type="date" class="form-control" :min="today" />
                </div>
              </div>
              <div class="modal-footer justify-content-center">
                <button :style="buttonMainStyle" class="btn btn-main px-5" type="submit">Confirmar</button>
              </div>
            </form>
          </div>
        </div>
      </div>
      <div v-if="showBoletaModal" class="modal fade show d-block" tabindex="-1" style="background:rgba(0,0,0,0.4);">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header modal-header-gradient d-flex justify-content-between align-items-center" :style="headerGradientStyle">
              <h2 class="modal-title fs-5">Boleta {{ boletaSeleccionada }}</h2>
              <button type="button" class="btn-close btn-close-white" aria-label="Cerrar" @click="showBoletaModal = false"></button>
            </div>
            <div class="modal-body text-center">
              <p class="fs-4">Estado: <span v-if="boletaSeleccionada && getBoletaEstado(boletaSeleccionada) === 'Pagada'" class="badge badge-paid">Pagada</span><span v-else-if="boletaSeleccionada && getBoletaEstado(boletaSeleccionada) === 'Apartada'" class="badge badge-warning">Apartada</span><span v-else-if="boletaSeleccionada && getBoletaEstado(boletaSeleccionada) === 'Ganadora'" class="badge badge-winner">Ganadora</span><span v-else class="badge badge-success">Disponible</span></p>
            </div>
            <div class="modal-footer justify-content-center">
              <button :style="buttonMainStyle" class="btn btn-main px-5" @click="adquirirBoleta">Adquirir boleta</button>
            </div>
          </div>
        </div>
      </div>
      <div v-if="showRegistroModal" class="modal fade show d-block" tabindex="-1" style="background:rgba(0,0,0,0.4);">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header modal-header-gradient d-flex justify-content-between align-items-center" :style="headerGradientStyle">
              <h2 class="modal-title fs-5">Dato de boleta {{ boletaSeleccionada }}</h2>
              <button type="button" class="btn-close btn-close-white" aria-label="Cerrar" @click="showRegistroModal = false"></button>
            </div>
            <form @submit.prevent="registrarBoleta">
              <div class="modal-body">
                <div v-if="errorMessage && errorType === 'registro'" class="alert alert-danger animate__animated animate__fadeIn mb-3 text-center fw-bold" style="font-size:1.1rem;">
                  {{ errorMessage }}
                </div>
                <div class="mb-3">
                  <label class="form-label">Nombre del comprador</label>
                  <input v-model="nombreComprador" type="text" class="form-control" />
                </div>
                <div class="mb-3">
                  <label class="form-label">Dirección del comprador</label>
                  <input v-model="direccionComprador" type="text" class="form-control" />
                </div>
                <div class="mb-3">
                  <label class="form-label">Número telefónico</label>
                  <input v-model="telefonoComprador" type="text" class="form-control" maxlength="10" @input="limitarTelefono($event)" />
                </div>
                <div class="mb-3">
                  <label class="form-label">Estado de la boleta</label>
                  <select v-model="estadoBoleta" class="form-select">
                    <option value="Apartada">Apartada</option>
                    <option value="Pagada">Pagada</option>
                  </select>
                </div>
              </div>
              <div class="modal-footer justify-content-center">
                <button :style="buttonMainStyle" class="btn btn-main px-5" type="submit">Registrar</button>
              </div>
            </form>
          </div>
        </div>
      </div>
      <div v-if="showBoletaRegistradaModal" class="modal fade show d-block" tabindex="-1" style="background:rgba(0,0,0,0.4);">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header modal-header-gradient d-flex justify-content-between align-items-center" :style="headerGradientStyle">
              <h2 class="modal-title fs-5">Boleta {{ boletaSeleccionada }}</h2>
              <button type="button" class="btn-close btn-close-white" aria-label="Cerrar" @click="showBoletaRegistradaModal = false"></button>
            </div>
            <div class="modal-body text-center">
              <p class="fs-4">Estado: <span class="badge bg-success">{{ estadoActualBoleta }}</span></p>
            </div>
            <div class="modal-footer justify-content-center d-flex flex-column gap-3">
              <button :style="buttonMainStyle" class="btn btn-main w-100" @click="verDatosParticipante">Ver datos del participante</button>
              <button :style="buttonMainStyle" class="btn btn-main w-100" @click="liberarBoleta">Liberar boleta</button>
              <button :style="buttonMainStyle" class="btn btn-main w-100" @click="elegirGanador" v-if="estadoActualBoleta === 'Pagada' || estadoActualBoleta === 'Ganadora'">Elegir ganador</button>
              <button v-else-if="estadoActualBoleta === 'Apartada'" :style="buttonMainStyle" class="btn btn-paid w-100" @click="pagarBoleta">Pagar boleta</button>
            </div>
          </div>
        </div>
      </div>
      <div v-if="showDatosParticipanteModal" class="modal fade show d-block" tabindex="-1" style="background:rgba(0,0,0,0.4);">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header modal-header-gradient d-flex justify-content-between align-items-center" :style="headerGradientStyle">
              <h2 class="modal-title fs-5">Datos del participante</h2>
              <button type="button" class="btn-close btn-close-white" aria-label="Cerrar" @click="showDatosParticipanteModal = false"></button>
            </div>
            <div class="modal-body">
              <div v-if="errorMessage && errorType === 'participante'" class="alert alert-danger animate__animated animate__fadeIn mb-3 text-center fw-bold" style="font-size:1.1rem;">
                {{ errorMessage }}
              </div>
              <div v-if="!editandoParticipante">
                <p><strong>Nombre:</strong> {{ datosParticipante.nombre }}</p>
                <p><strong>Dirección:</strong> {{ datosParticipante.direccion }}</p>
                <p><strong>Teléfono:</strong> {{ datosParticipante.telefono }}</p>
                <p><strong>Estado:</strong> {{ datosParticipante.estado }}</p>
              </div>
              <div v-else>
                <div class="mb-3">
                  <label class="form-label">Nombre</label>
                  <input v-model="datosParticipanteEdit.nombre" type="text" class="form-control" />
                </div>
                <div class="mb-3">
                  <label class="form-label">Dirección</label>
                  <input v-model="datosParticipanteEdit.direccion" type="text" class="form-control" />
                </div>
                <div class="mb-3">
                  <label class="form-label">Teléfono</label>
                  <input v-model="datosParticipanteEdit.telefono" type="text" class="form-control" maxlength="10" @input="limitarTelefono($event)" />
                </div>
              </div>
            </div>
            <div class="modal-footer justify-content-center">
              <button :style="buttonMainStyle" class="btn btn-main px-4" @click="editarParticipante" v-if="!editandoParticipante">Editar</button>
              <button :style="buttonMainStyle" class="btn btn-main px-4" @click="guardarEdicionParticipante" v-else>Guardar</button>
            </div>
          </div>
        </div>
      </div>
      <div v-if="showTablaBoletasModal" class="modal fade show d-block" tabindex="-1" style="background:rgba(0,0,0,0.4);">
        <div class="modal-dialog modal-lg">
          <div class="modal-content">
            <div class="modal-header modal-header-gradient d-flex justify-content-between align-items-center" :style="headerGradientStyle">
              <h2 class="modal-title fs-5">Listado de Boletas</h2>
              <button type="button" class="btn-close btn-close-white" aria-label="Cerrar" @click="showTablaBoletasModal = false"></button>
            </div>
            <div class="modal-body">
              <div v-if="registroBoletas.length === 0" class="text-center text-muted">No hay boletas registradas.</div>
              <div v-else class="table-responsive">
                <table class="table table-bordered table-striped align-middle">
                  <thead class="table-success">
                    <tr>
                      <th>Nombre Comprador</th>
                      <th>Dirección</th>
                      <th>Número Telefónico</th>
                      <th>Fecha Compra</th>
                      <th>Estado Boleta</th>
                      <th>N. Boleta</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="b in registroBoletas" :key="b.numero">
                      <td>{{ b.nombre }}</td>
                      <td>{{ b.direccion }}</td>
                      <td>{{ b.telefono }}</td>
                      <td>{{ b.fechaCompra || '-' }}</td>
                      <td>{{ b.estado }}</td>
                      <td>{{ b.numero }}</td>
                    </tr>
                    <tr v-if="registroBoletas.length > 0">
                      <td colspan="3" class="fw-bold text-end">Dinero recaudado (pagadas):</td>
                      <td colspan="3">{{ formatCurrency(totalPagadas) }}</td>
                    </tr>
                    <tr v-if="registroBoletas.length > 0">
                      <td colspan="3" class="fw-bold text-end">Deuda total (apartadas):</td>
                      <td colspan="3">{{ formatCurrency(totalApartadas) }}</td>
                    </tr>
                  </tbody>
                </table>
                <div class="d-flex justify-content-center mt-3">
                  <button :style="buttonMainStyle" class="btn btn-main" @click="generarPDF">Generar PDF</button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Modal de Personalización de Tema -->
      <div v-if="showPersonalizarModal" class="modal fade show d-block" tabindex="-1" style="background:rgba(0,0,0,0.4);">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header modal-header-gradient d-flex justify-content-between align-items-center" :style="headerGradientStyle">
              <h2 class="modal-title fs-5">Temas de personalización</h2>
              <button type="button" class="btn-close btn-close-white" aria-label="Cerrar" @click="showPersonalizarModal = false"></button>
            </div>
            <div class="modal-body">
              <h5 class="mb-3 fw-bold">Colores Header o Encabezados</h5>
              <div class="d-flex flex-column gap-3 mb-4">
                <div v-for="color in headerColors" :key="'header-' + color.name" class="d-flex align-items-center gap-3">
                  <span :style="{background: color.gradient, width: '2rem', height: '2rem', borderRadius: '50%', border: selectedHeaderColor === color.value ? '3px solid #222' : '2px solid #ccc', display: 'inline-block', boxShadow: selectedHeaderColor === color.value ? '0 0 8px ' + color.shadow : 'none', transition: 'all 0.2s'}"></span>
                  <button class="btn btn-outline-dark flex-grow-1 text-start" :class="{'fw-bold': selectedHeaderColor === color.value}" @click="cambiarColorHeader(color.value)">
                    {{ color.label }}
                  </button>
                </div>
              </div>
              <h5 class="mb-3 fw-bold">Colores Footer</h5>
              <div class="d-flex flex-column gap-3 mb-4">
                <div v-for="color in footerColors" :key="'footer-' + color.name" class="d-flex align-items-center gap-3">
                  <span :style="{background: color.gradient, width: '2rem', height: '2rem', borderRadius: '50%', border: selectedFooterColor === color.value ? '3px solid #222' : '2px solid #ccc', display: 'inline-block', boxShadow: selectedFooterColor === color.value ? '0 0 8px ' + color.shadow : 'none', transition: 'all 0.2s'}"></span>
                  <button class="btn btn-outline-dark flex-grow-1 text-start" :class="{'fw-bold': selectedFooterColor === color.value}" @click="cambiarColorFooter(color.value)">
                    {{ color.label }}
                  </button>
                </div>
              </div>
              <h5 class="mb-3 fw-bold">Colores Botones</h5>
              <div class="d-flex flex-column gap-3">
                <div v-for="color in buttonColors" :key="'btn-' + color.name" class="d-flex align-items-center gap-3">
                  <span :style="{background: color.gradient, width: '2rem', height: '2rem', borderRadius: '50%', border: selectedButtonColor === color.value ? '3px solid #222' : '2px solid #ccc', display: 'inline-block', boxShadow: selectedButtonColor === color.value ? '0 0 8px ' + color.shadow : 'none', transition: 'all 0.2s'}"></span>
                  <button class="btn btn-outline-dark flex-grow-1 text-start" :class="{'fw-bold': selectedButtonColor === color.value}" @click="cambiarColorBoton(color.value)">
                    {{ color.label }}
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>
    <footer class="footer-gradient text-center border-top footer-fixed" :style="footerGradientStyle">
      <p class="mb-0">&copy; 2025 Talonario</p>
    </footer>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import jsPDF from 'jspdf';
import autoTable from 'jspdf-autotable';
// Ya no es necesario importar SweetAlert2 en el script de Vue, porque está disponible globalmente por CDN

const showModal = ref(false);
const dato1 = ref('');
const dato2 = ref('');
const dato3 = ref('');
const dato4 = ref('');
const dato5 = ref('');
const cards = ref([]);
const editingIndex = ref(null);
const boletasCantidad = ref('');
const showBoletaModal = ref(false);
const boletaSeleccionada = ref(null);
const showRegistroModal = ref(false);
const nombreComprador = ref('');
const direccionComprador = ref('');
const telefonoComprador = ref('');
const estadoBoleta = ref('Apartada');
const registroBoletas = ref([]);
const showBoletaRegistradaModal = ref(false);
const showDatosParticipanteModal = ref(false);
const estadoActualBoleta = ref('');
const datosParticipante = ref({ nombre: '', direccion: '', telefono: '', estado: '' });
const editandoParticipante = ref(false);
const datosParticipanteEdit = ref({ nombre: '', direccion: '', telefono: '' });
const boletasConfirmadas = ref('');
const showTablaBoletasModal = ref(false);
const showPersonalizarModal = ref(false);
const selectedHeaderColor = ref('verde');
const selectedFooterColor = ref('verde');
const selectedButtonColor = ref('verde');

const headerColors = [
  {
    name: 'amarillo',
    value: 'amarillo',
    label: 'Color Amarillo',
    gradient: 'linear-gradient(90deg, #ffe259 0%, #ffa751 100%)',
    shadow: '#ffe25999'
  },
  {
    name: 'verde',
    value: 'verde',
    label: 'Color Verde',
    gradient: 'linear-gradient(90deg, #43e97b 0%, #38f9d7 100%)',
    shadow: '#43e97b99'
  },
  {
    name: 'rosa',
    value: 'rosa',
    label: 'Color Rosa',
    gradient: 'linear-gradient(90deg, #ff6a88 0%, #ff99ac 100%)',
    shadow: '#ff6a8899'
  },
  {
    name: 'azul',
    value: 'azul',
    label: 'Color Azul',
    gradient: 'linear-gradient(90deg, #2196f3 0%, #0d47a1 100%)',
    shadow: '#2196f399'
  }
];

const footerColors = headerColors;
const buttonColors = headerColors;

const loteriaOptions = {
  'Opción 1': 'Cruz Roja',
  'Oción 2': 'La Perla',
  'Opción 3': 'Santander',
  'Opción 4': 'Baloto'
};

const boletasFilas = computed(() => {
  const total = boletasConfirmadas.value ? parseInt(boletasConfirmadas.value) + 1 : 0;
  const filas = [];
  for (let i = 0; i < total; i += 10) {
    const fila = [];
    for (let j = 0; j < 10 && i + j < total; j++) {
      fila.push(i + j);
    }
    filas.push(fila);
  }
  return filas;
});

onMounted(() => {
  showModal.value = true;
});

// Para la fecha mínima de hoy
const today = ref(new Date().toISOString().split('T')[0]);
const errorMessage = ref('');
const errorType = ref(''); // 'modal', 'registro', 'participante'
let errorTimeout = null;

function mostrarError(msg, tipo = 'modal') {
  window.Swal.fire({
    icon: 'error',
    title: '¡Error!',
    text: msg,
    confirmButtonColor: '#38f9d7',
    background: '#fff',
    customClass: {
      popup: 'animate__animated animate__shakeX',
      confirmButton: 'btn btn-main'
    },
    didOpen: (popup) => {
      popup.style.zIndex = 3000;
    }
  });
}

function mostrarExito(msg) {
  window.Swal.fire({
    icon: 'success',
    title: '¡Éxito!',
    text: msg,
    confirmButtonColor: '#43e97b',
    background: '#fff',
    customClass: {
      popup: 'animate__animated animate__fadeIn',
      confirmButton: 'btn btn-main'
    },
    didOpen: (popup) => {
      popup.style.zIndex = 4000;
    }
  });
}

function handleSubmit() {
  if (!dato1.value || !dato2.value || !dato3.value || !dato4.value || !dato5.value) {
    mostrarError('Todos los campos son obligatorios, por favor completa la información.', 'modal');
    return;
  }
  if (!/^[\d]+$/.test(dato1.value)) {
    mostrarError('El valor del premio debe ser numérico.', 'modal');
    return;
  }
  if (!/^[\d]+$/.test(dato2.value)) {
    mostrarError('El valor de la boleta debe ser numérico.', 'modal');
    return;
  }
  if (parseFloat(dato1.value) < parseFloat(dato2.value)) {
    mostrarError('El premio de la lotería no puede ser menor al valor de la boleta.', 'modal');
    return;
  }
  if (dato5.value < today.value) {
    mostrarError('No se puede elegir una fecha anterior a hoy para jugar la lotería.', 'modal');
    return;
  }
  const cardData = {
    dato1: dato1.value,
    dato2: dato2.value,
    dato3: loteriaOptions[dato3.value] || dato3.value,
    dato4: dato5.value, // Guardar la fecha real en dato4
    dato5: dato4.value  // Guardar la cantidad de boletas en dato5
  };
  if (editingIndex.value !== null) {
    cards.value[editingIndex.value] = cardData;
    editingIndex.value = null;
    mostrarExito('Datos de la lotería actualizados correctamente');
  } else {
    cards.value.push(cardData);
    mostrarExito('Datos de la lotería registrados');
  }
  dato1.value = '';
  dato2.value = '';
  dato3.value = '';
  // dato4 y dato5 no se limpian para mantener la selección de boletas
  boletasConfirmadas.value = dato4.value; // Aquí se confirman las boletas
  showModal.value = false;
}

function editCard(idx) {
  const card = cards.value[idx];
  dato1.value = card.dato1;
  dato2.value = card.dato2;
  dato3.value = card.dato3;
  dato4.value = card.dato4 ? card.dato4.split(',').map(s => s.trim()) : card.dato4;
  dato5.value = card.dato5;
  editingIndex.value = idx;
  showModal.value = true;
}

function mostrarBoleta(num) {
  boletaSeleccionada.value = num;
  // Buscar si la boleta está registrada
  const registro = registroBoletas.value.find(b => b.numero === num);
  if (registro) {
    estadoActualBoleta.value = registro.estado;
    datosParticipante.value = { ...registro };
    showBoletaRegistradaModal.value = true;
  } else {
    showBoletaModal.value = true;
  }
}
function adquirirBoleta() {
  showBoletaModal.value = false;
  showRegistroModal.value = true;
}
function registrarBoleta() {
  if (!nombreComprador.value || !direccionComprador.value || !telefonoComprador.value || !estadoBoleta.value) {
    mostrarError('Todos los campos son obligatorios, por favor completa la información.', 'registro');
    return;
  }
  if (!/^\d+$/.test(telefonoComprador.value)) {
    mostrarError('El número telefónico del comprador debe ser numérico.', 'registro');
    return;
  }
  if (telefonoComprador.value.length !== 10) {
    mostrarError('El número telefónico del comprador debe tener 10 dígitos.', 'registro');
    return;
  }
  registroBoletas.value.push({
    numero: boletaSeleccionada.value,
    nombre: nombreComprador.value,
    direccion: direccionComprador.value,
    telefono: telefonoComprador.value,
    estado: estadoBoleta.value,
    fechaCompra: new Date().toLocaleDateString()
  });
  mostrarExito('Boleta registrada correctamente');
  // Limpiar campos
  nombreComprador.value = '';
  direccionComprador.value = '';
  telefonoComprador.value = '';
  estadoBoleta.value = 'Apartada';
  showRegistroModal.value = false;
}
function verDatosParticipante() {
  showDatosParticipanteModal.value = true;
}
function liberarBoleta() {
  registroBoletas.value = registroBoletas.value.filter(b => b.numero !== boletaSeleccionada.value);
  showBoletaRegistradaModal.value = false;
}
function elegirGanador() {
  // Cambiar el estado de la boleta seleccionada a Ganadora
  const idx = registroBoletas.value.findIndex(b => b.numero === boletaSeleccionada.value);
  if (idx !== -1) {
    registroBoletas.value[idx].estado = 'Ganadora';
    mostrarExito('¡La boleta ' + boletaSeleccionada.value + ' ha sido elegida como ganadora!');
  }
  showBoletaRegistradaModal.value = false;
}
function pagarBoleta() {
  const idx = registroBoletas.value.findIndex(b => b.numero === boletaSeleccionada.value);
  if (idx !== -1) {
    registroBoletas.value[idx].estado = 'Pagada';
    estadoActualBoleta.value = 'Pagada';
    mostrarExito('¡Boleta pagada exitosamente!');
  }
  showBoletaRegistradaModal.value = false;
}
function editarParticipante() {
  editandoParticipante.value = true;
  datosParticipanteEdit.value = {
    nombre: datosParticipante.value.nombre,
    direccion: datosParticipante.value.direccion,
    telefono: datosParticipante.value.telefono
  };
}
function guardarEdicionParticipante() {
  if (!datosParticipanteEdit.value.nombre || !datosParticipanteEdit.value.direccion || !datosParticipanteEdit.value.telefono) {
    mostrarError('Todos los campos son obligatorios, por favor completa la información.', 'participante');
    return;
  }
  if (!/^\d+$/.test(datosParticipanteEdit.value.telefono)) {
    mostrarError('En el campo "Teléfono" solo se aceptan números.', 'participante');
    return;
  }
  // Actualizar en registroBoletas
  const idx = registroBoletas.value.findIndex(b => b.numero === boletaSeleccionada.value);
  if (idx !== -1) {
    registroBoletas.value[idx].nombre = datosParticipanteEdit.value.nombre;
    registroBoletas.value[idx].direccion = datosParticipanteEdit.value.direccion;
    registroBoletas.value[idx].telefono = datosParticipanteEdit.value.telefono;
    datosParticipante.value.nombre = datosParticipanteEdit.value.nombre;
    datosParticipante.value.direccion = datosParticipanteEdit.value.direccion;
    datosParticipante.value.telefono = datosParticipanteEdit.value.telefono;
  }
  editandoParticipante.value = false;
  mostrarExito('Datos del participante actualizados correctamente');
}
function abrirTablaBoletas() {
  showTablaBoletasModal.value = true;
}

function generarPDF() {
  const doc = new jsPDF();
  // Centrar el título y hacerlo más grande
  doc.setFontSize(18);
  const pageWidth = doc.internal.pageSize.getWidth();
  const title = 'Listado de Boletas';
  const textWidth = doc.getTextWidth(title);
  doc.text(title, (pageWidth - textWidth) / 2, 18);
  doc.setFontSize(10);
  const columns = [
    'Nombre Comprador',
    'Dirección',
    'Número Telefónico',
    'Fecha Compra',
    'Estado Boleta',
    'N. Boleta'
  ];
  const rows = registroBoletas.value.map(b => [
    b.nombre,
    b.direccion,
    b.telefono,
    b.fechaCompra || '-',
    b.estado,
    b.numero
  ]);
  // Agregar filas de totales al final de la tabla
  if (registroBoletas.value.length > 0) {
    rows.push([
      { content: 'Dinero recaudado (pagadas):', colSpan: 3, styles: { halign: 'right', fontStyle: 'bold' } },
      { content: formatCurrency(totalPagadas.value), colSpan: 3, styles: { halign: 'left' } }
    ]);
    rows.push([
      { content: 'Deuda total (apartadas):', colSpan: 3, styles: { halign: 'right', fontStyle: 'bold' } },
      { content: formatCurrency(totalApartadas.value), colSpan: 3, styles: { halign: 'left' } }
    ]);
  }
  autoTable(doc, {
    head: [columns],
    body: rows,
    startY: 25,
    theme: 'grid',
    styles: { cellPadding: 2, fontSize: 10 },
    headStyles: { fillColor: [40, 167, 69] },
    didParseCell: function (data) {
      // Ajustar estilos de las filas de totales
      if (data.row.index >= registroBoletas.value.length) {
        data.cell.styles.fontStyle = 'bold';
        data.cell.styles.fillColor = [240, 240, 240];
      }
    }
  });
  doc.save('boletas.pdf');
}

const totalPagadas = computed(() => {
  const valor = parseFloat(cards.value.length > 0 ? cards.value[cards.value.length - 1].dato2 : dato2.value) || 0;
  return registroBoletas.value.filter(b => b.estado === 'Pagada').length * valor;
});
const totalApartadas = computed(() => {
  const valor = parseFloat(cards.value.length > 0 ? cards.value[cards.value.length - 1].dato2 : dato2.value) || 0;
  return registroBoletas.value.filter(b => b.estado === 'Apartada').length * valor;
});

function formatCurrency(value) {
  return new Intl.NumberFormat('es-CO', { style: 'currency', currency: 'COP', minimumFractionDigits: 0 }).format(value);
}

function getBoletaEstado(num) {
  const reg = registroBoletas.value.find(b => b.numero === num);
  if (!reg) return '';
  if (reg.estado === 'Ganadora') return 'Ganadora';
  return reg.estado;
}

function cambiarColorHeader(color) {
  selectedHeaderColor.value = color;
  // Aplicar el color seleccionado a los encabezados
  const headerElements = document.querySelectorAll('.header-gradient, .modal-header-gradient');
  headerElements.forEach(el => {
    el.style.background = headerColors.find(c => c.value === color).gradient;
  });
}

function cambiarColorFooter(color) {
  selectedFooterColor.value = color;
}

function cambiarColorBoton(color) {
  selectedButtonColor.value = color;
}

const headerGradientStyle = computed(() => {
  switch (selectedHeaderColor.value) {
    case 'amarillo':
      return { background: 'linear-gradient(90deg, #ffe259 0%, #ffa751 100%)', color: '#333' };
    case 'verde':
      return { background: 'linear-gradient(90deg, #43e97b 0%, #38f9d7 100%)', color: '#fff' };
    case 'rosa':
      return { background: 'linear-gradient(90deg, #ff6a88 0%, #ff99ac 100%)', color: '#fff' };
    case 'azul':
      return { background: 'linear-gradient(90deg, #2196f3 0%, #0d47a1 100%)', color: '#fff' };
    default:
      return { background: 'linear-gradient(90deg, #43e97b 0%, #38f9d7 100%)', color: '#fff' };
  }
});

const footerGradientStyle = computed(() => {
  switch (selectedFooterColor.value) {
    case 'amarillo':
      return { background: 'linear-gradient(90deg, #ffe259 0%, #ffa751 100%)', color: '#333', borderTop: '1px solid #ffa751' };
    case 'verde':
      return { background: 'linear-gradient(90deg, #43e97b 0%, #38f9d7 100%)', color: '#fff', borderTop: '1px solid #38f9d7' };
    case 'rosa':
      return { background: 'linear-gradient(90deg, #ff6a88 0%, #ff99ac 100%)', color: '#fff', borderTop: '1px solid #ff99ac' };
    case 'azul':
      return { background: 'linear-gradient(90deg, #2196f3 0%, #0d47a1 100%)', color: '#fff', borderTop: '1px solid #2196f3' };
    default:
      return { background: 'linear-gradient(90deg, #43e97b 0%, #38f9d7 100%)', color: '#fff', borderTop: '1px solid #38f9d7' };
  }
});

const buttonMainStyle = computed(() => {
  switch (selectedButtonColor.value) {
    case 'amarillo':
      return { background: 'linear-gradient(90deg, #ffe259 0%, #ffa751 100%)', color: '#333', border: '2px solid #ffa751', boxShadow: '0 0 8px #ffe25999' };
    case 'verde':
      return { background: 'linear-gradient(90deg, #43e97b 0%, #38f9d7 100%)', color: '#fff', border: '2px solid #38f9d7', boxShadow: '0 0 8px #38f9d799' };
    case 'rosa':
      return { background: 'linear-gradient(90deg, #ff6a88 0%, #ff99ac 100%)', color: '#fff', border: '2px solid #ff99ac', boxShadow: '0 0 8px #ff6a8899' };
    case 'azul':
      return { background: 'linear-gradient(90deg, #2196f3 0%, #0d47a1 100%)', color: '#fff', border: '2px solid #2196f3', boxShadow: '0 0 8px #2196f399' };
    default:
      return { background: 'linear-gradient(90deg, #43e97b 0%, #38f9d7 100%)', color: '#fff', border: '2px solid #38f9d7', boxShadow: '0 0 8px #38f9d799' };
  }
});

function limitarTelefono(event) {
  if (event.target.value.length > 10) {
    event.target.value = event.target.value.slice(0, 10);
  }
}
</script>

<style scoped>
body {
  background-color: #f8f9fa;
}
.modal {
  z-index: 2000;
}
.boletas-buttons-container {
  min-width: 320px;
  max-width: 100%;
}
.boleta-btn {
  min-width: 48px;
  flex-basis: 60px;
  margin-bottom: 0.5rem;
}
.btn-winner {
  background: linear-gradient(90deg, #ffd700 0%, #ffb300 100%) !important;
  color: #333 !important;
  border: 2px solid #ffb300 !important;
  font-weight: bold;
  box-shadow: 0 0 8px #ffd70099;
}
.btn-warning {
  background: linear-gradient(90deg, #ff7675 0%, #d63031 100%) !important;
  color: #fff !important;
  border: 2px solid #d63031 !important;
  font-weight: bold;
  box-shadow: 0 0 8px #ff767599;
}
.btn-paid {
  background: linear-gradient(90deg, #2196f3 0%, #0d47a1 100%) !important;
  color: #fff !important;
  border: 2px solid #1565c0 !important;
  font-weight: bold;
  box-shadow: 0 0 8px #2196f399;
}
.btn-main {
  /* El color y fondo ahora se controlan por :style en Vue */
  font-weight: bold;
  box-shadow: 0 0 8px #38f9d799;
}
.btn-secondary-main {
  background: linear-gradient(90deg, #bdbdbd 0%, #757575 100%) !important;
  color: #fff !important;
  border: 2px solid #757575 !important;
  font-weight: bold;
  box-shadow: 0 0 8px #bdbdbd99;
}
.card {
  border-radius: 1rem;
}
.main-card {
  min-height: 340px;
  max-width: 500px;
  min-width: 320px;
}
.side-card {
  border: 2px solid #0d6efd22;
  min-height: 340px;
  max-width: 500px;
  min-width: 320px;
}

.card-gradient {
  background: linear-gradient(90deg, #e0eafc 0%, #cfdef3 100%) !important;
}

.footer-clean {
  background: #fff !important;
  color: #333 !important;
  padding: 8px 0 4px 0 !important;
  margin-top: 0 !important;
  border-top: 1px solid #e0e0e0 !important;
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 100;
}
.footer-gradient {
  /* El color y fondo ahora se controlan por :style en Vue */
  padding: 12px 0 !important;
  margin-top: 1.5rem !important;
  border-top: 1px solid #38f9d7 !important;
  position: static;
}
.footer-fixed {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 100;
  padding: 10px 0 !important;
  margin-top: 0 !important;
}
.badge-success {
  background: linear-gradient(90deg, #43e97b 0%, #38f9d7 100%) !important;
  color: #fff !important;
}
.badge-warning {
  background: linear-gradient(90deg, #ff7675 0%, #d63031 100%) !important;
  color: #fff !important;
}
.badge-paid {
  background: linear-gradient(90deg, #2196f3 0%, #0d47a1 100%) !important;
  color: #fff !important;
}
.badge-winner {
  background: linear-gradient(90deg, #ffd700 0%, #ffb300 100%) !important;
  color: #333 !important;
}
.btn-close, .btn-close-white {
  filter: none !important;
  background: none !important;
  opacity: 1 !important;
  color: #111 !important;
  /* Forzar SVG negro para la X de Bootstrap */
  background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 16 16'%3e%3cpath stroke='black' stroke-linecap='round' stroke-miterlimit='10' stroke-width='2' d='M2 2l12 12M14 2L2 14'/%3e%3c/svg%3e") !important;
  background-size: 0.5em 0.5em;
  background-position: center;
  background-repeat: no-repeat;
  width: 0.9rem !important;
  height: 0.9rem !important;
  font-size: 0.7rem !important;
  min-width: 0.9rem !important;
  min-height: 0.9rem !important;
  padding: 0.05rem !important;
  margin-right: 0.5rem !important;
}
</style>

