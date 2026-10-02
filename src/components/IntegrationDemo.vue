<script setup>
import { ref, computed, onBeforeUnmount } from 'vue';

const scenarios = {
  validador: {
    label: 'Validador de documentos', short: 'Validador',
    action: 'Enviar un documento',
    steps: [
      'La plataforma publica el documento en la cola.',
      'RabbitMQ entrega el mensaje al microservicio de IA.',
      'El LLM analiza el documento y verifica sus datos.',
      'La plataforma recibe el resultado y lo muestra al usuario.',
    ],
    result: 'Documento verificado',
    link: { name: 'RabbitMQ', detail: 'Cola de mensajes', icon: 'queue' },
  },
  chatbot: {
    label: 'Chatbot interno', short: 'Chatbot',
    action: 'Enviar una pregunta',
    steps: [
      'La plataforma añade la pregunta a la sesión de conversación.',
      'La plataforma llama por HTTP al microservicio con la pregunta y el historial.',
      'El LLM genera la respuesta usando la memoria de la conversación.',
      'La plataforma guarda la respuesta en la sesión y la muestra.',
    ],
    result: 'Respuesta entregada',
    link: { name: 'API HTTP', detail: 'Petición y respuesta', icon: 'http' },
  },
};

const current = ref('validador');
const step = ref(-1); // -1 idle, 0..3 running, 4 done
let timers = [];

const scenario = computed(() => scenarios[current.value]);
const running = computed(() => step.value >= 0 && step.value < 4);
const dotAt = computed(() => [0, 1, 2, 0][Math.min(Math.max(step.value, 0), 3)]);

function clear() { timers.forEach(clearTimeout); timers = []; }
function run() {
  clear();
  const reduce = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  const gap = reduce ? 350 : 1100;
  step.value = 0;
  [1, 2, 3, 4].forEach((s, i) => timers.push(setTimeout(() => (step.value = s), gap * (i + 1))));
}
function pick(key) { clear(); current.value = key; step.value = -1; }
onBeforeUnmount(clear);

const nodes = computed(() => [
  { name: 'Plataforma', detail: 'PHP y Symfony', mine: true, icon: 'platform' },
  { ...scenario.value.link, mine: false },
  { name: 'Microservicio de IA', detail: 'LLM', mine: false, icon: 'ai' },
]);
const active = (i) => running.value && dotAt.value === i;
const HEX = '32,2 58,17 58,47 32,62 6,47 6,17';
</script>

<template>
  <div class="demo rounded-[22px] p-5 sm:p-7">
    <div class="flex flex-wrap items-center justify-between gap-3">
      <div class="flex gap-1 rounded-full border border-white/20 p-1" role="tablist" aria-label="Producto">
        <button v-for="(s, key) in scenarios" :key="key" role="tab" :aria-selected="current === key"
          class="rounded-full px-3.5 py-1.5 text-sm transition-colors"
          :class="current === key ? 'bg-white text-[var(--blue)] font-semibold' : 'text-white/70 hover:text-white'"
          @click="pick(key)"><span class="sm:hidden">{{ s.short }}</span><span class="hidden sm:inline">{{ s.label }}</span></button>
      </div>
      <button class="rounded-full bg-[var(--amber)] px-4 py-2 text-sm font-semibold text-[#221704] transition-opacity disabled:opacity-60"
        :disabled="running" @click="run">{{ running ? 'En camino…' : scenario.action }}</button>
    </div>

    <div class="relative mt-9 grid grid-cols-3 gap-2 sm:gap-6">
      <div class="rail pointer-events-none absolute left-[16.66%] right-[16.66%] top-8" aria-hidden="true"></div>
      <div v-if="running" aria-hidden="true"
        class="dot pointer-events-none absolute top-8 h-3.5 w-3.5 -translate-x-1/2 -translate-y-1/2 rounded-full bg-[var(--amber)]"
        :style="{ left: `${16.66 + dotAt * 33.33}%` }"></div>

      <div v-for="(n, i) in nodes" :key="n.name + i" class="relative flex flex-col items-center text-center">
        <div class="relative h-16 w-16">
          <svg viewBox="0 0 64 64" class="absolute inset-0 h-full w-full" aria-hidden="true">
            <polygon :points="HEX" class="transition-[fill] duration-300"
              :fill="active(i) ? 'rgba(255,190,80,.28)' : (n.mine ? 'rgba(255,255,255,.14)' : 'var(--blue)')"
              :stroke="n.mine ? '#fff' : 'rgba(255,255,255,.45)'" :stroke-width="n.mine ? 2 : 1.4"
              stroke-linejoin="round" />
          </svg>
          <svg viewBox="0 0 24 24" class="absolute left-1/2 top-1/2 h-6 w-6 -translate-x-1/2 -translate-y-1/2 text-white" fill="none" stroke="currentColor" stroke-width="1.6" aria-hidden="true">
            <template v-if="n.icon==='platform'"><rect x="3" y="4" width="18" height="14" rx="2"/><path d="M3 8h18M8 21h8"/></template>
            <template v-else-if="n.icon==='queue'"><rect x="3" y="6" width="4" height="12" rx="1"/><rect x="10" y="6" width="4" height="12" rx="1"/><rect x="17" y="6" width="4" height="12" rx="1"/></template>
            <template v-else-if="n.icon==='http'"><path d="M4 9h14l-3-3M20 15H6l3 3"/></template>
            <template v-else><path d="M12 3v3M12 18v3M3 12h3M18 12h3M6 6l2 2M16 16l2 2M6 18l2-2M16 8l2-2"/><circle cx="12" cy="12" r="3.5"/></template>
          </svg>
        </div>
        <p class="mt-3 text-sm font-semibold leading-tight text-white sm:text-base">{{ n.name }}</p>
        <p class="text-xs text-white/60 sm:text-sm">{{ n.detail }}</p>
        <p v-if="n.mine" class="mt-1.5 rounded-full bg-white px-2 py-0.5 text-[11px] font-semibold text-[var(--blue)]">Mi parte</p>
      </div>
    </div>

    <ol class="mt-8 space-y-2 border-t border-white/15 pt-6 text-[0.95rem]" aria-live="polite">
      <li v-for="(t, i) in scenario.steps" :key="t" class="flex gap-3 transition-colors duration-300"
        :class="step >= i ? 'text-white' : 'text-white/45'">
        <span class="w-4 shrink-0 tabular-nums" :class="step === i ? 'font-bold text-[var(--amber)]' : ''">{{ i + 1 }}</span>
        <span>{{ t }}</span>
      </li>
    </ol>
    <p class="mt-4 h-6 text-sm font-semibold text-[var(--amber)]" aria-live="polite">
      <span v-if="step === 4">✓ {{ scenario.result }}</span>
    </p>
  </div>
</template>

<style scoped>
.demo { background: color-mix(in srgb, var(--blue) 80%, #000 20%); border: 1px solid rgba(255,255,255,.18); box-shadow: 0 30px 60px -30px rgba(0,0,20,.55); }
.rail { height: 0; border-top: 1.5px dashed rgba(255,255,255,.35); }
.dot { transition: left 0.9s cubic-bezier(0.6, 0, 0.3, 1); box-shadow: 0 0 0 6px rgba(255,190,80,.25), 0 0 18px rgba(255,190,80,.7); }
</style>
