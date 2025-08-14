# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This is a Shopware 6 e-commerce platform for "1001 Frucht" (organic/fruit products store). The codebase uses a custom theme (Act1001FruchtTheme) that extends Shopware's base Storefront theme.

### Environments
- **Staging:** https://staging.1001frucht.de/ (Development environment, Shopware 6.7)
- **Live:** https://www.1001frucht.de/ (Production environment, Shopware 6.5, visual reference)

## Essential Commands

### Theme Development
```bash
# Navigate to theme directory
cd custom/plugins/Act1001FruchtTheme/src/Resources/app/storefront

# Install dependencies
npm install

# Lint SCSS files
npm run lint:scss

# Fix SCSS linting issues
npm run lint:scss-fix

# Run Lighthouse CI tests
npm run ci:lighthouse
```

### Shopware Commands
```bash
# Clear cache
bin/console cache:clear

# Compile theme
bin/console theme:compile

# Build storefront
bin/build-storefront.sh

# Run migrations
bin/console database:migrate --all
```

## Architecture & Key Patterns

### Theme Structure
The custom theme is located at `/custom/plugins/Act1001FruchtTheme/` and follows Shopware's theme structure:
- `src/Resources/views/storefront/` - Twig templates that override base templates
- `src/Resources/app/storefront/src/scss/` - SCSS files for styling
- `src/Resources/app/storefront/src/js/` - JavaScript plugins and components
- `src/Resources/config/` - Theme configuration

### Template Inheritance
Templates extend from Shopware's base using Twig's inheritance:
```twig
{% sw_extends '@Storefront/storefront/base.html.twig' %}
```

### JavaScript Plugin Pattern
Custom JS plugins follow Shopware's plugin system:
```javascript
export default class MyPlugin extends Plugin {
    init() {
        // Plugin initialization
    }
}
```

### Key Integration Points
- Payment providers configured in `/custom/plugins/` (PayPal, Mollie, Klarna, etc.)
- Pickware ERP integration for inventory and shipping
- Multiple marketing/analytics integrations (Google Analytics, Tag Manager)

### File Locations
- Media files: `/files/media/`
- Backups: `/files/backup/`
- Logs: `/var/log/`
- Cache: `/var/cache/`
- Compiled theme assets: `/public/theme/`

## Shopware 6.7 Migration Strategy

### Current Status
Das Act1001FruchtTheme wurde ursprünglich für Shopware 6.5 entwickelt und muss für die Kompatibilität mit Shopware 6.7 angepasst werden. Shopware 6.7 hat die Struktur der Produktdetailseiten grundlegend geändert.

### Arbeitsweise & Rolle von Claude
- **Beratend & Planend:** Claude unterstützt strategische Entscheidungen und schlägt Vorgehensweisen vor
- **Kollaborativ:** Alle Änderungen werden gemeinsam besprochen, kein eigenständiges Arbeiten ohne Rücksprache
- **Iterativ:** Schrittweise Migration mit regelmäßiger Überprüfung und Tests
- **Dokumentiert:** Alle Entscheidungen und Änderungen werden hier festgehalten

### 🔄 Standard-Workflow für Template-Divergenzen

#### 1. **Problem-Meldung durch User**
User berichtet Divergenz zwischen Live-Shop (6.5) und Staging (6.7)

#### 2. **Analyse-Phase**
1. **Playwright E2E Testing** (optional): Visuelle Überprüfung der gemeldeten Divergenz
2. **Custom Template Analyse**:
   ```bash
   # Suche Custom Twig Templates für betroffenen Bereich
   find custom/plugins/Act1001FruchtTheme -name "*.html.twig" | grep [bereich]
   ```
3. **SCSS Analyse** (falls relevant):
   ```bash
   # Prüfe Custom SCSS für betroffene Komponenten
   grep -r "[komponente]" custom/plugins/Act1001FruchtTheme/src/Resources/app/storefront/src/scss/
   ```

#### 3. **Vendor-Vergleich**
1. **Shopware 6.7 Struktur verstehen**:
   ```bash
   # Analysiere Vendor Templates für neuen Ansatz
   find vendor/shopware/storefront -name "*.html.twig" | grep [bereich]
   ```
2. **Strukturelle Änderungen identifizieren**: Template-Pfade, neue Blöcke, geänderte Variablen

#### 4. **Lösungsfindung**
1. **Breaking Changes Research**: Bei Bedarf online nach "Shopware 6.7 breaking changes [komponente]" suchen
2. **Minimaler Eingriff**: Eleganteste Lösung mit wenigsten Änderungen bevorzugen
3. **Template-Konformität**: Neue Shopware 6.7 Strukturen nutzen statt alte zu erzwingen

#### 5. **Implementation**
1. **Neue Templates erstellen** oder bestehende anpassen
2. **Alte Templates markieren**: `UNUSED_` Präfix für nicht mehr greifende Templates
3. **Custom-Logik dokumentieren**: Wichtige Anpassungen für spätere Migration festhalten

#### 6. **Deployment & Test**
1. Upload zum Staging-Server
2. Theme kompilieren
3. Cache leeren
4. Funktionalität testen
5. **Playwright Testing:** Visuelle Überprüfung auf https://staging.1001frucht.de/
6. **Live-Vergleich:** Mit https://www.1001frucht.de/ vergleichen für visuelle Referenz

### 🎯 Kern-Prinzipien der Migration
1. **Vendor-First**: Immer zuerst verstehen, wie Shopware 6.7 es macht
2. **Minimal-Invasiv**: Kleinste mögliche Änderung für maximalen Effekt
3. **Dokumentation**: Jede Custom-Anpassung wird festgehalten
4. **UNUSED_ Convention**: Alte Templates bleiben zur Analyse erhalten
5. **Test-Driven**: Änderungen werden auf Staging getestet bevor Live-Deploy

### Kritische Strukturänderungen in 6.7
1. **Produktdetailseiten:** Nutzen jetzt hauptsächlich CMS-System statt dedizierte Templates
2. **Template-Pfad:** `/vendor/shopware/storefront/Resources/views/storefront/page/content/product-detail.html.twig`
3. **CMS-Architektur:** Rendering über CMS-Blöcke und -Elemente
4. **Navigation-System:** Migration von `/layout/navigation/` zu `/layout/navbar/` System

### Problematische Theme-Überschreibungen
- `/page/product-detail/index.html.twig` - Existiert nicht mehr in 6.7
- Direkter Include von `cms-element-image-gallery.html.twig` mit hardcodierten Parametern
- Veraltete Template-Referenzen aus älteren Shopware-Versionen

### Bekannte SCSS-Kompatibilitätsprobleme
- **`$sw-footer-text-color`:** Variable existiert nicht mehr in 6.7 → Ersetzt durch `$gray-700`
- **Fehlende Shopware-spezifische Variablen:** Viele `$sw-*` Variablen wurden entfernt
- **Bootstrap-Migration:** Standard Bootstrap-Variablen verwenden statt Shopware-spezifische

### Migrations-Phasen
1. **Bestandsaufnahme:** Template-Audit und Identifikation kritischer Überschreibungen
2. **CMS-Anpassung:** Refactoring der Produktdetailseite für CMS-Kompatibilität  
3. **Schrittweise Migration:** Template für Template unter gemeinsamer Beratung

### Navigation System Migration (✅ ABGESCHLOSSEN)

#### Problem
- Shopware 6.7 verwendet neues `navbar`-System statt `navigation`-System
- Mega-Dropdown wurde automatisch aktiviert mit Bootstrap-Klassen und JavaScript-Triggern
- Custom Templates griffen nicht mehr, da Pfadstruktur geändert wurde

#### Lösung (implementiert)
1. **Mega-Dropdown deaktiviert**: Neues Template `/layout/navbar/navbar.html.twig` mit `{% set hasChildren = false %}`
2. **Template aufgeräumt**: Altes Template als `UNUSED_navigation.html.twig` markiert
3. **Custom-Anpassungen dokumentiert**:
   ```twig
   {# Alte Custom-Logik aus UNUSED_navigation.html.twig: #}
   - {% block layout_main_navigation_menu_flyout_wrapper %}{% endblock %}  # Flyout komplett deaktiviert
   - Custom haveGrandChildren Logik für mehrstufige Navigation
   - Produktseiten-spezifische Navigation mit page.product.categoryTree
   - Deprecated category.id Handling für v6.5 Kompatibilität
   ```

#### Template Management Regel
**UNUSED_ Präfix für veraltete Templates**: Nicht mehr greifende Custom-Templates werden mit `UNUSED_` Präfix markiert → Erlaubt Analyse der Custom-Anpassungen → Relevante Logik wird in neue Template-Struktur migriert

### Grundsätze
- Immer vor Änderungen: git add → commit → push
- Jede Annahme sofort testen oder dokumentieren
- Unsicherheiten offen kommunizieren
- Templates gegen 6.7 Vendor-Struktur prüfen
- CMS-konforme Lösungen bevorzugen

### Workflow & Deployment
- **Lokale Entwicklung:** Alle Änderungen werden zuerst lokal im Repository gemacht
- **Keine direkten Server-Edits:** Niemals Dateien direkt auf dem Staging-Server bearbeiten
- **Synchronisation:** Lokale Änderungen werden zum Staging-Server übertragen
- **Testing:** Nach lokalen Änderungen erfolgt Test der Theme-Kompilierung auf dem Staging-System
- **Staging-Server:** server1.1001frucht.de (aus .vscode/sftp.json)

#### Deployment-Methoden
1. **SFTP Plugin (VS Code/Cursor):** 
   - Konfiguriert mit `"uploadOnSave": true` in `.vscode/sftp.json`
   - **Limitation:** Funktioniert nur bei Speicherungen über IDE (Cursor/VS Code)
   - **Nicht aktiv:** Bei Terminal-basierten Datei-Edits (Edit-Tool)

2. **Manueller Upload (Terminal):**
   ```bash
   # Einzelne Datei
   scp [lokaler-pfad] c864675_w11_ssh1@server1.1001frucht.de:../../web/[ziel-pfad]
   
   # Ganzes Verzeichnis
   rsync -avz [lokales-verzeichnis]/ c864675_w11_ssh1@server1.1001frucht.de:../../web/[ziel-verzeichnis]/
   ```

3. **Empfohlener Workflow für Claude:**
   - Lokale Änderungen mit `code [datei-pfad]` (VS Code öffnen)
   - Manuelle Synchronisation per `scp`/`rsync`  
   - Theme-Kompilierung auf Server testen: `ssh [user]@[host] "cd ../../web && bin/console theme:compile"`
   
#### Standard-Deployment-Schritte
1. **Lokale Änderung:** `code [datei-pfad]` → Datei in VS Code bearbeiten
2. **Upload:** `scp [lokale-datei] [user]@server1.1001frucht.de:../../web/[ziel-pfad]`
3. **Theme kompilieren:** `ssh [user]@server1.1001frucht.de "cd ../../web && bin/console theme:compile"`
4. **Testen:** Website auf Änderungen prüfen
# important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.