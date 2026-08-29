# google-ads-reporting

Herramienta para consultar y generar reportes de Google Ads bajo demanda mediante la **Google Ads API**.

La aplicación permite obtener métricas de rendimiento de cuentas de Google Ads mediante cuatro métodos: terminal, Claude Code, formularios de GitHub Issues y solicitudes en texto libre mediante GitHub Issues.

Todas las modalidades utilizan la misma lógica de consulta y acceden a la API de Google Ads en el momento de ejecutar el reporte. No existen ejecuciones programadas ni periódicas y no se almacenan datos de las cuentas de Google Ads en el repositorio.

La herramienta está orientada exclusivamente a **reporting y consultas de solo lectura**. No crea, modifica ni elimina campañas, anuncios, grupos de anuncios o palabras clave.

## 1. Instalación

```bash
git clone <tu-repo>
cd google-ads-reporting
pip install -r requirements.txt
```

Requiere Python 3.9 o superior.

## 2. Credenciales

Para utilizar la herramienta se necesitan las credenciales de autenticación de Google Ads API:

| Dato                          | Descripción                                                                                          |
| ----------------------------- | ---------------------------------------------------------------------------------------------------- |
| `developer_token`             | Token de desarrollador obtenido desde el Centro de APIs de una cuenta de administrador de Google Ads |
| `client_id` / `client_secret` | Credenciales OAuth 2.0 obtenidas desde Google Cloud                                                  |
| `refresh_token`               | Token utilizado para mantener la autorización OAuth 2.0                                              |
| `login_customer_id`           | ID de la cuenta de administrador utilizada para realizar las llamadas a Google Ads API               |

Las credenciales reales deben mantenerse fuera del repositorio y nunca deben publicarse. La herramienta utiliza archivos locales o GitHub Secrets según el método de ejecución.

## 3. Métodos de uso

### A) Terminal

```bash
cp config/google-ads.yaml.example config/google-ads.yaml
```

Completá `config/google-ads.yaml` con tus datos reales. **Ese archivo nunca se sube al repositorio** y está incluido en `.gitignore`.

Ejemplo:

```bash
python main.py --metricas costo clics --nivel campaña --dias 7
```

#### Parámetros

| Flag            | Requerido | Descripción                                                                          |
| --------------- | --------- | ------------------------------------------------------------------------------------ |
| `--metricas`    | sí        | Una o más de: `costo`, `clics`, `impresiones`, `conversiones`, `ctr`, `cpc_promedio` |
| `--nivel`       | no        | `campaña` (default) o `grupo_de_anuncios`                                            |
| `--dias`        | no        | Ventana de días hacia atrás desde hoy. Default: 7                                    |
| `--campaña`     | no        | Filtra por nombre de campaña (coincidencia parcial)                                  |
| `--customer-id` | no        | Si no se pasa, utiliza el `login_customer_id` del archivo YAML                       |
| `--out`         | no        | Ruta para exportar además a CSV. Si no se pasa, no se escribe ningún archivo         |

#### Ejemplos

```bash
# Costo y clics por campaña, últimos 7 días
python main.py --metricas costo clics --nivel campaña --dias 7

# Conversiones y costo por grupo de anuncios durante los últimos 30 días
python main.py --metricas conversiones costo --nivel grupo_de_anuncios --dias 30 --campaña "Verano2026"

# Mostrar el reporte y exportarlo además a CSV
python main.py --metricas costo clics ctr --dias 14 --out reporte.csv
```

### B) Claude Code

Con `config/google-ads.yaml` completo, se puede solicitar un reporte directamente desde Claude Code.

Por ejemplo:

> "Traeme el costo y las conversiones de la campaña Verano2026 de los últimos 15 días."

Claude Code arma el comando correspondiente de `main.py`, lo ejecuta y devuelve el resultado interpretado.

Esta modalidad utiliza exactamente la misma lógica de consulta que la ejecución mediante terminal.

### C) GitHub Issues

Los reportes también pueden solicitarse mediante GitHub Issues, sin necesidad de abrir una terminal.

Al crear un Issue utilizando una de las plantillas disponibles, un GitHub Action ejecuta el reporte y publica el resultado como comentario en el mismo Issue.

No existen ejecuciones programadas: el workflow solamente se ejecuta cuando se crea un Issue compatible.

#### Configuración

En el repositorio, ir a:

**Settings → Secrets and variables → Actions**

y configurar los siguientes Secrets:

* `GOOGLE_ADS_DEVELOPER_TOKEN`
* `GOOGLE_ADS_CLIENT_ID`
* `GOOGLE_ADS_CLIENT_SECRET`
* `GOOGLE_ADS_REFRESH_TOKEN`
* `GOOGLE_ADS_LOGIN_CUSTOMER_ID`

Para utilizar la modalidad de texto libre con Claude, agregar además:

* `ANTHROPIC_API_KEY`

Las credenciales y claves se almacenan exclusivamente como GitHub Secrets y no forman parte del código fuente ni del repositorio.

#### C.1) Formulario estructurado

En:

**Issues → New Issue → Reporte de Google Ads**

se puede seleccionar:

* nivel del reporte;
* métricas;
* cantidad de días;
* campaña opcional.

El workflow transforma esos valores en los parámetros utilizados por `main.py` y ejecuta la consulta directamente contra Google Ads API.

Esta modalidad no utiliza modelos de IA para interpretar el pedido.

#### C.2) Texto libre

En:

**Issues → New Issue → Reporte de Google Ads (texto libre)**

se puede escribir una solicitud como:

> "Necesito el costo y las conversiones de la campaña Verano2026 de los últimos 15 días."

El workflow utiliza Claude para convertir el pedido en parámetros estructurados y posteriormente ejecuta la misma lógica de `main.py`.

Esta modalidad requiere `ANTHROPIC_API_KEY`.

Si el pedido es ambiguo o no puede transformarse correctamente en parámetros válidos, el workflow informa el problema mediante un comentario en el Issue.

> Si el repositorio llegara a ser público, conviene restringir quién puede crear Issues o ejecutar workflows para evitar consultas no autorizadas y el uso innecesario de cuotas de APIs externas.

## 4. Datos consultados

La herramienta utiliza Google Ads API para consultar métricas de rendimiento de campañas y grupos de anuncios.

Entre las métricas disponibles se encuentran:

* Costo
* Clics
* Impresiones
* Conversiones
* CTR
* CPC promedio

Los reportes se obtienen directamente desde Google Ads API en el momento de la ejecución.

La aplicación no modifica campañas, anuncios, grupos de anuncios, palabras clave ni otros recursos de Google Ads.

## 5. Estructura del repositorio

```text
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── reporte-ads.yml
│   │   └── reporte-ads-libre.yml
│   ├── scripts/
│   │   ├── common.py
│   │   ├── parse_and_run.py
│   │   └── interpret_and_run.py
│   └── workflows/
│       ├── reporte-ads.yml
│       └── reporte-ads-libre.yml
├── config/
│   └── google-ads.yaml.example
├── src/
│   ├── client.py
│   ├── queries.py
│   └── export.py
├── main.py
└── requirements.txt
```

## 6. Tratamiento de datos y seguridad

Las credenciales reales de Google Ads y Google Cloud no se almacenan en el repositorio.

El archivo local `config/google-ads.yaml` se encuentra excluido mediante `.gitignore`, mientras que los workflows de GitHub utilizan GitHub Secrets.

Las consultas a Google Ads API se realizan bajo demanda y los resultados no se almacenan automáticamente en el repositorio.

El parámetro `--out` permite generar un archivo CSV de forma explícita cuando el usuario lo solicita desde la terminal.

## 7. Notas técnicas

### Valores monetarios

Google Ads API devuelve determinados valores monetarios en micros. La aplicación realiza la conversión correspondiente para mostrar los importes en formato legible.

### Moneda

Cada reporte incluye la moneda de la cuenta consultada, obtenida directamente desde Google Ads API.

Los valores monetarios, como `costo` y `cpc_promedio`, se expresan en la moneda configurada para la cuenta.

### CTR

Google Ads API devuelve el CTR como una fracción decimal. La aplicación lo multiplica por 100 para mostrarlo en formato porcentual.

Por ejemplo:

```text
0.0523 → 5.23%
```

### Ejecución

Todos los métodos utilizan la misma lógica central de consulta.

No existen versiones diferentes del sistema de reporting: terminal, Claude Code y GitHub Issues solamente representan distintas formas de construir los parámetros y ejecutar el mismo proceso.

## 8. Objetivo del proyecto

El objetivo del proyecto es facilitar el acceso bajo demanda a métricas de rendimiento de Google Ads y automatizar la generación de reportes, manteniendo las consultas centralizadas y utilizando Google Ads API como fuente de datos.

La aplicación está diseñada para **consultar y reportar información de Google Ads mediante operaciones de lectura**, sin realizar cambios sobre las cuentas publicitarias.
