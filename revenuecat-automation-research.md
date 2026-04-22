# RevenueCat Offerings via REST API — Automation Research

**Datum:** 2026-04-21
**Frage:** Kann Claude autonom ein Offering + Product-Assignment anlegen, ohne Jurek's Dashboard-Login?
**Kurzantwort:** Technisch JA (v2 REST API), aber Jurek muss einmalig einen Secret API Key + Project ID liefern. Ohne Key = **NEIN**.

---

## 1. Gibt es die API?

**Ja.** RevenueCat REST API v2 hat volle CRUD-Endpoints für Offerings, Packages, Products, Entitlements.

Base URL: `https://api.revenuecat.com/v2`

### Relevante Endpoints

| Operation                       | Method + Path                                                                                      |
|---------------------------------|----------------------------------------------------------------------------------------------------|
| Offering anlegen                | `POST /v2/projects/{project_id}/offerings`                                                         |
| Package anlegen                 | `POST /v2/projects/{project_id}/offerings/{offering_id}/packages`                                  |
| Product an Package attachen     | `POST /v2/projects/{project_id}/offerings/{offering_id}/packages/{package_id}/actions/attach_products` |
| Product an Entitlement attachen | `POST /v2/projects/{project_id}/entitlements/{entitlement_id}/actions/attach_products`             |
| Product listen (bereits reg.)   | `GET  /v2/projects/{project_id}/products`                                                          |
| Entitlements listen             | `GET  /v2/projects/{project_id}/entitlements`                                                      |

Rate Limit: 60 req/min (Project Configuration domain).

## 2. Welcher API Key?

**Secret API Key (Prefix `sk_`)** — MUSS. Public App-Keys (`goog_…`, `appl_…`) funktionieren NICHT für v2.

- Secret Keys werden im Dashboard erzeugt unter **Project settings → API keys → + New secret API key**.
- Beim Erzeugen: **Version = V2** wählen + Permissions für `project_configuration:offerings:*`, `project_configuration:packages:*`, `project_configuration:products:read`, `project_configuration:entitlements:*`.
- Format: `sk_xxxxxxxxxxxxxxxxxxxxxxxx`.

Jureks aktuelle Keys in `app.config.js` sind nur:
- `EXPO_PUBLIC_REVENUECAT_ANDROID_API_KEY` (= public `goog_…`, nur SDK-Fetch, KEIN Schreibzugriff)
- `EXPO_PUBLIC_REVENUECAT_IOS_API_KEY` (= public `appl_…`, dito)
- `EXPO_PUBLIC_REVENUECAT_ENTITLEMENT_ID`

Kein Secret Key vorhanden — **nicht in `.claude/`, nicht in `app.config.js`, kein `.env` im em-app Ordner**.

## 3. Konkrete curl-Commands

### Variables (Jurek muss liefern)
```bash
export RC_SK="sk_XXXXXXXXXXXXXXXXXXXX"          # Secret v2 API Key (Dashboard)
export RC_PROJECT_ID="proj_XXXXXXXX"            # Dashboard → Project settings → General
export RC_PRODUCT_ID="em_schattenkurs"          # Play Store product id
export RC_ENTITLEMENT_ID="pro"                  # RC-interne ID (NICHT der Anzeigename!)
```

### Step 1 — Check ob Product überhaupt in RC registriert ist
```bash
curl -s "https://api.revenuecat.com/v2/projects/$RC_PROJECT_ID/products" \
  -H "Authorization: Bearer $RC_SK" \
  -H "Accept: application/json"
# suchen nach "store_identifier": "em_schattenkurs"
```
Wenn leer → Product ist im RC Dashboard nicht registriert. Das ist vermutlich die Root Cause vom Fehler.

### Step 2 — Offering anlegen
```bash
curl -X POST "https://api.revenuecat.com/v2/projects/$RC_PROJECT_ID/offerings" \
  -H "Authorization: Bearer $RC_SK" \
  -H "Content-Type: application/json" \
  -d '{
    "lookup_key": "default",
    "display_name": "Emotionale Meisterschaft Pro",
    "is_current": true
  }'
# Response enthält "id": "ofrng_XXXX" → als $RC_OFFERING_ID merken
```

### Step 3 — Package anlegen
```bash
curl -X POST "https://api.revenuecat.com/v2/projects/$RC_PROJECT_ID/offerings/$RC_OFFERING_ID/packages" \
  -H "Authorization: Bearer $RC_SK" \
  -H "Content-Type: application/json" \
  -d '{
    "lookup_key": "$rc_lifetime",
    "display_name": "Lifetime Access",
    "position": 1
  }'
# Response → "id": "pkg_XXXX"
```
Lookup-Keys sind vordefiniert: `$rc_monthly`, `$rc_annual`, `$rc_lifetime`, `$rc_weekly`, etc. Custom erlaubt.

### Step 4 — Product an Package attachen
```bash
# Erst Product-RC-ID ermitteln (aus Step 1 "id"-Feld, nicht store_identifier)
curl -X POST "https://api.revenuecat.com/v2/projects/$RC_PROJECT_ID/offerings/$RC_OFFERING_ID/packages/$RC_PACKAGE_ID/actions/attach_products" \
  -H "Authorization: Bearer $RC_SK" \
  -H "Content-Type: application/json" \
  -d '{
    "products": [{"product_id": "prod_XXXXXXXX"}]
  }'
```

### Step 5 — Product an Entitlement attachen
```bash
curl -X POST "https://api.revenuecat.com/v2/projects/$RC_PROJECT_ID/entitlements/$RC_ENTITLEMENT_ID/actions/attach_products" \
  -H "Authorization: Bearer $RC_SK" \
  -H "Content-Type: application/json" \
  -d '{
    "product_ids": ["prod_XXXXXXXX"]
  }'
```

## 4. Blocker: Product-Registration ist NICHT via API möglich (in den meisten Fällen)

**WICHTIG:** Der Fehler `"no Play Store products registered in the RevenueCat dashboard"` bedeutet: Product `em_schattenkurs` ist im RC Dashboard noch nicht als Product-Eintrag importiert.

Products werden via **Play Store Import** registriert — RC liest die Products direkt aus der Google Play API (mit dem Service Account JSON, das Jurek hochgeladen hat). Das passiert im Dashboard unter **Products → Import Products from Play Store**. Via REST API nur begrenzt möglich (v2 hat `POST /products` aber der empfohlene Weg bleibt Store-Import, weil Preise/Localizations gezogen werden müssen).

---

## 5. Dashboard-Weg (Fallback, wenn kein Secret Key)

**Pfad in RevenueCat Dashboard** (app.revenuecat.com):

1. **Login** → Project auswählen (Emotionale Meisterschaft)
2. **Products** (Sidebar) → **"+ New"** oder **"Import from Play Store"**
   - Wenn `em_schattenkurs` fehlt: Import klicken → Product auswählen → Speichern
   - Wenn vorhanden: weiter
3. **Entitlements** (Sidebar) → prüfen ob eines mit ID `pro` (oder whatever `EXPO_PUBLIC_REVENUECAT_ENTITLEMENT_ID` gesetzt ist) existiert
   - Wenn nicht: **"+ New Entitlement"** → ID: `pro` → Display Name: "Emotionale Meisterschaft Pro" → Save
   - Product verknüpfen: Entitlement öffnen → **Attach Products** → `em_schattenkurs` auswählen
4. **Offerings** (Sidebar) → **"+ New Offering"**
   - Identifier: `default` → Display Name: "Emotionale Meisterschaft Pro" → Save
   - Als "Current" markieren (Toggle)
5. **Offering öffnen** → **"+ New Package"**
   - Identifier: `$rc_lifetime` (oder passend zu Product-Typ)
   - Attach Product: `em_schattenkurs`
6. **App** neu laden → `Purchases.getOfferings()` sollte jetzt das Offering liefern

**Dauer:** 3-7 min wenn Service Account + Play Store Products bereits durchlaufen.

---

## 6. Was Jurek JETZT klären / liefern muss

Das kann Claude **nicht** autonom checken:

- [ ] **Dashboard-Screenshot** von `Products` — ist `em_schattenkurs` dort? Status "Ready"?
- [ ] **Dashboard-Screenshot** von `Entitlements` — existiert eines? Welche ID? Ist Product verknüpft?
- [ ] **Dashboard-Screenshot** von `Offerings` — leer? oder existiert eines das nicht als "Current" markiert ist?
- [ ] **Project ID** aus Project settings → General (Format: `proj_xxxxxxxx`)
- [ ] **Secret API Key erzeugen** (falls Automation-Weg gewünscht) → Format `sk_xxx`, Version V2, Permissions: `project_configuration:*` (write)
- [ ] **Service Account JSON für Play Store** bereits in RC hochgeladen? (Settings → Apps → Play Store App → Service Account)

Ohne diese Infos: autonom NICHT lösbar.

Mit Secret Key + Project ID → Claude kann alles außer dem Play-Store-Product-Import scripten.

## 7. Alternative — RevenueCat MCP Server

Jurek hat den `revenuecat` Skill installiert (vermutlich via RC MCP: read-only Metrics + Docs). Der offizielle RevenueCat MCP (`@revenuecat/mcp` bzw. https://github.com/iamhenry/revenuecat-mcp) kann v2 Writes wenn mit Secret Key konfiguriert. Jureks aktueller `revenuecat` Skill ist laut Beschreibung **read-only** ("metrics, customer data, docs"). Für Write-Ops bräuchte er einen zusätzlichen MCP-Server mit Secret-Key-Config oder direkte curl-Calls.

---

## Saved to `.env.revenuecat.example`

Siehe separate Datei: `C:/Users/Jurek/emotionale-meisterschaft-app/.env.revenuecat.example`

---

**Autonom fixbar: NO** (ohne Secret Key + Project ID von Jurek). Mit beiden Werten → YES, komplett scriptbar in ~2 min.

## Quellen

- [RevenueCat API v2 Docs](https://www.revenuecat.com/docs/api-v2)
- [RevenueCat API Reference](https://www.revenuecat.com/reference/revenuecat-rest-api)
- [API Keys & Authentication](https://www.revenuecat.com/docs/projects/authentication)
- [Product Setup Tutorial](https://www.revenuecat.com/tutorials/using-revenuecats-new-rest-api-product-setup/)
- [RevenueCat MCP Blog](https://www.revenuecat.com/blog/company/introducing-revenuecat-mcp/)
- [RevenueCat MCP GitHub (community)](https://github.com/iamhenry/revenuecat-mcp)
