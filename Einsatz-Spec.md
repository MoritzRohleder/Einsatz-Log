# Einsatz-Spec

> Spezifikation für die Webanwendung **Einsatz-Log** zur nachträglichen Dokumentation von Feuerwehreinsätzen.
> Status: **v0.1** – erster Entwurf, noch nicht freigegeben.
> Getroffene Entscheidungen stehen in [Kapitel 13.1](#131-entscheidungen), offene Punkte in [Kapitel 13.2](#132-offene-fragen) (im Text mit `❓` markiert).
> Stellen, die nicht ausdrücklich entschieden, sondern aus dem Haushaltsprojekt übernommen oder als Vorschlag ergänzt wurden, sind mit *(Vorschlag)* gekennzeichnet.

---

## Inhaltsverzeichnis

1. [Ziel und Zweck](#1-ziel-und-zweck)
2. [Leitprinzipien](#2-leitprinzipien)
3. [Begriffe](#3-begriffe)
4. [Rollen und Berechtigungen](#4-rollen-und-berechtigungen)
5. [Datenmodell](#5-datenmodell)
6. [Regeln](#6-regeln)
7. [Funktionale Anforderungen](#7-funktionale-anforderungen)
8. [Abläufe](#8-abläufe)
9. [Seiten und Navigation](#9-seiten-und-navigation)
10. [Technologie und Architektur](#10-technologie-und-architektur)
11. [Nicht-funktionale Anforderungen](#11-nicht-funktionale-anforderungen)
12. [Abgrenzung (nicht im ersten Release)](#12-abgrenzung-nicht-im-ersten-release)
13. [Entscheidungen und offene Fragen](#13-entscheidungen-und-offene-fragen)
14. [Meilensteine](#14-meilensteine)
15. [Lizenz](#15-lizenz)
16. [Änderungshistorie](#16-änderungshistorie)

---

## 1. Ziel und Zweck

Die Anwendung dient dazu, **Feuerwehreinsätze nach der Rückkehr zu dokumentieren**. Es gibt **eine gemeinsame Liste** von Einsätzen; jeder Hauptnutzer pflegt dazu seine **eigene Teilnahme** (Status, Ende, Bericht).

Kernfragen, die die Anwendung beantworten soll:

- Welche Einsätze gab es, und bei welchen war ich (oder eine andere Person) dabei?
- Wo habe ich den Bericht noch nicht geschrieben?
- Wie viele Einsätze gab es in einem Zeitraum, aufgeteilt nach Kennzeichnung und Stichwort?
- Wie oft war eine Person dabei, wie oft bewusst nicht, wie oft nicht aktiv – und wie viele Einsatzstunden kamen zusammen?

## 2. Leitprinzipien

*(Vorschlag, analog zum Haushaltsprojekt)*

| # | Prinzip | Bedeutung |
|---|---------|-----------|
| P1 | **Generisch statt maßgeschneidert** | Die Software kennt keine konkreten Personen, Wehren oder Stichworte. Kennzeichnungen und Stichworte werden in der Anwendung gepflegt; B, H und C mit 1–5 sind nur Startdaten. |
| P2 | **Einfach bleiben** | Kleine, server-gerenderte Webanwendung. Kein SPA-Framework, kein Build-Schritt fürs Frontend. |
| P3 | **Nachvollziehbarkeit** | Statistiken werden aus den gespeicherten Einsätzen und Teilnahmen berechnet, nicht separat gespeichert. Änderungen an gemeinsamen Feldern stehen in der Änderungshistorie. |
| P4 | **Daten gehören uns** | Selbst gehostet im Heimnetz, Daten liegen lokal als JSON-Dateien, Export und Backup sind einfach. |
| P5 | **Austauschbare Datenhaltung** | Die Anwendung greift nie direkt auf Dateien zu, sondern nur über eine Speicherschicht (siehe [10.3](#103-speicherschicht)). |
| P6 | **Sichtbarkeit im Server** | Was eine Rolle nicht sehen darf (Adresse, weitere Infos), wird schon im Server entfernt – auch in Statistik und Export. |

## 3. Begriffe

| Begriff | Definition |
|---------|------------|
| **Nutzer** | Eine Person mit eigenem Login (Nutzername + Passwort). Nutzer registrieren sich selbst. Jeder Nutzer hat genau eine **Rolle**. |
| **Admin** | Rolle des ersten registrierten Nutzers. Es gibt **genau einen** Admin. Der Admin ist zugleich Hauptnutzer. |
| **Hauptnutzer** | Rolle für Personen, die an Einsätzen teilnehmen: legt Einsätze an, bearbeitet sie und pflegt die eigene Teilnahme. |
| **Eingeschränkter Nutzer** | Rolle jedes neu registrierten Nutzers: darf nur lesen, sieht keine Adresse und keine weiteren Infos. |
| **Einsatz** | Ein Eintrag in der gemeinsamen Liste mit den gemeinsamen Feldern Kennzeichnung, Stichwort, Adresse, Beginn und weitere Infos. |
| **Kennzeichnung** | Buchstabe plus Zahl, z. B. `B2`. Buchstaben und der erlaubte Zahlenbereich je Buchstabe werden vom Admin gepflegt. |
| **Stichwort** | Kurzbezeichnung des Einsatzes, z. B. „Wohnungsbrand“. Jedes Stichwort gehört zu **einem Buchstaben** der Kennzeichnung. |
| **Beginn** | Datum und Uhrzeit der Alarmierung. Gilt für alle Teilnehmer gleich. |
| **Teilnahme** | Die Angaben **eines Hauptnutzers** zu **einem Einsatz**: Status, Ende, Bericht. |
| **Status** | `Offen` (noch nichts eingetragen), `Komme`, `Komme nicht`, `Nicht verfügbar`. |
| **Ende** | Datum und Uhrzeit, zu der der Einsatz für diesen Hauptnutzer endete. Teil der Teilnahme. |
| **Bericht** | Freitext des Hauptnutzers zu seinem Einsatz. Teil der Teilnahme. |
| **Bericht offen** | Markierung einer Teilnahme mit Status `Komme`, deren Bericht leer ist. |
| **Ausgewählte Person** | Der Hauptnutzer, dessen Teilnahmen die Übersicht gerade zeigt (siehe [6.3](#63-ausgewählte-person)). |
| **Änderungshistorie** | Protokoll der Änderungen an den gemeinsamen Feldern eines Einsatzes. |

## 4. Rollen und Berechtigungen

| Recht | Eingeschränkt | Hauptnutzer | Admin |
|-------|:---:|:---:|:---:|
| Übersicht, Detailansicht, Statistik, Export | ✔ (ohne Adresse / weitere Infos) | ✔ | ✔ |
| Adresse und weitere Infos sehen | – | ✔ | ✔ |
| Status, Ende und Berichte aller Hauptnutzer sehen | ✔ | ✔ | ✔ |
| Einsatz anlegen | – | ✔ | ✔ |
| Gemeinsame Felder eines Einsatzes bearbeiten | – | ✔ | ✔ |
| Eigene Teilnahme pflegen | – | ✔ | ✔ |
| Stichwort im Dropdown ergänzen | – | ✔ | ✔ |
| Einsatz löschen | – | – | ✔ |
| Änderungshistorie sehen | – | – | ✔ |
| Kennzeichnungen (Buchstaben, Zahlenbereiche) pflegen | – | – | ✔ |
| Stichworte bearbeiten und löschen | – | – | ✔ |
| Nutzer befördern, zurückstufen, löschen | – | – | ✔ |
| Admin-Rolle übergeben | – | – | ✔ |

Regeln:

| Regel | Beschreibung |
|-------|--------------|
| R-01 | Der **erste** Nutzer, der sich registriert, wird Admin. Jeder weitere Nutzer ist zunächst **eingeschränkt**. |
| R-02 | Es gibt immer **genau einen** Admin. Der Admin kann die Rolle an einen anderen Nutzer übergeben; ❓ welche Rolle der bisherige Admin danach hat (Vorschlag: Hauptnutzer). |
| R-03 | Der Admin kann eingeschränkte Nutzer zu Hauptnutzern befördern und Hauptnutzer wieder zurückstufen. |
| R-04 | Der Admin kann Nutzer löschen. ❓ Darf der Admin sich selbst löschen bzw. zurückstufen (Vorschlag: nein, erst Rolle übergeben)? |
| R-05 | ❓ Was passiert mit den Teilnahmen (Status, Ende, Bericht) eines Hauptnutzers, wenn er **zurückgestuft** oder **gelöscht** wird? |
| R-06 | Die Sichtbarkeit wird bei **jedem** Zugriff im Server geprüft. Für eingeschränkte Nutzer werden Adresse und weitere Infos gar nicht erst an die Seite, die Statistik oder den Export übergeben. |
| R-07 | Ruft ein Nutzer eine Seite auf, die seine Rolle nicht erlaubt, antwortet der Server mit **404** *(Vorschlag, analog Haushaltsprojekt)*. |

## 5. Datenmodell

Das Modell ist unabhängig von der Speicherform beschrieben. Im JSON-Speicher entspricht jede Entität einer Sammlung (Datei), siehe [10.3](#103-speicherschicht).
IDs sind zufällige UUIDs (`crypto.randomUUID()`). Feldnamen sind ein *(Vorschlag)*.

### 5.1 Überblick

Pfeile zeigen in Richtung des Verweises: `A ──► B` heißt „A speichert die ID von B“.

```
 ┌─────────────────┐        ┌──────────────────┐
 │ Kennzeichnung   │◄───────│ Stichwort        │
 │ (designations)  │ desig- │ (keywords)       │
 └────────▲────────┘ nation └────────▲─────────┘
          │ designation_id           │ keyword_id
          │                          │
        ┌─┴──────────────────────────┴─┐        ┌────────────────────┐
        │ Einsatz (incidents)          │◄───────│ Änderungshistorie  │
        └──────────────▲───────────────┘ inci-  │ (incident_history) │
                       │ incident_id     dent_id└─────────┬──────────┘
                       │                                  │ user_id
              ┌────────┴─────────┐                        ▼
              │ Teilnahme        │──── user_id ────►┌───────────┐
              │ (participations) │                  │ Nutzer    │
              └──────────────────┘                  │ (users)   │
                                                    └───────────┘
```

### 5.2 Entitäten

#### Nutzer (`users`)

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `id` | UUID | |
| `username` | Text | eindeutig, wird überall angezeigt |
| `password_hash` | Text | `crypto.scrypt` mit Salt |
| `role` | `admin` \| `main` \| `restricted` | Rolle (Kapitel 4) |
| `theme` | `auto` \| `light` \| `dark` | Darstellung, Standard `auto` (folgt dem System) |
| `selected_person_id` | UUID \| `null` | nur für eingeschränkte Nutzer: gespeicherte Auswahl (6.3) |
| `created_at` | Zeitstempel | |

#### Kennzeichnung (`designations`)

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `id` | UUID | |
| `letter` | Text | z. B. `B`; eindeutig |
| `min` / `max` | Ganzzahl | erlaubter Zahlenbereich, z. B. 1–5 |
| `deleted_at` | Zeitstempel \| `null` | gesetzt = für neue Einsätze nicht mehr wählbar (6.5) |

Startdaten beim ersten Start: `B` 1–5, `H` 1–5, `C` 1–5.
❓ Bedeutung der Buchstaben als Beschriftung mitführen (z. B. „B – Brand“)?

#### Stichwort (`keywords`)

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `id` | UUID | |
| `designation_id` | UUID | Buchstabe, zu dem das Stichwort gehört |
| `name` | Text | eindeutig je Buchstabe |
| `deleted_at` | Zeitstempel \| `null` | gesetzt = für neue Einsätze nicht mehr wählbar (6.5) |

❓ Sollen beim ersten Start Stichworte als Startdaten angelegt werden, oder beginnt die Liste leer?

#### Einsatz (`incidents`)

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `id` | UUID | |
| `designation_id` | UUID \| `null` | Buchstabe der Kennzeichnung |
| `designation_number` | Ganzzahl \| `null` | Zahl der Kennzeichnung, innerhalb `min`–`max` |
| `keyword_id` | UUID \| `null` | Stichwort (muss zum Buchstaben gehören) |
| `address` | Text | Einsatzadresse, Freitext |
| `start_at` | Datum + Uhrzeit \| `null` | Beginn (Alarmzeit) |
| `info` | Text | weitere Informationen, Freitext |
| `created_by` | UUID | anlegender Nutzer |
| `created_at`, `updated_at` | Zeitstempel | |

Es gibt **keine Pflichtfelder** (E-08).

#### Teilnahme (`participations`)

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `id` | UUID | |
| `incident_id` | UUID | |
| `user_id` | UUID | Hauptnutzer |
| `status` | `come` \| `not_coming` \| `unavailable` | `Komme`, `Komme nicht`, `Nicht verfügbar` |
| `end_at` | Datum + Uhrzeit \| `null` | Ende |
| `report` | Text | Bericht |
| `updated_at` | Zeitstempel | |

Höchstens eine Teilnahme je Einsatz und Nutzer. **Gibt es keine Teilnahme, ist der Status `Offen`** – sie wird erst beim ersten Speichern angelegt *(Vorschlag)*.

#### Änderungshistorie (`incident_history`)

| Feld | Typ | Beschreibung |
|------|-----|--------------|
| `id` | UUID | |
| `incident_id` | UUID | |
| `user_id` | UUID | wer geändert hat |
| `at` | Zeitstempel | wann |
| `field` | Text | welches gemeinsame Feld |
| `old_value` / `new_value` | Text | alter und neuer Wert (lesbar, z. B. `B2`, Stichwortname) |

❓ Wird das **Anlegen** eines Einsatzes ebenfalls in der Historie vermerkt?
❓ Was passiert mit der Historie, wenn der Admin einen Einsatz löscht (mitlöschen oder behalten)?

## 6. Regeln

### 6.1 Status und Sichtbarkeit von Ende und Bericht

- Ende und Bericht sind nur bei Status `Komme` sichtbar und ausfüllbar.
- Wechselt der Status von `Komme` auf einen anderen Wert, **bleiben Ende und Bericht gespeichert**, werden aber ausgeblendet – auch in Statistik und Export. Bei Rückkehr zu `Komme` erscheinen sie wieder.

### 6.2 Bericht offen

Eine Teilnahme ist „Bericht offen“, wenn Status = `Komme` und der Bericht leer ist (nur Leerzeichen zählen als leer *(Vorschlag)*). Die Markierung sehen alle Rollen.

### 6.3 Ausgewählte Person

Übersicht (Karten, Status-Filter, „Bericht offen“) zeigt die Teilnahmen **einer ausgewählten Person** (ein Hauptnutzer oder der Admin).

| Rolle | Standard | Gespeichert? |
|-------|----------|--------------|
| Hauptnutzer / Admin | man selbst | nein – bei jeder neuen Sitzung wieder „man selbst“ |
| Eingeschränkter Nutzer | der Admin | ja, über Sitzungen hinweg (`users.selected_person_id`) |

### 6.4 Doppelte Einsätze

Beim Anlegen warnt die Anwendung, wenn bereits ein Einsatz mit **gleicher Kennzeichnung** (Buchstabe und Zahl) und einem **Beginn höchstens 5 Minuten** daneben existiert. Die Warnung zeigt den gefundenen Einsatz; der Nutzer kann trotzdem anlegen oder zum bestehenden Einsatz wechseln *(Vorschlag)*.
❓ Gilt die Warnung auch beim nachträglichen Bearbeiten, oder nur beim Anlegen?

### 6.5 Gelöschte Kennzeichnungen und Stichworte

- Löscht der Admin einen Buchstaben oder ein Stichwort, **behalten bestehende Einsätze** den Eintrag und zeigen ihn weiter an.
- Für **neue** Einsätze ist er nicht mehr wählbar. ❓ Beim Bearbeiten eines bestehenden Einsatzes, der den Eintrag hat: bleibt er dort wählbar (Vorschlag: ja, solange er nicht geändert wird)?
- ❓ Wird ein Zahlenbereich verkleinert (z. B. B 1–5 → 1–3), behalten bestehende Einsätze ihre Zahl (Vorschlag: ja, analog).
- Bearbeitet der Admin ein Stichwort (Umbenennen), gilt der neue Name auch für bestehende Einsätze *(Vorschlag)*.

### 6.6 Statistik

Grundlage sind alle Einsätze, deren **Beginn** im gewählten Zeitraum liegt. Der Zeitraum ist frei wählbar (damit auch pro Monat oder Jahr).
❓ Wie werden Einsätze **ohne Beginn** behandelt (es gibt keine Pflichtfelder)?

Für die Auswertung werden die Status zu drei Gruppen zusammengefasst:

| Gruppe | Status |
|--------|--------|
| **Komme** | `Komme` |
| **Komme nicht** | `Komme nicht` |
| **Nicht aktiv** ❓ (oder „Inaktiv“) | `Nicht verfügbar` und `Offen` |

**Ansicht „Gesamt“**

| Kennzahl | Bedeutung |
|----------|-----------|
| Anzahl Einsätze | alle Einsätze im Zeitraum, pro Monat bzw. Jahr |
| nach Kennzeichnung | Anzahl je Buchstabe ❓ (oder je Buchstabe + Zahl) |
| häufigste Stichworte | über alle Einsätze im Zeitraum |
| Statusgruppen | **je Person im Vergleich** |
| Teilnahmequote | **je Person im Vergleich** |
| Einsatzstunden | **je Person im Vergleich** |

**Ansicht „Person X“** – alle Kennzahlen aufgeteilt nach den drei Statusgruppen dieser Person:

| Kennzahl | Bedeutung |
|----------|-----------|
| Anzahl Einsätze | je Statusgruppe, pro Monat bzw. Jahr |
| nach Kennzeichnung | je Statusgruppe |
| häufigste Stichworte | je Statusgruppe |
| Teilnahmequote | Anteil „Komme“ an **allen** Einsätzen im Zeitraum (Offen zählt als „Nicht aktiv“) |
| Einsatzstunden | Summe von Beginn bis eigenem Ende über alle Teilnahmen mit Status `Komme` |

❓ Einsatzstunden, wenn das Ende fehlt oder vor dem Beginn liegt: nicht mitzählen und als Hinweis anzeigen (Vorschlag)?
❓ Zählen bei einer Person auch Einsätze, die **vor ihrer Registrierung bzw. Beförderung** lagen, als „Nicht aktiv“?

Statistik und Export enthalten für **alle Rollen** nur, was der eingeschränkte Nutzer sieht – nie Adresse oder weitere Infos.

### 6.7 Formate

*(Vorschlag)* Datum `TT.MM.JJJJ`, Uhrzeit `HH:MM`, Zeitzone über `TZ` (Standard `Europe/Berlin`). Kennzeichnung wird als Buchstabe + Zahl ohne Leerzeichen angezeigt (`B2`). Einsatzstunden mit einer Nachkommastelle bzw. als `h:mm` ❓.

## 7. Funktionale Anforderungen

Priorität: **M** = Muss (erstes Release), **S** = Soll, **K** = Kann (später). Die Prioritäten sind ein *(Vorschlag)*.

### 7.1 Nutzer und Login

| ID | Anforderung | Prio |
|----|-------------|------|
| F-01 | Registrieren mit Nutzername und Passwort (mit Wiederholung); erster Nutzer wird Admin, alle weiteren eingeschränkt (R-01) | M |
| F-02 | Anmelden, Abmelden | M |
| F-03 | Alle Seiten außer Login, Registrierung, Handbuch und Health-Check nur angemeldet erreichbar | M |
| F-04 | Eigenes Passwort ändern | S |
| F-05 | Hell/Dunkel: Standard folgt der Systemeinstellung; jeder Nutzer kann im Profil Automatisch, Hell oder Dunkel wählen; Wahl wird pro Nutzer gespeichert | M |
| F-06 | Handbuch in der Anwendung (aus `docs/wiki`), analog Haushaltsprojekt | S |

### 7.2 Verwaltung (Admin)

| ID | Anforderung | Prio |
|----|-------------|------|
| F-10 | Nutzerliste mit Rolle; befördern, zurückstufen, löschen (R-03, R-04) | M |
| F-11 | Admin-Rolle an einen anderen Nutzer übergeben (R-02) | M |
| F-12 | Kennzeichnungen pflegen: Buchstaben anlegen, Zahlenbereich je Buchstabe festlegen, löschen (6.5) | M |
| F-13 | Stichworte bearbeiten und löschen (6.5) | M |
| F-14 | Änderungshistorie eines Einsatzes ansehen | M |
| F-15 | Einsatz löschen (mit Bestätigung; dabei werden alle Teilnahmen mitgelöscht) | M |

### 7.3 Einsätze

| ID | Anforderung | Prio |
|----|-------------|------|
| F-20 | Einsatz anlegen: Kennzeichnung (Buchstabe + Zahl im erlaubten Bereich), Stichwort, Adresse, Beginn, weitere Infos; keine Pflichtfelder | M |
| F-21 | Stichwort-Auswahl abhängig vom gewählten Buchstaben; neues Stichwort direkt im Dropdown ergänzen | M |
| F-22 | Beim Anlegen die eigene Teilnahme gleich mit erfassen (Status, Ende, Bericht) *(Vorschlag)* | S |
| F-23 | Warnung bei möglichem doppeltem Einsatz (6.4) | M |
| F-24 | Gemeinsame Felder bearbeiten (alle Hauptnutzer); jede Änderung wird in der Änderungshistorie protokolliert | M |
| F-25 | Eigene Teilnahme pflegen: Status, Ende und Bericht (Ende und Bericht nur bei `Komme`, 6.1) | M |

### 7.4 Übersicht und Detailansicht

| ID | Anforderung | Prio |
|----|-------------|------|
| F-30 | Übersicht als Karten bzw. Listeneinträge mit Datum & Uhrzeit, Kennzeichnung, Stichwort, Status der ausgewählten Person und Markierung „Bericht offen“ | M |
| F-31 | Auswahl der Person (6.3) | M |
| F-32 | Filter: Status (inkl. `Offen`), Kennzeichnung, Stichwort, „Bericht offen“, Zeitraum (Beginn von–bis) | M |
| F-33 | Standardfilter: Status `Komme`; Sortierung nach Datum & Uhrzeit, neuester oben; umkehrbar | M |
| F-34 | Knopf „Neuer Einsatz“ in der Übersicht (nur Hauptnutzer/Admin) | M |
| F-35 | Detailansicht: alle sichtbaren Felder, dazu Status, Ende und Bericht **aller** Hauptnutzer mit Namen | M |
| F-36 | Bearbeiten aus der Detailansicht heraus | M |

❓ Filter als Mehrfachauswahl (wie im Haushaltsprojekt, 9.1) oder als Einzelauswahl?
❓ Filter Kennzeichnung: nach Buchstabe, nach Buchstabe + Zahl, oder beides?
❓ Sollen Filter und Sortierung der Übersicht beim Wechsel zur Detailansicht und zurück erhalten bleiben (Vorschlag: ja, über die Adresse)?

### 7.5 Statistik und Export

| ID | Anforderung | Prio |
|----|-------------|------|
| F-40 | Statistikseite mit frei wählbarem Zeitraum und Umschalter „Gesamt / Person X“ (6.6) | M |
| F-41 | Diagramme zu allen Kennzahlen, zusätzlich Werte als Tabelle | M |
| F-42 | Export als **CSV** | M |
| F-43 | Export als **PDF** | M |
| F-44 | Beim Export wählbar: **alle** Einsätze oder die **aktuell gefilterte** Liste; und **Gesamt** oder **Person X** | M |
| F-45 | Export enthält nur Felder, die der eingeschränkte Nutzer sieht: Kennzeichnung, Stichwort, Beginn, Status, Ende, Bericht (6.6) | M |

❓ Aufbau des Exports bei „Gesamt“: eine Zeile je Einsatz **und** Person, oder eine Zeile je Einsatz mit einer Spalte je Person?
❓ Inhalt des PDF: nur die Liste, oder Liste plus Statistik?

### 7.6 Daten

| ID | Anforderung | Prio |
|----|-------------|------|
| F-50 | Sicherung: vor jedem Schreiben Kopie in `data/backup/` (analog Haushaltsprojekt) | M |
| F-51 | Health-Check `GET /health` | M |

## 8. Abläufe

### 8.1 Erster Start

1. Anwendung startet mit leerem Datenordner; Startdaten B, H, C (je 1–5) werden angelegt.
2. Erster Nutzer registriert sich → Admin.
3. Weitere Nutzer registrieren sich → eingeschränkt. Der Admin befördert, wer teilnimmt, zum Hauptnutzer.

### 8.2 Nach einem Einsatz dokumentieren

1. Hauptnutzer öffnet die Übersicht und prüft, ob der Einsatz schon angelegt ist.
2. Falls nicht: „Neuer Einsatz“ → Kennzeichnung, Stichwort (ggf. neu ergänzen), Adresse, Beginn, weitere Infos.
   Gibt es einen ähnlichen Einsatz (6.4), erscheint eine Warnung.
3. Eigene Teilnahme: Status setzen; bei `Komme` Ende und Bericht eintragen. Bericht kann später nachgetragen werden – bis dahin „Bericht offen“.

### 8.3 Bericht nachtragen

1. Filter „Bericht offen“ zeigt alle eigenen Einsätze ohne Bericht.
2. Einsatz öffnen → Teilnahme bearbeiten → Bericht speichern.

### 8.4 Eingeschränkter Nutzer liest mit

1. Nach dem Login zeigt die Übersicht die Einsätze mit den Teilnahmen des Admins (bzw. der zuletzt gewählten Person).
2. Person wechseln → Auswahl bleibt für die nächste Sitzung gespeichert.

## 9. Seiten und Navigation

*(Vorschlag)*

| Seite | Route | Inhalt |
|-------|-------|--------|
| Login | `GET/POST /login` | |
| Registrieren | `GET/POST /registrieren` | |
| Abmelden | `POST /logout` | |
| Übersicht | `GET /` | Karten mit Filter, Sortierung und Personenauswahl (Query-Parameter) |
| Neuer Einsatz | `GET/POST /einsaetze/neu` | inkl. Warnung bei Doppelung |
| Detailansicht | `GET /einsaetze/:id` | alle sichtbaren Felder und Teilnahmen |
| Einsatz bearbeiten | `GET/POST /einsaetze/:id/bearbeiten` | gemeinsame Felder |
| Eigene Teilnahme | `GET/POST /einsaetze/:id/teilnahme` | Status, Ende, Bericht |
| Einsatz löschen | `POST /einsaetze/:id/loeschen` | nur Admin |
| Änderungshistorie | `GET /einsaetze/:id/historie` | nur Admin |
| Statistik | `GET /statistik?von=&bis=&person=` | |
| Export | `GET /export.csv`, `GET /export.pdf` | gleiche Parameter wie Übersicht bzw. `alle=1`, `person=` |
| Nutzerverwaltung | `GET/POST /admin/nutzer` | nur Admin |
| Kennzeichnungen | `GET/POST /admin/kennzeichnungen` | nur Admin |
| Stichworte | `GET/POST /admin/stichworte` | nur Admin; Ergänzen auch im Einsatzformular |
| Profil | `GET/POST /profil` | Darstellung, Passwort |
| Handbuch | `GET /handbuch`, `/handbuch/:seite` | ohne Anmeldung lesbar |
| Health-Check | `GET /health` | `{"status":"ok"}` |

Änderungen und Löschungen laufen über `POST` (Muster *Post/Redirect/Get*).

## 10. Technologie und Architektur

Technisch analog zum Projekt [Haushaltsprojekt](https://github.com/MoritzRohleder/Haushaltsprojekt) (E-13): Struktur, Bausteine und Betrieb werden übernommen.

### 10.1 Komponenten

| Baustein | Wahl |
|----------|------|
| Laufzeit | Node.js, aktuelle LTS (`engines: >=22`) |
| Web-Framework | Express 5 |
| Template-Engine | EJS |
| Sitzungen | `express-session` mit eigenem Store in `data/sessions.json` |
| Passwort-Hashing | `crypto.scrypt` |
| Datenhaltung | JSON-Dateien hinter einer Speicherschicht |
| CSS | Pico.css (über npm, kein CDN) + `public/css/theme.css` |
| Client-JS | nur wo nötig, Vanilla JS |
| Diagramme | auf dem Server erzeugtes SVG, ohne Bibliothek |
| Handbuch | `marked` |
| PDF-Erzeugung | ❓ (z. B. eine Bibliothek auf dem Server, oder eine Druckansicht, die der Browser als PDF speichert) |
| Tests | `node:test` |

### 10.2 Schichten

```
 routes/        ← Eingaben lesen, prüfen, Service aufrufen, View rendern
 services/      ← Fachlogik: Rollen/Sichtbarkeit, Teilnahmen, Doppelungs-Prüfung, Historie, Statistik, Export
 repositories/  ← ein Repository pro Entität
 storage/       ← austauschbarer Adapter (heute JSON)
```

### 10.3 Speicherschicht

Wie im Haushaltsprojekt: Schnittstelle `findAll`, `findById`, `insert`, `update`, `remove`, `transaction`; eine Datei pro Sammlung (`users`, `designations`, `keywords`, `incidents`, `participations`, `incident_history`); alles im Arbeitsspeicher; Schreiben nacheinander und atomar; Sicherung vor dem Überschreiben; Migrationen über `meta.json`; `data/` steht in `.gitignore`.

### 10.4 Login und Sicherheit

Wie im Haushaltsprojekt: Passwort mindestens 8 Zeichen, Session-ID nach Login erneuert, Cookie `httpOnly` / `sameSite: lax` / `secure` bei HTTPS, Sitzungen 30 Tage, CSRF-Token in allen Formularen, Login-Bremse, neutrale Fehlermeldung, Sicherheits-Header (CSP ohne Inline-Skripte), kein Open Redirect, CSV-Formel-Entschärfung.

### 10.5 Projektstruktur

```
Einsatz-Log/
├── Einsatz-Spec.md
├── src/ (app.js, server.js, config.js, storage/, repositories/, services/, routes/, middleware/, utils/)
├── views/ (partials/, auth/, incidents/, admin/, statistics.ejs, profile.ejs)
├── public/ (css/theme.css, js/)
├── docs/wiki/
├── test/
├── Dockerfile, docker-compose.yml, .env.example
└── .github/workflows/ (ci.yml, release.yml)
```

### 10.6 Konfiguration

| Variable | Default | Bedeutung |
|----------|---------|-----------|
| `PORT` | `3000` | HTTP-Port |
| `DATA_DIR` | `./data` (Docker: `/app/data`) | Datenordner |
| `STORAGE` | `json` | Speicher-Adapter |
| `SESSION_SECRET` | zufällig, in `data/.session-secret` | Schlüssel für das Login-Cookie |
| `COOKIE_SECURE` | `false` | `true` bei HTTPS |
| `TRUST_PROXY` | `false` | hinter einem Reverse Proxy |
| `TZ` | `Europe/Berlin` | Zeitzone |
| `EINSATZ_VERSION` | `latest` | nur Docker Compose: Image-Version |
| `HOST_PORT` | `3000` | nur Docker Compose: Port auf dem Server |

### 10.7 Gestaltung und Farben

- Pico.css, alle Farben als Variablen in `theme.css`, Hell und Dunkel jeweils festgelegt; `data-theme` am `<html>`-Element.
- Farbe ist nie das einzige Merkmal: Status zusätzlich als Text bzw. Symbol.
- ❓ Farbpalette (Primärfarbe und Farben für die Status) – Vorschlag folgt bei Bedarf.

### 10.8 Betrieb mit Docker

Wie im Haushaltsprojekt: `node:24-alpine`, unprivilegierter Nutzer, Volume `/app/data`, Health-Check, `docker-compose.yml` mit fertigem Image `ghcr.io/moritzrohleder/einsatz-log` (`pull_policy: always`), fester Projektname `einsatz-log`, `.env.example`.

### 10.9 CI/CD

Wie im Haushaltsprojekt: `ci.yml` (Tests mit Node 22 und 24, Docker-Build mit Start- und Health-Prüfung), `release.yml` (Image für amd64 und arm64 nach ghcr.io mit Tags `X.Y.Z`, `X.Y`, `X`, `latest`).

### 10.10 Handbuch

Wie im Haushaltsprojekt: Quelle `docs/wiki/`, in der Anwendung unter `/handbuch`, Test prüft interne Links.

## 11. Nicht-funktionale Anforderungen

| ID | Anforderung |
|----|-------------|
| N-01 | **Sprache**: Deutsch. |
| N-02 | **Responsive**: auf dem Smartphone nutzbar, vor allem Erfassen und Bericht schreiben. |
| N-03 | **Sicherheit**: Ausgaben escaped; serverseitige Validierung; Rollen und Sichtbarkeit (Kapitel 4) bei jedem Zugriff im Server geprüft. |
| N-04 | **Betrieb**: im Heimnetz (z. B. Raspberry Pi), per Docker. Die Registrierung ist offen; neue Nutzer sehen sofort die Daten eines eingeschränkten Nutzers. Das ist für den Heimnetz-Betrieb akzeptiert (E-05). |
| N-05 | **Datenschutz**: Adressen, weitere Infos und Berichte können personenbezogene Daten enthalten. Bei Erreichbarkeit aus dem Internet nur über HTTPS. |
| N-06 | **Datensicherung**: Der Ordner `data/` ist das Einzige, was gesichert werden muss. |
| N-07 | **Testbarkeit**: Rollen und Sichtbarkeit, Statistikregeln, Doppelungs-Prüfung, Historie und der JSON-Adapter sind durch Tests abgedeckt. |

## 12. Abgrenzung (nicht im ersten Release)

*(Vorschlag – aus den besprochenen, aber nicht gewählten Ideen)*

- Keine Schnellerfassung bzw. kein Einlesen von Alarmtexten.
- Keine Fahrzeuge, Funktionen oder Atemschutz-Angaben.
- Keine Einsatznummer der Leitstelle.
- Keine Fotos oder Dateianhänge.
- Keine strukturierte Adresse (Straße, PLZ, Ort) und keine Karte.
- Keine Volltextsuche.
- Keine Freischaltung neuer Nutzer vor dem ersten Lesen.
- Kein „Passwort vergessen“ per E-Mail.

## 13. Entscheidungen und offene Fragen

### 13.1 Entscheidungen

| # | Thema | Entscheidung |
|---|-------|--------------|
| E-01 | Einsatzliste | Eine gemeinsame Liste; Status, Ende und Bericht pflegt jeder Hauptnutzer selbst (Teilnahme). |
| E-02 | Rollen | Admin (genau einer, zugleich Hauptnutzer), Hauptnutzer, eingeschränkter Nutzer; von Haupt- und eingeschränkten Nutzern beliebig viele. |
| E-03 | Eingeschränkte Nutzer | Nur lesen; sehen Kennzeichnung, Stichwort, Beginn, Status, Ende, Bericht und „Bericht offen“, nicht Adresse und weitere Infos. |
| E-04 | Registrierung | Offen; erster Nutzer wird Admin, alle weiteren sind eingeschränkt. |
| E-05 | Nutzerverwaltung | Admin kann befördern, zurückstufen, löschen und die Admin-Rolle übergeben. |
| E-06 | Kennzeichnung | Buchstabe + Zahl; Buchstaben und Zahlenbereich je Buchstabe pflegt nur der Admin. Start: B, H, C mit 1–5. |
| E-07 | Stichwort | Gebunden an den Buchstaben; im Dropdown ergänzbar; Bearbeiten und Löschen nur Admin. |
| E-08 | Pflichtfelder | Keine. |
| E-09 | Adresse, weitere Infos, Bericht | Freitext. |
| E-10 | Beginn und Ende | Beginn (Alarmzeit) gemeinsam; Ende je Teilnahme, nur bei `Komme`. |
| E-11 | Status | `Offen` (Standard), `Komme`, `Komme nicht`, `Nicht verfügbar`. |
| E-12 | Statuswechsel weg von `Komme` | Ende und Bericht bleiben gespeichert, werden ausgeblendet. |
| E-13 | Technik | Analog Haushaltsprojekt, inkl. Docker, CI, Release-Workflow und Handbuch. |
| E-14 | Gelöschte Einträge | Bestehende Einsätze behalten gelöschte Kennzeichnungen/Stichworte, neue können sie nicht wählen. |
| E-15 | Löschen von Einsätzen | Nur der Admin. |
| E-16 | Änderungshistorie | Nur für gemeinsame Felder; wer, wann, Feld, alt → neu; sichtbar nur für den Admin. |
| E-17 | Doppelte Einsätze | Warnung bei gleicher Kennzeichnung und Beginn höchstens 5 Minuten auseinander. |
| E-18 | Detailansicht | Status und Berichte aller Hauptnutzer sichtbar, mit Namen. |
| E-19 | Ausgewählte Person | Hauptnutzer: standardmäßig man selbst, je Sitzung zurückgesetzt. Eingeschränkt: standardmäßig der Admin, gespeichert. |
| E-20 | Filter und Sortierung | Status, Kennzeichnung, Stichwort, „Bericht offen“, Zeitraum; Sortierung nach Datum & Uhrzeit; Standard: `Komme`, neuester oben. |
| E-21 | Statistik | Frei wählbarer Zeitraum, mit Diagrammen; „Gesamt“ und „Person X“ wie in 6.6; Offen zählt als „Nicht aktiv“. |
| E-22 | Export | CSV und PDF; alle oder gefiltert; Gesamt oder Person X; nur Felder des eingeschränkten Nutzers. |
| E-23 | Darstellung | Hell/Dunkel je Nutzer, Standard folgt dem System. |
| E-24 | Betrieb | Im Heimnetz. |
| E-25 | Name | „Einsatz-Log“. |
| E-26 | Lizenz | Die im Repository liegende GNU GPL v3. |

### 13.2 Offene Fragen

| # | Frage | Stelle |
|---|-------|--------|
| O-01 | Bezeichnung der Statusgruppe: „Nicht aktiv“ oder „Inaktiv“? | 6.6 |
| O-02 | Rolle des bisherigen Admins nach Übergabe? | R-02 |
| O-03 | Darf der Admin sich selbst löschen/zurückstufen? | R-04 |
| O-04 | Teilnahmen eines zurückgestuften oder gelöschten Hauptnutzers? | R-05 |
| O-05 | Bedeutung der Buchstaben als Beschriftung mitführen? | 5.2 |
| O-06 | Startliste für Stichworte? | 5.2 |
| O-07 | Anlegen in der Historie vermerken? Historie beim Löschen eines Einsatzes? | 5.2 |
| O-08 | Doppelungswarnung auch beim Bearbeiten? | 6.4 |
| O-09 | Gelöschte Einträge beim Bearbeiten bestehender Einsätze; verkleinerter Zahlenbereich? | 6.5 |
| O-10 | Einsätze ohne Beginn in der Statistik? | 6.6 |
| O-11 | Einsatzstunden bei fehlendem/ungültigem Ende? Darstellung der Stunden? | 6.6, 6.7 |
| O-12 | Einsätze vor Registrierung/Beförderung einer Person in deren Statistik? | 6.6 |
| O-13 | Statistik nach Kennzeichnung: je Buchstabe oder je Buchstabe + Zahl? | 6.6 |
| O-14 | Filter als Mehrfach- oder Einzelauswahl? Kennzeichnungsfilter nach Buchstabe und/oder Zahl? Filter beim Zurückkehren erhalten? | 7.4 |
| O-15 | Aufbau des Exports bei „Gesamt“; Inhalt des PDF? | 7.5 |
| O-16 | Art der PDF-Erzeugung? | 10.1 |
| O-17 | Farbpalette? | 10.7 |
| O-18 | Prioritäten (7), Abgrenzung (12) und Meilensteine (14) bestätigen. | 7, 12, 14 |

## 14. Meilensteine

*(Vorschlag)*

| # | Meilenstein | Inhalt |
|---|-------------|--------|
| M0 | **Spezifikation** | Dieses Dokument abstimmen, offene Fragen klären, freigeben. |
| M1 | **Grundgerüst** | `package.json`, Express + EJS, Pico + `theme.css`, Layout, Konfiguration, Speicherschicht mit Startdaten und Tests. |
| M2 | **Login und Rollen** | F-01 bis F-03, F-05, Rollenprüfung (Kapitel 4) mit Tests. |
| M3 | **Verwaltung** | F-10 bis F-13: Nutzer, Kennzeichnungen, Stichworte. |
| M4 | **Einsätze** | F-20 bis F-25, F-14, F-15: Anlegen, Bearbeiten mit Historie, Teilnahme, Doppelungswarnung, Löschen. |
| M5 | **Übersicht** | F-30 bis F-36: Karten, Personenauswahl, Filter, Sortierung, Detailansicht. → **Kern fertig** |
| M6 | **Statistik und Export** | F-40 bis F-45. |
| M7 | **Betrieb** | Sicherheit (10.4), Docker, CI/CD, Handbuch (F-06), F-04. → **Version 1.0.0** |

## 15. Lizenz

Das Projekt steht unter der **GNU General Public License v3.0** (siehe [`LICENSE`](LICENSE)); in der `package.json` entsprechend `"license": "GPL-3.0-or-later"`.

## 16. Änderungshistorie

| Version | Änderungen |
|---------|------------|
| v0.1 | Erster Entwurf aus dem im Chat abgestimmten Konzept. |
