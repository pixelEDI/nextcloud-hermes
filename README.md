# Hermes × Nextcloud Talk

Dieses Projekt dokumentiert die Anbindung des [Hermes Agent](https://hermes-agent.nousresearch.com/) an einen Nextcloud-Talk-Raum.

Hermes empfängt Textnachrichten aus Nextcloud Talk, verarbeitet sie über das konfigurierte Modell und antwortet im selben Raum. Das Plugin ist eine unabhängige Community-Integration und kein offizielles Nextcloud-Produkt.

## Architektur

![Architektur von Hermes mit Nextcloud Talk](hermes-nextcloud-architektur%20Screenshot.png)

Die Dokumentation beschreibt das Setup für das begleitende Video.

## Voraussetzungen

- laufender Hermes Agent auf einem Linux-Server
- konfiguriertes Modell, zum Beispiel über OpenRouter
- öffentlich erreichbare HTTPS-Domain für den Webhook
- Nextcloud mit Nextcloud Talk und `occ`
- gemeinsames Webhook-Secret in Hermes und Nextcloud

Der Webhook kann intern auf `127.0.0.1` lauschen. Ein Reverse Proxy, zum Beispiel nginx, stellt ihn unter HTTPS bereit.

## Installation

Der Katalogeintrag ist derzeit noch nicht verfügbar. Für die manuelle Installation:

```bash
git clone https://github.com/Cliff3c/hermes-nextcloud-talk.git
cd hermes-nextcloud-talk
hermes plugins validate ./plugin --install-deps --json
```

Vor einer bestehenden Installation sollte ein Backup erstellt werden. Nicht zwei Instanzen derselben Bot-Identität gleichzeitig starten.

Die Konfiguration erfolgt über Umgebungsvariablen. Eine Vorlage steht in [`.env.example`](.env.example). Secrets gehören ausschließlich in die lokale Konfiguration und dürfen nicht in Git eingecheckt werden.

Ein Secret kann beispielsweise so erzeugt werden:

```bash
openssl rand -hex 32
```

## Nextcloud-Talk-Bot registrieren

Auf dem Nextcloud-Server zunächst die verfügbaren Optionen prüfen und anschließend den Bot registrieren:

```bash
sudo -u www-data php occ help talk:bot:install
sudo -u www-data php occ talk:bot:install
```

Callback-URL und Secret müssen auf beiden Seiten exakt übereinstimmen. Zum ersten Test sollte nur ein eigener Talk-Raum verwendet werden.

Beispiel für die öffentliche Callback-URL:

```text
https://hermes.example.de/nextcloud-talk-webhook
```

## Test und Fehlersuche

1. Im Test-Raum `/help` senden.
2. Eine normale Nachricht senden.
3. Prüfen, ob Hermes antwortet und ein Eintrag im Hermes-Gateway-Log erscheint.

Typische Ursachen:

- `401`: Secret oder Signatur stimmt nicht überein.
- keine Antwort: DNS, TLS-Zertifikat, Reverse Proxy, Callback-Pfad oder Gateway-Status prüfen.
- doppelte Antworten: Das Plugin darf nicht gleichzeitig in mehreren Profilen als Listener aktiv sein.

Aktuell werden Textnachrichten unterstützt. Anhänge, Reaktionen und rekonstruierter Reply-Kontext sind nicht implementiert.

## Mehrere Hermes-Profile

Für unterschiedliche Aufgaben können getrennte Hermes-Profile verwendet werden, zum Beispiel `pixel`, `nexus` oder `ledger`. Das Routing sollte über die jeweilige Nextcloud-Talk-Raum-ID erfolgen. Pro Raum sollte genau ein aktiver Listener antworten.

```bash
hermes profile create pixel --clone-from default
hermes profile create nexus --clone-from default
hermes profile create ledger --clone-from default
```

Die Persönlichkeit eines Profils wird in dessen `SOUL.md` beschrieben. Zugangsdaten und Raum-IDs bleiben lokal auf dem Server.

## Weiterführende Dokumentation

- [Hermes: Nextcloud-Talk-Plugin](https://hermes-agent.nousresearch.com/docs/plugins/nextcloud-talk)
- [Hermes Agent](https://hermes-agent.nousresearch.com/)
- [`hermes-nextcloud-architektur.excalidraw`](hermes-nextcloud-architektur.excalidraw) – bearbeitbare Architekturzeichnung

## Status

Dieses Repository ist eine begleitende Dokumentation zum Setup und Video. Hermes- und Nextcloud-Versionen, Plugin-Optionen und CLI-Kommandos können sich ändern; im Zweifel gelten die aktuellen offiziellen Hermes-Dokumente.

Weitere Projekte findest du auf [pixeledi.eu](https://pixeledi.eu).
