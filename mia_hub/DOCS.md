# MIA Hub

Der Hub verbindet Home Assistant mit MIA. Er baut nur **ausgehende** Verbindungen auf: die
Kopplung über HTTPS und danach den Ereigniskanal über NATS mit gegenseitiger Zertifikatsprüfung.
Im Heimnetz öffnet er keinen Port. Home Assistant erreicht er über die Supervisor-API. Ein
langlebiges Zugriffstoken braucht es nicht, und es steht nie in MIA.

## Installation

1. Einstellungen → Add-ons → Add-on-Store → oben rechts ⋮ → **Repositories** → die Adresse
   `https://github.com/JochenWilFloor/mia-hub-addon` hinzufügen.
2. Im Add-on-Store erscheint „MIA Hub“. Installieren. Neue Versionen bietet Home Assistant danach
   wie bei jedem Add-on als Update an.
3. Das Image liegt in der GitHub Container Registry. Solange das Paket nicht öffentlich ist,
   unter Einstellungen → Add-ons → Add-on-Store → ⋮ → „Registries“ die Registry `ghcr.io` mit
   einem Lesezugang (Token mit `read:packages`) eintragen.

### Umzug vom lokalen Add-on

Das lokale Add-on (Ordner `/addons/mia_hub`) und das Add-on aus dem Repository sind für Home
Assistant zwei verschiedene Add-ons. Schlüssel und Zertifikat liegen je Add-on unter `/data` und
ziehen nicht mit. Deshalb einmal neu koppeln; Räume, Geräte und Auswahl in MIA bleiben erhalten,
weil sie am Haushalt hängen, nicht am Hub.

1. Im lokalen Add-on unter „Konfiguration“ **Adresse von MIA** und **Gerätename** notieren.
2. Das Add-on aus dem Repository installieren (oben) und dieselben Werte eintragen.
3. In MIA (Owner) unter Zuhause einen **neuen Kopplungscode** erzeugen und im neuen Add-on bei
   **Kopplungscode** eintragen. Das lokale Add-on **stoppen**, dann das neue starten.
4. Im Protokoll des neuen Add-ons erscheint „Als Home-Assistant-Add-on gestartet“, danach das
   Verbinden mit MIA. Die neue Kopplung widerruft die alte automatisch.
5. Läuft der neue Hub (in MIA „verbunden“), das lokale Add-on deinstallieren und den Ordner
   `/addons/mia_hub` löschen.

**Rückfall:** Klappt die neue Kopplung nicht, das neue Add-on stoppen, im lokalen Add-on einen
neuen Kopplungscode eintragen und es wieder starten.

**Notfallweg ohne Repository:** den Ordner `mia_hub` aus `hub/addon/` nach `/addons/mia_hub`
kopieren (Add-on „Samba share“ oder „Advanced SSH & Web Terminal“), dann im Add-on-Store ⋮ →
„Nach Updates suchen“; er erscheint unter „Lokale Add-ons“.

## Koppeln

1. In MIA (Owner) unter Zuhause einen Kopplungscode erzeugen.
2. Im Add-on unter „Konfiguration“ den Code bei **Kopplungscode** eintragen, speichern und das
   Add-on starten.
3. Im Protokoll des Add-ons erscheint „Als Home-Assistant-Add-on gestartet“, danach das
   Verbinden mit MIA. In MIA steht der Hub dann als verbunden.

Der Code gilt einmal und nur kurz. Ist er abgelaufen, meldet das Protokoll „Kopplung abgewiesen“:
neuen Code erzeugen und eintragen. Schlüssel und Zertifikat liegen im Add-on unter `/data`.
Deinstallieren löscht sie; danach muss neu gekoppelt werden.

## Neustart nach Ausfällen

Im Add-on unter „Info“ die Schalter **Beim Booten starten** und **Watchdog** einschalten. Dann gilt:

- Verliert der Hub die Verbindung, verbindet er sich von selbst neu.
- Scheitert er mehrmals hintereinander, startet Home Assistant das Add-on neu, ebenso nach einem
  Update oder einer Reparatur von Home Assistant.

Ein Neustart von Hand ist dann nicht mehr nötig.

## Zustand der Integrationen (ab 0.9.0)

Alle 5 Minuten prüft der Hub, ob Integrationen in Home Assistant ausgefallen sind oder eine neue
Anmeldung verlangen, und meldet das an MIA. Er meldet nur den technischen Namen der Integration und
eine Zahl, nie Titel, Konten oder Zugangsdaten.

- **Neu laden** darf der Hub eine ausgefallene Integration nur, wenn der Owner das in MIA erlaubt
  hat (Standard aus). Dann versucht er es **einmal** und danach frühestens nach 24 Stunden wieder.
- **Neu anmelden** muss man sich in Home Assistant selbst, unter Einstellungen → Reparaturen. Der
  Hub lädt in diesem Fall nichts neu.

## Aufwachen über das iPhone

Siehe `hub/agent/EINRICHTUNG-IPHONE.md` (#1157).

## Bewegung fürs Aufwachen (ab 0.10.0)

Damit MIA das Aufwachen auch ohne Handy erkennt, meldet der Hub Bewegungen an Bewegungsmeldern,
die in MIA **eingeschaltet** sind (Menü → Peripherie → Geräte). Neu erkannte Geräte sind aus.

- Gemeldet werden nur die Kennung des Melders und die Uhrzeit, kein Raum, keine Person und kein
  Messwert. Jeder Melder meldet höchstens einmal pro Minute.
- MIA nutzt das nur, um das Aufwachen zu erkennen und das Briefing vorzubereiten. Gesprochen wird
  es erst, wenn ein Signal von dir dazukommt, zum Beispiel deine erste Nutzung von MIA.
- Ohne Einwilligung „Zuhause“ meldet der Hub nichts.
