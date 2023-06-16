<script setup>
  import { ref, reactive, watch, onMounted, computed } from "vue";
  import { generarId } from "./helpers";
  import Swal from 'sweetalert2'
  import Presupuesto from "./components/Presupuesto.vue";
  import ControlPresupuesto from "./components/ControlPresupuesto.vue";
  import Modal from "./components/Modal.vue";
  import Gasto from "./components/Gasto.vue";
  import Filtros from "./components/Filtros.vue";
  
  import iconoNuevoGasto from "./assets/img/nuevo-gasto.svg";

  const modal = reactive({
    mostrar: false,
    animar: false,
  });

  const presupuesto = ref(0);
  const disponible = ref(0);
  const gastado = ref(0);
  const filtro = ref('');

  const gasto = reactive({
    nombre: '',
    cantidad: '',
    categoria: '',
    id: null,
    fecha: Date.now()
  });

  const gastos = ref([]);

  // Guardar en LocalStorage los gatos y el presupuesto
  const guardarLocalStorage = () => {
    localStorage.setItem('gastos', JSON.stringify(gastos.value));
    localStorage.setItem('presupuesto', JSON.stringify(presupuesto.value));
  }

  onMounted(() => {
    const gastosStorage = JSON.parse(localStorage.getItem('gastos'));
    const presupuestoStorage = JSON.parse(localStorage.getItem('presupuesto'));
    if (gastosStorage && presupuestoStorage) {
      gastos.value = gastosStorage;
      presupuesto.value = presupuestoStorage;
    }
  })
  
  watch(gastos, () => {
    const totalGastado = gastos.value.reduce((total, gasto) => gasto.cantidad + total, 0);
    gastado.value = totalGastado;
    
    const totalDisponible = presupuesto.value - totalGastado;
    disponible.value = totalDisponible;
    guardarLocalStorage();
  }, { 
    deep: true 
  })

  watch(modal, () => {
    if(!modal.mostrar) {
      // Reiniciar objeto
      reiniciarStateGasto();
    }
  }, { 
    deep: true 
  })

  const definirPresupuesto = (cantidad) => {
    presupuesto.value = cantidad;
    disponible.value = cantidad;
    guardarLocalStorage();
  };

  const mostrarModal = () => {
    modal.mostrar = true;
    setTimeout(() => {
      modal.animar = true;
    }, 300);
  };

  const ocultarModal = () => {
    modal.animar = false;

    setTimeout(() => {
      modal.mostrar = false;
    }, 300);
  };

  const guardarGasto = () => {
    if(gasto.id) {
      // editando
      const { id } = gasto;
      const indice = gastos.value.findIndex(gasto => gasto.id === id);
      gastos.value[indice] = { ...gasto}
    } else {
      // agregando
      gastos.value.push({
        ...gasto,
        id: generarId(),
      });
    }

    ocultarModal();
    reiniciarStateGasto();
  }

  const reiniciarStateGasto = () => {
    Object.assign(gasto, {
      nombre: '',
      cantidad: '',
      categoria: '',
      id: null,
      fecha: Date.now()
    });
  }

  const seleccionarGasto = id => {
    const gastoEditar = gastos.value.filter(gasto => gasto.id === id)[0];
    Object.assign(gasto, gastoEditar);
    mostrarModal();
  }


  const eliminarGasto = () => {
    Swal.fire({
      title: '¿Estás seguro?',
      text: "Un gasto eliminado no se puede recuperar",
      icon: 'warning',
      showCancelButton: true,
      confirmButtonColor: '#3B82F6',
      cancelButtonColor: '#EF4444',
      confirmButtonText: 'Si, eliminar',
      cancelButtonText: 'Cancelar'
    }).then((result) => {
      if (result.isConfirmed) {
        Swal.fire(
          'Eliminado',
          'El gasto ha sido eliminado',
          'success'
        )
        
        gastos.value = gastos.value.filter(gastoState => gastoState.id !== gasto.id);
        ocultarModal();
      }
    })
  }

  const gastosFiltrados = computed(() => {
    if(filtro.value) {
      return gastos.value.filter(gasto => gasto.categoria === filtro.value);
    }
    return gastos.value;
  })

  const resetApp = () => {
    gastos.value = [];
    presupuesto.value = 0;
    disponible.value = 0;
    gastado.value = 0;
    localStorage.clear();
  }
</script>

<template>
  <div 
    :class="{fijar: modal.mostrar}"
  >
    <header>
      <h1>Planificador de Gastos</h1>
      <div class="contenedor-header contenedor sombra">
        <Presupuesto 
          v-if="presupuesto === 0"
          @definir-presupuesto="definirPresupuesto"
        />
        <ControlPresupuesto 
          v-else
          :presupuesto="presupuesto"
          :disponible="disponible"
          :gastado="gastado"
          @reset-app="resetApp"
        />
      </div>
    </header>

    <main v-if="presupuesto > 0">
      <Filtros 
        v-model:filtro = "filtro"
      />
      <div class="listado-gastos contenedor">
        <h2>{{gastosFiltrados.length > 0 ? 'Gastos' : 'No hay gastos' }}</h2>

        <Gasto 
          v-for="gasto in gastosFiltrados"
          :key="gasto.id"
          :gasto="gasto"
          @seleccionar-gasto="seleccionarGasto"
        />
      </div>
      <div class="crear-gasto">
        <img 
          :src="iconoNuevoGasto" 
          alt="icono nuevo gasto"
          @click="mostrarModal"
        >
      </div>
      <Modal 
        v-if="modal.mostrar"
        @cerrar-modal="ocultarModal"
        @guardar-gasto="guardarGasto"
        @eliminar-gasto="eliminarGasto"
        :modal="modal"
        :disponible="disponible"
        :id="gasto.id"
        v-model:nombre="gasto.nombre"
        v-model:cantidad="gasto.cantidad"
        v-model:categoria="gasto.categoria"
      />
    </main>
  </div>
</template>

<style>
  :root {
    --azul: #3B82F6;
    --blanco: #FFF;
    --gris-claro: #F5F5F5;
    --gris: #94a3b8;
    --gris-oscuro: #64748B;
    --negro: #000;
  }
  html {
    font-size: 62.5%;
    box-sizing: border-box;
  }
  *,
  *::before,
  *:after {
    box-sizing: inherit;
  }
  body {
    font-size:  1.6rem;
    font-family: "Lato", sans-serif;
    background-color: var(--gris-claro);
  }
  h1 { 
    font-size: 4rem;
  }
  h2 {
    font-size: 3rem;
  }
  .fijar {
    overflow: hidden;
    height: 100vh;
  }
  header {
    background-color: var(--azul);
  }
  header h1 {
    padding: 3rem 0;
    margin: 0;
    color: var(--blanco);
    text-align: center;
  }
  .contenedor {
    width: 90%;
    max-width: 80rem;
    margin: 0 auto;
  }
  .contenedor-header {
    margin-top: -5rem;
    transform: translateY(5rem);
    padding: 5rem;
  }
  .sombra {
    box-shadow: 0px 10px 15px -3px rgba(0,0,0,0.1);
    background-color: var(--blanco);
    border-radius: 1.2rem;
    padding: 5rem;
  }

  .crear-gasto {
    position: fixed;
    bottom: 5rem;
    right: 5rem;
  }
  .crear-gasto img {
    width: 5rem;
    cursor: pointer;
  }
  .listado-gastos {
    margin-top: 10rem;
  }
  .listado-gastos h2 {
    font-weight: 900;
    color: var(--gris-oscuro);
  }
</style>
