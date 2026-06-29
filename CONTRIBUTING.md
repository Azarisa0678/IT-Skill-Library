# Beitragen zur IT Skill Library

Danke für dein Interesse! Jeder Beitrag hilft, die Library besser zu machen.

---

## Arten von Beiträgen

### 🐛 Fehler melden
- Falscher Befehl / veraltete Syntax
- Falsche regulatorische Angabe
- Fehlendes Referenzmodul

→ [Bug melden](../../issues/new?template=bug-report.md)

### 💡 Neues Referenzmodul vorschlagen
- Neues Thema das fehlt (z.B. neues Tool, neue Regulation)
- Neuer Sektor (z.B. Öl & Gas, Luft- und Raumfahrt)

→ [Modul vorschlagen](../../issues/new?template=skill-request.md)

### 📝 Bestehende Inhalte verbessern
- Veraltete Befehle aktualisieren
- Beispiele ergänzen
- Deutsche Übersetzungen verbessern

---

## Workflow

```
1. Fork erstellen
2. Branch anlegen: git checkout -b fix/modbus-example
3. Änderung vornehmen
4. Commit: git commit -m "fix: Modbus-Beispiel für IEC 62351 aktualisiert"
5. Push: git push origin fix/modbus-example
6. Pull Request öffnen
```

---

## Referenzmodul erstellen

Jedes Referenzmodul ist eine Markdown-Datei unter `skills/<skill-name>/references/`.

**Struktur:**
```markdown
# Modulname — Referenzmodul

Kurze Beschreibung (1-2 Sätze).

---

## Abschnitt 1

Inhalt mit Code-Beispielen...

---

## Weiterführende Ressourcen

- Link 1
- Link 2
```

**Qualitätskriterien:**
- ✅ Produktionsreife Code-Beispiele (nicht nur Pseudocode)
- ✅ DACH-Regulierung berücksichtigt wo relevant
- ✅ Deutsche Kommentare in Code-Beispielen
- ✅ Konkrete Werkzeuge / Befehle statt abstrakte Konzepte
- ✅ Checklisten am Ende des Moduls

---

## Commit-Konventionen

```
feat:  Neues Referenzmodul
fix:   Korrektur eines Fehlers
docs:  README / Dokumentation
chore: Strukturänderungen
```

---

## Fragen?

Einfach ein [Issue öffnen](../../issues/new) oder eine [Discussion starten](../../discussions).
