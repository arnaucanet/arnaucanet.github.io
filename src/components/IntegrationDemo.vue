<script setup>
import { ref, computed } from 'vue';

const scenarios = {
  validador: {
    label: 'Validador de documentos', short: 'Validador',
    steps: [
      'La plataforma publica el documento en la cola.',
      'RabbitMQ entrega el mensaje al microservicio de IA.',
      'El LLM analiza el documento y verifica sus datos.',
      'La plataforma recibe el resultado y lo muestra al usuario.',
    ],
    link: { name: 'RabbitMQ', detail: 'Cola de mensajes', icon: 'queue' },
  },
  chatbot: {
    label: 'Chatbot interno', short: 'Chatbot',
    steps: [
      'La plataforma añade la pregunta a la sesión de conversación.',
      'La plataforma llama por HTTP al microservicio con la pregunta y el contexto de la sesión.',
      'El microservicio genera la respuesta con el LLM.',
      'La plataforma guarda la respuesta en la sesión y la muestra.',
    ],
    link: { name: 'API HTTP', detail: 'Petición y respuesta', icon: 'http' },
  },
};

const current = ref('validador');
const scenario = computed(() => scenarios[current.value]);
const nodes = computed(() => [
  { name: 'Plataforma', detail: 'PHP y Symfony', mine: true, icon: 'platform' },
  { ...scenario.value.link, mine: false },
  { name: 'Microservicio de IA', detail: 'LLM', mine: false, icon: 'ai' },
]);
const HEX = '32,2 58,17 58,47 32,62 6,47 6,17';
</script>

<template>
  <div class="demo rounded-[22px] p-5 sm:p-7">
    <div class="flex w-fit gap-1 rounded-full border border-white/12 p-1" role="tablist" aria-label="Producto">
      <button v-for="(s, key) in scenarios" :key="key" role="tab" :aria-selected="current === key"
        class="rounded-full px-3.5 py-1.5 text-sm transition-colors"
        :class="current === key ? 'bg-[var(--accent)] font-semibold text-[var(--on-accent)]' : 'text-white/65 hover:text-white'"
        @click="current = key"><span class="sm:hidden">{{ s.short }}</span><span class="hidden sm:inline">{{ s.label }}</span></button>
    </div>

    <div class="relative mt-9 grid grid-cols-3 gap-2 sm:gap-6">
      <div class="rail pointer-events-none absolute left-[16.66%] right-[16.66%] top-8" aria-hidden="true"></div>
      <div v-for="(n, i) in nodes" :key="n.name + i" class="relative flex flex-col items-center text-center">
        <div class="relative h-16 w-16">
          <svg viewBox="0 0 64 64" class="absolute inset-0 h-full w-full" aria-hidden="true">
            <polygon :points="HEX" :fill="n.mine ? 'rgba(242,181,68,.14)' : '#141B19'"
              :stroke="n.mine ? 'var(--accent)' : 'rgba(255,255,255,.3)'" :stroke-width="n.mine ? 2 : 1.4" stroke-linejoin="round" />
          </svg>
          <svg viewBox="0 0 24 24" class="absolute left-1/2 top-1/2 h-6 w-6 -translate-x-1/2 -translate-y-1/2" :class="n.mine ? 'text-[var(--accent)]' : 'text-white/85'" fill="none" stroke="currentColor" stroke-width="1.6" aria-hidden="true">
            <template v-if="n.icon==='platform'"><rect x="3" y="4" width="18" height="14" rx="2"/><path d="M3 8h18M8 21h8"/></template>
            <template v-else-if="n.icon==='queue'"><rect x="3" y="6" width="4" height="12" rx="1"/><rect x="10" y="6" width="4" height="12" rx="1"/><rect x="17" y="6" width="4" height="12" rx="1"/></template>
            <template v-else-if="n.icon==='http'"><path d="M4 9h14l-3-3M20 15H6l3 3"/></template>
            <template v-else><path d="M12 3v3M12 18v3M3 12h3M18 12h3M6 6l2 2M16 16l2 2M6 18l2-2M16 8l2-2"/><circle cx="12" cy="12" r="3.5"/></template>
          </svg>
        </div>
        <p class="mt-3 text-sm font-semibold leading-tight text-white sm:text-base">{{ n.name }}</p>
        <p class="text-xs text-white/55 sm:text-sm">{{ n.detail }}</p>
        <p v-if="n.mine" class="mt-1.5 rounded-full bg-[var(--accent)] px-2 py-0.5 text-[11px] font-semibold text-[var(--on-accent)]">Mi parte</p>
      </div>
    </div>

    <ol class="mt-8 space-y-2 border-t border-white/10 pt-6 text-[0.95rem] text-white/85">
      <li v-for="(t, i) in scenario.steps" :key="t" class="flex gap-3">
        <span class="w-4 shrink-0 tabular-nums font-semibold text-[var(--accent)]">{{ i + 1 }}</span>
        <span>{{ t }}</span>
      </li>
    </ol>
  </div>
</template>

<style scoped>
.demo { background: rgba(22, 30, 28, .85); border: 1px solid rgba(255,255,255,.1); box-shadow: 0 40px 80px -40px rgba(0,0,0,.8); backdrop-filter: blur(4px); }
.rail { height: 0; border-top: 1.5px dashed rgba(255,255,255,.22); }
</style>
