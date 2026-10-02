<script setup>
import { ref, computed } from 'vue';

const scenarios = {
  validador: {
    label: 'Validador de documentos', short: 'Validador',
    steps: [
      'La plataforma publica el documento.',
      'RabbitMQ lo entrega al microservicio.',
      'El LLM lo analiza y verifica.',
      'La plataforma muestra el resultado.',
    ],
    link: { name: 'RabbitMQ', detail: 'Cola de mensajes', icon: 'queue' },
    info: [
      'Integración en Symfony con DDD hexagonal: publica el documento, recoge el resultado y lo muestra.',
      'Canal entre plataforma e IA: publicación de documentos y recepción de resultados.',
      'Desarrollado por un compañero. Mi parte: integrar su respuesta en el producto.',
    ],
  },
  chatbot: {
    label: 'Chatbot interno', short: 'Chatbot',
    steps: [
      'La plataforma añade la pregunta a la sesión.',
      'La envía por HTTP con el contexto.',
      'El LLM genera la respuesta.',
      'La plataforma la guarda y la muestra.',
    ],
    link: { name: 'API HTTP', detail: 'Petición y respuesta', icon: 'http' },
    info: [
      'Integración en Symfony, gestión de sesiones y componente de chat en Vue.js.',
      'Envía la pregunta con el contexto de la sesión y guarda la respuesta.',
      'Desarrollado por un compañero. Mi parte: integrar sus respuestas en el producto.',
    ],
  },
};

const current = ref('validador');
const selected = ref(0);
const scenario = computed(() => scenarios[current.value]);
const nodes = computed(() => [
  { name: 'Plataforma', detail: 'PHP y Symfony', mine: true, icon: 'platform' },
  { ...scenario.value.link, mine: false },
  { name: 'Servicio de IA', detail: 'LLM', mine: false, icon: 'ai' },
]);
const HEX = '32,2 58,17 58,47 32,62 6,47 6,17';
function pickScenario(key) { current.value = key; }
</script>

<template>
  <div class="demo rounded-[22px] p-5 sm:p-7">
    <div class="flex w-fit gap-1 rounded-full border border-white/12 p-1" role="tablist" aria-label="Producto">
      <button v-for="(s, key) in scenarios" :key="key" role="tab" :aria-selected="current === key"
        class="rounded-full px-3.5 py-1.5 text-sm transition-colors"
        :class="current === key ? 'bg-[var(--accent)] font-semibold text-[var(--on-accent)]' : 'text-white/65 hover:text-white'"
        @click="pickScenario(key)"><span class="sm:hidden">{{ s.short }}</span><span class="hidden sm:inline">{{ s.label }}</span></button>
    </div>

    <div class="relative mt-8 grid grid-cols-3 gap-2 sm:gap-6">
      <div class="rail pointer-events-none absolute left-[16.66%] right-[16.66%] top-[2.6rem]" aria-hidden="true"></div>
      <button v-for="(n, i) in nodes" :key="n.name + i" type="button" :aria-pressed="selected === i"
        class="node group relative flex flex-col items-center rounded-xl px-1 pb-2 pt-2 text-center"
        :class="selected === i ? 'is-selected' : ''" @click="selected = i">
        <div class="relative h-16 w-16 transition-transform duration-200 group-hover:-translate-y-0.5">
          <svg viewBox="0 0 64 64" class="absolute inset-0 h-full w-full" aria-hidden="true">
            <polygon :points="HEX"
              :fill="selected === i ? 'color-mix(in srgb, var(--accent) 18%, #0F1211)' : (n.mine ? 'color-mix(in srgb, var(--accent) 10%, #0F1211)' : '#0F1211')"
              :stroke="selected === i || n.mine ? 'var(--accent)' : 'rgba(255,255,255,.3)'" :stroke-width="selected === i ? 2.4 : (n.mine ? 1.8 : 1.4)" stroke-linejoin="round" />
          </svg>
          <svg viewBox="0 0 24 24" class="absolute left-1/2 top-1/2 h-6 w-6 -translate-x-1/2 -translate-y-1/2" :class="selected === i || n.mine ? 'text-[var(--accent)]' : 'text-white/85'" fill="none" stroke="currentColor" stroke-width="1.6" aria-hidden="true">
            <template v-if="n.icon==='platform'"><rect x="3" y="4" width="18" height="14" rx="2"/><path d="M3 8h18M8 21h8"/></template>
            <template v-else-if="n.icon==='queue'"><rect x="3" y="6" width="4" height="12" rx="1"/><rect x="10" y="6" width="4" height="12" rx="1"/><rect x="17" y="6" width="4" height="12" rx="1"/></template>
            <template v-else-if="n.icon==='http'"><path d="M4 9h14l-3-3M20 15H6l3 3"/></template>
            <template v-else><path d="M12 3v3M12 18v3M3 12h3M18 12h3M6 6l2 2M16 16l2 2M6 18l2-2M16 8l2-2"/><circle cx="12" cy="12" r="3.5"/></template>
          </svg>
        </div>
        <span class="mt-3 text-sm font-semibold leading-tight text-white sm:text-base">{{ n.name }}</span>
        <span class="text-xs text-white/55 sm:text-sm">{{ n.detail }}</span>
        <span v-if="n.mine" class="mt-1.5 rounded-full bg-[var(--accent)] px-2 py-0.5 text-[11px] font-semibold text-[var(--on-accent)]">Mi parte</span>
      </button>
    </div>

    <div class="mt-4 rounded-xl border border-white/10 bg-white/[0.03] p-4" aria-live="polite">
      <p class="text-sm font-semibold text-[var(--accent)]">{{ nodes[selected].name }}</p>
      <p class="mt-1 text-[0.95rem] leading-relaxed text-white/85">{{ scenario.info[selected] }}</p>
    </div>

    <ol class="mt-6 space-y-2 text-[0.95rem] text-white/75">
      <li v-for="(t, i) in scenario.steps" :key="t" class="flex gap-3">
        <span class="w-4 shrink-0 tabular-nums font-semibold text-[var(--accent)]">{{ i + 1 }}</span>
        <span>{{ t }}</span>
      </li>
    </ol>
  </div>
</template>

<style scoped>
.demo { background: var(--surface); border: 1px solid rgba(255,255,255,.1); box-shadow: 0 40px 80px -40px rgba(0,0,0,.8); }
.rail { height: 0; border-top: 1.5px dashed rgba(255,255,255,.22); }
.node { transition: background-color .2s ease; }
.node:hover { background: rgba(255,255,255,.03); }
.node.is-selected { background: rgba(255,255,255,.04); }
</style>
