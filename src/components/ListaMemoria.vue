<script setup lang="ts">
import { ref, computed } from 'vue'

interface TareaMemoria {
  id: number
  titulo: string
  prioridad: 'Alta' | 'Media' | 'Baja'
  completada: boolean
}

const emit = defineEmits<{
  (e: 'evento', tipo: string, detalle: string): void
}>()

const items = ref<TareaMemoria[]>([
  { id: 1, titulo: 'Configurar repositorio en GitHub', prioridad: 'Alta', completada: true },
  { id: 2, titulo: 'Integrar Tailwind CSS v4 en Vite', prioridad: 'Alta', completada: true },
  { id: 3, titulo: 'Probar reactividad de listas en memoria', prioridad: 'Media', completada: false }
])

const nuevoTitulo = ref('')
const nuevaPrioridad = ref<TareaMemoria['prioridad']>('Media')
const soloPendientes = ref(false)

const itemsVisibles = computed(() =>
  soloPendientes.value ? items.value.filter((i) => !i.completada) : items.value
)

const agregarItem = () => {
  const texto = nuevoTitulo.value.trim()
  if (!texto) return
  items.value.unshift({
    id: Date.now(),
    titulo: texto,
    prioridad: nuevaPrioridad.value,
    completada: false
  })
  emit('evento', 'Crear Objeto', `"${texto}" (${nuevaPrioridad.value})`)
  nuevoTitulo.value = ''
}

const alternarCompletada = (item: TareaMemoria) => {
  item.completada = !item.completada
  emit('evento', 'Toggle Objeto', `${item.titulo} → ${item.completada ? 'Completada' : 'Pendiente'}`)
}

const borrarItem = (item: TareaMemoria) => {
  items.value = items.value.filter((i) => i.id !== item.id)
  emit('evento', 'Eliminar Objeto', `Eliminado "${item.titulo}"`)
}
</script>

<template>
  <div class="space-y-4">
    <form @submit.prevent="agregarItem" class="flex flex-wrap gap-3 bg-slate-900/70 p-4 rounded-xl border border-slate-800">
      <input
        v-model="nuevoTitulo"
        type="text"
        placeholder="Nuevo objeto en memoria..."
        class="flex-1 min-w-[220px] bg-slate-950 border border-slate-700 rounded-lg px-3 py-2 text-sm text-slate-100 focus:outline-none focus:border-emerald-500"
      />
      <select v-model="nuevaPrioridad" class="bg-slate-950 border border-slate-700 rounded-lg px-3 py-2 text-sm text-slate-200">
        <option value="Alta">Prioridad Alta</option>
        <option value="Media">Prioridad Media</option>
        <option value="Baja">Prioridad Baja</option>
      </select>
      <button type="submit" class="bg-emerald-600 hover:bg-emerald-500 text-white text-sm font-medium px-4 py-2 rounded-lg cursor-pointer transition">
        Añadir a Memoria
      </button>
    </form>

    <div class="flex justify-between items-center text-xs text-slate-400 px-1">
      <span>Objetos en RAM: <strong class="text-slate-200">{{ items.length }}</strong></span>
      <label class="flex items-center gap-2 cursor-pointer">
        <input v-model="soloPendientes" type="checkbox" class="accent-emerald-500" />
        Mostrar solo pendientes
      </label>
    </div>

    <ul class="space-y-2">
      <li
        v-for="item in itemsVisibles"
        :key="item.id"
        class="flex items-center justify-between p-3.5 rounded-xl border border-slate-800 bg-slate-900/60 hover:border-slate-700 transition"
      >
        <div class="flex items-center gap-3">
          <input
            type="checkbox"
            :checked="item.completada"
            @change="alternarCompletada(item)"
            class="w-4 h-4 accent-emerald-500 cursor-pointer"
          />
          <span :class="item.completada ? 'line-through text-slate-500' : 'text-slate-100'" class="text-sm font-medium">
            {{ item.titulo }}
          </span>
          <span
            :class="{
              'bg-rose-500/15 text-rose-400': item.prioridad === 'Alta',
              'bg-amber-500/15 text-amber-400': item.prioridad === 'Media',
              'bg-slate-700/50 text-slate-300': item.prioridad === 'Baja'
            }"
            class="text-[11px] px-2 py-0.5 rounded font-medium"
          >
            {{ item.prioridad }}
          </span>
        </div>
        <button @click="borrarItem(item)" class="text-xs text-slate-400 hover:text-rose-400 cursor-pointer px-2 py-1">
          Quitar
        </button>
      </li>
    </ul>
  </div>
</template>