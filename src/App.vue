<script setup lang="ts">
import { ref } from 'vue'
import TablaEnVivo from './components/TablaEnVivo.vue'
import GaleriaDinamica from './components/GaleriaDinamica.vue'
import ListaMemoria from './components/ListaMemoria.vue'

interface RegistroEvento {
  id: number
  hora: string
  tipo: string
  detalle: string
}

const pestanaActiva = ref<'tabla' | 'galeria' | 'memoria'>('tabla')
const totalEventos = ref(0)
const contadorManual = ref(0)
const paso = ref(1)
const historial = ref<RegistroEvento[]>([])

const registrarEvento = (tipo: string, detalle: string) => {
  totalEventos.value++
  historial.value.unshift({
    id: totalEventos.value,
    hora: new Date().toLocaleTimeString(),
    tipo,
    detalle
  })
  if (historial.value.length > 5) historial.value.pop()
}

const cambiarPestana = (nueva: 'tabla' | 'galeria' | 'memoria') => {
  pestanaActiva.value = nueva
  registrarEvento('Navegación', `Pestaña: ${nueva.toUpperCase()}`)
}

const modificarContador = (delta: number) => {
  contadorManual.value += delta
  registrarEvento('Contador Manual', `${delta > 0 ? '+' : ''}${delta} (Total: ${contadorManual.value})`)
}

const reiniciarTelemetria = () => {
  totalEventos.value = 0
  contadorManual.value = 0
  historial.value = []
}
</script>

<template>
  <div class="min-h-screen bg-slate-950 text-slate-100 p-4 md:p-8 font-sans">
    <div class="max-w-6xl mx-auto space-y-6">
      <header class="bg-slate-900 border border-slate-800 rounded-2xl p-5 flex flex-wrap items-center justify-between gap-4">
        <div>
          <h1 class="text-xl font-bold text-white tracking-tight">Vue 3 • Panel Interactivo de Eventos</h1>
          <p class="text-xs text-slate-400">Composition API + TypeScript + Tailwind CSS</p>
        </div>

        <div class="flex flex-wrap items-center gap-3">
          <div class="flex items-center gap-2 bg-slate-950 border border-slate-800 rounded-xl px-3 py-2">
            <span class="text-xs text-slate-400">Contador:</span>
            <button @click="modificarContador(-paso)" class="w-7 h-7 rounded bg-slate-800 hover:bg-slate-700 font-bold cursor-pointer">-</button>
            <span class="font-mono font-bold text-emerald-400 min-w-[2.5rem] text-center">{{ contadorManual }}</span>
            <button @click="modificarContador(paso)" class="w-7 h-7 rounded bg-slate-800 hover:bg-slate-700 font-bold cursor-pointer">+</button>
            <select v-model.number="paso" class="bg-slate-900 border border-slate-700 rounded px-1.5 py-1 text-xs text-slate-300">
              <option :value="1">x1</option>
              <option :value="5">x5</option>
              <option :value="10">x10</option>
            </select>
          </div>

          <div class="bg-emerald-500/10 border border-emerald-500/30 rounded-xl px-4 py-2 text-center">
            <div class="text-[11px] uppercase tracking-wider text-emerald-400 font-semibold">Eventos Capturados</div>
            <div class="text-lg font-mono font-bold text-white">{{ totalEventos }}</div>
          </div>

          <button @click="reiniciarTelemetria" class="text-xs text-slate-400 hover:text-white px-3 py-2 rounded-lg border border-slate-800 hover:bg-slate-800 cursor-pointer">
            Reiniciar
          </button>
        </div>
      </header>

      <div v-if="historial.length > 0" class="flex flex-wrap items-center gap-2 text-xs bg-slate-900/50 border border-slate-800/80 rounded-xl p-3">
        <span class="text-slate-500 font-semibold uppercase">Últimos eventos:</span>
        <span v-for="ev in historial" :key="ev.id" class="bg-slate-800/90 text-slate-300 px-2.5 py-1 rounded-md">
          <strong class="text-emerald-400">[{{ ev.tipo }}]</strong> {{ ev.detalle }}
        </span>
      </div>

      <nav class="flex gap-2 border-b border-slate-800 pb-3">
        <button
          @click="cambiarPestana('tabla')"
          :class="pestanaActiva === 'tabla' ? 'bg-emerald-600 text-white' : 'bg-slate-900 text-slate-400 hover:text-white'"
          class="px-4 py-2 rounded-lg text-sm font-medium transition cursor-pointer"
        >
          1. Tabla en Vivo
        </button>
        <button
          @click="cambiarPestana('galeria')"
          :class="pestanaActiva === 'galeria' ? 'bg-emerald-600 text-white' : 'bg-slate-900 text-slate-400 hover:text-white'"
          class="px-4 py-2 rounded-lg text-sm font-medium transition cursor-pointer"
        >
          2. Galería Dinámica
        </button>
        <button
          @click="cambiarPestana('memoria')"
          :class="pestanaActiva === 'memoria' ? 'bg-emerald-600 text-white' : 'bg-slate-900 text-slate-400 hover:text-white'"
          class="px-4 py-2 rounded-lg text-sm font-medium transition cursor-pointer"
        >
          3. Lista en Memoria
        </button>
      </nav>

      <main>
        <TablaEnVivo v-if="pestanaActiva === 'tabla'" @evento="registrarEvento" />
        <GaleriaDinamica v-else-if="pestanaActiva === 'galeria'" @evento="registrarEvento" />
        <ListaMemoria v-else @evento="registrarEvento" />
      </main>
    </div>
  </div>
</template>