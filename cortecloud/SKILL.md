---
name: cortecloud
description: >-
  Usar para cualquier trabajo en Cortecloud (cortecloud.com, cualquier
  país/tienda): crear o editar servicios de despiece o módulos 3D (Hellomob),
  cualquier mueble o proyecto, asignar tableros/cantos, importar piezas, cotizar
  en una o varias tiendas, ver aprovechamiento/sobrante o Generar piezas. No hay
  conector MCP.
---
# Cortecloud

Plataforma de corte de tableros: el usuario diseña (piezas sueltas o módulos 3D), elige una tienda/barraca/maderera y la tienda cotiza, corta y canta. **No hay conector MCP**: todo va por el navegador de la computadora con un subagente `computerUse`. Archivos (CSV/JSON de piezas, croquis) se generan con Shell.

## 0. Antes de tocar el navegador
1. Cargar `box-desktop`.
2. Leer memoria del usuario: país, zona o tiendas preferidas, material habitual, medidas estándar, servicios existentes (número `#`), email de la cuenta Cortecloud si se conoce.
3. Resolver la cuenta (§1) antes del primer dispatch.
4. **Hard stops en todo dispatch:** no Autorizar, Pagar, Confirmar pedido ni aceptar términos de compra sin ok explícito del usuario en este chat. *Generar presupuesto* y *Generar piezas* solo cuando el usuario lo pide (no cobran, pero pueden mandar el pedido a revisión de la tienda).

## 1. Cuenta y login (genérico, nunca credenciales en la skill)
Cargar `sign-in` y `vault` y seguir su orden:
1. **Vault:** `Vault list` kind login y buscar una entrada para `cortecloud.com` / `professional.cortecloud.com`. Si hay, pasar al subagente las **fill references exactas** (`vault_<id>_username`, `vault_<id>_password`). Sin las referencias en el texto del task, el relleno seguro falla.
2. **Gestor de contraseñas:** si el vault no tiene, `GetCredentialProviderStatus`; con 1Password conectado, `RequestOnePasswordLogins` para Cortecloud.
3. **Preguntar al usuario:** si tiene cuenta pero no está guardada, que la cargue con el formulario seguro (`in-chat-forms`, `RequestUserForm` kind login; puede guardarla en el vault). Nunca pedir la contraseña en el chat.
4. **Sin cuenta:** preguntar si crear una. Datos que pide el registro: nombre, apellido, email, país, perfil profesional (p. ej. Aficionado / Carpintero / Diseñador) y a veces teléfono (algunas tiendas lo exigen para vincular). Contraseña generada con `Vault save` kind login (no inventarla). Verificar email si llega código (puede necesitar al usuario).
5. Login Google/SSO, 2FA o captcha que no se pueda pasar: `request_box_help`.
- Teléfono, dirección o email del usuario: tomarlos del vault, no de memoria.
- La sesión puede caerse (sobre todo al Guardar en Hellomob): reentrar con la misma credencial, verificar si el cambio quedó antes de repetirlo.

## 2. URLs
- `https://cortecloud.com` puede abrir un selector de país; elegir el del usuario. La app vive en `https://professional.cortecloud.com/`.
- Lista de servicios: `#/plans`. Servicio: `#/plans/<id>/view` (pestañas Piezas, Clientes, Tableros, Tapacantos, Componentes, Sobras).
- Editor 3D Hellomob: Opciones → Editar proyecto → `#/hellomobweb/<id>`.

## 3. Elegir tienda
- Buscar tiendas por ciudad/zona al crear el servicio. Respetar la zona que pida el usuario.
- Cada tienda tiene su catálogo de tableros y cantos (marcas, espesores, tamaño de placa): el mismo material puede no existir en otra.
- Primera vez con una tienda: **«Solicitud de vínculo en revisión»** hasta que la tienda apruebe (llega mail). Sin totales mientras tanto. Dar al usuario el teléfono de la tienda si se muestra.
- **Un servicio queda atado a su tienda: no se puede duplicar ni mover.** Para otra tienda: servicio nuevo, reimportar piezas (§6) y reasignar materiales de su catálogo.

## 4. Tipos de servicio
| Tipo | Cuándo | Notas |
|---|---|---|
| Servicio completo / despiece | Ya hay lista de piezas (croquis, DXF, cálculo) | Piezas L×A, cantidad, material, cantos por lado. |
| Servicio con módulos (Hellomob) | Diseñar en 3D desde biblioteca (placares, cocinas, baño, escritorios, estanterías…) | Genera piezas con *Generar piezas*. |
| Otros (producir N veces, sobras, componentes) | Repetir un servicio, usar retazos propios, herrajes | Ver §7. |

Si el usuario aprobó cotas exactas, el despiece es más fiel; Hellomob es más rápido pero impone sus módulos (holguras, suplementos, rodapiés en pasos fijos).

## 5. Flujo recomendado (cualquier mueble)
1. **Requisitos y croquis** con cotas (Shell/Python → PNG legible; DXF si lo piden). Ok del usuario antes de cargar.
2. **Despiece** propio en JSON/CSV: función, L, A, cantidad, material, cantos (L1, L2, A1, A2), mueble/cliente. Sumar m² por material.
3. **Crear o abrir el servicio** en la tienda correcta.
4. Cargar piezas (§6) o armar módulos (§8). Guardar seguido.
5. **Materiales** a todo (§9).
6. Primer mueble → **captura al usuario** antes de replicar.
7. Cotizar / *Generar presupuesto* / *Generar piezas* solo a pedido. Reportar totales, estado y capturas.

## 6. Cargar piezas (despiece)
- Preferir **importar** desde archivo (CSV/Excel/JSON que acepte la tienda) generado con Shell; verificar conteo y m² tras importar.
- Reutilizar el mismo archivo para otra tienda.
- Revisar cantos en los lados correctos.
- «Consumo» en Tableros = **m² de piezas**, no cantidad de placas.

## 7. Presupuesto, aprovechamiento y sobrantes
- **Placas, nesting y % de aprovechamiento aparecen recién tras *Generar presupuesto* (o *Generar piezas*)**. Antes, estimar: m² piezas ÷ m² placa = mínimo teórico; real +10–20 % por sierra, veta y piezas largas. Aclarar que es estimación.
- Pestaña **Sobras**: cargar *retazos que el usuario ya tiene* (tablero + medidas) para aprovecharlos; no muestra el sobrante del corte.
- Totales suelen separar material y servicio (corte + canto); herrajes, mecanizado y flete normalmente **no** incluidos.
- Estados: Guardado, Diseñando, Solicitud de vínculo en revisión, Esperando autorización (= falta que el usuario Autorice: hard stop).
- Comparar tiendas: un servicio por tienda con el mismo despiece; tabla con material, total, placas, % sobrante, plazo.

## 8. Hellomob (módulos 3D)
- Editor lento (20–60 s por cambio); si Playwright da timeout, usar clicks de Computer.
- Ajustar: medidas generales, rodapié (dropdown en pasos, p. ej. 50/60/70/80 mm: elegir el más cercano y avisar), columnas y sub-módulos (repisas, cajones, percheros, divisores).
- **Colisiones** («Piezas colisionando con componentes», 3D rojo/naranja) bloquean *Generar piezas*: resolver antes.
- Tras cambiar sub-módulos, revisar que todas las familias tengan tablero (Cuerpo, Repisa, Fondo, Recubrimiento, Separador, Cajones, Frentes, Fondo cajones…).
- Duplicar un módulo dentro del servicio para un mueble parecido; borrar módulos viejos.

### Patrones aprendidos (placares)
- Puertas corredizas integradas: **«Armario corredero con N columnas (puertas apoyadas/suspendidas)»**. «Armario con columnas editables» no trae puertas.
- Repisas a todo el ancho de un vano: dividir el cuerpo y usar sub-módulo **Repisas**, no ensanchar el vano de un «perchero con repisa inferior» (colisiona).
- Cajones en corredero: suplementos laterales «Sin» suele colisionar; «Ambos» funciona. Avisar al usuario.
- Fondo de cajón 3–6 mm; cuerpo/repisas en el espesor principal.

## 9. Materiales y cantos
- Buscar por espesor + color/nombre; si la marca pedida no está, proponer la equivalente más cercana y decirlo.
- Canto: espesor (0.4, 1, 2 mm) y ancho ≥ espesor del tablero + margen (22 mm para 18; algunas tiendas solo 45).
- Anotar tamaño de placa (p. ej. 2750×1830/1850) para estimaciones.

## 10. Dispatch al subagente (plantilla)
Incluir: URL exacta o tienda + tipo a crear; fill references de la credencial (o «ya logueado, si pide login reportar URL y parar»); medidas en mm (W×H×D); layout interno; material/espesor/canto; ruta del archivo a importar; hard stops; qué reportar (estado, piezas/módulos, m², totales, colisiones, rutas de capturas). Un mueble o un paso por dispatch.

## 11. Errores comunes
| Síntoma | Solución |
|---|---|
| Selector de país | Elegir país o ir a professional.cortecloud.com |
| No hay credencial | §1: vault → 1Password → formulario seguro → crear cuenta |
| Sesión cae al Guardar | Reentrar, verificar, reintentar |
| Sin totales | Vínculo en revisión o falta *Generar presupuesto* |
| Piezas colisionando | Cambiar sub-módulo / suplementos / holguras |
| 3D naranja / «Selecciona el tablero» | Reasignar material a esa familia |
| Material no existe en tienda | Equivalente del catálogo, avisar |
| Otra tienda | Servicio nuevo + reimportar piezas |

## 12. Al terminar
Guardar en memoria número de servicio, tienda, estado y decisiones del usuario (no credenciales ni teléfonos). Sumar a esta skill aprendizajes genéricos, sin datos personales.
