<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue';

const DEMO = 'https://arnaucanet.github.io/request-journey/';

// Datos sacados de los escenarios de Request Journey. Latencias simuladas, no medidas.
const scenarios = [
  {
    id: 'primera-visita', label: 'Primera visita',
    metric: '456 ms', metricLabel: 'carga completa, con todas las cachés vacías',
    rows: [
      ['Navegador', 'Chrome', 'No conoce la IP', 'ok'],
      ['DNS', 'Route 53', 'Resuelve el nombre', 'ok'],
      ['CDN', 'CloudFront', 'MISS: pide el archivo a S3', 'ok'],
      ['Balanceador', 'ALB', 'Elige un servidor', 'ok'],
      ['API', 'EC2', 'Busca en la caché', 'ok'],
      ['Caché', 'Redis', 'MISS', 'ok'],
      ['Base de datos', 'RDS', '1 consulta', 'mark'],
    ],
  },
  {
    id: 'segunda-visita', label: 'Segunda visita',
    metric: '113 ms', metricLabel: 'la misma página, con las cachés llenas',
    rows: [
      ['Navegador', 'Chrome', 'IP y conexión guardadas; JS y CSS en disco', 'mark'],
      ['DNS', 'Route 53', 'No hace falta', 'skip'],
      ['CDN', 'CloudFront', 'HIT: sirve el HTML', 'mark'],
      ['Balanceador', 'ALB', 'Elige un servidor', 'ok'],
      ['API', 'EC2', 'Busca en la caché', 'ok'],
      ['Caché', 'Redis', 'HIT', 'mark'],
      ['Base de datos', 'RDS', 'No se entera', 'skip'],
    ],
  },
  {
    id: 'n-mas-1', label: 'Web lenta (N+1)',
    metric: '51 consultas', metricLabel: 'en una sola petición: 486 ms de espera. Con un JOIN, 1 consulta y 140 ms',
    rows: [
      ['Navegador', 'Chrome', 'GET /api/products', 'ok'],
      ['CDN', 'CloudFront', 'Pasa la API al balanceador', 'ok'],
      ['Balanceador', 'ALB', 'Elige un servidor', 'ok'],
      ['API', 'EC2', 'Un bucle pide las reseñas de cada producto', 'bad'],
      ['Caché', 'Redis', 'MISS, recién desplegado', 'ok'],
      ['Base de datos', 'RDS', '1 + 50 consultas', 'bad'],
    ],
  },
  {
    id: 'redis-caido', label: 'Se cae Redis',
    metric: '8 de 8', metricLabel: 'peticiones llegan a la base de datos. La web sigue viva, pero más lenta',
    rows: [
      ['Navegador', 'Chrome', 'Ráfaga de 8 peticiones', 'ok'],
      ['CDN', 'CloudFront', 'Pasa la API al balanceador', 'ok'],
      ['Balanceador', 'ALB', 'Reparte entre los servidores', 'ok'],
      ['API', 'EC2', 'Captura ECONNREFUSED y sigue', 'mark'],
      ['Caché', 'Redis', 'Caído', 'bad'],
      ['Base de datos', 'RDS', '8 consultas', 'bad'],
    ],
  },
];

const HEX = '32,3 57,17.5 57,46.5 32,61 7,46.5 7,17.5';
const STEP_MS = 340;

const currentId = ref(scenarios[0].id);
const scenario = computed(() => scenarios.find((s) => s.id === currentId.value));
// Sin JavaScript o con movimiento reducido se ve el recorrido entero desde el principio
const reached = ref(scenarios[0].rows.length);
const done = computed(() => reached.value >= scenario.value.rows.length);
let timer = null;

function play() {
  clearInterval(timer);
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
    reached.value = scenario.value.rows.length;
    return;
  }
  reached.value = 0;
  timer = setInterval(() => {
    reached.value += 1;
    if (done.value) clearInterval(timer);
  }, STEP_MS);
}

function pick(id) {
  currentId.value = id;
  play();
}

onMounted(play);
onBeforeUnmount(() => clearInterval(timer));
</script>

<template>
  <div class="trace rounded-[22px] p-5 sm:p-6">
    <div class="flex items-center justify-between gap-3 font-mono text-xs text-white/55">
      <span>GET https://tienda.example/</span>
      <button type="button" class="shrink-0 rounded px-1 underline decoration-white/25 underline-offset-4 hover:text-white" @click="play">Repetir</button>
    </div>

    <div class="mt-4 flex flex-wrap gap-1.5" role="group" aria-label="Escenario">
      <button v-for="s in scenarios" :key="s.id" type="button" :aria-pressed="currentId === s.id"
        class="rounded-full border px-3 py-1 text-sm transition-colors"
        :class="currentId === s.id ? 'border-[var(--accent)] bg-[var(--accent)] font-semibold text-[var(--on-accent)]' : 'border-white/12 text-white/70 hover:text-white'"
        @click="pick(s.id)">{{ s.label }}</button>
    </div>

    <ol class="hops mt-5">
      <li v-for="([name, service, state, kind], i) in scenario.rows" :key="scenario.id + name"
        class="hop" :class="[`is-${kind}`, i < reached ? 'is-reached' : '']">
        <svg viewBox="0 0 64 64" class="hex" aria-hidden="true"><polygon :points="HEX" stroke-width="6" stroke-linejoin="round" /></svg>
        <div class="min-w-0">
          <p class="text-sm font-semibold leading-tight text-white">{{ name }}</p>
          <p class="font-mono text-[11px] leading-tight text-white/45">{{ service }}</p>
        </div>
        <p class="state text-right text-[0.8rem] leading-snug">{{ state }}</p>
      </li>
    </ol>

    <div class="result mt-5 grid gap-x-4 gap-y-1 border-t border-white/10 pt-4 sm:grid-cols-[auto_1fr] sm:items-center" :class="done ? 'is-done' : ''" aria-live="polite">
      <p class="whitespace-nowrap text-3xl font-extrabold tracking-tight text-[var(--accent)]">{{ scenario.metric }}</p>
      <p class="text-sm leading-snug text-white/70">{{ scenario.metricLabel }}</p>
    </div>

    <div class="mt-4 flex flex-wrap items-center justify-between gap-x-4 gap-y-2">
      <a :href="`${DEMO}?escenario=${scenario.id}`" target="_blank" rel="noopener" class="inline-flex items-center gap-1.5 text-sm font-semibold underline decoration-[var(--accent)] underline-offset-4 hover:text-[var(--accent)]">
        Verlo en 3D
        <svg viewBox="0 0 24 24" class="h-3.5 w-3.5" fill="none" stroke="currentColor" stroke-width="2.2" aria-hidden="true"><path d="M7 17 17 7M8 7h9v9"/></svg>
      </a>
      <span class="text-xs text-white/45">Request Journey · latencias simuladas</span>
    </div>
  </div>
</template>

<style scoped>
.trace { background: var(--surface); border: 1px solid rgba(255,255,255,.1); box-shadow: 0 40px 80px -40px rgba(0,0,0,.8); }
.hop {
  position: relative;
  display: grid;
  grid-template-columns: 1.5rem minmax(6.25rem, 7.5rem) 1fr;
  align-items: center;
  gap: .75rem;
  min-height: 2.75rem;
  padding: .3rem 0;
  opacity: .28;
  transition: opacity .25s ease;
}
.hop.is-reached { opacity: 1; }
.hop.is-skip.is-reached { opacity: .45; }
/* Cable entre un equipo y el siguiente */
.hop:not(:last-child)::after {
  content: '';
  position: absolute;
  left: calc(.75rem - .75px);
  top: calc(50% + .8rem);
  height: calc(100% - 1.6rem);
  border-left: 1.5px dashed rgba(255,255,255,.2);
}
.hex { width: 1.5rem; height: 1.5rem; fill: #0F1211; stroke: rgba(255,255,255,.4); transition: fill .25s ease, stroke .25s ease; }
.state { color: rgba(255,255,255,.72); }
.is-reached.is-ok .hex { stroke: rgba(255,255,255,.75); }
.is-reached.is-mark .hex { stroke: var(--accent); fill: color-mix(in srgb, var(--accent) 35%, #0F1211); }
.is-reached.is-mark .state { color: var(--accent); }
.is-reached.is-bad .hex { stroke: var(--danger); fill: color-mix(in srgb, var(--danger) 35%, #0F1211); }
.is-reached.is-bad .state { color: var(--danger); }
.result { opacity: 0; transition: opacity .3s ease; }
.result.is-done { opacity: 1; }
</style>
