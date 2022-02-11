# Dokumentationsaufträge

## Historisches zu Datenbanken

### SQL (Structured Query Language)

Mithilfe von ```SELECT, INSERT, DELETE und UPDATE``` fragt man Daten ab, setzt neue in die Datenabnk ein, bearbeitet oder löscht sie.

### SQL DDL (Data Definition Language)

Mithilfe von ```CREATE und DROP``` erstellt man oder löscht eine Datenstruktur. (Tabellen)

### SQL DML (Data Manipulation Language)

Mithilfe von ```ALTER``` kann man Spalten (Attribute) hinzufügen, bearbeiten oder löschen.

## Begrifflichkeiten und Abkürzungen

### Transaktion

Eine Transaktion beinhaltet ein oder mehrere SQL-Statements. Diese werden nacheinander ausgeführt. Falls ein Befehl in dieser Transakation fehlschlägt, kann ein Rollback gemacht werden, ohne dass produktive Daten auf der Datenbank verändert  wurden. Wenn die Transaktion erfolgreich ist, kann sie commited werden und auf die Datenbank übernommen werden.

### ACID

* A = Atomicity / Abgeschlossenheit
* C = Consistency / Konsistenz
* I = Isolation / Isolation
* D = Durability / Dauerhaftigkeit

**Atomicity** beschreibt die Zielsetzung, dass Datenveränderungen mittels einer Transaktion ganz oder gar nicht ausgeführt werden. Falls ein Befehl in der Transaktion fehlschlägt, werden alle anderen Befehle rückgängig gemacht (Rollback).

**Consistency** beschreibt die Einhaltung der Integritätsbedingungen. Das heisst, dass die Relationen (Beziehungen in andere Tabellen z.b User --> Projekte) korrekt sind.

**Isolation** beschreibt, dass jede Befehlsfolge (Transaktion) nacheinander ausgeführt wird. So wird nicht ausversehen etwas negativ verändert / überschrieben. Deshalb werden Tabellen für diese Zeit gelockt / gesperrt. Dies reduziert aber massiv die Performance. Man wartet bis jemand seine Daten vollständig in die DB geschrieben hat.

**Durability** beschreibt, dass die Daten nach einer erfolgreichen Transaktion dauerhaft in der Datenbank gespeichert sind.

### BASE