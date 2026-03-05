# INSTALAR 9ROUTER — FALLBACK DE MODELOS PARA CLAUDE CODE

## QUE ES ESTO

Guia completa para instalar **9router** como proxy local que permite usar **GPT-5.3-Codex** (via ChatGPT Plus) como fallback cuando se agota la cuota de Claude Code.

**Requisitos del usuario:**
- Suscripcion Claude Max ($100/mes) con Claude Code CLI instalado
- Suscripcion ChatGPT Plus (para acceder a Codex via OAuth)
- Windows 10/11
- Node.js >= v18

**Que logras:**
- Seguir usando `claude` en la terminal exactamente igual
- Cuando se agota la cuota de Claude, cambiar manualmente a Codex
- NO pierdes historial, sesiones, CLAUDE.md, skills ni nada del ecosistema Claude Code
- 9router solo se usa como fallback manual, NO como intermediario permanente

**Seguridad:**
- Open source MIT: https://github.com/decolua/9router
- 100% local: corre en localhost:20128
- Sin facturacion propia: tu pagas directo a Anthropic y OpenAI
- Los tokens OAuth se guardan localmente en tu PC

---

## PASO 1 — VERIFICAR NODE.JS

```bash
node --version
npm --version
```

**IMPORTANTE:** Node.js debe ser >= v18. Si no lo tienes, instalalo desde https://nodejs.org/

---

## PASO 2 — INSTALAR 9ROUTER

```bash
npm install -g 9router
```

Verificar:
```bash
9router --version
```

Debe mostrar la version (ej: 0.3.31). Si da error de "command not found", cierra y abre la terminal.

---

## PASO 3 — CREAR CARPETA DE DATOS

```bash
mkdir C:\Users\TU_USUARIO\9router-data
```

> **NOTA:** Reemplaza `TU_USUARIO` por tu nombre de usuario de Windows en TODOS los pasos siguientes.

---

## PASO 4 — CREAR SCRIPT DE INICIO

Crea el archivo `C:\Users\TU_USUARIO\iniciar-9router.bat` con este contenido:

```batch
@echo off
echo ==========================================
echo  9ROUTER - Iniciando proxy de fallback
echo  Dashboard: http://localhost:20128/dashboard
echo ==========================================
set JWT_SECRET=tu-secreto-jwt-aqui
set INITIAL_PASSWORD=tu-password-aqui
set DATA_DIR=C:\Users\TU_USUARIO\9router-data
set PORT=20128
set HOSTNAME=127.0.0.1
set NODE_ENV=production
set NEXT_PUBLIC_BASE_URL=http://localhost:20128
set API_KEY_SECRET=tu-api-secret-aqui
set MACHINE_ID_SALT=tu-salt-aqui
echo Iniciando 9router...
9router
echo.
echo === 9router se detuvo. Presiona una tecla para cerrar ===
pause
```

### ERRORES COMUNES EN ESTE PASO:

1. **La terminal se cierra inmediatamente:** Esto pasa si NO tienes el `pause` al final. El `pause` mantiene la ventana abierta para que veas el error si 9router falla.

2. **La password para el dashboard:** La password para entrar al dashboard web es la que defines en `INITIAL_PASSWORD`. NO es "123456" ni nada generico. Recuerda lo que pusiste ahi.

---

## PASO 5 — CREAR SCRIPTS DE CAMBIO DE MODO

### Script para volver a Claude real:
Crea `C:\Users\TU_USUARIO\claude-modo-normal.bat`:

```batch
@echo off
echo Modo: CLAUDE REAL (Anthropic directo)
setx ANTHROPIC_BASE_URL ""
setx ANTHROPIC_API_KEY ""
echo Variables limpiadas. Reinicia la terminal y usa: claude
```

### Script para cambiar a Codex via 9router:
Crea `C:\Users\TU_USUARIO\claude-modo-codex.bat`:

```batch
@echo off
echo Modo: CODEX via 9router (fallback)
echo IMPORTANTE: 9router debe estar corriendo en otra terminal
setx ANTHROPIC_BASE_URL "http://localhost:20128"
setx ANTHROPIC_API_KEY "cualquier-valor-dummy"
echo Listo. Reinicia la terminal y usa: claude
echo Dashboard disponible en: http://localhost:20128/dashboard
```

### Script para ver en que modo estas:
Crea `C:\Users\TU_USUARIO\claude-estado.bat`:

```batch
@echo off
echo === MODO ACTUAL DE CLAUDE CODE ===
if "%ANTHROPIC_BASE_URL%"=="" (
    echo MODO NORMAL: Claude real - Anthropic directo
) else (
    echo MODO FALLBACK: 9router activo en %ANTHROPIC_BASE_URL%
)
echo.
echo Para cambiar:
echo   claude-modo-normal.bat  = Claude real
echo   claude-modo-codex.bat   = Codex via 9router
```

---

## PASO 6 — PRIMERA EJECUCION DE 9ROUTER

1. Haz doble clic en `iniciar-9router.bat`
2. Se abrira una terminal con un menu interactivo:
   ```
   Choose Interface (v0.3.xx)
   Server: http://localhost:20128

   > Web UI (Open in Browser)
     Terminal UI (Interactive CLI)
     Hide to Tray (Background)
     Exit
   ```
3. **Elige "Web UI (Open in Browser)"** — esto abre el dashboard en tu navegador

### IMPORTANTE — LA TERMINAL DEBE QUEDARSE ABIERTA

La terminal que se abre es el servidor de 9router. **NO LA CIERRES.** Si la cierras, 9router se apaga. Minimizala y dejala ahi.

4. En el navegador, ve a `http://localhost:20128/dashboard`
5. Te pedira password — usa la que definiste en `INITIAL_PASSWORD` en el paso 4
6. Ve a **Providers > Codex** y haz login con tu cuenta de ChatGPT Plus via OAuth

---

## PASO 7 — ACTIVAR MODO CODEX EN CLAUDE CODE

### PROBLEMA CRITICO: CONFLICTO DE AUTENTICACION

Cuando ejecutas `claude-modo-codex.bat` y luego abres `claude`, veras este warning:

```
Auth conflict: Both a token (claude.ai) and an API key (ANTHROPIC_API_KEY) are set.
```

Esto significa que Claude Code tiene DOS credenciales:
- Tu token OAuth de claude.ai (login normal)
- La API key dummy que apunta a 9router

**Claude Code prioriza el token OAuth**, asi que aunque pongas el modelo custom, sigue usando Claude real, NO Codex.

### SOLUCION: HACER /logout ANTES DE USAR CODEX

1. Ejecuta `claude-modo-codex.bat` en una terminal
2. **Cierra esa terminal y abre una nueva**
3. Ejecuta `claude`
4. Dentro de Claude Code, escribe `/logout`
5. Te preguntara si quieres usar API key — di **SI**
6. Ahora `claude` usara la API key que apunta a 9router
7. Escribe `/model cx/gpt-5.3-codex` para seleccionar el modelo de Codex

### PREGUNTA FRECUENTE: "/logout borra mi historial?"

**NO.** `/logout` solo elimina el token de autenticacion OAuth. No toca:
- Tu historial de sesiones
- Tus CLAUDE.md y configuraciones
- Tus skills y MCPs
- Tus archivos en ~/.claude/

Es como cerrar sesion en Gmail — no pierdes los correos.

### COMO SABER SI REALMENTE ESTAS USANDO CODEX

- `/model cx/gpt-5.3-codex` es solo un nombre de texto. Claude Code lo acepta siempre, **no valida si el modelo existe**. Asi que escribir el nombre NO confirma nada.
- Si arriba en Claude Code dice **"Sonnet 4.6 - API Usage Billing"**, sigues usando Claude real.
- La forma mas confiable: revisa el **Usage** en el dashboard de 9router (`http://localhost:20128/dashboard`). Si muestra consumo, las peticiones estan pasando por ahi.
- Tambien revisa **Usage > Codex** en el dashboard. Si muestra actividad, Codex esta respondiendo.

---

## PASO 8 — VOLVER A CLAUDE REAL

Cuando quieras volver a usar Claude con tu cuota normal:

1. Ejecuta `claude-modo-normal.bat`
2. Cierra la terminal de 9router (ahi si puedes cerrarla)
3. **Cierra tu terminal actual y abre una nueva**
4. Ejecuta `claude` — te pedira login de nuevo con tu cuenta de claude.ai
5. Todo vuelve a la normalidad

---

## FLUJO COMPLETO DE USO

```
MODO NORMAL (dia a dia):
  Usas: claude
  Modelo: Claude real (Sonnet/Opus via Anthropic)
  9router: APAGADO

CUANDO SE AGOTA LA CUOTA:
  1. Abre terminal nueva → iniciar-9router.bat → elige Web UI
  2. Dashboard → Providers → Codex → OAuth login (solo la primera vez)
  3. Abre otra terminal → claude-modo-codex.bat
  4. Cierra y abre tu terminal de trabajo
  5. claude → /logout → acepta API key → /model cx/gpt-5.3-codex
  6. Trabaja normalmente

CUANDO VUELVE LA CUOTA DE CLAUDE:
  1. claude-modo-normal.bat
  2. Cierra terminal de 9router
  3. Cierra y abre terminal de trabajo
  4. claude → login normal
```

---

## MODELOS DISPONIBLES EN 9ROUTER CON CHATGPT PLUS

| Modelo | Uso |
|--------|-----|
| `cx/gpt-5.3-codex` | El mejor para coding |
| `cx/gpt-5.2-codex` | Alternativa |

Para cambiar modelo dentro de Claude Code:
```
/model cx/gpt-5.3-codex
```

---

## ARCHIVOS CREADOS (REFERENCIA)

| Archivo | Funcion |
|---------|---------|
| `iniciar-9router.bat` | Arranca el servidor 9router |
| `claude-modo-codex.bat` | Cambia Claude Code a usar 9router |
| `claude-modo-normal.bat` | Vuelve a Claude real |
| `claude-estado.bat` | Muestra en que modo estas |
| `9router-data\` | Carpeta de datos de 9router |

---

## NOTAS IMPORTANTES

- **NO** configures 9router como servicio de Windows — solo inicialo manualmente cuando lo necesites
- **NO** modifiques nada en `~/.claude/` manualmente
- **Siempre reinicia la terminal** despues de ejecutar los scripts de cambio de modo (los `setx` solo aplican a terminales nuevas)
- La primera vez que conectes Codex en el dashboard, necesitas hacer OAuth login. Las siguientes veces ya recuerda la sesion.
- Si el dashboard no carga (`ERR_CONNECTION_REFUSED`), verifica que la terminal de 9router siga abierta y muestre el menu o el log del servidor

---

## LINKS

- Repo 9router: https://github.com/decolua/9router
- npm: https://www.npmjs.com/package/9router
- Dashboard: http://localhost:20128/dashboard (solo cuando 9router esta corriendo)
