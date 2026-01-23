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
        placeholder="Buscar por nombre, tipo o marca..."
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
              <span class="card-precio">{{ p.precio }}€</span>
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
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.titulo-principal {
  color: #2c3e50;
  text-align: center;
  margin-bottom: 30px;
  font-size: 2rem;
  border-bottom: 3px solid #3498db;
  padding-bottom: 10px;
}

.subtitulo {
  color: #34495e;
  margin-bottom: 15px;
  font-size: 1.1rem;
}

/* ============================================================
   BÚSQUEDA
   ============================================================ */
.busqueda-container {
  display: flex;
  gap: 10px;
  margin-bottom: 30px;
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
}

.search-input {
  flex: 1;
  padding: 12px 15px;
  border: 2px solid #ddd;
  border-radius: 8px;
  font-size: 16px;
  transition: border-color 0.3s;
}

.search-input:focus {
  outline: none;
  border-color: #3498db;
  box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.2);
}

/* ============================================================
   BOTONES
   ============================================================ */
.btn {
  padding: 12px 20px;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  transition: all 0.3s ease;
  font-weight: 600;
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

.btn-buscar {
  background-color: #3498db;
  color: white;
}

.btn-buscar:hover {
  background-color: #2980b9;
}

.btn-filtrar {
  background-color: #2ecc71;
  color: white;
}

.btn-filtrar:hover {
  background-color: #27ae60;
}

.icono {
  font-size: 18px;
}

/* ============================================================
   FILTROS
   ============================================================ */
.filtros-container {
  background: #f8f9fa;
  padding: 20px;
  border-radius: 12px;
  margin-bottom: 30px;
  border: 1px solid #e9ecef;
}

.filtros-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 15px;
  align-items: end;
}

.input-filtro, .select-filtro {
  padding: 10px 12px;
  border: 1px solid #ced4da;
  border-radius: 6px;
  font-size: 14px;
  width: 100%;
  box-sizing: border-box;
}

.select-filtro {
  background-color: white;
  cursor: pointer;
}

/* ============================================================
   PRODUCTOS
   ============================================================ */
.grid-productos {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 25px;
  margin: 30px 0;
}

.card-producto {
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 5px 15px rgba(0,0,0,0.08);
  transition: all 0.3s ease;
  border: 1px solid #e9ecef;
}

.card-producto:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 25px rgba(0,0,0,0.15);
}

.card-imagen-container {
  position: relative;
  height: 180px;
  overflow: hidden;
}

.card-imagen {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s;
}

.card-producto:hover .card-imagen {
  transform: scale(1.05);
}

.card-badge {
  position: absolute;
  top: 10px;
  right: 10px;
  background: #27ae60;
  color: white;
  padding: 5px 10px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 600;
}

.bajo-stock {
  background: #e74c3c;
}

.card-contenido {
  padding: 20px;
}

.card-titulo {
  color: #2c3e50;
  margin: 0 0 10px 0;
  font-size: 1.1rem;
  line-height: 1.3;
}

.card-descripcion {
  color: #7f8c8d;
  font-size: 14px;
  line-height: 1.5;
  margin-bottom: 15px;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.card-precio-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.card-precio {
  color: #2c3e50;
  font-size: 1.4rem;
  font-weight: bold;
}

.card-iva {
  color: #95a5a6;
  font-size: 12px;
}

/* ============================================================
   CARGA
   ============================================================ */
.carga-container {
  text-align: center;
  padding: 60px 20px;
}

.spinner {
  width: 50px;
  height: 50px;
  border: 5px solid #f3f3f3;
  border-top: 5px solid #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 20px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.sin-resultados {
  text-align: center;
  padding: 40px;
  color: #7f8c8d;
  font-size: 18px;
  background: #f8f9fa;
  border-radius: 12px;
  border: 2px dashed #dee2e6;
}

/* ============================================================
   PAGINACIÓN
   ============================================================ */
.paginacion-container {
  margin-top: 40px;
  padding-top: 20px;
  border-top: 1px solid #e9ecef;
}

.paginacion-info {
  text-align: center;
  color: #6c757d;
  margin-bottom: 15px;
  font-size: 14px;
}

.paginacion-botones {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 8px;
}

.btn-paginacion, .btn-pagina {
  padding: 8px 15px;
  border: 1px solid #dee2e6;
  background: white;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
  font-size: 14px;
}

.btn-paginacion:hover:not(:disabled),
.btn-pagina:hover:not(.activa) {
  background: #f8f9fa;
  border-color: #adb5bd;
}

.btn-paginacion:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-pagina.activa {
  background-color: #3498db;
  color: white;
  border-color: #3498db;
  font-weight: bold;
}
</style>