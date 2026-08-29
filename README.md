# google-ads-api-info

# google-ads-reporting

Reportes de Google Ads a demanda, en 4 formas posibles: terminal, Claude
Code, o abriendo un Issue en GitHub (con formulario o en texto libre).
Ninguna corre sola (sin cron), y ninguna guarda datos en el repo: cada
corrida consulta la API en el momento y muestra el resultado.

## 1. Instalación

```bash
git clone <tu-repo>
cd google-ads-reporting
pip install -r requirements.txt
```

Requiere Python 3.9 o superior.

## 2. Credenciales

Necesitás 4 cosas de Google antes de poder usar esto:

| Dato | Dónde se consigue |
|---|---|
| `developer_token` | Google Ads → Herramientas → Centro de API (gratis, aprobación manual) |
| `client_id` / `client_secret` | Google Cloud Console → credenciales OAuth2 (tipo "Desktop App") |
| `refresh_token` | Se genera una vez corriendo el flujo OAuth2 con el client_id/secret de arriba |
| `login_customer_id` | El ID de tu cuenta de Google Ads (o cuenta MCC si administrás varias) |

Con eso, según el método que uses (ver abajo), completás las credenciales
de una forma u otra.

## 3. Métodos de uso

### A) Terminal

```bash
cp config/google-ads.yaml.example config/google-ads.yaml
```

Completá `config/google-ads.yaml` con tus datos reales. **Ese archivo nunca
se sube al repo** (está en `.gitignore`).

```bash
python main.py --metricas costo clics --nivel campaña --dias 7
```

#### Parámetros

| Flag | Requerido | Descripción |
|---|---|---|
| `--metricas` | sí | Una o más de: `costo`, `clics`, `impresiones`, `conversiones`, `ctr`, `cpc_promedio` |
| `--nivel` | no | `campaña` (default) o `grupo_de_anuncios` |
| `--dias` | no | Ventana de días hacia atrás desde hoy. Default: 7 |
| `--campaña` | no | Filtra por nombre de campaña (coincidencia parcial) |
| `--customer-id` | no | Si no se pasa, usa el `login_customer_id` del yaml |
| `--out` | no | Ruta para exportar además a CSV. Si no se pasa, no se escribe ningún archivo |

#### Ejemplos

```bash
# Costo y clics por campaña, últimos 7 días
python main.py --metricas costo clics --nivel campaña --dias 7

# Conversiones por grupo de anuncios de una campaña puntual, últimos 30 días
python main.py --metricas conversiones costo --nivel grupo_de_anuncios --dias 30 --campaña "Verano2026"

# Además de mostrarlo, exportar a CSV
python main.py --metricas costo clics ctr --dias 14 --out reporte.csv
```

### B) Claude Code

Con `config/google-ads.yaml` completo (mismo paso que en Terminal), le
pedís algo en el chat (ej. "traeme el costo y las conversiones de la
campaña Verano2026 de los últimos 15 días") y Claude Code arma el comando
de `main.py` correspondiente, lo corre, y te devuelve el reporte ya
interpretado.

### C) Issues de GitHub

Pensado para pedir un reporte sin abrir la terminal: abrís un Issue nuevo,
un GitHub Action corre el reporte solo y te contesta como comentario en el
mismo Issue, después lo cierra. Hay dos plantillas, según cómo prefieras
pedirlo — conviven en el mismo repo, cada una con su propio disparador.

**Setup único, antes de poder usar cualquiera de las dos:**

1. Andá a tu repo en GitHub → **Settings → Secrets and variables →
   Actions** y creá estos 5 Secrets (los mismos datos de la sección de
   Credenciales, arriba):
   - `GOOGLE_ADS_DEVELOPER_TOKEN`
   - `GOOGLE_ADS_CLIENT_ID`
   - `GOOGLE_ADS_CLIENT_SECRET`
   - `GOOGLE_ADS_REFRESH_TOKEN`
   - `GOOGLE_ADS_LOGIN_CUSTOMER_ID`
2. Si además querés la vía de **texto libre** (C.2), sumá un sexto Secret:
   - `ANTHROPIC_API_KEY` — tu API key de Claude (console.anthropic.com).
     Sin este Secret, esa vía específica no funciona (te lo va a avisar
     en el comentario del Issue), pero la del formulario (C.1) funciona
     igual sin necesitarlo.
3. Listo — no hace falta tocar nada más, los workflows
   (`.github/workflows/reporte-ads.yml` y `reporte-ads-libre.yml`) ya
   están armados para leerlos.

#### C.1) Formulario estructurado

En tu repo, pestaña **Issues → New Issue → "Reporte de Google Ads"**,
completás el formulario (nivel, métricas, días, campaña opcional), y a
los pocos segundos tenés la respuesta como comentario. No hace falta que
nadie interprete texto libre: el formulario ya entrega los datos en un
formato fijo, así que esta vía nunca falla por una mala interpretación
— y no tiene costo de por medio (no llama a ningún modelo de IA).

#### C.2) Texto libre (con Claude)

Para cuando preferís escribir el pedido como en un chat, en vez de llenar
un formulario: **Issues → New Issue → "Reporte de Google Ads (texto
libre)"**, y en el único campo escribís algo como *"necesito el costo y
las conversiones de la campaña Verano2026 de los últimos 15 días"*. El
workflow le pasa ese texto a Claude (modelo `claude-haiku-4-5`, elegido
por ser rápido y barato para esto) para traducirlo a los mismos
parámetros que usa `main.py`, y sigue el mismo camino que el formulario
de ahí en adelante.

Como hay interpretación de por medio:
- Cada Issue de este tipo implica una llamada a la API de Claude (costo
  mínimo por llamada, pero no es cero como en C.1).
- Si el pedido es ambiguo o el modelo no devuelve algo interpretable, el
  comentario de respuesta te lo va a decir en vez de fallar en silencio
  — probá reformular más simple, o usá el formulario (C.1) para ese caso.

> Si el repo llegara a ser público en algún momento, conviene restringir
> quién puede disparar estos workflows (por ejemplo, solo colaboradores),
> para que nadie externo pueda abrir Issues, consultar tu cuenta de Ads, y
> —en el caso de C.2— gastar tu cuota de la API de Claude.

## 4. Estructura del repo

```
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── reporte-ads.yml         # formulario estructurado (C.1)
│   │   └── reporte-ads-libre.yml   # campo de texto libre (C.2)
│   ├── scripts/
│   │   ├── common.py            # arma el comando de main.py, lo corre y guarda el resultado
│   │   ├── parse_and_run.py     # C.1: lee el formulario y llama a common.py
│   │   └── interpret_and_run.py # C.2: le pide a Claude los parámetros y llama a common.py
│   └── workflows/
│       ├── reporte-ads.yml         # dispara con la plantilla C.1
│       └── reporte-ads-libre.yml   # dispara con la plantilla C.2
├── config/
│   └── google-ads.yaml.example   # plantilla de credenciales (sin datos reales)
├── src/
│   ├── client.py    # inicializa el cliente autenticado de la API
│   ├── queries.py   # arma la consulta GAQL según lo que se pida
│   └── export.py    # ejecuta la consulta y devuelve/exporta los resultados
├── main.py          # punto de entrada (línea de comandos)
└── requirements.txt
```

## Notas

- **Los valores son los mismos que muestra la plataforma**, no cálculos
  propios. La API de Google Ads devuelve los montos en "micros" (valor ×
  1.000.000); el código aplica la misma conversión que usa la plataforma
  para mostrarlos en pantalla, nada más.
- **Moneda**: cada reporte incluye una columna `moneda` con el código de
  la cuenta consultada (ej. `USD`, `ARS`), tomado siempre de la API en el
  momento — nunca asumido ni hardcodeado. Todos los valores monetarios
  (`costo`, `cpc_promedio`) están en esa moneda.
- **CTR**: la API lo devuelve como fracción (`0.0523`); acá se multiplica
  por 100 para que coincida con el formato que ves en la plataforma
  (`5.23`, es decir 5.23%).
- No hay ningún cron ni corrida periódica: el único disparador automático
  son los dos Actions de Issues, y solo actúan cuando vos abrís uno.
  Ningún método guarda datos en el repo salvo que uses `--out` manualmente.
- **Las 4 formas de uso corren el mismo `main.py`** — no hay versiones
  distintas de la lógica del reporte, solo distintas formas de armar los
  parámetros y disparar la corrida (un flag de terminal, lo que arma
  Claude Code, un formulario, o texto libre interpretado por Claude).
