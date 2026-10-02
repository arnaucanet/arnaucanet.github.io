<script setup>
import { ref, computed, onBeforeUnmount } from 'vue';

const scenarios = {
  validador: {
    label: 'Validador de documentos',
    action: 'Enviar un documento',
    steps: [
      'La plataforma publica el documento en la cola.',
      'RabbitMQ entrega el mensaje al microservicio de IA.',
      'El LLM analiza el documento y verifica sus datos.',
      'La plataforma recibe el resultado y lo muestra al usuario.',
    ],
    result: 'Documento verificado',
  },
  chatbot: {
    label: 'Chatbot interno',
    action: 'Enviar una pregunta',
    steps: [
      'La plataforma añade la pregunta a la sesión de conversación.',
      'RabbitMQ entrega el mensaje con el historial al microservicio.',
      'El LLM genera la respuesta usando la memoria de la conversación.',
      'La plataforma guarda la respuesta en la sesión y la muestra.',
    ],
    result: 'Respuesta entregada',
  },
};

const current = ref('validador');
const step = ref(-1); // -1 idle, 0..3 running, 4 done
let timers = [];

const scenario = computed(() => scenarios[current.value]);
const running = computed(() => step.value >= 0 && step.value < 4);

// dot position: 0 platform, 1 queue, 2 ai
const dotAt = computed(() => [0, 1, 2, 0][Math.min(Math.max(step.value, 0), 3)]);
const dotVisible = computed(() => step.value >= 0 && step.value < 4);

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

const nodes = [
  { name: 'Plataforma', detail: 'PHP y Symfony', mine: true },
  { name: 'RabbitMQ', detail: 'Cola de mensajes', mine: false },
  { name: 'Microservicio de IA', detail: 'LLM', mine: false },
];
const active = (i) => step.value >= 0 && step.value < 4 && dotAt.value === i;
</script>

<template>
  <div class="rounded-2xl border border-line bg-surface p-5 sm:p-7">
    <div class="flex flex-wrap items-center justify-between gap-3">
      <div class="flex gap-1 rounded-full border border-line p-1" role="tablist" aria-label="Producto">
        <button v-for="(s, key) in scenarios" :key="key" role="tab" :aria-selected="current === key"
          class="rounded-full px-3.5 py-1.5 text-sm transition-colors"
          :class="current === key ? 'bg-ink text-bg' : 'text-muted hover:text-ink'"
          @click="pick(key)">{{ s.label }}</button>
      </div>
      <button class="rounded-full bg-accent px-4 py-2 text-sm font-semibold text-bg disabled:opacity-60"
        :disabled="running" @click="run">{{ running ? 'En camino…' : scenario.action }}</button>
    </div>

    <div class="relative mt-8 grid grid-cols-3 gap-3 sm:gap-6">
      <!-- rail -->
      <div class="pointer-events-none absolute left-[16.66%] right-[16.66%] top-7 h-px bg-line" aria-hidden="true"></div>
      <div v-if="dotVisible" aria-hidden="true"
        class="dot pointer-events-none absolute top-7 h-3.5 w-3.5 -translate-x-1/2 -translate-y-1/2 rounded-full bg-signal"
        :style="{ left: `${16.66 + dotAt * 33.33}%` }"></div>

      <div v-for="(n, i) in nodes" :key="n.name" class="relative flex flex-col items-center text-center">
        <div class="grid h-14 w-14 place-items-center rounded-xl border-2 bg-surface transition-colors"
          :class="[n.mine ? 'border-accent' : 'border-line', active(i) ? 'bg-signal/15' : '']">
          <svg v-if="i===0" viewBox="0 0 24 24" class="h-6 w-6" fill="none" stroke="currentColor" stroke-width="1.6"><rect x="3" y="4" width="18" height="14" rx="2"/><path d="M3 8h18M8 21h8"/></svg>
          <svg v-else-if="i===1" viewBox="0 0 24 24" class="h-6 w-6" fill="none" stroke="currentColor" stroke-width="1.6"><rect x="3" y="6" width="4" height="12" rx="1"/><rect x="10" y="6" width="4" height="12" rx="1"/><rect x="17" y="6" width="4" height="12" rx="1"/></svg>
          <svg v-else viewBox="0 0 24 24" class="h-6 w-6" fill="none" stroke="currentColor" stroke-width="1.6"><path d="M12 3v3M12 18v3M3 12h3M18 12h3M6 6l2 2M16 16l2 2M6 18l2-2M16 8l2-2"/><circle cx="12" cy="12" r="3.5"/></svg>
        </div>
        <p class="mt-3 text-sm font-semibold leading-tight sm:text-base">{{ n.name }}</p>
        <p class="text-xs text-muted sm:text-sm">{{ n.detail }}</p>
        <p v-if="n.mine" class="mt-1 text-xs font-medium text-accent">Mi parte</p>
      </div>
    </div>

    <ol class="mt-8 space-y-1.5 text-[0.95rem]" aria-live="polite">
      <li v-for="(t, i) in scenario.steps" :key="t" class="flex gap-3 transition-colors"
        :class="step >= i ? 'text-ink' : 'text-muted/60'">
        <span class="w-4 shrink-0 tabular-nums" :class="step === i ? 'text-signal font-semibold' : ''">{{ i + 1 }}</span>
        <span>{{ t }}</span>
      </li>
    </ol>
    <p class="mt-4 h-6 text-sm font-semibold text-accent" aria-live="polite">
      <span v-if="step === 4">✓ {{ scenario.result }}</span>
    </p>
  </div>
</template>

<style scoped>
.dot { transition: left 0.9s cubic-bezier(0.6, 0, 0.3, 1); box-shadow: 0 0 0 5px color-mix(in srgb, var(--signal) 25%, transparent); }
</style>
