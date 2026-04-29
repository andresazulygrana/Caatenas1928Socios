# Caatenas1928Socios

Sistema de gestion de socios y cuotas para Club Atletico Atenas, San Carlos, Uruguay.

Este repositorio contiene la exportacion textual de objetos GeneXus de la KB `Caatenas1928`. La aplicacion tiene dos superficies principales:

- Portal publico del socio: consulta por cedula, registro de nuevo socio y pago de cuota.
- Backoffice administrativo: gestion de socios, tipos de cuota, cuotas, importacion, configuracion de pago y diagnostico Mercado Pago.

## Estado Actual

Fecha de referencia: 2026-04-29.

Estado funcional probado:

- Backoffice principal operativo.
- `BuscarSocio` operativo:
  - Sin cedula: muestra mensaje y no navega.
  - Cedula existente con cuota pendiente: navega a `PantallaPago`.
  - Cedula existente sin cuota pendiente: informa que no hay cuota para pagar.
  - Cedula inexistente: informa que no es socio y permite registrarse.
  - Boton `Registrarme`: navega a `DatosNuevoSocio`.
- `DatosNuevoSocio` operativo:
  - Valida cedula, nombre, apellido, email, celular y tipo de cuota.
  - Email y celular son obligatorios.
  - Muestra solo tipos de cuota activos.
  - Crea socio y primera cuota pendiente.
- `PantallaPago` operativo:
  - Muestra cuota pendiente, datos del socio y boton de pago.
  - Crea preferencia Checkout Pro de Mercado Pago.
- `ConfirmacionPago` operativo para pagos aprobados.
- Mercado Pago sandbox:
  - Pago con dinero disponible funciona correctamente.
  - Pago con tarjeta sandbox esta en consulta con Mercado Pago: la preferencia y la merchant order se crean, pero Mercado Pago no crea `Payment`; por eso no hay `status_detail`.
- `AdminConfiguracionPago` operativo para configurar token, Base URL publica y correo.
- `AdminProcesarPagoSandbox` operativo como pantalla tecnica de diagnostico.

## Directorios Importantes

KB local:

```text
C:\Users\andres.cabrera\GeneXus\KBs\Caatenas1928
```

Repositorio Git real:

```text
C:\Users\andres.cabrera\GeneXus\KBs\Caatenas1928\src
```

Remoto:

```text
https://github.com/andresazulygrana/Caatenas1928Socios.git
```

Importante: el repositorio vive dentro del directorio `src` de la KB porque ahi GeneXus/MCP exporta los objetos como texto. Esto es intencional.

No versionar:

- `NetModel`
- `Data*`
- `GXSPC*`
- `GXRESTSPC`
- `Locks`
- `tmp_*`
- logs
- archivos de base local
- archivos con credenciales reales

## Regla Fundamental de Trabajo

Todo cambio de la aplicacion debe hacerse en objetos GeneXus usando MCP y skills.

No editar codigo generado en `NetModel`.

No modificar directamente archivos generados por build.

No ejecutar reorg, impact database, build all o build one sin confirmacion explicita.

El usuario normalmente ejecuta el build desde GeneXus IDE.

## Skills y MCP

Skill obligatorio para trabajo de KB:

- `nexa`

Skill util para problemas MCP/import/export:

- `gx-kb-troubleshooting`

MCP esperado:

```text
GeneXus Next MCP en localhost:8001/mcp
Instalacion indicada: C:\GeneXusVersions\GenexusNextAppNativeBeta
```

Al usar herramientas MCP, el `rootDirectory` correcto es el directorio de la KB, no el repo `src`:

```text
C:\Users\andres.cabrera\GeneXus\KBs\Caatenas1928
```

Correcto:

```json
{
  "rootDirectory": "C:\\Users\\andres.cabrera\\GeneXus\\KBs\\Caatenas1928"
}
```

Incorrecto:

```json
{
  "rootDirectory": "C:\\Users\\andres.cabrera\\GeneXus\\KBs\\Caatenas1928\\src"
}
```

Usar `forceSave=false` salvo que haya una razon concreta. En esta KB, `forceSave=true` ya genero confusion con imports que parecian correctos pero no refrescaban como se esperaba.

Flujo recomendado para cambios:

1. Leer objetos actuales desde `src`.
2. Exportar desde KB si hay duda de sincronizacion:

```text
export_kb_to_text
rootDirectory = C:\Users\andres.cabrera\GeneXus\KBs\Caatenas1928
```

3. Modificar los archivos de objeto GeneXus exportados.
4. Validar:

```text
validate_kb_text_files
```

5. Importar:

```text
import_text_to_kb
forceSave = false
```

6. Reexportar el objeto y verificar que el cambio quedo en la KB.
7. Pedir al usuario que haga build desde IDE.
8. Probar en navegador.
9. Commit en Git.

## Git

Ultimo commit estable subido:

```text
80228f2 Stabilize Atenas portal and Mercado Pago sandbox flow
```

Tag de respaldo antes de ese estado:

```text
pre-mp-flow-baseline-20260429
```

Comandos utiles:

```powershell
git -C "C:\Users\andres.cabrera\GeneXus\KBs\Caatenas1928\src" status --short
git -C "C:\Users\andres.cabrera\GeneXus\KBs\Caatenas1928\src" log --oneline --decorate -5
git -C "C:\Users\andres.cabrera\GeneXus\KBs\Caatenas1928\src" diff --stat
```

## Modelo Principal

### Socio

Representa un socio del club.

Atributos importantes:

- `SocioId`
- `SocioCedula`
- `SocioNombre`
- `SocioApellido`
- `SocioEmail`
- `SocioTelefono`
- `SocioFechaNacimiento`
- `TipoCuotaId`
- `SocioEstado`
- `SocioFechaAlta`
- `SocioOrigen`

Regla de negocio:

- La cedula es el identificador natural del socio.
- Debe evitarse duplicar socios por cedula.
- El portal permite registro publico, pero valida que no exista socio previo.

### TipoCuota

Define planes o categorias de cuota.

Atributos importantes:

- `TipoCuotaId`
- `TipoCuotaDescripcion`
- `TipoCuotaImporte`
- `TipoCuotaActivo`

Regla de negocio:

- El portal de registro solo debe mostrar tipos activos.
- Cambios de importe aplican a cuotas nuevas, no a cuotas ya generadas.

### Cuota

Representa una cuota de socio.

Atributos importantes:

- `CuotaId`
- `SocioId`
- `TipoCuotaId`
- `CuotaPeriodo`
- `CuotaImporte`
- `CuotaEstado`
- `CuotaFechaVencimiento`
- `CuotaFechaPago`
- `CuotaMPPaymentId`
- `CuotaMPExternalUrl`
- `CuotaMPEstado`

Estados:

- `P`: pendiente
- `G`: pagada
- `V`: vencida

## Portal Publico

### BuscarSocio

Entrada publica del portal.

Flujo correcto:

1. Usuario ingresa cedula.
2. Si no ingresa cedula, mostrar mensaje.
3. Si la cedula existe como socio:
   - Si tiene cuota pendiente, ir a `PantallaPago`.
   - Si no tiene cuota pendiente, informar que no hay cuota para pagar.
4. Si la cedula no existe, informar que no es socio.
5. Si el usuario presiona `Registrarme`, ir a `DatosNuevoSocio`.

Notas:

- No usar `Return` en eventos del Web Panel para cortar validaciones; anteriormente eso provoco navegacion/retorno incorrecto en navegador.
- La validacion de cedula acepta solo numeros, con limpieza de puntos y guion cuando corresponde.

### DatosNuevoSocio

Formulario publico para registro.

Validaciones actuales:

- Cedula obligatoria y valida.
- Nombre obligatorio.
- Apellido obligatorio.
- Email obligatorio.
- Celular obligatorio.
- Tipo de cuota obligatorio y activo.
- No crear socio si la cedula ya existe.

UX actual:

- Las cuotas activas se muestran como opciones.
- Al seleccionar plan, la informacion seleccionada queda visible cerca de las opciones.

### PantallaPago

Pantalla de estado de cuenta y pago.

Responsabilidades:

- Recibir `SocioId`.
- Buscar cuota pendiente del socio.
- Mostrar importe, periodo, vencimiento y datos del socio.
- Invocar `PagarCuota` para crear preferencia Mercado Pago.
- Redirigir al `sandbox_init_point` o `init_point` segun configuracion.

### ConfirmacionPago

Pantalla de retorno de Mercado Pago.

Responsabilidades:

- Recibir parametros de retorno:
  - `MPEstado`
  - `SocioId`
  - `CuotaId`
  - `payment_id`
  - `collection_status`
  - `external_reference`
  - `merchant_order_id`
  - `preference_id`
- Mostrar resultado para el usuario.
- Para `approved`, marcar cuota como pagada si corresponde.
- Mostrar acceso a comprobante.

Pendiente recomendado:

- Blindar idempotencia entre retorno y webhook.
- Validar monto, moneda y `external_reference` contra `CuotaId`.
- Evitar doble envio de comprobante si entran return y webhook.

## Backoffice

### AdminDashboard

Panel general administrativo.

Muestra resumen de socios, cuotas y recaudacion.

### AdminSocios

Listado y gestion de socios.

Incluye filtros y acceso a detalle.

### AdminSocioDetalle

Detalle administrativo de socio e historial de cuotas.

### AdminImportarSocios

Importacion de socios desde archivo.

Debe mantener deduplicacion por cedula.

### AdminTipoCuota

Listado y gestion de tipos de cuota.

### AdminTipoCuotaDetalle

Detalle/ABM de tipo de cuota.

### AdminCuotasNuevo

Pantalla nueva para reemplazar `AdminCuotas`, creada porque `AdminCuotas` quedo con problemas de layout/catalogo.

Estado:

- `AdminCuotasNuevo` esta operativo.
- `General.UI.SidebarItemsDP` apunta a `AdminCuotasNuevo`.
- El usuario elimino `AdminCuotas` desde IDE, pero pueden quedar archivos historicos exportados si no se limpian con cuidado.

Precaucion:

- No recrear `AdminCuotas` accidentalmente desde archivos viejos.
- Si se importa todo desde texto, verificar que no vuelva a aparecer `AdminCuotas` si no se desea.

### AdminConfiguracionPago

Configuracion operativa de Mercado Pago y correo.

Campos relevantes:

- Nombre de configuracion.
- Access token Mercado Pago.
- Base URL publica.
- Estado activo.
- SMTP host, puerto, usuario, clave, remitente, nombre remitente, SSL/seguro.

Base URL publica usada en pruebas:

```text
https://91e1-2800-a4-24e3-e200-e44a-4c6d-e1c0-13fd.ngrok-free.app/Caatenas1928.NET
```

Webhook esperado:

```text
https://91e1-2800-a4-24e3-e200-e44a-4c6d-e1c0-13fd.ngrok-free.app/Caatenas1928.NET/awebhookmercadopago.aspx
```

No guardar tokens reales en Git.

### AdminProcesarPagoSandbox

Pantalla tecnica para pruebas Mercado Pago.

Funciones:

- Procesar manualmente un `payment_id`.
- Diagnosticar un `payment_id`.
- Buscar pagos por `CuotaId` usando `external_reference`.
- Diagnosticar preferencia por `PreferenceId`.
- Buscar merchant order por `PreferenceId`.
- Abrir portal para `SocioId` de prueba.

Archivos de diagnostico generados en runtime:

```text
NetModel\web\mp_diagnostico_preferencia.txt
NetModel\web\mp_diagnostico_orden.txt
NetModel\web\mp_diagnostico_cuota.txt
```

Estos archivos son runtime/debug y no deben commitearse.

## Mercado Pago

Integracion actual:

- Checkout Pro sandbox.
- Se crea preferencia desde `PagarCuota`.
- Se usa `external_reference = CuotaId`.
- `back_urls` apuntan a `ConfirmacionPago`.
- `notification_url` apunta a `awebhookmercadopago.aspx`.
- `ProcesarPagoMercadoPago` consulta `GET /v1/payments/{payment_id}` para verificar estado real.

Prueba aprobada con dinero disponible:

```text
SocioId=232
CuotaId=32
preference_id=3322900544-a807e118-5a41-4389-8d8a-82c3b8d0b1ed
payment_id=156989967858
collection_status=approved
payment_type=account_money
merchant_order_id=40387446731
```

Caso tarjeta sandbox en consulta:

```text
SocioId=231
CuotaId=31
preference_id=3322900544-3204a3a8-37b5-4ff3-94af-7d2459fa296c
merchant_order_id=40415488432
```

Diagnostico del caso tarjeta:

- Preferencia HTTP 200.
- Merchant order HTTP 200.
- `status = opened`.
- `order_status = payment_required`.
- `is_test = true`.
- `payments = []`.
- No existe `payment_id`.
- No hay `status_detail` porque Mercado Pago no creo el objeto `Payment`.

Conclusion:

- El circuito base funciona porque `account_money` aprueba y retorna correctamente.
- El problema de tarjeta parece estar dentro del flujo sandbox/card de Mercado Pago o en una condicion que Mercado Pago debe explicar.

## Correo

Correo automatico previsto:

- Alta de socio.
- Comprobante de pago.

Reglas:

- Email del socio es obligatorio.
- Celular del socio es obligatorio.
- No romper el envio actual al ajustar datos enviados a Mercado Pago.
- Si se modifica la preferencia MP para no enviar `payer.email`, no debe afectar `SocioEmail` ni los correos internos del sistema.

Pendiente recomendado:

- Evitar doble envio si `ConfirmacionPago` y webhook procesan el mismo pago.
- Registrar estado de envio de correo si se agrega auditoria.

## Objetos de Diagnostico Mercado Pago

Procedures agregados:

- `DiagnosticarPagoMercadoPago`
- `DiagnosticarPagosMercadoPagoPorCuota`
- `DiagnosticarPreferenciaMercadoPago`
- `DiagnosticarOrdenMercadoPagoPorPreferencia`

Uso:

- Diagnosticar rechazo o aprobacion de Mercado Pago sin depender solo del navegador.
- Obtener JSON crudo para enviar a soporte de Mercado Pago.

## Prompt Inicial de Referencia

El prompt original del proyecto esta en:

```text
C:\Users\andres.cabrera\Downloads\atenas_prompt_v4.md
```

Resumen del prompt:

- Construir sistema completo de gestion de socios del Club Atletico Atenas.
- Portal publico sin login.
- Backoffice administrativo.
- Cedula como identificador natural.
- Transacciones principales: `Socio`, `TipoCuota`, `Cuota`.
- Mercado Pago para cobro de cuotas.
- Importacion de socios desde Excel.
- Diseno institucional con azul marino y rojo.
- Mobile-first en portal del socio.

Nota: el prompt inicial indicaba otra KB/directorio de arranque. La KB activa actual es `Caatenas1928` en:

```text
C:\Users\andres.cabrera\GeneXus\KBs\Caatenas1928
```

## Handoff Anterior

Handoff leido:

```text
C:\Users\andres.cabrera\GeneXus\KBs\Caatenas1928\HANDOFF_2026-04-28.md
```

Punto clave del handoff:

- `AdminCuotas` tenia un problema de layout/catalogo.
- Se decidio avanzar con `AdminCuotasNuevo` para no seguir chocando contra ese objeto.
- El backoffice restante estaba razonablemente estable.

## Pendientes Recomendados

Prioridad alta:

1. Blindar `ConfirmacionPago` y `WebhookMercadoPago`:
   - Idempotencia.
   - Validacion de `payment_id`.
   - Validacion de `external_reference`.
   - Validacion de monto y moneda.
   - Evitar doble mail/comprobante.
2. Mejorar auditoria de pagos:
   - `payment_id`
   - `preference_id`
   - `merchant_order_id`
   - estado MP
   - fecha
   - monto
   - medio de pago
3. Esperar respuesta de Mercado Pago sobre tarjetas sandbox.

Prioridad media:

1. Pulir UX de comprobante.
2. Mejorar visuales menores de backoffice.
3. Revisar importacion de socios con archivos reales.
4. Automatizar generacion mensual de cuotas.

Prioridad baja:

1. Limpieza de objetos historicos si ya no se usan.
2. Revisar consistencia entre panels web y sdpanels.

## Como Arrancar una Nueva Sesion

1. Confirmar MCP:

```powershell
codex mcp list
```

2. Confirmar repo:

```powershell
git -C "C:\Users\andres.cabrera\GeneXus\KBs\Caatenas1928\src" status --short
```

3. Usar skill `nexa`.
4. Abrir la KB:

```text
C:\Users\andres.cabrera\GeneXus\KBs\Caatenas1928
```

5. Trabajar con MCP usando:

```text
rootDirectory = C:\Users\andres.cabrera\GeneXus\KBs\Caatenas1928
```

6. No tocar `NetModel`.
7. No hacer build/reorg sin confirmacion.
8. Despues de importar cambios, pedir al usuario que haga build desde IDE.

## URLs Locales Usadas

Base local:

```text
http://localhost:8082/Caatenas1928.NET
```

Portal:

```text
http://localhost:8082/Caatenas1928.NET/buscarsocio
```

Configuracion pago:

```text
http://localhost:8082/Caatenas1928.NET/adminconfiguracionpago
```

Procesar sandbox:

```text
http://localhost:8082/Caatenas1928.NET/adminprocesarpagosandbox
```

## Criterios de Cuidado

- No asumir que un import MCP funciono solo porque devuelve OK: reexportar o probar.
- No usar `Return` en eventos de Web Panel para cortar validaciones de usuario.
- No tocar DB/reorg/build sin autorizacion.
- No commitear secretos.
- No recrear `AdminCuotas` accidentalmente.
- No romper flujo ya probado de `BuscarSocio`, `DatosNuevoSocio`, `PantallaPago`, `ConfirmacionPago`.
- Separar problemas de Mercado Pago de problemas de la app: si `account_money` funciona, el circuito base esta probado.
