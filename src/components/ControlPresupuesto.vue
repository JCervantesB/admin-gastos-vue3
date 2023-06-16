<script setup>
    import { computed } from "vue";
    import CircleProgress from "vue3-circle-progress";
    import "vue3-circle-progress/dist/circle-progress.css";
    import { formatearCantidad } from '../helpers'

    const props = defineProps({
        presupuesto: {
            type: Number,
            required: true
        },
        disponible: {
            type: Number,
            required: true
        },
        gastado: {
            type: Number,
            required: true
        }
    })

    defineEmits(['reset-app'])

    const porcentaje = computed(() => {
        return parseInt(((props.presupuesto - props.disponible) / props.presupuesto) * 100)
    })

    const barColor = computed(() => {
    const porcentajeActual = porcentaje.value;

        if (porcentajeActual >= 0 && porcentajeActual <= 55) {
        return '#3b82f6'; // Azul
        } else if (porcentajeActual > 55 && porcentajeActual <= 80) {
        return '#f6983b'; // Naranja
        } else if (porcentajeActual > 80 && porcentajeActual >= 100) {
        return '#DC2626'; // Rojo
        }
    });
    const emptyColor = computed(() => {
        if (porcentaje.value > 100) {
        return '#DC2626'; 
        } else {
        return '#e1e1e1'; 
        }
    });
</script>

<template>
    <div class="dos-columnas">
        <div class="contenedor-grafico">
            <p class="porcentaje" :style="{ color: barColor }">
                {{porcentaje}}%
            </p>
            <CircleProgress
                :percent="porcentaje"
                :size="250"
                :border-width="20"
                :border-bg-width="20"
                :fill-color="barColor"
                :empty-color="emptyColor"          
            />
        </div>

        <div class="contenedor-presupuesto">
            <button
                class="reset-app"
                @click="$emit('reset-app')"
            >
            Reiniciar App</button>

            <p>
                <span>Presupuesto:</span>
                {{ formatearCantidad(presupuesto) }}
            </p>
            <p :class="{excedido: disponible < 0}">
                <span>Disponible:</span>
                {{ formatearCantidad(disponible) }}
            </p>
            <p>
                <span>Gastado:</span>
                {{ formatearCantidad(gastado) }}
            </p>
        </div>
    </div>
</template>

<style scoped>
    .contenedor-grafico {
        margin: 0 auto;
        position: relative;
    }
    .porcentaje {
        position: absolute;
        margin: auto;
        top: calc(50% - 1.5rem);
        left: 0;
        right: 0;
        font-size: 3rem;
        font-weight: 900;
        text-align: center;
        z-index: 100;
    }
    .dos-columnas {
        display: flex;
        flex-direction: column;
    }
    .dos-columnas > :first-child {
        margin-bottom: 3rem;
    }

    @media (min-width: 768px) {
        .dos-columnas {
            flex-direction: row;
            gap: 4rem;
            align-items: center;
        }

        .dos-columnas > :first-child {
            margin-bottom: 0;
        }
    }
    .reset-app {
        background-color: #DB2777;
        border: none;
        padding: 1rem;
        width: 100%;
        color: var(--blanco);
        font-weight: 900;
        text-transform: uppercase;
        border-radius: 1rem;
        transition-property: background-color;
        transition-duration: .3s;
    }
    .reset-app:hover {
        background-color: #cc1265;
        cursor: pointer;
    }

    .contenedor-presupuesto {
        width: 100%;
    }

    .contenedor-presupuesto p {
        font-size: 2.4rem;
        text-align: center;
        color: var(--gris-oscuro);
    }
    @media (min-width: 768px) {
        .contenedor-presupuesto p {
            text-align: left;
        }
    }
    p.excedido{
        color: #DC2626;
    }
    .contenedor-presupuesto span {
        font-weight: 900;
        color: var(--azul);
    }
</style>