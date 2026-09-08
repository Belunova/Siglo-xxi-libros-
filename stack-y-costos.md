# Siglo XXI Libros — Estructura, Stack Tecnológico y Costos

## 1. Arquitectura general del proyecto

```
                    ┌─────────────────────────┐
                    │   CATÁLOGO MAESTRO       │
                    │ (título, ISBN, precio,   │
                    │  stock, origen, portada) │
                    └───────────┬──────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
   Alta automática         Alta manual            Fuentes externas
   (scrape Dalsa +         (mostrador,             (Google Books:
   detección de mail       ISBN scan)              portada + sinopsis)
   enviado a Dalsa)
        │                       │                       │
        └───────────────────────┴───────────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
  WOOCOMMERCE (venta)    PANEL INTERNO            CUENTA SOCIA
  WordPress + plugin     Airtable — Socias/       Airtable Interface o
  Points&Rewards         préstamos · Pedidos/     portal WooCommerce —
  Home·Catálogo·Ficha·   stock · Ventas           Chequera·Sellos·
  Carrito·Pago·Pedidos                             Mis pedidos
        │                       │
        └───────────┬───────────┘
                     │
    Mercado Pago (plugin oficial) + Make (sincroniza stock/pedidos)
         → WhatsApp / Mail (avisos por canal preferido)
```

**Los 3 sistemas se despliegan en paralelo, no en etapas** — se construyen y lanzan juntos:

| Sistema | Para quién | Se implementa en |
|---|---|---|
| E-commerce | Cualquier visitante | WordPress + WooCommerce |
| Panel interno | Vendedora/dueña | Airtable |
| Cuenta socia | Socias con membresía | Dentro de "Mi cuenta" de WooCommerce (custom endpoint), con Airtable como motor de datos atrás — Make sincroniza en ambas direcciones |

---

## 2. Stack tecnológico por capa

| Capa | Herramienta | Por qué |
|---|---|---|
| **Base de datos / Catálogo Maestro** | Airtable | Vista tipo app para la vendedora (no una planilla fría), automations nativas para sellos/chequera, buen soporte de campo tipo "escáner de código de barras" desde celular |
| **E-commerce (tienda, carrito, checkout)** | WordPress + WooCommerce | Ecosistema maduro de e-commerce (15+ años), no reinventa carrito/checkout/cupones desde cero, plugin oficial de Mercado Pago, mejor SEO de fábrica, mantenible por cualquier desarrollador WordPress a futuro |
| **Fidelización (sellos → 50% OFF)** | Plugin de Points & Rewards (WooCommerce) | Reemplaza directo lo que mockeamos a mano — hay versión gratuita para arrancar, con upgrade pago si se necesita más adelante |
| **Chequera de préstamo (retirar libro, 30 días, validado por plan)** | Custom — Airtable + Make | Esto no lo resuelve ningún plugin porque es un modelo de negocio propio de Siglo XXI, no un e-commerce estándar |
| **Automatizaciones** | Make | Scraping del catálogo Dalsa, detección de mail enviado, OCR de remitos, sincronización de stock WooCommerce ↔ Airtable, notificaciones por canal preferido |
| **Pagos** | Mercado Pago (plugin oficial de WooCommerce) | Estándar en Argentina, mantenido por MP mismo — no hay que programar la integración a mano |
| **Portadas y sinopsis** | Google Books API | Gratuita, buena cobertura en español, un solo llamado trae título/autor/editorial/sinopsis/portada |
| **Notificaciones** | Gmail (Make) + WhatsApp (Whapi, opcional) | Según el canal preferido que elija cada socia/cliente en el alta |
| **Hosting** | Hosting WordPress administrado (ej. SiteGround, Hostinger Business) | Necesario para correr WordPress/WooCommerce — no aplica el hosting de Lovable/Netlify en este esquema |

---

## 3. Costos mensuales estimados (infraestructura, en USD)

Estos son costos de **servicios/licencias**, no incluyen el trabajo de diseño e implementación. Son precios de lista verificados en las páginas oficiales — Belu, conviene reconfirmarlos al momento de contratar porque cambian con frecuencia.

### Costo mensual — sistema completo (venta + panel interno + cuenta socia)

| Servicio | Plan sugerido | Costo mensual |
|---|---|---|
| Airtable | Team (1 editora) | ~US$20–24 |
| Hosting WordPress administrado | Plan business (soporta WooCommerce sin fricción) | ~US$10–25 |
| WordPress + WooCommerce (core) | — | US$0 (open source) |
| Plugin Mercado Pago para WooCommerce | Oficial | US$0 |
| Plugin Points & Rewards (sellos) | Versión gratuita para arrancar | US$0 (upgrade pago opcional ~US$8-16/mes si se necesita más adelante) |
| Make | Core (10.000 operaciones) | ~US$10,59 |
| Google Books API | Gratis | US$0 |
| Dominio (.com.ar) | — | ~US$10–15/año (no mensual) |
| **Total aproximado** | | **~US$41–60/mes** |

**Nota sobre volumen:** si con las 3 automatizaciones corriendo juntas (catálogo, chequera/sellos, ventas) se supera el volumen de operaciones de Make Core, el siguiente escalón es Pro (~US$18,82/mes) — no debería hacer falta al arrancar, pero vale la pena tenerlo previsto. Si además se activa WhatsApp (Whapi) como canal de aviso, ese costo se cotiza aparte según el proveedor.

### Costo variable por venta (no es un abono, es por transacción)

| Concepto | Costo |
|---|---|
| Mercado Pago — tarjeta de crédito (acreditación inmediata) | ~3,99% + IVA por venta |
| Mercado Pago — tarjeta de débito | ~3,29% + IVA por venta |
| Cuotas sin interés (si se activa) | Comisión adicional variable según cantidad de cuotas |

---

## 4. Supuestos y lo que falta confirmar

- Los precios de Airtable y Make están **facturados anualmente**; pagando mes a mes suelen ser ~20% más caros.
- No incluye el trabajo de diseño, desarrollo e implementación del proyecto (eso va en el Project Charter aparte, con el pricing de NOVAH).
- No incluye WhatsApp Business API / Whapi de forma fija — se cotiza según el proveedor una vez definido si el canal se activa.
- El dominio (.com.ar o el que corresponda) es un costo anual, no mensual — lo dejé aparte para no mezclarlo con el abono mensual.
- Los % de Mercado Pago pueden variar por promociones temporales (por ejemplo, comisión reducida los primeros 30 días de cuenta nueva) — vale la pena confirmarlo en el momento de dar de alta la cuenta.
- **La chequera de préstamo no tiene equivalente en plugins de WooCommerce** — es desarrollo custom vía Airtable + Make, sin importar qué frontend se use. El plugin de Points & Rewards solo cubre la parte de sellos/fidelización, no el préstamo.
- El diseño visual custom que ya mockeamos (la cinta de frases, el talonario, la paleta del logo) se implementa en WordPress vía child theme + CSS — es más trabajo de implementación que en un builder tipo Lovable, pero sin dependencia de un proveedor específico a futuro.
- **La cuenta de socia integrada a "Mi cuenta" de WooCommerce** (chequera + sellos + pedidos en una sola pestaña dentro del sitio) requiere un endpoint custom de WooCommerce que lea/escriba en Airtable vía Make — esto suma horas de desarrollo respecto a tenerlo como un Airtable Interface separado, pero evita que la socia tenga que saltar a otra URL para ver su cuenta. Como los 3 sistemas se construyen en paralelo, esto va contemplado desde el arranque del proyecto, no como un agregado posterior.

## 5. Extensibilidad futura

- **Facturación electrónica (ERP / AFIP):** el stack está armado para sumar esto después sin rehacer nada — cada venta ya pasa por Make, así que solo hay que agregar un paso que dispare la creación de la factura hacia el ERP que use la clienta (Tango, Xubio, Colppy, ContaPYMEs, etc., si tienen API) o directo contra el webservice de AFIP (WSFE) si no. Falta confirmar cuál sistema usa o piensa usar la librería para dimensionar el esfuerzo real.
