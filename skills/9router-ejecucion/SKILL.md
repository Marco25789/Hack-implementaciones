---
name: 9router-ejecucion
description: >
  Gestiona el cambio entre Claude real y Codex (GPT-5.3) via 9router como fallback
  cuando se agota la cuota de Claude Code. Usar cuando el usuario diga "activar codex",
  "activar fallback", "usar codex", "9router", "cambiar a codex", "volver a claude",
  "modo codex", "modo normal", "fallback", o "/fallback".
---

# 9router Fallback - Guia paso a paso

Este skill guia al usuario paso a paso para activar/desactivar el fallback de Codex via 9router.
Tratar al usuario como principiante: explicar cada paso, mostrar exactamente que hacer, y esperar confirmacion antes de avanzar.

## Al invocar este skill

Usar AskUserQuestion:

Pregunta: "Que necesitas?"
Opciones:
- "Activar Codex (usar GPT cuando se agota Claude)"
- "Volver a Claude normal"
- "Ver en que modo estoy"

---

## FLUJO: ACTIVAR CODEX

### Paso 1 - Verificar instalacion

Ejecutar:
```bash
9router --version
```

Si falla: "9router no esta instalado. Ejecuto npm install -g 9router por ti?"
Si el usuario acepta, ejecutar npm install -g 9router y verificar de nuevo.

### Paso 2 - Leer la password del dashboard

Ejecutar:
```bash
grep "INITIAL_PASSWORD" /c/Users/User/iniciar-9router.bat
```

Guardar el valor para mostrarlo despues. Si el archivo no existe, informar al usuario que necesita crear iniciar-9router.bat (ver referencia en references/).

### Paso 3 - Pedir al usuario que inicie 9router

Mostrar al usuario EXACTAMENTE esto:

"Necesito que hagas esto en tu PC:

1. Abre una terminal NUEVA (clic derecho escritorio > Abrir terminal, o busca 'cmd' en inicio)
2. Escribe esto y dale Enter:
   iniciar-9router.bat
3. Te saldra un menu asi:
   > Web UI (Open in Browser)
     Terminal UI (Interactive CLI)
     Hide to Tray (Background)
     Exit
4. Selecciona 'Web UI (Open in Browser)' (la primera opcion)
5. IMPORTANTE: Esa terminal debe quedarse abierta. NO LA CIERRES. Minimizala.

Dime cuando hayas hecho estos pasos."

Usar AskUserQuestion: "Ya iniciaste 9router?"
- "Si, esta corriendo"
- "La terminal se cerro sola"
- "No entiendo que hacer"

Si "la terminal se cerro sola": Verificar que iniciar-9router.bat tiene 'pause' al final. Leer el archivo y corregirlo si es necesario.
Si "no entiendo": Repetir las instrucciones de forma mas simple.

### Paso 4 - Verificar que 9router responde

Ejecutar:
```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:20128/ 2>&1
```

Si devuelve 200 o 307: "Perfecto, 9router esta corriendo."
Si falla: "9router no esta respondiendo. Revisa que la terminal siga abierta y muestre el menu o logs."

### Paso 5 - Abrir el dashboard y dar la clave

Mostrar al usuario:

"Ahora abre tu navegador y ve a esta direccion:
http://localhost:20128/dashboard

Te pedira una clave. Tu clave es: [valor de INITIAL_PASSWORD del paso 2]

Escribela y dale Enter para entrar al dashboard."

Usar AskUserQuestion: "Pudiste entrar al dashboard?"
- "Si, ya estoy dentro"
- "La clave no funciona"
- "Me dice ERR_CONNECTION_REFUSED"

Si "la clave no funciona": Releer el .bat y mostrar la clave exacta otra vez.
Si "ERR_CONNECTION_REFUSED": 9router no esta corriendo, volver al paso 3.

### Paso 6 - Conectar Codex

Mostrar al usuario:

"Bien, ahora dentro del dashboard:

1. Busca la seccion 'Providers' (proveedores) en el menu lateral o en la pagina principal
2. Busca 'Codex' o 'OpenAI Codex'
3. Haz clic en 'Connect' o 'Login'
4. Te abrira una ventana de login de OpenAI/ChatGPT
5. Inicia sesion con tu cuenta de ChatGPT Plus
6. Acepta los permisos

ATENCION - PASO IMPORTANTE QUE CONFUNDE:
Despues de aceptar los permisos, NO te saldra una pagina de exito.
Te saldra una pagina que parece un ERROR o 'no se pudo conectar' o similar.
Esto es NORMAL, no te preocupes.

Lo que tienes que hacer:
7. Mira la BARRA DE DIRECCIONES del navegador (donde dice la URL)
8. COPIA TODA ESA URL completa (Ctrl+L para seleccionarla, luego Ctrl+C)
9. Vuelve a la pestana del dashboard de 9router
10. Veras un MODAL o ventana emergente pidiendo que pegues la URL
11. PEGA la URL completa ahi (Ctrl+V)
12. Dale clic en 'Connect' o 'Confirmar'
13. Ahora si deberia mostrar que Codex esta conectado

Dime cuando hayas conectado Codex."

Usar AskUserQuestion: "Codex conectado?"
- "Si, ya lo conecte"
- "Me sale una pagina de error despues de los permisos"
- "No encuentro donde conectar Codex"
- "Otro problema"

Si "pagina de error": Decir "Es normal. Copia la URL COMPLETA de esa pagina de error (la de la barra de direcciones), vuelve al dashboard de 9router, pegala en el modal que aparecio, y dale en conectar."
Si "no encuentro": Sugerir revisar el dashboard, puede estar en Settings > Providers o en la pagina principal.
Si "otro problema": Preguntar que error muestra para ayudar.

### Paso 7 - Configurar variables de entorno

Decir: "Ahora voy a configurar las variables de entorno para que Claude Code use 9router."

Escribir script temporal y ejecutar:
```bash
cat > /c/Users/User/temp-env.ps1 << 'PSEOF'
[System.Environment]::SetEnvironmentVariable('ANTHROPIC_BASE_URL', 'http://localhost:20128', 'User')
[System.Environment]::SetEnvironmentVariable('ANTHROPIC_API_KEY', 'router-dummy-key', 'User')
Write-Host "OK: Variables configuradas"
PSEOF
powershell -ExecutionPolicy Bypass -File "C:/Users/User/temp-env.ps1"
rm /c/Users/User/temp-env.ps1
```

Verificar que diga "OK: Variables configuradas".

### Paso 8 - Instrucciones finales (MUY IMPORTANTE)

Mostrar al usuario EXACTAMENTE esto:

"Todo configurado! Ahora sigue estos pasos (no necesitas cerrar nada):

1. Abre una terminal NUEVA (busca 'cmd' en inicio)
   (No cierres esta ni ninguna otra ventana)

2. En la nueva terminal escribe: claude

3. Cuando Claude Code abra, escribe: /logout

4. Te preguntara algo sobre API key. Responde: SI

5. Escribe: /model cx/gpt-5.3-codex

6. Hazle una pregunta de prueba para verificar

7. Abre http://localhost:20128/dashboard > Usage
   Si muestra consumo, Codex esta funcionando correctamente

LISTO! Ya estas usando Codex via 9router.

RECUERDA: La terminal de 9router debe quedarse abierta siempre.
Cuando quieras volver a Claude normal, escribe /fallback y elige 'Volver a Claude normal'."

---

## FLUJO: VOLVER A CLAUDE NORMAL

### Paso A - Limpiar variables

Decir: "Voy a eliminar las variables de 9router para que Claude vuelva a funcionar normal."

Escribir script temporal y ejecutar:
```bash
cat > /c/Users/User/temp-env.ps1 << 'PSEOF'
[System.Environment]::SetEnvironmentVariable('ANTHROPIC_BASE_URL', $null, 'User')
[System.Environment]::SetEnvironmentVariable('ANTHROPIC_API_KEY', $null, 'User')
Write-Host "OK: Variables eliminadas"
PSEOF
powershell -ExecutionPolicy Bypass -File "C:/Users/User/temp-env.ps1"
rm /c/Users/User/temp-env.ps1
```

CRITICO: NUNCA usar setx para limpiar. setx con "" deja las variables como cadena vacia y causa "Auth conflict". SIEMPRE usar PowerShell con $null.

### Paso B - Verificar

Ejecutar:
```bash
powershell -ExecutionPolicy Bypass -File - << 'PSEOF'
$b = [System.Environment]::GetEnvironmentVariable('ANTHROPIC_BASE_URL', 'User')
$k = [System.Environment]::GetEnvironmentVariable('ANTHROPIC_API_KEY', 'User')
if ($b -or $k) { Write-Host "ERROR: Variables aun existen - BASE_URL=$b API_KEY=$k" } else { Write-Host "OK: Variables completamente eliminadas" }
PSEOF
```

Si dice ERROR: Intentar la eliminacion de nuevo.
Si dice OK: Continuar.

### Paso C - Instrucciones finales

Mostrar al usuario:

"Variables eliminadas! Ahora:

1. Si tienes la terminal de 9router abierta, ya puedes cerrarla
   (Ya no la necesitas)

2. Abre una terminal NUEVA (no necesitas cerrar las demas)

3. Escribe: claude

4. Te pedira iniciar sesion. Entra con tu cuenta de claude.ai

5. IMPORTANTE: Claude Code recordara el modelo cx/gpt-5.3-codex de antes.
   Cambialo al modelo normal escribiendo:
   /model opus
   o si prefieres Sonnet:
   /model sonnet

6. LISTO! Ya estas de vuelta con Claude real.

Si ves el error 'Auth conflict', significa que las variables no se limpiaron bien.
En ese caso, vuelve a ejecutar /fallback y elige 'Volver a Claude normal' otra vez."

---

## FLUJO: VER ESTADO

Ejecutar ambos comandos:

```bash
powershell -ExecutionPolicy Bypass -Command "Write-Host 'BASE_URL:'; [System.Environment]::GetEnvironmentVariable('ANTHROPIC_BASE_URL', 'User'); Write-Host 'API_KEY:'; [System.Environment]::GetEnvironmentVariable('ANTHROPIC_API_KEY', 'User')"
```

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:20128/ 2>&1
```

Reportar de forma clara:

- Variables vacias + curl falla:
  "Estas en MODO NORMAL: usando Claude real de Anthropic. Todo bien."

- Variables con valor + curl responde (200/307):
  "Estas en MODO CODEX: 9router activo, las peticiones van a Codex."

- Variables con valor + curl falla:
  "HAY UN PROBLEMA: Las variables apuntan a 9router pero 9router no esta corriendo.
   Opcion 1: Inicia 9router (iniciar-9router.bat)
   Opcion 2: Vuelve a modo normal (ejecuta /fallback > Volver a Claude normal)"

- Variables vacias + curl responde:
  "9router esta corriendo pero no lo estas usando. Estas en modo Claude normal. Si quieres usar Codex, ejecuta /fallback > Activar Codex."
