<script setup lang="ts">
import { ref, computed } from 'vue'

interface Empleado {
  id: number
  nombre: string
  rol: string
  departamento: string
  estado: 'Activo' | 'En Pausa' | 'Revisión'
  tareas: number
}

const emit = defineEmits<{
  (e: 'evento', tipo: string, detalle: string): void
}>()

const empleados = ref<Empleado[]>([
  { id: 101, nombre: 'Elena Rostova', rol: 'Arquitecta Frontend', departamento: 'Ingeniería', estado: 'Activo', tareas: 14 },
  { id: 102, nombre: 'Carlos Mendoza', rol: 'Diseñador UX/UI', departamento: 'Producto', estado: 'Activo', tareas: 9 },
  { id: 103, nombre: 'Sofía Alarcón', rol: 'Ingeniera DevOps', departamento: 'Infraestructura', estado: 'En Pausa', tareas: 21 },
  { id: 104, nombre: 'Mateo Silva', rol: 'Analista QA', departamento: 'Ingeniería', estado: 'Revisión', tareas: 7 },
  { id: 105, nombre: 'Valeria Gómez', rol: 'Product Manager', departamento: 'Producto', estado: 'Activo', tareas: 18 }
])

const busqueda = ref('')
const filtroDepartamento = ref('Todos')
const filtroEstado = ref('Todos')
const ordenCampo = ref<keyof Empleado>('id')
const ordenAsc = ref(true)

const departamentos = ['Todos', 'Ingeniería', 'Producto', 'Infraestructura']
const estados = ['Todos', 'Activo', 'En Pausa', 'Revisión']

const empleadosFiltrados = computed(() => {
  return empleados.value
    .filter((emp) => {
      const coincideTexto =
        emp.nombre.toLowerCase().includes(busqueda.value.toLowerCase()) ||
        emp.rol.toLowerCase().includes(busqueda.value.toLowerCase())
      const coincideDepto = filtroDepartamento.value === 'Todos' || emp.departamento === filtroDepartamento.value
      const coincideEstado = filtroEstado.value === 'Todos' || emp.estado === filtroEstado.value
      return coincideTexto && coincideDepto && coincideEstado
    })
    .sort((a, b) => {
      const valA = a[ordenCampo.value]
      const valB = b[ordenCampo.value]
      if (valA < valB) return ordenAsc.value ? -1 : 1
      if (valA > valB) return ordenAsc.value ? 1 : -1
      return 0
    })
})

const cambiarOrden = (campo: keyof Empleado) => {
  if (ordenCampo.value === campo) {
    ordenAsc.value = !ordenAsc.value
  } else {
    ordenCampo.value = campo
    ordenAsc.value = true
  }
  emit('evento', 'Orden Tabla', `Ordenado por ${campo} (${ordenAsc.value ? 'ASC' : 'DESC'})`)
}

const rotarEstado = (emp: Empleado) => {
  const ciclo: Empleado['estado'][] = ['Activo', 'En Pausa', 'Revisión']
  const siguiente = ciclo[(ciclo.indexOf(emp.estado) + 1) % ciclo.length]!
  emp.estado = siguiente
  emit('evento', 'Cambio Estado', `${emp.nombre} → ${siguiente}`)
}

const eliminarRegistro = (emp: Empleado) => {
  empleados.value = empleados.value.filter((item) => item.id !== emp.id)
  emit('evento', 'Eliminar Fila', `Se eliminó a ${emp.nombre}`)
}

const agregarAleatorio = () => {
  const nombres = ['Diego Vargas', 'Lucía Morales', 'Andrés Castro', 'Camila Ortiz']
  const roles = ['Fullstack Dev', 'Data Engineer', 'Scrum Master', 'Security Lead']
  const deptos = ['Ingeniería', 'Producto', 'Infraestructura']
  const nuevo: Empleado = {
    id: Math.floor(200 + Math.random() * 800),
    nombre: nombres[Math.floor(Math.random() * nombres.length)]!,
    rol: roles[Math.floor(Math.random() * roles.length)]!,
    departamento: deptos[Math.floor(Math.random() * deptos.length)]!,
    estado: 'Activo',
    tareas: Math.floor(1 + Math.random() * 20)
  }
  empleados.value.unshift(nuevo)
  emit('evento', 'Nuevo Registro', `Añadido ${nuevo.nombre} (#${nuevo.id})`)
}
</script>

<template>
  <div class="space-y-4">
    <div class="flex flex-wrap items-center justify-between gap-3 bg-slate-900/70 p-4 rounded-xl border border-slate-800">
      <div class="flex flex-wrap items-center gap-3 flex-1">
        <input
          v-model="busqueda"
          @input="emit('evento', 'Búsqueda', `Filtro: '${busqueda}'`)"
          type="text"
          placeholder="Buscar por nombre o rol..."
          class="bg-slate-950 border border-slate-700 rounded-lg px-3 py-2 text-sm text-slate-100 focus:outline-none focus:border-emerald-500 w-64"
        />
        <select
          v-model="filtroDepartamento"
          @change="emit('evento', 'Filtro Depto', filtroDepartamento)"
          class="bg-slate-950 border border-slate-700 rounded-lg px-3 py-2 text-sm text-slate-200"
        >
          <option v-for="d in departamentos" :key="d" :value="d">{{ d }}</option>
        </select>
        <select
          v-model="filtroEstado"
          @change="emit('evento', 'Filtro Estado', filtroEstado)"
          class="bg-slate-950 border border-slate-700 rounded-lg px-3 py-2 text-sm text-slate-200"
        >
          <option v-for="e in estados" :key="e" :value="e">{{ e }}</option>
        </select>
      </div>
      <button
        @click="agregarAleatorio"
        class="bg-emerald-600 hover:bg-emerald-500 text-white text-sm font-medium px-4 py-2 rounded-lg transition cursor-pointer"
      >
        + Simular Registro
      </button>
    </div>

    <div class="overflow-x-auto rounded-xl border border-slate-800 bg-slate-900/50">
      <table class="w-full text-left border-collapse text-sm">
        <thead>
          <tr class="border-b border-slate-800 bg-slate-900 text-slate-400 select-none">
            <th @click="cambiarOrden('id')" class="p-3 cursor-pointer hover:text-white">ID ↕</th>
            <th @click="cambiarOrden('nombre')" class="p-3 cursor-pointer hover:text-white">Colaborador ↕</th>
            <th @click="cambiarOrden('departamento')" class="p-3 cursor-pointer hover:text-white">Departamento ↕</th>
            <th @click="cambiarOrden('estado')" class="p-3 cursor-pointer hover:text-white">Estado (Clic) ↕</th>
            <th @click="cambiarOrden('tareas')" class="p-3 cursor-pointer hover:text-white">Tareas ↕</th>
            <th class="p-3 text-right">Acciones</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-slate-800/70">
          <tr v-for="emp in empleadosFiltrados" :key="emp.id" class="hover:bg-slate-800/40 transition">
            <td class="p-3 font-mono text-slate-400">#{{ emp.id }}</td>
            <td class="p-3">
              <div class="font-medium text-slate-100">{{ emp.nombre }}</div>
              <div class="text-xs text-slate-400">{{ emp.rol }}</div>
            </td>
            <td class="p-3 text-slate-300">{{ emp.departamento }}</td>
            <td class="p-3">
              <button
                @click="rotarEstado(emp)"
                :class="{
                  'bg-emerald-500/15 text-emerald-400 border-emerald-500/30': emp.estado === 'Activo',
                  'bg-amber-500/15 text-amber-400 border-amber-500/30': emp.estado === 'En Pausa',
                  'bg-sky-500/15 text-sky-400 border-sky-500/30': emp.estado === 'Revisión'
                }"
                class="px-2.5 py-1 rounded-full text-xs font-medium border cursor-pointer hover:opacity-80 transition"
              >
                {{ emp.estado }}
              </button>
            </td>
            <td class="p-3 font-mono text-slate-200">{{ emp.tareas }}</td>
            <td class="p-3 text-right">
              <button
                @click="eliminarRegistro(emp)"
                class="text-rose-400 hover:text-rose-300 text-xs font-medium px-2 py-1 rounded hover:bg-rose-500/10 cursor-pointer"
              >
                Eliminar
              </button>
            </td>
          </tr>
          <tr v-if="empleadosFiltrados.length === 0">
            <td colspan="6" class="p-8 text-center text-slate-500">
              No hay registros que coincidan con los filtros actuales.
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>