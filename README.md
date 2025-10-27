<div dir="ltr">

# 🪙 Sistema de Ahorro en Oro – Patria (Venezuela)

Este proyecto automatiza el seguimiento, cálculo y notificación de tu
****Plan de Ahorro en Oro**** en la plataforma ****Patria****, alineado
con las reglas oficiales del ****Banco Central de Venezuela (BCV)****:

- ✅ ****Precio del oro****: Basado en el ****LBMA Gold Price AM****
  (London Bullion Market).
- ✅ ****Tasa de cambio****: Usa la ****tasa oficial del BCV**** (no
  tasas de mercado).
- ✅ ****Amortización****: Calculada como el ****25% del valor del
  lingote**** en la fecha de cobro.
- ✅ ****Notificaciones****: Correo + Telegram el día de la
  amortización.
- ✅ ****Panel web****: Acceso móvil en tiempo real vía GitHub Pages.

------------------------------------------------------------------------

## 📁 Estructura del proyecto

<div dir="ltr">

<div dir="ltr">

<div dir="ltr">

<div dir="ltr">

<div dir="ltr">

<div dir="ltr">

/

</div>

<div dir="ltr">

├── index.html \# Panel web para móvil (GitHub Pages)

</div>

<div dir="ltr">

├── README.md \# Este archivo

</div>

<div dir="ltr">

└── Google Sheets \# Hoja de cálculo (ver sección de uso)

</div>

</div>

</div>

</div>

</div>

</div>

------------------------------------------------------------------------

## 🛠️ Requisitos

1.  ****Cuenta de Google**** (para Google Sheets y Apps Script).
2.  ****Cuenta de Telegram**** (opcional, para notificaciones).
3.  ****Cuenta de GitHub**** (para alojar el panel web).

------------------------------------------------------------------------

## 🔧 Configuración paso a paso

### 1. Crear la hoja de cálculo en Google Sheets

#### Hoja: `Inversiones`

<div dir="ltr">

<div dir="ltr">

| COLUMNA | ENCABEZADO           | TIPO / FÓRMULA                     |
|---------|----------------------|------------------------------------|
| A       | `REFERENCIA`         | Texto (ej:`Inv-001`)               |
| B       | `FECHA INVERSIÓN`    | Fecha                              |
| C       | `PRESENTACIÓN`       | Lista desplegable:`0.15g,0.25g`    |
| D       | `INVERSIÓN (VES)`    | Número (ingreso manual)            |
| E       | `FECHA AMORTIZACIÓN` | `=IF(B2<>"", B2 + 90, )`           |
| F       | `%`                  | `0.25`                             |
| G       | `AMORTIZACIÓN (VES)` | ***(actualizado por script)***     |
| H       | `REINVERTIR`         | Casilla de verificación (opcional) |

</div>

</div>

> ****Zona de precios (columna K)****
>
> - `K2`: Fecha de última actualización
> - `K3`: Precio oro (USD/gramo)
> - `K4`: Precio oro (VES/gramo) ← ****usado en cálculos****
> - `K5`: Precio oro (USD/onza)
> - `K6`: Tasa BCV (VES/USD)
> - `K7`: Valor de 0.15g en VES
> - `K8`: Valor de 0.25g en VES
> - `K9`: `=SUMA(D2:D)` → Total invertido
> - `K10`: `=SUMA(G2:G)` → Total amortizaciones

#### Hoja: `Amortizaciones` (opcional)

- Para seguimiento consolidado (no esencial para el núcleo).

#### Hoja: `Panel`

- En `A1`: `=ARRAYFORMULA(Inversiones!J2:K10)`
- Usada para exportar datos limpios a la web.

------------------------------------------------------------------------

### 2. Configurar Google Apps Script

Ve a ****Extensiones → Apps Script**** y crea dos funciones:

#### 📌 Script 1: `updateMarketData()`

Actualiza precios del oro y tasa BCV.

<div dir="ltr">

<div dir="ltr">

javascript

</div>

<div dir="ltr">

<div dir="ltr">

<div dir="ltr">

<div dir="ltr">

<div dir="ltr">

function updateMarketData() {

</div>

<div dir="ltr">

const sheet =
SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Inversiones');

</div>

<div dir="ltr">

const goldApiKey = "TU_CLAVE_GOLD_API"; // Regístrate en
https://www.goldapi.io

</div>

<div dir="ltr">

const goldApiUrl = "https://www.goldapi.io/api/XAU/USD";

</div>

<div dir="ltr">

const bcvApiUrl = "https://bcvapi.tech/api/v1/dolar";

</div>

<div dir="ltr">

try {

</div>

<div dir="ltr">

// Oro en USD

</div>

<div dir="ltr">

const goldOptions = { "headers": { "x-access-token": goldApiKey } };

</div>

<div dir="ltr">

const goldResponse = UrlFetchApp.fetch(goldApiUrl, goldOptions);

</div>

<div dir="ltr">

const goldData = JSON.parse(goldResponse.getContentText());

</div>

<div dir="ltr">

const pricePerOzUsd = goldData.price;

</div>

<div dir="ltr">

const pricePerGramUsd = pricePerOzUsd / 31.1035;

</div>

<div dir="ltr">

// Tasa BCV

</div>

<div dir="ltr">

let exchangeRateVesPerUsd = 147.08; // valor por defecto

</div>

<div dir="ltr">

try {

</div>

<div dir="ltr">

const bcvResponse = UrlFetchApp.fetch(bcvApiUrl);

</div>

<div dir="ltr">

const bcvData = JSON.parse(bcvResponse.getContentText());

</div>

<div dir="ltr">

if (typeof bcvData.tasa === 'number') exchangeRateVesPerUsd =
bcvData.tasa;

</div>

<div dir="ltr">

} catch (e) { console.warn("BCV API no disponible"); }

</div>

<div dir="ltr">

// Cálculos en VES

</div>

<div dir="ltr">

const pricePerGramVes = pricePerGramUsd \* exchangeRateVesPerUsd;

</div>

<div dir="ltr">

const porcion_0_15g = pricePerGramVes \* 0.15;

</div>

<div dir="ltr">

const porcion_0_25g = pricePerGramVes \* 0.25;

</div>

<div dir="ltr">

// Escribir en hoja

</div>

<div dir="ltr">

sheet.getRange('K2').setValue(new Date());

</div>

<div dir="ltr">

sheet.getRange('K3').setValue(pricePerGramUsd);

</div>

<div dir="ltr">

sheet.getRange('K4').setValue(pricePerGramVes);

</div>

<div dir="ltr">

sheet.getRange('K5').setValue(pricePerOzUsd);

</div>

<div dir="ltr">

sheet.getRange('K6').setValue(exchangeRateVesPerUsd);

</div>

<div dir="ltr">

sheet.getRange('K7').setValue(porcion_0_15g);

</div>

<div dir="ltr">

sheet.getRange('K8').setValue(porcion_0_25g);

</div>

<div dir="ltr">

sheet.getRange('K2').setNumberFormat("dd/mm/yyyy hh:mm:ss");

</div>

<div dir="ltr">

sheet.getRange('K3:K8').setNumberFormat("#,##0.00");

</div>

<div dir="ltr">

} catch (e) {

</div>

<div dir="ltr">

console.error("Error:", e.toString());

</div>

<div dir="ltr">

sheet.getRange('K2').setValue("Error");

</div>

<div dir="ltr">

sheet.getRange('K3:K8').clearContent();

</div>

<div dir="ltr">

}

</div>

<div dir="ltr">

}

</div>

</div>

</div>

</div>

</div>

</div>

> 🔑 ****Obtén tu clave GoldAPI****:
> <a href="https://www.goldapi.io/" dir="ltr">https://www.goldapi.io</a>

#### 📌 Script 2: `actualizarYCongelarAmortizaciones()`

Actualiza amortizaciones y envía notificaciones.

<div dir="ltr">

<div dir="ltr">

javascript

</div>

<div dir="ltr">

<div dir="ltr">

<div dir="ltr">

<div dir="ltr">

<div dir="ltr">

function actualizarYCongelarAmortizaciones() {

</div>

<div dir="ltr">

} catch (ex) { continue; }

</div>

<div dir="ltr">

let gramos = 0;

</div>

<div dir="ltr">

if (c === "0.15g") gramos = 0.15;

</div>

<div dir="ltr">

else if (c === "0.25g") gramos = 0.25;

</div>

<div dir="ltr">

else continue;

</div>

<div dir="ltr">

const valor = k4 \* gramos \* 0.25;

</div>

<div dir="ltr">

const amortStr = Utilities.formatDate(fechaValida, timezone,
"yyyy-MM-dd");

</div>

<div dir="ltr">

if (amortStr \<= todayStr) {

</div>

<div dir="ltr">

if (typeof g !== 'number' \|\| g \<= 0) {

</div>

<div dir="ltr">

sheet.getRange(row, 7).setValue(valor);

</div>

<div dir="ltr">

updates++;

</div>

<div dir="ltr">

const fechaAmortFormateada = Utilities.formatDate(fechaValida, timezone,
"dd/MM/yyyy");

</div>

<div dir="ltr">

const montoFormateado = valor.toLocaleString("es-VE", {
minimumFractionDigits: 2 });

</div>

<div dir="ltr">

notificaciones.push({ porcion: c, monto: valor, montoFormateado, fecha:
fechaAmortFormateada });

</div>

<div dir="ltr">

}

</div>

<div dir="ltr">

} else {

</div>

<div dir="ltr">

sheet.getRange(row, 7).setValue(valor);

</div>

<div dir="ltr">

updates++;

</div>

<div dir="ltr">

}

</div>

<div dir="ltr">

}

</div>

<div dir="ltr">

// Notificaciones

</div>

<div dir="ltr">

if (notificaciones.length \> 0) {

</div>

<div dir="ltr">

// Correo

</div>

<div dir="ltr">

const asunto = \`🔔 Amortización recibida - Plan de Ahorro en Oro\`;

</div>

<div dir="ltr">

let cuerpoCorreo = \`Se han recibido las siguientes
amortizaciones:\n\n\`;

</div>

<div dir="ltr">

notificaciones.forEach(item =\> {

</div>

<div dir="ltr">

const iconoPorcion = item.porcion === "0.15g" ? "🥇" : "🪙";

</div>

<div dir="ltr">

cuerpoCorreo += \`\${iconoPorcion} Porción: \${item.porcion}\n\`;

</div>

<div dir="ltr">

cuerpoCorreo += \`📅 Fecha: \${item.fecha}\n\`;

</div>

<div dir="ltr">

cuerpoCorreo += \`💵 Monto: \${item.montoFormateado} VES\n\n\`;

</div>

<div dir="ltr">

});

</div>

<div dir="ltr">

cuerpoCorreo += \`✅ Acción sugerida: Reinvierte hoy si estás en ventana
(15–20).\`;

</div>

<div dir="ltr">

MailApp.sendEmail(emailDestino, asunto, cuerpoCorreo);

</div>

<div dir="ltr">

console.log(\`📧 Correo enviado\`);

</div>

<div dir="ltr">

// Telegram

</div>

<div dir="ltr">

let mensajeTelegram = \`🔔 \*Amortización recibida\*\n\n\`;

</div>

<div dir="ltr">

notificaciones.forEach(item =\> {

</div>

<div dir="ltr">

const iconoPorcion = item.porcion === "0.15g" ? "🥇" : "🪙";

</div>

<div dir="ltr">

mensajeTelegram += \`\${iconoPorcion} Porción: \${item.porcion}\n\`;

</div>

<div dir="ltr">

mensajeTelegram += \`📅 Fecha: \${item.fecha}\n\`;

</div>

<div dir="ltr">

mensajeTelegram += \`💵 Monto: \*\${item.montoFormateado} VES\*\n\n\`;

</div>

<div dir="ltr">

});

</div>

<div dir="ltr">

mensajeTelegram += \`✅ \*Acción sugerida: Reinvierte hoy si estás en
ventana (15–20).\*\`;

</div>

<div dir="ltr">

const telegramUrl =
\`https://api.telegram.org/bot\${TELEGRAM_TOKEN}/sendMessage\`;

</div>

<div dir="ltr">

const payload = {

</div>

<div dir="ltr">

method: "POST",

</div>

<div dir="ltr">

headers: { "Content-Type": "application/json" },

</div>

<div dir="ltr">

payload: JSON.stringify({ chat_id: TELEGRAM_CHAT_ID, text:
mensajeTelegram, parse_mode: "Markdown" })

</div>

<div dir="ltr">

};

</div>

<div dir="ltr">

try { UrlFetchApp.fetch(telegramUrl, payload); console.log(\`📲 Telegram
enviado\`); }

</div>

<div dir="ltr">

catch (e) { console.error("❌ Telegram falló:", e); }

</div>

<div dir="ltr">

}

</div>

<div dir="ltr">

console.log(\`✅ Actualizadas \${updates} amortizaciones.\`);

</div>

<div dir="ltr">

}

</div>

</div>

</div>

</div>

</div>

</div>

#### 🕒 Configurar triggers

- `updateMarketData()` → cada ****6 horas****
- `actualizarYCongelarAmortizaciones()` → ****diario a las 8:00 AM****

------------------------------------------------------------------------

### 3. Panel web en GitHub Pages (`index.html`)

<div dir="ltr">

<div dir="ltr">

html

</div>

<div dir="ltr">

<div dir="ltr">

<div dir="ltr">

<div dir="ltr">

<div dir="ltr">

\<!DOCTYPE html\>

</div>

<div dir="ltr">

\<html lang="es"\>

</div>

<div dir="ltr">

\<head\>

</div>

<div dir="ltr">

\<meta charset="UTF-8" /\>

</div>

<div dir="ltr">

\<meta name="viewport" content="width=device-width,
initial-scale=1.0"/\>

</div>

<div dir="ltr">

\<title\>Ahorro en Oro\</title\>

</div>

<div dir="ltr">

\<style\>

</div>

<div dir="ltr">

body { background: \#0f0f0f; color: \#e0e0e0; font-family: system-ui;
font-size: 13px; margin: 0; padding: 20px; }

</div>

<div dir="ltr">

h1 { color: \#ffd700; text-align: center; font-size: 1.3em; margin: 0 0
18px; }

</div>

<div dir="ltr">

.card { background: \#1a1a1a; border-radius: 10px; padding: 16px;
max-width: 400px; margin: 0 auto; box-shadow: 0 2px 8px rgba(0,0,0,0.4);
}

</div>

<div dir="ltr">

.row { display: flex; padding: 10px 0; border-bottom: 1px solid
\#2a2a2a; }

</div>

<div dir="ltr">

.row:last-child { border: none; }

</div>

<div dir="ltr">

.label { color: \#ffb74d; flex: 1; }

</div>

<div dir="ltr">

.value { color: white; text-align: right; flex: 1; }

</div>

<div dir="ltr">

.footer { text-align: center; color: \#777; font-size: 12px; margin-top:
16px; }

</div>

<div dir="ltr">

\</style\>

</div>

<div dir="ltr">

\</head\>

</div>

<div dir="ltr">

\<body\>

</div>

<div dir="ltr">

\<h1\>📊 Ahorro en Oro – Patria\</h1\>

</div>

<div dir="ltr">

\<div class="card" id="datos"\>\<div class="row"\>\<div
class="label"\>Cargando...\</div\>\</div\>\</div\>

</div>

<div dir="ltr">

\<div class="footer"\>Actualizado en tiempo real\</div\>

</div>

<div dir="ltr">

\<script\>

</div>

<div dir="ltr">

const CSV_URL =
"https://docs.google.com/spreadsheets/d/e/TU_ID/pub?gid=757563170&single=true&output=csv";

</div>

<div dir="ltr">

async function cargarDatos() {

</div>

<div dir="ltr">

try {

</div>

<div dir="ltr">

const res = await fetch(CSV_URL);

</div>

<div dir="ltr">

const txt = await res.text();

</div>

<div dir="ltr">

const lines = txt.trim().split('\n').filter(l =\> l.trim());

</div>

<div dir="ltr">

let html = '';

</div>

<div dir="ltr">

lines.forEach(line =\> {

</div>

<div dir="ltr">

const parts = line.split(/,(?=(?:(?:\[^"\]\*"){2})\*\[^"\]\*\$)/);

</div>

<div dir="ltr">

if (parts.length \>= 2) {

</div>

<div dir="ltr">

const label = parts\[0\].replace(/^"(.\*)"\$/, '\$1');

</div>

<div dir="ltr">

const value = parts\[1\].replace(/^"(.\*)"\$/, '\$1');

</div>

<div dir="ltr">

html += \`\<div class="row"\>\<div class="label"\>\${label}\</div\>\<div
class="value"\>\${value}\</div\>\</div\>\`;

</div>

<div dir="ltr">

}

</div>

<div dir="ltr">

});

</div>

<div dir="ltr">

document.getElementById('datos').innerHTML = html \|\| '\<div
class="row"\>\<div class="label"\>Sin datos\</div\>\</div\>';

</div>

<div dir="ltr">

} catch (e) {

</div>

<div dir="ltr">

document.getElementById('datos').innerHTML = '\<div class="row"\>\<div
class="label"\>❌ Error\</div\>\</div\>';

</div>

<div dir="ltr">

}

</div>

<div dir="ltr">

}

</div>

<div dir="ltr">

cargarDatos();

</div>

<div dir="ltr">

setInterval(cargarDatos, 300000);

</div>

<div dir="ltr">

\</script\>

</div>

<div dir="ltr">

\</body\>

</div>

<div dir="ltr">

\</html\>

</div>

</div>

</div>

</div>

</div>

</div>

> 📌 ****Para usarlo****:
>
> 1.  Crea hoja `Panel` con `=ARRAYFORMULA(Inversiones!J2:K10)`
> 2.  Publica `Panel` como CSV: ****Archivo → Compartir → Publicar en la
>     web → CSV****
> 3.  Reemplaza `CSV_URL` con tu enlace real.

------------------------------------------------------------------------

## 🌐 APIs utilizadas

<div dir="ltr">

<div dir="ltr">

| SERVICIO                 | URL                                                                    | USO                      |
|--------------------------|------------------------------------------------------------------------|--------------------------|
| ****GoldAPI****          | <a href="https://www.goldapi.io/" dir="ltr">https://www.goldapi.io</a> | Precio del oro (LBMA)    |
| ****BCV API****          | <a href="https://bcvapi.tech/" dir="ltr">https://bcvapi.tech</a>       | Tasa oficial del BCV     |
| ****Google Sheets****    | \-                                                                     | Almacenamiento y cálculo |
| ****Telegram Bot API**** | <a href="https://core.telegram.org/bots/api" dir="ltr">https://core.telegram.org/bots/api</a> | Notificaciones           |

</div>

</div>

> 💡 ****Tasa BCV actual****: `147.08 VES/USD` (según `bcvapi.tech`,
> 2025-08-30) 💡 ****Tasa de mercado (exchangerate-api.com)****:
> `112.13 VES/USD` (no se usa en este proyecto)

------------------------------------------------------------------------

## 📝 Notas importantes

- El ****25% de amortización**** se calcula sobre el ****valor del
  lingote en la fecha de cobro****, no sobre tu inversión inicial.
- Los valores se ****congelan automáticamente**** el día de la
  amortización.
- El sistema es ****ilimitado en el tiempo****: añade tantas filas como
  necesites.

------------------------------------------------------------------------

## 💡 Inversiones en fecha de amortización

Si realizas una nueva inversión el mismo día que recibes una amortización, **regístrala como una nueva fila** en la hoja `Inversiones`. El sistema automáticamente:

- Calculará su fecha de amortización (`fecha + 90 días`).
- Incluirá su valor en el total consolidado si coincide con otras amortizaciones futuras.

> **No se suman amortizaciones pasadas**: cada inversión genera su propia amortización independiente.

#### Ejemplo:
- **16/09/2025**: Inviertes 1,200 VES en 0.15g → se amortiza el **16/12/2025**.
- **16/12/2025**: Recibes 1,500 VES (amortización) y ese mismo día inviertes 1,500 VES en 0.25g.
  - Esta **nueva inversión** se registra en una **nueva fila**.
  - Su fecha de amortización será: **16/03/2026**.
- **16/03/2026**: Recibirás la amortización correspondiente a la inversión del 16/12/2025 (calculada con el precio del oro de ese día).

La amortización del 16/12/2025 **no se suma** a la del 16/03/2026; son eventos independientes.

------------------------------------------------------------------------

## 🙏 Agradecimientos

- A las APIs públicas que hacen posible este proyecto.
- A la comunidad de desarrolladores venezolanos por compartir
  conocimiento.

------------------------------------------------------------------------

## 📜 Licencia

Este proyecto es de uso personal y educativo. Puedes adaptarlo
libremente, pero ****no para uso comercial sin autorización****.

</div>
