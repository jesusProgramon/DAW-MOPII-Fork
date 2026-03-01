<!-- ============================================================
   COMPONENTE Productos.vue
   --------------------------------------------------------------
   Vista principal del catálogo:
   - Consulta productos al backend Flask mediante api.js
   - Soporta búsqueda, filtros, ordenación y paginación
   - Usa Composition API (Vue 3)
   ============================================================ -->

<template>
  <div class="catalogo-container">
    <h2 class="titulo-principal">Catálogo de Productos</h2>

    <!-- ===============================
         BÚSQUEDA GENERAL
         =============================== -->
    <div class="busqueda-container">
      <input
        type="text"
        v-model="terminoBusqueda"
        placeholder="¿Qué estás buscando?"
        @keyup.enter="accionEncontrar"
        class="search-input"
      />
      <button @click="accionEncontrar" class="btn btn-buscar">
        <span class="icono">🔍</span> Buscar
      </button>
    </div>

    <!-- ===============================
         FILTROS AVANZADOS
         =============================== -->
    <div class="filtros-container">
      <h3 class="subtitulo">Filtros Avanzados</h3>
      <div class="filtros-grid">
        <input type="text" v-model="filtroTipo" placeholder="Tipo (motosierra, taladro…)" class="input-filtro" />
        <input type="text" v-model="filtroMarca" placeholder="Marca (STIHL, Makita…)" class="input-filtro" />
        <input type="number" v-model.number="precioMin" placeholder="Precio mínimo €" class="input-filtro" />
        <input type="number" v-model.number="precioMax" placeholder="Precio máximo €" class="input-filtro" />
        
        <select v-model="orden" class="select-filtro">
          <option value="">Ordenar por...</option>
          <option value="asc">Precio: Menor a Mayor</option>
          <option value="desc">Precio: Mayor a Menor</option>
        </select>
        
        <button @click="accionFiltrar" class="btn btn-filtrar">
          <span class="icono">⚙️</span> Aplicar Filtros
        </button>
      </div>
    </div>

    <!-- ===============================
         ESTADO DE CARGA
         =============================== -->
    <div v-if="loading" class="carga-container">
      <div class="spinner"></div>
      <p>Cargando productos...</p>
    </div>

    <!-- ===============================
         LISTA DE PRODUCTOS
         =============================== -->
    <div v-else>
      <div v-if="productos.length === 0" class="sin-resultados">
        <p>No se encontraron productos con los filtros seleccionados.</p>
      </div>
      
      <div v-else class="grid-productos">
        <div v-for="p in productos" :key="p.id" class="card-producto">
          <div class="card-imagen-container">
            <img :src="'/img/' + p.imagen" :alt="p.nombre" class="card-imagen" />
            <div class="card-badge" :class="{ 'bajo-stock': p.stock < 10 }">
              Stock: {{ p.stock }}
            </div>
          </div>
          <div class="card-contenido">
            <h3 class="card-titulo">{{ p.nombre }}</h3>
            <p class="card-descripcion">{{ p.descripcion }}</p>
            <div class="card-precio-container">
              <span class="card-precio">{{ p.precio }} Euros</span>
              <span class="card-iva">IVA incluido</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ===============================
         PAGINACIÓN
         =============================== -->
    <div class="paginacion-container" v-if="totalPaginas > 1">
      <div class="paginacion-info">
        Página {{ paginaActual }} de {{ totalPaginas }} • {{ totalResultados }} productos
      </div>
      
      <div class="paginacion-botones">
        <button 
          @click="cambiarPagina(paginaActual - 1)" 
          :disabled="paginaActual === 1"
          class="btn-paginacion"
        >
          ← Anterior
        </button>

        <button
          v-for="n in totalPaginas"
          :key="n"
          @click="cambiarPagina(n)"
          :class="['btn-pagina', { activa: n === paginaActual }]"
        >
          {{ n }}
        </button>

        <button 
          @click="cambiarPagina(paginaActual + 1)" 
          :disabled="paginaActual === totalPaginas"
          class="btn-paginacion"
        >
          Siguiente →
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
/* ============================================================
   IMPORTS
   ============================================================ */
import { ref } from "vue"
import {
  obtenerProductos,
  filtrarProductos,
  buscarProductos
} from "@/services/api"

/* ============================================================
   VARIABLES REACTIVAS
   ============================================================ */
const productos = ref([])
const loading = ref(true)
const terminoBusqueda = ref("")
const filtroTipo = ref("")
const filtroMarca = ref("")
const precioMin = ref(null)
const precioMax = ref(null)
const orden = ref("")
const paginaActual = ref(1)
const porPagina = ref(10)
const totalPaginas = ref(1)
const totalResultados = ref(0)

/* ============================================================
   FUNCIONES
   ============================================================ */
const cargarProductos = async () => {
  loading.value = true
  try {
    const data = await filtrarProductos({
      pagina: paginaActual.value,
      por_pagina: porPagina.value,
      tipo: filtroTipo.value,
      marca: filtroMarca.value,
      precio_min: precioMin.value,
      precio_max: precioMax.value,
      ordenar: orden.value
    })

    productos.value = data.productos
    paginaActual.value = data.pagina_actual
    totalPaginas.value = data.total_paginas
    totalResultados.value = data.total_resultados
  } catch (e) {
    console.error("Error cargando productos:", e)
    productos.value = []
  }
  loading.value = false
}

const accionEncontrar = async () => {
  paginaActual.value = 1
  if (!terminoBusqueda.value.trim()) {
    cargarProductos()
    return
  }

  loading.value = true
  try {
    const resultados = await buscarProductos(terminoBusqueda.value)
    productos.value = resultados
    totalResultados.value = resultados.length
    totalPaginas.value = Math.ceil(resultados.length / porPagina.value)
  } catch (e) {
    console.error("Error en la búsqueda:", e)
  }
  loading.value = false
}

const accionFiltrar = () => {
  paginaActual.value = 1
  cargarProductos()
}

const cambiarPagina = (nuevaPagina) => {
  if (nuevaPagina < 1 || nuevaPagina > totalPaginas.value) return
  paginaActual.value = nuevaPagina
  cargarProductos()
}

/* ============================================================
   CARGA INICIAL
   ============================================================ */
cargarProductos()
</script>

<style scoped>
/* ============================================================
   ESTILOS GENERALES
   ============================================================ */
.catalogo-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 24px 20px;
  font-family: 'Inter', 'Segoe UI', system-ui, sans-serif;
  color: #1e293b;
}

.titulo-principal {
  color: #0f172a;
  text-align: center;
  margin-bottom: 32px;
  font-size: 2.2rem;
  font-weight: 600;
  letter-spacing: -0.02em;
  border-bottom: 3px solid #3b82f6;
  padding-bottom: 12px;
  display: inline-block;
  width: 100%;
}

.subtitulo {
  color: #334155;
  margin-bottom: 16px;
  font-size: 1.1rem;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

/* ============================================================
   BÚSQUEDA
   ============================================================ */
.busqueda-container {
  display: flex;
  gap: 12px;
  margin-bottom: 32px;
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
}

.search-input {
  flex: 1;
  padding: 14px 18px;
  border: 2px solid #e2e8f0;
  border-radius: 12px;
  font-size: 16px;
  transition: all 0.2s ease;
  background-color: #f8fafc;
}

.search-input:focus {
  outline: none;
  border-color: #3b82f6;
  background-color: #ffffff;
  box-shadow: 0 0 0 4px rgba(59, 130, 246, 0.15);
}

/* ============================================================
   BOTONES
   ============================================================ */
.btn {
  padding: 14px 24px;
  border: none;
  border-radius: 12px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  transition: all 0.2s ease;
  line-height: 1;
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
}

.btn:active {
  transform: translateY(0);
}

.btn-buscar {
  background-color: #3b82f6;
  color: white;
}

.btn-buscar:hover {
  background-color: #2563eb;
}

.btn-filtrar {
  background-color: #10b981;
  color: white;
}

.btn-filtrar:hover {
  background-color: #059669;
}

.icono {
  font-size: 1.2em;
}

/* ============================================================
   FILTROS
   ============================================================ */
.filtros-container {
  background: #ffffff;
  padding: 24px;
  border-radius: 16px;
  margin-bottom: 32px;
  border: 1px solid #e2e8f0;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.02);
}

.filtros-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 16px;
  align-items: end;
}

.input-filtro,
.select-filtro {
  padding: 12px 14px;
  border: 1px solid #cbd5e1;
  border-radius: 10px;
  font-size: 14px;
  width: 100%;
  box-sizing: border-box;
  background-color: #f8fafc;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.input-filtro:focus,
.select-filtro:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.select-filtro {
  background-color: white;
  cursor: pointer;
}

/* ============================================================
   TARJETAS DE PRODUCTO
   ============================================================ */
.grid-productos {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 32px;
  margin: 32px 0;
}

.card-producto {
  background: #ffffff;
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 10px 20px -5px rgba(0, 0, 0, 0.05);
  transition: transform 0.25s ease, box-shadow 0.25s ease;
  border: 1px solid #f1f5f9;
  display: flex;
  flex-direction: column;
}

.card-producto:hover {
  transform: translateY(-6px);
  box-shadow: 0 20px 30px -8px rgba(0, 0, 0, 0.15);
}

.card-imagen-container {
  position: relative;
  height: 200px;
  overflow: hidden;
  background-color: #f1f5f9;
}

.card-imagen {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.4s ease;
}

.card-producto:hover .card-imagen {
  transform: scale(1.08);
}

.card-badge {
  position: absolute;
  top: 12px;
  right: 12px;
  background: #10b981;
  color: white;
  padding: 6px 12px;
  border-radius: 30px;
  font-size: 0.8rem;
  font-weight: 600;
  letter-spacing: 0.3px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.bajo-stock {
  background: #ef4444;
}

.card-contenido {
  padding: 20px 18px 22px;
  flex: 1;
  display: flex;
  flex-direction: column;
}

.card-titulo {
  color: #0f172a;
  margin: 0 0 8px 0;
  font-size: 1.2rem;
  font-weight: 600;
  line-height: 1.3;
}

.card-descripcion {
  color: #475569;
  font-size: 0.9rem;
  line-height: 1.5;
  margin-bottom: 16px;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
  flex: 1;
}

.card-precio-container {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  margin-top: auto;
  padding-top: 12px;
  border-top: 1px solid #e2e8f0;
}

.card-precio {
  color: #0f172a;
  font-size: 1.5rem;
  font-weight: 700;
  line-height: 1;
}

.card-iva {
  color: #64748b;
  font-size: 0.75rem;
  font-weight: 500;
}

/* ============================================================
   ESTADOS DE CARGA / SIN RESULTADOS
   ============================================================ */
.carga-container {
  text-align: center;
  padding: 80px 20px;
}

.spinner {
  width: 48px;
  height: 48px;
  border: 4px solid #e2e8f0;
  border-top: 4px solid #3b82f6;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
  margin: 0 auto 20px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.sin-resultados {
  text-align: center;
  padding: 60px 20px;
  color: #64748b;
  font-size: 1.1rem;
  background: #f8fafc;
  border-radius: 24px;
  border: 2px dashed #cbd5e1;
}

/* ============================================================
   PAGINACIÓN
   ============================================================ */
.paginacion-container {
  margin-top: 48px;
  padding-top: 24px;
  border-top: 1px solid #e2e8f0;
}

.paginacion-info {
  text-align: center;
  color: #64748b;
  margin-bottom: 16px;
  font-size: 0.9rem;
  font-weight: 500;
}

.paginacion-botones {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 10px;
  /* Eliminado el fondo azul que causaba conflicto */
  background-color: transparent;
  color: inherit;
}

.btn-paginacion,
.btn-pagina {
  padding: 10px 16px;
  border: 1px solid #e2e8f0;
  background: white;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.15s ease;
  font-size: 0.9rem;
  font-weight: 500;
  color: #334155;
  min-width: 44px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.btn-paginacion:hover:not(:disabled),
.btn-pagina:hover:not(.activa) {
  background: #f1f5f9;
  border-color: #94a3b8;
  color: #0f172a;
}

.btn-paginacion:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  background: #f1f5f9;
}

.btn-pagina.activa {
  background-color: #3b82f6;
  color: white;
  border-color: #3b82f6;
  font-weight: 600;
  box-shadow: 0 4px 10px rgba(59, 130, 246, 0.3);
}
</style>