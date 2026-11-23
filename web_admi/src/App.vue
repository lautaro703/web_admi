<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue'
import * as THREE from 'three'
// @ts-expect-error vanta no incluye tipos oficiales de TypeScript
import NET from 'vanta/dist/vanta.net.min'

interface Tema {
  subtitulo: string
  detalle: string
}

interface Modulo {
  id: string
  etiqueta: string
  titulo: string
  descripcion: string
  temas: Tema[]
  materiales: string[]
}

interface Filmina {
  imagen: string
  titulo: string
  descripcion: string
}

const menuAbierto = ref(false)
const seccionActiva = ref('portada')
const filminaActual = ref(0)
const progresoScroll = ref(0)
const vantaRef = ref<HTMLElement | null>(null)
let vantaEffect: { destroy: () => void } | null = null

const modulos: Modulo[] = [
  {
    id: 'mod-1',
    etiqueta: 'Módulo I',
    titulo: 'Fundamentos de Gestión y Rol Profesional',
    descripcion:
      'El pilar de la administración moderna: perfil del asistente, organización del trabajo y estructura de la empresa.',
    temas: [
      {
        subtitulo: 'El Asistente Administrativo',
        detalle: 'Funciones operativas, atención, archivo y coordinación entre áreas.',
      },
      {
        subtitulo: 'Circuitos y Organización',
        detalle: 'Flujo de información interna y externa en compras, ventas, pagos y cobranzas.',
      },
    ],
    materiales: ['Guía teórica M1 - Asistente Administrativo', 'Actividad práctica inicial (A-1)'],
  },
  {
    id: 'mod-2',
    etiqueta: 'Módulo II',
    titulo: 'Documentación Comercial y Facturación Electrónica',
    descripcion:
      'Identificación, emisión y control de comprobantes respaldatorios y facturación electrónica.',
    temas: [
      {
        subtitulo: 'Facturas Electrónicas',
        detalle: 'Tipos de comprobantes (A, B, C), requisitos fiscales, CAE y validación.',
      },
      {
        subtitulo: 'Comprobantes de Respaldo',
        detalle: 'Remitos, órdenes de compra, notas de débito, notas de crédito y recibos.',
      },
    ],
    materiales: [
      'Manual de Facturas Electrónicas',
      'Guía M-2 y Ejercitación A-M2',
      'Cheque / Sistema de información',
    ],
  },
  {
    id: 'mod-3',
    etiqueta: 'Módulo III',
    titulo: 'Cheques, Cuentas Contables y Constancias de CUIT',
    descripcion:
      'Manejo de cheques, clasificación de cuentas contables y lectura de constancias fiscales.',
    temas: [
      {
        subtitulo: 'Cheques y Operatoria',
        detalle: 'Tipos de cheques, emisión, endosos, plazos y registro de información.',
      },
      {
        subtitulo: 'Cuentas Contables y CUIT',
        detalle: 'Clasificación de cuentas y verificación de Constancias de Inscripción de CUIT.',
      },
    ],
    materiales: [
      'Guía práctica de Cheques y Anexo 2025',
      'Planilla de Cuentas Contables (Ejemplo)',
      'Constancias de CUIT y Actividad práctica Cierre',
    ],
  },
  {
    id: 'mod-4',
    etiqueta: 'Módulo IV',
    titulo: 'Higiene y Seguridad en el Trabajo',
    descripcion:
      'Prevención de riesgos, ergonomía en el puesto administrativo y seguridad laboral.',
    temas: [
      {
        subtitulo: 'Ergonomía y Prevención',
        detalle: 'Postura, iluminación, pausas y orden en el espacio de trabajo.',
      },
      {
        subtitulo: 'Seguridad Institucional',
        detalle: 'Medidas preventivas, señalética y protocolos ante emergencias.',
      },
    ],
    materiales: ['Manual M-4: Higiene y Seguridad en el trabajo'],
  },
  {
    id: 'mod-5-6',
    etiqueta: '2do Trayecto · Módulos 5 y 6',
    titulo: 'Administración Contable y Ejercitación Práctica',
    descripcion:
      'Aplicación práctica de registraciones contables, actividades y casos de empresas trabajados en clase.',
    temas: [
      {
        subtitulo: 'Registración y Actividades',
        detalle: 'Resolución de tareas, cierre y ejercitación extra de administración contable.',
      },
      {
        subtitulo: 'Casos Prácticos',
        detalle: 'Análisis y práctica sobre "Empresa Azul" y "Estrella Fugaz".',
      },
    ],
    materiales: [
      'Cuadernillo M 5 y 6 - Administración Contable',
      'Casos: Empresa Azul y Estrella Fugaz',
      'Actividades Adm Contable, Tarea y Ejercitación extra',
    ],
  },
]

// Cargá acá las imágenes cuando estén listas en public/img/
const filminas = ref<Filmina[]>([])

const tieneFilminas = computed(() => filminas.value.length > 0)
const filminaActiva = computed(() => filminas.value[filminaActual.value])

function irASeccion(id: string) {
  seccionActiva.value = id
  menuAbierto.value = false
  const elemento = document.getElementById(id)
  if (elemento) {
    elemento.scrollIntoView({ behavior: 'smooth' })
  }
}

function volverArriba() {
  seccionActiva.value = 'portada'
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

function filminaSiguiente() {
  if (!tieneFilminas.value) return
  filminaActual.value = (filminaActual.value + 1) % filminas.value.length
}

function filminaAnterior() {
  if (!tieneFilminas.value) return
  filminaActual.value = (filminaActual.value - 1 + filminas.value.length) % filminas.value.length
}

function actualizarScroll() {
  const scrollTop = window.scrollY
  const alturaTotal = document.documentElement.scrollHeight - window.innerHeight
  progresoScroll.value = alturaTotal > 0 ? Math.min(100, (scrollTop / alturaTotal) * 100) : 0

  const ids = [...modulos.map((m) => m.id), 'trabajo-practico']
  for (const id of ids) {
    const el = document.getElementById(id)
    if (el) {
      const rect = el.getBoundingClientRect()
      if (rect.top <= 150 && rect.bottom >= 150) {
        seccionActiva.value = id
        return
      }
    }
  }
  if (scrollTop < 120) {
    seccionActiva.value = 'portada'
  }
}

onMounted(() => {
  window.addEventListener('scroll', actualizarScroll, { passive: true })

  if (vantaRef.value) {
    vantaEffect = NET({
      el: vantaRef.value,
      THREE,
      color: 0x1a2a3a,
      backgroundColor: 0xfcfaf7,
      points: 10.0,
      maxDistance: 24.0,
      spacing: 16.0,
      showDots: false,
      speed: 0.75,
    })
  }
})

onUnmounted(() => {
  window.removeEventListener('scroll', actualizarScroll)
  if (vantaEffect) {
    vantaEffect.destroy()
  }
})
</script>

<template>
  <div class="pagina-administrativa">
    <div ref="vantaRef" class="vanta-background"></div>

    <div class="contenido-pagina">
      <div class="barra-progreso" :style="{ width: `${progresoScroll}%` }"></div>

      <header class="encabezado">
        <div class="encabezado-limitado">
          <button type="button" class="marca" @click="volverArriba">
            <span class="logo-box" aria-hidden="true">
              <svg viewBox="0 0 36 36" width="24" height="24" fill="none">
                <path
                  d="M5 10.5C5 9.12 6.12 8 7.5 8H14L16.5 11H28.5C29.88 11 31 12.12 31 13.5V25.5C31 26.88 29.88 28 28.5 28H7.5C6.12 28 5 26.88 5 25.5V10.5Z"
                  stroke="currentColor"
                  stroke-width="1.8"
                  stroke-linejoin="round"
                />
                <path
                  d="M11 22L16 17.5L19.5 20.5L25 15"
                  stroke="#c87d3b"
                  stroke-width="2"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                />
                <circle cx="25" cy="15" r="2" fill="#c87d3b" />
              </svg>
            </span>
            <span class="marca-textos">
              <span class="marca-titulo">Auxiliar en Administración</span>
              <span class="marca-subtitulo">Gestión Administrativa · Contenidos del Curso</span>
            </span>
          </button>

          <button
            class="boton-menu"
            type="button"
            :aria-expanded="menuAbierto"
            aria-label="Abrir menú principal"
            @click="menuAbierto = !menuAbierto"
          >
            <span class="icono-hamburguesa" :class="{ abierto: menuAbierto }"></span>
          </button>

          <nav class="navegacion" :class="{ abierta: menuAbierto }">
            <button
              v-for="modulo in modulos"
              :key="modulo.id"
              type="button"
              class="nav-item"
              :class="{ activo: seccionActiva === modulo.id }"
              @click="irASeccion(modulo.id)"
            >
              {{ modulo.etiqueta }}
            </button>
            <button
              type="button"
              class="nav-item nav-tp"
              :class="{ activo: seccionActiva === 'trabajo-practico' }"
              @click="irASeccion('trabajo-practico')"
            >
              Trabajo Práctico
            </button>
          </nav>
        </div>
      </header>

      <section id="portada" class="portada-hero">
        <div class="hero-contenido">
          <span class="hero-kicker">Trayecto Formativo · Auxiliar Administrativo</span>
          <h2>Organización administrativa, comprobantes y práctica contable</h2>
          <p>
            Recorré los temas vistos en clase por sección y accedé al espacio del trabajo
            práctico al final de la página.
          </p>
          <div class="hero-acciones">
            <button type="button" class="btn-hero" @click="irASeccion('mod-1')">
              Ir a los contenidos
            </button>
            <button type="button" class="btn-hero-suave" @click="irASeccion('trabajo-practico')">
              Sección Trabajo Práctico
            </button>
          </div>
        </div>
      </section>

      <main class="cuerpo-principal">
        <article
          v-for="(modulo, index) in modulos"
          :key="modulo.id"
          :id="modulo.id"
          class="seccion-modulo"
        >
          <div class="modulo-header">
            <div class="modulo-meta-fila">
              <span class="modulo-label">{{ modulo.etiqueta }}</span>
              <span class="modulo-indice">0{{ index + 1 }}</span>
            </div>
            <h3>{{ modulo.titulo }}</h3>
            <p class="modulo-resumen">{{ modulo.descripcion }}</p>
          </div>

          <div class="bloque-temas">
            <div v-for="(tema, idx) in modulo.temas" :key="idx" class="tarjeta-tema">
              <h4>{{ tema.subtitulo }}</h4>
              <p>{{ tema.detalle }}</p>
            </div>
          </div>

          <aside v-if="modulo.materiales.length" class="caja-material">
            <strong>Material trabajado en el curso</strong>
            <ul>
              <li v-for="(mat, i) in modulo.materiales" :key="i">{{ mat }}</li>
            </ul>
          </aside>
        </article>

        <section id="trabajo-practico" class="seccion-tp">
          <div class="tp-encabezado">
            <span class="tp-tag">Trabajo Práctico</span>
            <h3>Filminas de Presentación</h3>
            <p>Espacio reservado para cargar las imágenes del trabajo práctico más adelante.</p>
          </div>

          <template v-if="tieneFilminas && filminaActiva">
            <div class="visor-multimedia">
              <button
                type="button"
                class="ctrl-flecha"
                aria-label="Filmina anterior"
                @click="filminaAnterior"
              >
                <span>‹</span>
              </button>

              <figure class="filmina-contenedor">
                <div class="imagen-marco">
                  <img :src="filminaActiva.imagen" :alt="filminaActiva.titulo" />
                </div>
                <figcaption class="filmina-pie">
                  <div class="pie-texto">
                    <span class="filmina-contador">
                      {{ filminaActual + 1 }} / {{ filminas.length }}
                    </span>
                    <h4>{{ filminaActiva.titulo }}</h4>
                  </div>
                  <p>{{ filminaActiva.descripcion }}</p>
                </figcaption>
              </figure>

              <button
                type="button"
                class="ctrl-flecha"
                aria-label="Filmina siguiente"
                @click="filminaSiguiente"
              >
                <span>›</span>
              </button>
            </div>

            <nav class="miniaturas-nav" aria-label="Seleccionar filmina">
              <button
                v-for="(item, idx) in filminas"
                :key="item.imagen"
                type="button"
                class="miniatura-btn"
                :class="{ actual: filminaActual === idx }"
                @click="filminaActual = idx"
              >
                0{{ idx + 1 }}
              </button>
            </nav>
          </template>

          <div v-else class="tp-pendiente">
            <p class="tp-pendiente-titulo">Las filminas se cargarán próximamente</p>
            <p class="tp-pendiente-texto">
              Cuando tengas listas las imágenes en <code>public/img/</code>, agregalas al arreglo
              <code>filminas</code> dentro de <code>src/App.vue</code> para activar el visor.
            </p>
          </div>
        </section>
      </main>

      <footer class="pie-sitio">
        <p>Curso de Auxiliar en Administración · Material de Estudio</p>
        <button type="button" class="btn-subir" @click="volverArriba">↑ Subir</button>
      </footer>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,600;9..144,700&family=Inter:wght@400;500;600;700&display=swap');

.pagina-administrativa {
  --cream: #fcfaf7;
  --slate: #1a2a3a;
  --slate-muted: #526b82;
  --accent: #b87333;
  --accent-hover: #965c28;
  --surface: rgba(255, 255, 255, 0.95);
  --border: #e3e8ef;
  --text: #273240;
  --text-light: #607084;

  min-height: 100vh;
  position: relative;
  overflow-x: hidden;
  color: var(--text);
  font-family: 'Inter', sans-serif;
  display: flex;
  flex-direction: column;
}

.vanta-background {
  position: fixed;
  inset: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  opacity: 0.72;
  pointer-events: none;
}

.contenido-pagina {
  position: relative;
  z-index: 1;
  flex: 1;
  display: flex;
  flex-direction: column;
}

.barra-progreso {
  position: fixed;
  top: 0;
  left: 0;
  height: 3px;
  background: linear-gradient(90deg, var(--accent), #df9a5b);
  z-index: 1100;
  transition: width 0.1s linear;
}

.encabezado {
  position: sticky;
  top: 0;
  z-index: 1000;
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(14px);
  border-bottom: 1px solid var(--border);
  padding: 0 1rem;
}

.encabezado-limitado {
  max-width: 1160px;
  margin: 0 auto;
  padding: 0.72rem 0;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  flex-wrap: wrap;
}

.marca {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  background: transparent;
  border: none;
  padding: 0;
  text-align: left;
  cursor: pointer;
  color: var(--slate);
}

.logo-box {
  width: 42px;
  height: 42px;
  border-radius: 12px;
  background: linear-gradient(145deg, #ffffff, #f2efe9);
  border: 1px solid #d9e0e8;
  box-shadow: 0 4px 12px rgba(26, 42, 58, 0.08);
  color: var(--slate);
  display: grid;
  place-items: center;
  flex-shrink: 0;
  transition: transform 0.2s ease, border-color 0.2s ease;
}

.marca:hover .logo-box {
  transform: translateY(-1px);
  border-color: rgba(184, 115, 51, 0.55);
}

.marca-textos {
  display: flex;
  flex-direction: column;
}

.marca-titulo {
  font-family: 'Fraunces', serif;
  font-size: 1.18rem;
  font-weight: 700;
  line-height: 1.15;
  color: var(--slate);
}

.marca-subtitulo {
  font-size: 0.74rem;
  color: var(--slate-muted);
  margin-top: 0.12rem;
}

.navegacion {
  display: flex;
  align-items: center;
  gap: 0.35rem;
  flex-wrap: wrap;
}

.nav-item {
  background: transparent;
  border: 1px solid transparent;
  color: var(--slate);
  font-family: 'Inter', sans-serif;
  font-size: 0.83rem;
  font-weight: 600;
  padding: 0.45rem 0.78rem;
  border-radius: 9px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.nav-item:hover {
  background: rgba(26, 42, 58, 0.06);
}

.nav-item.activo {
  background: var(--slate);
  color: #ffffff;
  box-shadow: 0 4px 10px rgba(26, 42, 58, 0.16);
}

.nav-tp {
  border-color: rgba(184, 115, 51, 0.45);
  color: var(--accent);
}

.nav-tp:hover,
.nav-tp.activo {
  background: var(--accent);
  border-color: var(--accent);
  color: #ffffff;
}

.portada-hero {
  padding: 4.5rem 1rem 3.8rem;
  text-align: center;
}

.hero-contenido {
  max-width: 740px;
  margin: 0 auto;
}

.hero-kicker {
  display: inline-block;
  font-size: 0.73rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.14em;
  color: var(--accent);
  background: rgba(184, 115, 51, 0.1);
  border: 1px solid rgba(184, 115, 51, 0.22);
  padding: 0.3rem 0.75rem;
  border-radius: 999px;
  margin-bottom: 0.9rem;
}

.portada-hero h2 {
  font-family: 'Fraunces', serif;
  font-size: clamp(1.95rem, 4.2vw, 2.85rem);
  color: var(--slate);
  margin: 0 0 1rem;
  line-height: 1.15;
  font-weight: 700;
}

.portada-hero p {
  font-size: 1.04rem;
  color: var(--slate-muted);
  margin: 0 auto 2rem;
  max-width: 600px;
}

.hero-acciones {
  display: flex;
  justify-content: center;
  gap: 0.9rem;
  flex-wrap: wrap;
}

.btn-hero,
.btn-hero-suave {
  font-family: 'Inter', sans-serif;
  font-size: 0.94rem;
  font-weight: 600;
  padding: 0.85rem 1.6rem;
  border-radius: 10px;
  cursor: pointer;
  transition: transform 0.2s ease, background-color 0.2s ease, border-color 0.2s ease;
}

.btn-hero {
  background: var(--slate);
  color: #ffffff;
  border: none;
  box-shadow: 0 8px 20px rgba(26, 42, 58, 0.16);
}

.btn-hero:hover {
  transform: translateY(-2px);
  background: #24394f;
}

.btn-hero-suave {
  background: rgba(255, 255, 255, 0.9);
  color: var(--slate);
  border: 1px solid var(--border);
}

.btn-hero-suave:hover {
  transform: translateY(-2px);
  border-color: var(--accent);
}

.cuerpo-principal {
  max-width: 1000px;
  width: 100%;
  margin: 0 auto 3.2rem;
  padding: 0 1rem;
  display: flex;
  flex-direction: column;
  gap: 2.1rem;
}

.seccion-modulo {
  background: var(--surface);
  padding: 2.2rem;
  border-radius: 18px;
  border: 1px solid var(--border);
  box-shadow: 0 12px 30px rgba(26, 42, 58, 0.05);
  transition: transform 0.2s ease, border-color 0.2s ease;
  scroll-margin-top: 88px;
}

.seccion-modulo:hover {
  transform: translateY(-2px);
  border-color: rgba(184, 115, 51, 0.35);
}

.modulo-header {
  margin-bottom: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 0.45rem;
}

.modulo-meta-fila {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.75rem;
}

.modulo-label {
  font-size: 0.72rem;
  text-transform: uppercase;
  letter-spacing: 0.14em;
  color: var(--accent);
  font-weight: 700;
}

.modulo-indice {
  font-size: 0.78rem;
  font-weight: 700;
  color: var(--slate-muted);
  background: #f4f6f9;
  border: 1px solid var(--border);
  padding: 0.18rem 0.55rem;
  border-radius: 999px;
}

.seccion-modulo h3 {
  font-family: 'Fraunces', serif;
  font-size: 1.72rem;
  color: var(--slate);
  margin: 0;
  font-weight: 700;
}

.modulo-resumen {
  font-size: 0.99rem;
  color: var(--text-light);
  margin: 0.2rem 0 0;
  max-width: 680px;
}

.bloque-temas {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 1.15rem;
  margin-bottom: 1.5rem;
}

.tarjeta-tema {
  background: #faf9f6;
  border: 1px solid #ece7de;
  padding: 1.25rem 1.35rem;
  border-radius: 12px;
}

.tarjeta-tema h4 {
  font-size: 1.02rem;
  margin: 0 0 0.4rem;
  color: var(--slate);
  font-weight: 600;
}

.tarjeta-tema p {
  font-size: 0.92rem;
  color: var(--text-light);
  margin: 0;
  line-height: 1.55;
}

.caja-material {
  background: #fdfaf5;
  border-left: 3.5px solid var(--accent);
  border-top: 1px solid #f1eadf;
  border-right: 1px solid #f1eadf;
  border-bottom: 1px solid #f1eadf;
  padding: 0.95rem 1.3rem;
  border-radius: 10px;
}

.caja-material strong {
  color: var(--slate);
  font-size: 0.86rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.caja-material ul {
  margin: 0.45rem 0 0;
  padding-left: 1.2rem;
  color: var(--text);
  font-size: 0.92rem;
}

.seccion-tp {
  background: var(--surface);
  padding: 2.3rem;
  border-radius: 18px;
  border: 1.5px solid rgba(184, 115, 51, 0.55);
  box-shadow: 0 14px 32px rgba(26, 42, 58, 0.06);
  scroll-margin-top: 88px;
}

.tp-encabezado {
  text-align: center;
  margin-bottom: 1.5rem;
}

.tp-tag {
  display: inline-block;
  background: rgba(184, 115, 51, 0.12);
  color: var(--accent);
  font-size: 0.72rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  padding: 0.25rem 0.7rem;
  border-radius: 999px;
  margin-bottom: 0.45rem;
}

.tp-encabezado h3 {
  font-family: 'Fraunces', serif;
  font-size: 1.85rem;
  color: var(--slate);
  margin: 0 0 0.4rem;
}

.tp-encabezado p {
  color: var(--text-light);
  margin: 0;
}

.tp-pendiente {
  text-align: center;
  padding: 2.4rem 1.5rem;
  border: 1.5px dashed rgba(184, 115, 51, 0.45);
  border-radius: 14px;
  background: #fffdfb;
}

.tp-pendiente-titulo {
  margin: 0 0 0.45rem;
  font-weight: 700;
  color: var(--slate);
  font-size: 1.05rem;
}

.tp-pendiente-texto {
  margin: 0;
  color: var(--text-light);
  font-size: 0.92rem;
}

.tp-pendiente code {
  background: #f1f5f9;
  padding: 0.15rem 0.4rem;
  border-radius: 5px;
  color: var(--slate);
}

.visor-multimedia {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1.25rem;
}

.filmina-contenedor {
  flex: 1;
  margin: 0;
  border: 1px solid var(--border);
  border-radius: 12px;
  overflow: hidden;
  background: var(--cream);
}

.imagen-marco {
  width: 100%;
  aspect-ratio: 16 / 9;
  background: var(--slate);
  display: grid;
  place-items: center;
}

.imagen-marco img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.filmina-pie {
  padding: 1.25rem 1.5rem;
  background: #ffffff;
}

.pie-texto {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  flex-wrap: wrap;
  margin-bottom: 0.35rem;
}

.filmina-contador {
  font-size: 0.72rem;
  font-weight: 700;
  color: #ffffff;
  background: var(--accent);
  padding: 0.2rem 0.55rem;
  border-radius: 4px;
}

.filmina-pie h4 {
  margin: 0;
  font-size: 1.1rem;
  color: var(--slate);
}

.filmina-pie p {
  margin: 0;
  font-size: 0.92rem;
  color: var(--text-light);
}

.ctrl-flecha {
  background: #ffffff;
  border: 1px solid var(--border);
  color: var(--slate);
  width: 46px;
  height: 46px;
  border-radius: 50%;
  cursor: pointer;
  display: grid;
  place-items: center;
  font-size: 1.75rem;
  line-height: 1;
  transition: all 0.2s ease;
}

.ctrl-flecha:hover {
  background: var(--slate);
  color: #ffffff;
  border-color: var(--slate);
}

.miniaturas-nav {
  display: flex;
  justify-content: center;
  gap: 0.6rem;
  flex-wrap: wrap;
}

.miniatura-btn {
  background: #ffffff;
  border: 1px solid var(--border);
  min-width: 42px;
  height: 38px;
  padding: 0 0.6rem;
  border-radius: 8px;
  cursor: pointer;
  font-family: 'Inter', sans-serif;
  font-weight: 600;
  font-size: 0.85rem;
  transition: all 0.2s ease;
}

.miniatura-btn.actual {
  background: var(--accent);
  border-color: var(--accent);
  color: #ffffff;
}

.pie-sitio {
  text-align: center;
  padding: 2.2rem 1rem;
  background: rgba(244, 246, 249, 0.92);
  border-top: 1px solid var(--border);
  color: var(--text-light);
  font-size: 0.9rem;
  margin-top: auto;
}

.pie-sitio p {
  margin: 0;
}

.btn-subir {
  margin-top: 0.85rem;
  background: #ffffff;
  border: 1px solid var(--border);
  color: var(--slate);
  padding: 0.45rem 0.95rem;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
}

.boton-menu {
  display: none;
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 6px;
}

.icono-hamburguesa {
  display: block;
  width: 22px;
  height: 2px;
  background: var(--slate);
  position: relative;
  transition: background 0.2s ease;
}

.icono-hamburguesa::before,
.icono-hamburguesa::after {
  content: '';
  display: block;
  width: 100%;
  height: 100%;
  background: var(--slate);
  position: absolute;
  transition: transform 0.3s ease;
}

.icono-hamburguesa::before {
  top: -6px;
}

.icono-hamburguesa::after {
  top: 6px;
}

.icono-hamburguesa.abierto {
  background: transparent;
}

.icono-hamburguesa.abierto::before {
  transform: translateY(6px) rotate(45deg);
}

.icono-hamburguesa.abierto::after {
  transform: translateY(-6px) rotate(-45deg);
}

@media (max-width: 900px) {
  .boton-menu {
    display: block;
  }

  .navegacion {
    display: none;
    width: 100%;
    flex-direction: column;
    align-items: stretch;
    gap: 0.4rem;
    padding: 0.75rem 0 0.25rem;
  }

  .navegacion.abierta {
    display: flex;
  }

  .nav-item {
    text-align: left;
    padding: 0.65rem 0.9rem;
  }

  .seccion-modulo,
  .seccion-tp {
    padding: 1.4rem;
  }

  .visor-multimedia {
    flex-direction: column;
  }

  .ctrl-flecha {
    display: none;
  }
}
</style>