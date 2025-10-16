<template>
  <div
    class="container mx-auto mt-2 p-3 my-1 border rounded-0 shadow-sm min-vh-75 bg-light"
  >
    <h3 class="text-center my-2">Gestión de Clientes</h3>
    <!-- Formulario -->
    <form @submit.prevent="guardarCliente" class="mb-4">
      <!-- DNI con validación visual -->
      <div class="mb-3 row align-items-center">
        <!-- Columna DNI -->
        <div class="col-md-4 d-flex align-items-center">
          <label for="dni" class="form-label mb-0 w-25">DNI: </label>
          <div class="flex-grow-1">
            <input
              type="text"
              id="dni"
              v-model="nuevoCliente.dni"
              @blur="validarDni"
              class="form-control w-auto"
              :class="{ 'is-invalid': !dniValido }"
              required
            />
            <div v-if="!dniValido" class="invalid-feedback">
              DNI o NIE inválido.
            </div>
          </div>
        </div>

        <!-- Columna Fecha de Alta a la derecha -->
        <div
          class="col-md-4 ms-auto d-flex align-items-center justify-content-end"
        >
          <label for="fechaAlta" class="form-label me-2 mb-0 text-nowrap"
            >Fecha de Alta:</label
          >
          <input
            type="date"
            id="fechaAlta"
            v-model="nuevoCliente.fechaAlta"
            class="form-control w-auto"
          />
        </div>
      </div>

      <!-- FILA Nombre y Apellidos -->
      <div class="mb-3 row g-3 align-items-center">
        <!-- Nombre -->
        <div class="col-md-5 d-flex align-items-center">
          <label for="nombre" class="form-label mb-0 text-nowrap w-25"
            >Nombre:</label
          >
          <input
            type="text"
            id="nombre"
            v-model="nuevoCliente.nombre"
            class="form-control flex-grow-1"
            @blur="capitalizarTexto('nombre')"
            required
          />
        </div>

        <!-- Apellidos -->
        <div class="col-md-6 d-flex align-items-center ms-5">
          <label for="apellidos" class="form-label me-4 mb-0 text-nowrap"
            >Apellidos:</label
          >
          <input
            type="text"
            id="apellidos"
            v-model="nuevoCliente.apellidos"
            class="form-control flex-grow-1"
            @blur="capitalizarTexto('apellidos')"
            required
          />
        </div>
      </div>

      <!-- FILA Email y Móvil -->
      <div class="mb-3 row g-3 align-items-center">
        <!-- Email -->
        <div class="col-md-5 d-flex align-items-center">
          <label for="email" class="form-label mb-0 text-nowrap w-25"
            >Email:</label
          >
          <input
            type="email"
            id="email"
            v-model="nuevoCliente.email"
            class="form-control flex-grow-1"
            @blur="validarEmail('email')"
            :class="{ 'is-invalid': !emailValido }"
            required
          />
        </div>

        <!-- Móvil -->
        <div class="col-md-3 d-flex align-items-center">
          <label for="movil" class="form-label me-5 ms-5 mb-0 text-nowrap"
            >Móvil:</label
          >
          <input
            type="tel"
            id="movil"
            v-model="nuevoCliente.movil"
            @blur="validarMovil('movil')"
            class="form-control flex-grow-1 text-center"
            :class="{ 'is-invalid': !movilValido }"
          />
        </div>
      </div>

      <!-- FILA Dirección, Provincia y Municipio -->
      <div class="mb-3 row g-3 align-items-center">
        <!-- Dirección -->
        <div class="col-md-5 d-flex align-items-center">
          <label for="direccion" class="form-label mb-0 w-25 text-nowrap"
            >Dirección:</label
          >
          <input
            type="text"
            id="direccion"
            v-model="nuevoCliente.direccion"
            @blur="capitalizarTexto('direccion')"
            class="form-control flex-grow-1"
          />
        </div>

        <!-- Provincia -->
        <div class="col-md-3 d-flex align-items-center">
          <label for="provincia" class="form-label me-4 ms-5 mb-0 text-nowrap"
            >Provincia:</label
          >
          <select
            id="provincia"
            v-model="nuevoCliente.provincia"
            class="form-select flex-grow-1 w-25"
            @change="filtrarMunicipios"
          >
            <option disabled value="">Seleccione provincia</option>
            <option v-for="prov in provincias" :key="prov.id" :value="prov.nm">
              {{ prov.nm }}
            </option>
          </select>
        </div>

        <!-- Municipio -->
        <div class="col-md-3 d-flex align-items-center">
          <label for="municipio" class="form-label me-2 ms-4 mb-0 text-nowrap"
            >Municipio:</label
          >
          <select
            id="municipio"
            v-model="nuevoCliente.municipio"
            class="form-select flex-grow-1 w-auto"
          >
            <option disabled value="">Seleccione municipio</option>
            <option
              v-for="mun in municipiosFiltrados"
              :key="mun.id"
              :value="mun.nm"
            >
              {{ mun.nm }}
            </option>
          </select>
        </div>
      </div>

      <!-- Histórico -->
      <div class="d-flex justify-content-end mb-2">
        <input
          type="checkbox"
          id="historico"
          v-model="mostrarHistorico"
          class="form-check-input"
          @change="cargarClientes"
        />
        <label for="historico" class="form-check-label ms-3 me-5 mb-0"
          >Histórico</label
        >
      </div>

      <!-- Botón centrado -->
      <div class="text-center">
        <button
          type="submit"
          class="btn btn-primary border-0 shadow-none rounded-0"
        >
          Cargar
        </button>
      </div>
    </form>
    <!-- Lista de Clientes -->
    <div class="table-responsive">
      <h4 class="text-center w-100">Listado Clientes</h4>
      <table
        class="table table-bordered table-striped table-hover table-sm w-100 align-middle"
      >
        <thead class="table-primary">
          <tr>
            <th class="text-center">ID</th>
            <th class="text-center">Apellidos</th>
            <th class="text-center">Nombre</th>
            <th class="text-center">Móvil</th>
            <th class="text-center">Municipio</th>
            <th class="text-center">Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(cliente, index) in clientes" :key="index">
            <th scope="row" class="text-center">{{ index + 1 }}</th>
            <td>{{ cliente.apellidos }}</td>
            <td>{{ cliente.nombre }}</td>
            <td class="text-center">{{ cliente.movil }}</td>
            <td class="text-center">{{ cliente.municipio }}</td>
            <td class="align-middle text-center">
              <button
                @click="eliminarCliente(cliente.movil)"
                class="btn btn-danger btn-sm ms-4 border-0 shadow-none rounded-0"
                title="Eliminar cliente"
                aria-label="Eliminar cliente"
              >
                <i class="bi bi-trash"></i>
              </button>
              <button
                @click="editarCliente(cliente.movil)"
                class="btn btn-warning btn-sm border-0 dow-none shadow-none rounded-0"
                title="Editar cliente"
              >
                <i class="bi bi-pencil"></i>
              </button>
              <button
                v-if="cliente.historico === false"
                @click="activarCliente(cliente)"
                class="btn btn-secondary btn-sm ms-2 border-0 shadow-none rounded-0"
                title="Activar cliente"
              >
                <i class="bi bi-person-check"></i>
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import provmuniData from "@/data/provmuni.json";
import Swal from "sweetalert2";
import {
  getClientes,
  addCliente,
  updateCliente,
  deleteCliente,
} from "@/api/clientes.js";

// SCRIPTS CRUD //

const nuevoCliente = ref({
  dni: "",
  nombre: "",
  apellidos: "",
  email: "",
  movil: "",
  direccion: "",
  provincia: "",
  municipio: "",
  fechaAlta: "",
  historico: true, // luego lo cambiamos a false
});

// Funcion lisar clientes con get

const editando = ref(false); // Estado de edición
const clienteEditandoId = ref(null); // ID del cliente que se está editando
const mostrarHistorico = ref(false);
// Cargar clientes al montar el componente

const clientes = ref([]);
// Zona Cargar clientes Al Montar el componente
onMounted(async () => {
  cargarClientes();
});

const cargarClientes = () => {
  getClientes(mostrarHistorico.value).then((data) => {
    clientes.value = data;
  });
  Swal.fire({
    icon: "success",
    title: "Listando Clientes",
    showConfirmButton: false,
    timer: 1500,
  });
};

const guardarCliente = async () => {
  // Validar duplicados solo si estás creando (no si editando)

  if (!editando.value) {
    const duplicado = clientes.value.find(
      (cliente) =>
        cliente.dni === nuevoCliente.value.dni ||
        cliente.movil === nuevoCliente.value.movil ||
        cliente.email === nuevoCliente.value.email
    );
    if (duplicado) {
      Swal.fire({
        icon: "error",
        title: "DNI, móvil o email duplicados",
        showConfirmButton: false,
        timer: 2000,
      });
      return;
    }
  }

  // Confirmación antes de guardar
  const result = await Swal.fire({
    title: editando.value
      ? "¿Desea modificar este cliente?"
      : "¿Desea grabar este cliente?",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: editando.value ? "Modificar" : "Grabar",
    cancelButtonText: "Cancelar",
  });

  if (!result.isConfirmed) return;
  //  cliente.fechaAlta = formatearFechaParaInput(cliente.fechaAlta);
  try {
    if (editando.value) {
      // Validar campos
      // Modificar cliente (PUT)+

      const clienteActualizado = await updateCliente(
        clienteEditandoId.value,
        nuevoCliente.value
      );

      // Actualiza el cliente en la lista local
      const index = clientes.value.findIndex(
        (c) => c.id === clienteEditandoId.value
      );
      if (index !== -1) clientes.value[index] = clienteActualizado;
      Swal.fire({
        icon: "success",
        title: "Cliente modificado",
        showConfirmButton: false,
        timer: 1500,
      });
    } else {
      // Agregar cliente (POST)

      const clienteAgregado = await addCliente(nuevoCliente.value);
      clientes.value.push(clienteAgregado);
      Swal.fire({
        icon: "success",
        title: "Cliente agregado",
        showConfirmButton: false,
        timer: 1500,
      });
    }

    // Reset formulario y estado
    nuevoCliente.value = {
      dni: "",
      nombre: "",
      apellidos: "",
      email: "",
      movil: "",
      direccion: "",
      provincia: "",
      municipio: "",
      fechaAlta: "",
      historico: true,
    };
    editando.value = true;
    clienteEditandoId.value = null;

    // Reset validaciones si tienes (dniValido, movilValido, etc)
    dniValido.value = true;
    movilValido.value = true;
    emailValido.value = true;

    // Refrescar lista completa (opcional)
    clientes.value = await getClientes();
  } catch (error) {
    console.error("Error al guardar cliente:", error);
    Swal.fire({
      icon: "error",
      title: "Error al guardar cliente",
      text: "Inténtelo de nuevo o contacte con el administrador.",
      showConfirmButton: false,
      timer: 1500,
    });
  }
};

// Función para activar cliente (poner historico en true)
const activarCliente = async (cliente) => {
  const confirmacion = await Swal.fire({
    title: `¿Activar cliente ${cliente.nombre} ${cliente.apellidos}?`,
    icon: "question",
    showCancelButton: true,
    confirmButtonText: "Activar",
    cancelButtonText: "Cancelar",
  });

  if (!confirmacion.isConfirmed) return;

  try {
    // Crear una copia del cliente con historico en true
    const clienteActivado = { ...cliente, historico: true };

    // Llamar a la API para actualizar
    const actualizado = await updateCliente(cliente.id, clienteActivado);

    // Actualizar la lista local (opcional, también puedes volver a cargar todo)
    const index = clientes.value.findIndex((c) => c.id === cliente.id);
    if (index !== -1) {
      clientes.value[index] = actualizado;
    }

    Swal.fire({
      icon: "success",
      title: "Cliente reactivado",
      showConfirmButton: false,
      timer: 1500,
    });

    // Recargar lista actualizada
    await cargarClientes();
  } catch (error) {
    console.error("Error al reactivar cliente:", error);
    Swal.fire({
      icon: "error",
      title: "Error al activar cliente",
      text: "Por favor, intenta de nuevo.",
      timer: 1500,
    });
  }
};

const agregarCliente = () => {
  clientes.value.push({ ...nuevoCliente.value });
  // Reiniciar el formulario
  nuevoCliente.value = {
    dni: "",
    nombre: "",
    apellidos: "",
    email: "",
    movil: "",
    direccion: "",
    provincia: "",
    municipio: "",
    fechaAlta: "",
    historico: false,
  };
};

// Funcion Eliminar Cliente con patch (histórico a false)
const eliminarCliente = async (movil) => {
  // Refrescar lista desde la API
  clientes.value = await getClientes();
  // Buscar cliente completo (que incluye el ID)
  const clienteAEliminar = clientes.value.find(
    (cliente) => cliente.movil === movil
  );

  if (!clienteAEliminar) {
    Swal.fire({
      icon: "error",
      title: "Cliente no encontrado",
      showConfirmButton: false,
      timer: 1500,
    });
    return;
  }

  // Pedir confirmación antes de eliminar
  const result = await Swal.fire({
    title: `¿Eliminar al cliente ${clienteAEliminar.nombre} ${clienteAEliminar.apellidos}?`,
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "Sí, eliminar",
    cancelButtonText: "Cancelar",
  });

  // Si no confirma, salir
  if (!result.isConfirmed) return;

  // Si confirma, eliminar cliente usando la API y movil como ID
  await deleteCliente(clienteAEliminar.id);
  // Refrescar la lista desde la "API"
  clientes.value = await getClientes();

  Swal.fire({
    icon: "success",
    title: "Cliente eliminado",
    showConfirmButton: false,
    timer: 1500,
  });
};

// Función Editar Cliente (carga datos en el formulario)
const editarCliente = (movil) => {
  const cliente = clientes.value.find((c) => c.movil === movil);
  if (!cliente) {
    Swal.fire({
      icon: "error",
      title: "Cliente no encontrado",
      showConfirmButton: false,
      timer: 1500,
    });
    return;
  }

  // Copiar datos al formulario
  nuevoCliente.value = { ...cliente }; // 🔁 Aquí cargas el formulario con los datos
  editando.value = true;
  // Formatear fecha para el input type="date"
  nuevoCliente.value.fechaAlta = formatearFechaParaInput(cliente.fechaAlta);
  // Actualiza municipios filtrados según la provincia seleccionada
  filtrarMunicipios();
  nuevoCliente.value.municipio = cliente.municipio; // 🟢 Ahora estamos en modo edición
  clienteEditandoId.value = cliente.id;
};

// SCRIPS AUXILIARES

// Estado de validez del DNI/NIE si la estructura de datos es más compleja se usa reactive
const dniValido = ref(true); // Por defecto es válido y no muestra error al iniciar

// Función para validar DNI y NIE
const validarDniNie = (valor) => {
  const letras = "TRWAGMYFPDXBNJZSQVHLCKE";
  const dniRegex = /^[0-9]{8}[A-Z]$/;
  const nieRegex = /^[XYZ][0-9]{7}[A-Z]$/;

  valor = valor.toUpperCase();

  if (dniRegex.test(valor)) {
    const numero = parseInt(valor.slice(0, 8), 10);
    const letra = valor.charAt(8);
    return letra === letras[numero % 23]; //sale con true si es válido
  } else if (nieRegex.test(valor)) {
    const nie = valor.replace("X", "0").replace("Y", "1").replace("Z", "2");
    const numero = parseInt(nie.slice(0, 8), 10);
    const letra = valor.charAt(8);
    return letra === letras[numero % 23]; //sale con true si es válido
  }
  return false;
};

// Función única: capitaliza y asigna en el mismo paso
const capitalizarTexto = (campo) => {
  const texto = nuevoCliente.value[campo] ?? "";
  nuevoCliente.value[campo] = texto
    .toLowerCase()
    .split(" ")
    .map((palabra) => {
      if (!palabra) return "";
      return palabra.charAt(0).toLocaleUpperCase() + palabra.slice(1);
    })
    .join(" ");
};

// control email

const emailValido = ref(true);
const validarEmail = () => {
  const email = nuevoCliente.value.email.trim();
  // Expresión simple para email válido
  const regex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
  emailValido.value = regex.test(email);
};

// Provincias y municipios

const provincias = ref(provmuniData.provincias); // Array de provincias
const municipios = ref(provmuniData.municipios); // Array de municipios para filtrarlos
const municipiosFiltrados = ref([]); // vacío pero contendrá los municipios filtrados

const filtrarMunicipios = () => {
  // nombre de la provincia elegida en el <select>
  const nombreProv = nuevoCliente.value.provincia;

  // 1️⃣ buscar en provincias el objeto con ese nombre
  const prov = provincias.value.find((p) => p.nm === nombreProv);
  if (!prov) {
    municipiosFiltrados.value = [];
    return;
  }

  // 2️⃣ los dos primeros dígitos del id de la provincia
  const codigoProv = prov.id.slice(0, 2);

  // 3️⃣ filtrar los municipios cuyo id empiece por esos dos dígitos
  municipiosFiltrados.value = municipios.value.filter((m) =>
    m.id.startsWith(codigoProv)
  );

  // 4️⃣ opcional: resetear el municipio si ya no corresponde
  nuevoCliente.value.municipio = "";
};

const movilValido = ref(true);
// Validar al salir del campo
const validarDni = () => {
  nuevoCliente.value.dni = nuevoCliente.value.dni.trim().toUpperCase();
  dniValido.value = validarDniNie(dni);
};

const validarMovil = () => {
  const movil = nuevoCliente.value.movil.trim();
  // Expresión para móvil español (9 dígitos, empieza por 6, 7, 8 o 9)
  const regex = /^[6789]\d{8}$/;
  movilValido.value = regex.test(movil) || movil === "";
};

// conversor fecha
function formatearFechaParaInput(fecha) {
  if (!fecha) return '';

  // Detecta formato dd/mm/yyyy
  if (fecha.includes('/')) {
    const [dd, mm, yyyy] = fecha.split('/');
    return `${yyyy}-${mm.padStart(2, '0')}-${dd.padStart(2, '0')}`;
  }

  // Detecta formato yyyy-mm-dd
  if (fecha.includes('-')) {
    const partes = fecha.split('-');
    if (partes.length === 3) return fecha; // ya formato ISO
  }

  return '';
}
</script>

<style scoped>
.is-invalid {
  border-color: #f28b82 !important;
  background-color: #ffe6e6;
}

.invalid-feedback {
  display: block;
}

.gestion-clientes {
  width: 95%;
  max-width: none;
  margin: 0 auto;
  padding: 2rem 0;
}

.form-control {
  width: 100%;
}
</style>
