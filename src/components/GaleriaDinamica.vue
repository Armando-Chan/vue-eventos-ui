<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

interface Foto {
  id: number
  titulo: string
  categoria: string
  url: string
  likes: number
}

const emit = defineEmits<{
  (e: 'evento', tipo: string, detalle: string): void
}>()

const fotos = ref<Foto[]>([
  { id: 1, titulo: 'Arquitectura Minimalista', categoria: 'Diseño', url: 'https://images.unsplash.com/photo-1513694203232-719a280e022f?auto=format&fit=crop&w=900&q=80', likes: 12 },
  { id: 2, titulo: 'Espacio de Trabajo Nocturno', categoria: 'Setup', url: 'https://images.unsplash.com/photo-1498050108023-c5249f4df085?auto=format&fit=crop&w=900&q=80', likes: 28 },
  { id: 3, titulo: 'Montañas al Amanecer', categoria: 'Naturaleza', url: 'https://images.unsplash.com/photo-1464822759023-fed622ff2c3b?auto=format&fit=crop&w=900&q=80', likes: 45 },
  { id: 4, titulo: 'Geometría Urbana', categoria: 'Ciudad', url: 'https://images.unsplash.com/photo-1486406146926-c627a92ad1ab?auto=format&fit=crop&w=900&q=80', likes: 19 }
])

const indiceActivo = ref(0)
const modoHover = ref(false)
const lightboxAbierto = ref(false)

const seleccionar = (idx: number, origen: string) => {
  if (indiceActivo.value === idx) return
  indiceActivo.value = idx
  emit('evento', `Galería (${origen})`, `Vista: ${fotos.value[idx]!.titulo}`)
}

const navegar = (delta: number) => {
  const total = fotos.value.length
  indiceActivo.value = (indiceActivo.value + delta + total) % total
  emit('evento', 'Navegación Galería', `Imagen #${indiceActivo.value + 1}`)
}

const darLike = (foto: Foto) => {
  foto.likes++
  emit('evento', 'Me Gusta', `+1 like en "${foto.titulo}"`)
}

const manejarTeclado = (e: KeyboardEvent) => {
  if (e.key === 'ArrowRight') navegar(1)
  if (e.key === 'ArrowLeft') navegar(-1)
  if (e.key === 'Escape' && lightboxAbierto.value) lightboxAbierto.value = false
}

onMounted(() => window.addEventListener('keydown', manejarTeclado))
onUnmounted(() => window.removeEventListener('keydown', manejarTeclado))
</script>

<template>
  <div class="space-y-4">
    <div class="flex items-center justify-between bg-slate-900/70 p-3 rounded-xl border border-slate-800">
      <span class="text-xs text-slate-400">Usa las flechas <kbd class="px-1.5 py-0.5 bg-slate-800 rounded text-slate-200">←</kbd> <kbd class="px-1.5 py-0.5 bg-slate-800 rounded text-slate-200">→</kbd> del teclado o selecciona una miniatura.</span>
      <label class="flex items-center gap-2 text-sm text-slate-300 cursor-pointer select-none">
        <input v-model="modoHover" type="checkbox" class="accent-emerald-500" />
        Previsualizar al pasar el ratón (@mouseover)
      </label>
    </div>

    <div class="grid grid-cols-1 lg:grid-cols-3 gap-4">
      <div class="lg:col-span-2 relative rounded-xl overflow-hidden border border-slate-800 bg-slate-950 group">
        <img
          :src="fotos[indiceActivo]!.url"
          :alt="fotos[indiceActivo]!.titulo"
          @click="lightboxAbierto = true; emit('evento', 'Lightbox', 'Vista ampliada abierta')"
          class="w-full h-80 object-cover cursor-zoom-in transition duration-300 group-hover:scale-[1.01]"
        />
        <div class="p-4 bg-slate-900/90 flex items-center justify-between">
          <div>
            <span class="text-xs uppercase tracking-wider text-emerald-400 font-semibold">{{ fotos[indiceActivo]!.categoria }}</span>
            <h3 class="text-lg font-bold text-white">{{ fotos[indiceActivo]!.titulo }}</h3>
          </div>
          <div class="flex items-center gap-2">
            <button @click="darLike(fotos[indiceActivo]!)" class="px-3 py-1.5 rounded-lg bg-rose-500/15 text-rose-400 hover:bg-rose-500/25 text-sm font-medium cursor-pointer transition">
              ♥ {{ fotos[indiceActivo]!.likes }}
            </button>
            <button @click="navegar(-1)" class="px-3 py-1.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-slate-200 cursor-pointer">←</button>
            <button @click="navegar(1)" class="px-3 py-1.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-slate-200 cursor-pointer">→</button>
          </div>
        </div>
      </div>

      <div class="grid grid-cols-2 gap-3 content-start">
        <button
          v-for="(foto, idx) in fotos"
          :key="foto.id"
          @click="seleccionar(idx, 'Click')"
          @mouseover="modoHover && seleccionar(idx, 'Hover')"
          :class="idx === indiceActivo ? 'ring-2 ring-emerald-500 opacity-100' : 'opacity-60 hover:opacity-90'"
          class="relative rounded-xl overflow-hidden border border-slate-800 bg-slate-900 text-left transition cursor-pointer"
        >
          <img :src="foto.url" :alt="foto.titulo" class="w-full h-28 object-cover" />
          <div class="p-2 text-xs font-medium text-slate-200 truncate">{{ foto.titulo }}</div>
        </button>
      </div>
    </div>

    <div
      v-if="lightboxAbierto"
      @click.self="lightboxAbierto = false"
      class="fixed inset-0 z-50 bg-black/85 backdrop-blur-sm flex items-center justify-center p-6"
    >
      <div class="max-w-4xl w-full bg-slate-900 rounded-2xl overflow-hidden border border-slate-700">
        <img :src="fotos[indiceActivo]!.url" class="w-full max-h-[75vh] object-contain bg-black" />
        <div class="p-4 flex justify-between items-center">
          <span class="font-semibold text-white">{{ fotos[indiceActivo]!.titulo }}</span>
          <button @click="lightboxAbierto = false" class="px-4 py-1.5 bg-slate-800 hover:bg-slate-700 rounded-lg text-sm text-white cursor-pointer">
            Cerrar (Esc)
          </button>
        </div>
      </div>
    </div>
  </div>
</template>