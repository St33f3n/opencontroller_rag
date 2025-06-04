
1. Grundarchitektur - Drei Hauptkomponenten:
- UI Thread: Menüdarstellung und Einstellungsverwaltung
- Gamepad Event Thread: Verarbeitet Controller-Input
- Control Loop: Zentrale State Machine Steuerung mit statum crate

2. UI-System:
- State Machine verwaltet Menüzustände (MainMenu, SubMenu, Settings, etc.)
- Dual-Input über Gamepad und Touchpad
- Touchpad wird als Linux-Maus-Event behandelt (einfache Implementation)
- Erweiterbar für spätere Input-Methoden durch Abstraktionsebene

3. Gamepad Event Verarbeitung:
- 130ms Zeitfenster für Event-Sammlung
- Events innerhalb des Fensters werden gesammelt und analysiert
- Validierung von Tastendrücken:
  * Mindestens 30ms Druckzeit für validen Input
  * Kürzere Drücke werden gefiltert
  * Mehrere valide Drücke im Zeitfenster ermöglichen Tastenkombinationen

4. Timing-Überlegungen:
- Menschliche Reaktionszeit ~100-150ms für taktile Reize
- 130ms Fenster liegt unter wahrnehmbarer Verzögerung (~200ms)
- Frame-übergreifende Events sind unkritisch wegen menschlicher Reaktionszeit
- Intervall kann bei Bedarf angepasst werden

5. Vorteile des Systems:
- Robuste Erkennung von Einzel- und Kombitasten
- Filtert unbeabsichtigte kurze Inputs
- Skalierbare Architektur
- Klare Trennung der Zuständigkeiten
- Einfache Erweiterbarkeit

6. State Machine (statum crate):
- Verwaltet Programmzustände
- Reagiert auf validierte Events aus dem Gamepad Loop
- Ermöglicht vorhersagbare Zustandsübergänge
- Trennt Logik von Input-Verarbeitung
