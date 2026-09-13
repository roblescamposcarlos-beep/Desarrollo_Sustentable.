# Cómo configurar la automatización en Make

Objetivo: cuando alguien llena el formulario de `index.html` (Nombre, Correo,
Opinión), Make debe:

1. Recibir esos datos desde la página web.
2. Guardarlos como una fila nueva en Google Sheets.
3. Enviar un correo automático de agradecimiento a esa persona.

Esto se logra con un escenario de **3 módulos**: `Webhooks` → `Google Sheets` → `Email` (o `Gmail`).

---

## Módulo 1 — Webhooks: Custom webhook (disparador)

Este módulo es el que "escucha" cuando alguien envía el formulario.

1. En Make, crea un **Escenario nuevo** (Scenarios → Create a new scenario).
2. Haz clic en el `+` y busca **Webhooks**.
3. Elige **Custom webhook**.
4. Haz clic en **Add** para crear un webhook nuevo:
   - Dale un nombre, por ejemplo `Opinion Nube`.
   - Data structure: déjalo vacío por ahora, se determina solo.
   - Guarda.
5. Make te dará una URL como:
   `https://hook.us1.make.com/xxxxxxxxxxxxxxxxxxxxxxxx`
   Cópiala.
6. Abre `index.html` y reemplaza la constante `WEBHOOK_URL` (cerca del final
   del archivo, dentro del `<script>`) con esa URL exacta.
7. Vuelve a Make y deja el escenario en modo **"Redetermine data structure"**
   (o simplemente deja el módulo esperando). Ahora ve a tu página web, llena
   el formulario una vez con datos de prueba y envíalo. Eso hace que Make
   reciba un ejemplo real y detecte automáticamente los campos `nombre`,
   `correo` y `opinion`.
8. Verás esos 3 campos disponibles para usarlos en los siguientes módulos.

---

## Módulo 2 — Google Sheets: Add a Row

Este módulo guarda la respuesta en una hoja de cálculo.

1. Antes de nada, crea (o abre) una hoja en **Google Sheets** con estas
   columnas en la primera fila: `Nombre`, `Correo`, `Opinion`, `Fecha`
   (la fecha es opcional pero útil).
2. En Make, haz clic en el `+` a la derecha del módulo de Webhooks.
3. Busca **Google Sheets** y elige la acción **Add a Row**.
4. Conecta tu cuenta de Google (Add → inicia sesión → autoriza permisos de
   Sheets).
5. Configura:
   - **Spreadsheet**: selecciona el archivo que creaste.
   - **Sheet**: selecciona la pestaña (por defecto `Sheet1` u `Hoja 1`).
   - **Table contains headers**: actívalo (Yes), así Make te deja mapear
     por nombre de columna en vez de por número.
6. En los campos que aparecen (Nombre, Correo, Opinion...), haz clic en cada
   uno y selecciona el valor correspondiente del **Módulo 1** (el círculo
   morado de Webhooks): `nombre` → columna Nombre, `correo` → columna
   Correo, `opinion` → columna Opinion.
7. Si agregaste columna `Fecha`, usa la función `now` de Make o el campo de
   fecha del webhook.

---

## Módulo 3 — Email: Send an Email (agradecimiento automático)

Este módulo envía el correo de agradecimiento a quien llenó el formulario.

1. Haz clic en el `+` después del módulo de Google Sheets.
2. Busca **Email** (el módulo genérico que no necesita conexión, ideal para
   empezar rápido) — o **Gmail** si prefieres que el correo salga desde tu
   cuenta de Gmail y tenga tu firma.
   - Con **Email**: elige la acción **Send an Email**, y en el paso previo
     Make te pedirá configurar tu propio servidor SMTP o usar el que Make
     ofrece por defecto según tu plan.
   - Con **Gmail** (recomendado, más simple): elige **Send an Email**,
     conecta tu cuenta de Gmail (Add → inicia sesión → autoriza permisos).
3. Configura el correo:
   - **To**: haz clic y selecciona el campo `correo` del Módulo 1
     (Webhooks) — así el correo llega exactamente a quien llenó el
     formulario.
   - **Subject**: por ejemplo, `¡Gracias por compartir tu opinión!`
   - **Content / Body** (texto sugerido, puedes personalizarlo):
     ```
     Hola {{nombre}},

     ¡Gracias por participar en nuestra encuesta sobre el impacto
     ambiental del almacenamiento en la nube! Tu opinión fue registrada
     correctamente.

     Tu respuesta:
     "{{opinion}}"

     Saludos.
     ```
   - Para insertar `{{nombre}}` y `{{opinion}}` reales, haz clic en el
     campo de texto y selecciona esas variables del Módulo 1 (no las
     escribas a mano, selecciónalas desde el panel de mapeo de Make).

---

## Activar el escenario

1. Arriba a la izquierda del escenario, activa el interruptor **ON/OFF**
   para dejarlo encendido (el webhook solo funciona si el escenario está
   activo, o si lo estás probando con "Run once").
2. Mientras haces pruebas, puedes usar **Run once** en la esquina inferior
   y luego enviar el formulario desde `index.html` para ver los 3 módulos
   ejecutarse en tiempo real (aparecerá un número junto a cada módulo con
   la cantidad de operaciones).
3. Verifica:
   - Que la fila nueva aparezca en tu Google Sheet.
   - Que el correo de agradecimiento llegue a la bandeja de entrada (revisa
     también spam la primera vez).

---

## Resumen del flujo

```
[Formulario en index.html]
        │  (fetch POST)
        ▼
[Webhooks: Custom webhook]  ← Módulo 1, recibe nombre/correo/opinion
        │
        ▼
[Google Sheets: Add a Row]  ← Módulo 2, guarda la respuesta
        │
        ▼
[Gmail/Email: Send an Email] ← Módulo 3, agradece a la persona
```

Con estos 3 módulos conectados y el escenario activado, cada vez que alguien
llene el formulario de la página web, su respuesta quedará guardada
automáticamente en Excel/Google Sheets y recibirá un correo de
agradecimiento sin que tengas que hacer nada manual.
