# zzzz-mail-templates

Shopware-6-Mail-Templates (Twig/HTML) für FourZed – Bestell- und Versandmails auf Deutsch und Englisch.
Design-Quelle: Figma „Email Templates & Flows“.

## Templates

| Nr.  | Datei                               | Shopware-Mail-Typ                | Flow-Trigger                                | Figma     |
|------|-------------------------------------|----------------------------------|---------------------------------------------|-----------|
| 1.1a | `1.1a-order-confirmation-prepayment`| `order_confirmation_mail`        | `checkout.order.placed`                     | CUSTOM    |
| 1.1b | `1.1b-order-confirmation-instant`   | `order_confirmation_mail`        | `checkout.order.placed`                     | CUSTOM    |
| 1.1c | `1.1c-order-confirmation-ratepay`   | `order_confirmation_mail`        | `checkout.order.placed`                     | CUSTOM    |
| 1.2  | `1.2-shipping-confirmation`         | `order_delivery.state.shipped`   | `state_enter.order_delivery.state.shipped`  | CUSTOM    |
| 1.3  | `1.3-partial-shipment`              | `order_delivery.state.shipped`   | `state_enter.order_delivery.state.shipped`  | DEFAULT   |
| 1.4  | `1.4-order-cancelled`               | `order.state.cancelled`          | `state_enter.order.state.cancelled`         | DEFAULT   |
| 1.5  | `1.5-order-completed`               | `order.state.completed`          | `state_enter.order.state.completed`         | DEFAULT   |

Jedes Template gibt es als `.de.html` und `.en.html`. Unter `_preview/` liegen dieselben Mails mit Beispieldaten gerendert, zum Anschauen im Browser.

## Regelbaum (Flow Builder / Rule Builder)

```
Flow „Bestellung aufgegeben“  ·  checkout.order.placed
├── WENN Regel „Zahlungsart = Vorkasse“                 → 1.1a Bestellbestätigung Vorkasse
├── SONST WENN Regel „Zahlungsart = Ratepay Rechnung“   → 1.1c Bestellbestätigung Ratepay
└── SONST (PayPal, Karte, … – sofort bezahlt)           → 1.1b Bestellbestätigung Instant

Flow „Lieferung versandt“  ·  state_enter.order_delivery.state.shipped
├── WENN Regel „Bestellung hat mehrere Lieferungen“     → 1.3 Teilversand   (Karla-Link nur für dieses Paket)
└── SONST                                               → 1.2 Versandbestätigung (Karla-Link)

Flow „Bestellung storniert“  ·  state_enter.order.state.cancelled    → 1.4 Bestellung storniert
Flow „Bestellung abgeschlossen“  ·  state_enter.order.state.completed → 1.5 Bestellung abgeschlossen
```

- Die Mails 1.1a/b/c enthalten **keinen** Tracking-Link. Tracking gibt es nur in 1.2 und 1.3.
- Tracking läuft über Karla: `{shopUrl}/tracking/?orderNumber=…&zipCode=…&ref=karla`. Eine Carrier-Sendungsnummer ist nicht nötig.
- Für die Regel „mehrere Lieferungen“ bringt der Rule Builder standardmäßig keine Bedingung mit (für 6.7 prüfen). Es braucht also eine eigene Regel, oder 1.3 wird an den Status `order_delivery.state.shipped_partially` gehängt.

## Einbau in Shopware

1. Admin → Einstellungen → E-Mail-Templates → neues Template zum passenden Mail-Typ anlegen und das HTML einfügen (DE/EN je Sprache).
2. Admin → Einstellungen → Flow Builder → Flow gemäß Regelbaum anlegen und als Aktion „E-Mail senden“ mit dem Template wählen.
3. Regeln (Zahlungsart, Lieferungen) vorher im Rule Builder anlegen.

## Konfiguration

Die Werte stehen oben in jedem Template als `{% set %}`-Block:

- `shopUrl`, `shopName`, `supportMail`, Social-Links, Rechtstexte-URLs
- `paymentDays = 7` (Vorkasse-Zahlungsfrist, auch in 1.4 zitiert), `paymentTermDays = 14` (Ratepay)
- 1.1a: Bankverbindung (`bankIban`, `bankBic`, …) – **ist ein Platzhalter**, vor Go-Live durch echte Daten ersetzen.

## Offen

- **Assets:** Die PNGs (Pattern-Band, Wordmark, Icons) aus Figma exportieren und nach `{shopUrl}/media/email` hochladen. Die Liste steht als `TODO assets` in jedem Template. Keine SVGs verwenden, weil Outlook und Gmail sie nicht rendern.
- **1.5:** Der Figma-Frame ist eine Kopie von 1.2. Der Text ist deshalb ein Vorschlag (Rückgabefenster statt Tracking).
- **1.3:** Die Regel bzw. der Trigger für Teilversand ist noch festzulegen (siehe oben).
