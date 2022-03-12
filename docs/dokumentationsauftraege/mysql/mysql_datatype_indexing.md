# MySQL Datentypen und Indexierung

## Datentypen

Hier eine ganze Liste von MySQL Datentypen. Die meisten Datentypen sind von gleicher Art (z.B INTEGER), aber mit unterschiedlicher Speichergrösse. Je nach Anwendungsfall kann man so viel Speicher sparen. Bei vielen Datentypen kann man auch Optionen angeben, z.B bei Fliesskommazahlen wie viel Nachkommastellen es haben darf, bei Integer z.B Zahlen, welche keine Vorzeichen haben (unsigned) (also anstatt -128-127, 0-255).
Nicht aufgelistet ist hier der ```BOOLEAN```. Für diesen verwendet man oft den ```TINYINT``` und setzt einfach 0 oder 1 ein.

| Datentyp  | Speicherplatz  | Optionen                | Beschreibung                                                                                                                                                                              |
| --------- | -------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| TINYINT   | 1 Byte         |   \[(M)\] \[U\] \[Z\]   | Ganzzahlen von 0 bis 255 oder von -128 bis 127.                                                                                                                                           |
| SMALLINT  | 2 Bytes        |   \[(M)\] \[U\] \[Z\]   | Ganzzahlen von 0 bis 65.535 oder von -32.768 bis 32.767.                                                                                                                                  |
| MEDIUMINT | 3 Bytes        |   \[(M)\] \[U\] \[Z\]   | Ganzzahlen von 0 bis 16.777.215 oder von -8.388.608 bis 8.388.607.                                                                                                                        |
| INT       | 4 Bytes        |   \[(M)\] \[U\] \[Z\]   | Ganzzahlen von 0 bis ~4,3 Mill. oder von -2.147.483.648 bis 2.147.483.647.                                                                                                                |
| INTEGER   | 4 Bytes        |   \[(M)\] \[U\] \[Z\]   | Alias für INT.                                                                                                                                                                            |
| BIGINT    | 8 Bytes        |   \[(M)\] \[U\] \[Z\]   | Ganzzahlen von 0 bis 2^64-1 oder von -(2^63) bis (2^63)-1.                                                                                                                                |
| FLOAT     | 4 Bytes        |   \[(M,D)\] \[U\] \[Z\] | Fließkommazahl mit Vorzeichen. Wertebereich von -(3,402823466×10<sup>38</sup>) bis -(1,175494351×10<sup>\-38</sup>), 0 und 1,175494351×10<sup>\-38</sup> bis 3,402823466×10<sup>38</sup>. |
| DOUBLE    | 8 Bytes        |   \[(M,D)\] \[U\] \[Z\] | Fließkommazahl mit Vorzeichen. Wertebereich von -(1,79769×10<sup>308</sup>) bis -(2.22507×10<sup>\-308</sup>), 0 und 2.22507×10<sup>\-308</sup> bis 1,79769×10<sup>308</sup>              |
| REAL      | 8 Bytes        |   \[(M,D)\] \[U\] \[Z\] | Alias für DOUBLE.                                                                                                                                                                         |
| DECIMAL   | M+x Bytes      |   \[(M,D)\] \[U\] \[Z\] | Fließkommazahl mit Vorzeichen. Speicherbedarf: x=1 wenn D=0, sonst x=2. Ab MySQL 5.1 binär gespeichert, zuvor als String.                                                                 |
| NUMERIC   | M+x Bytes      |   \[(M,D)\] \[U\] \[Z\] | Alias für DECIMAL.                                                                                                                                                                        |
| DATE      | 3 Bytes        |   -                     | Datum im Format 'YYYY-MM-DD'. Wertebereich von 01.01.1000 bis 31.12.9999.                                                                                                                 |
| DATETIME  | 8 Bytes        |   -                     | Datumsangabe im Format 'YYYY-MM-DD hh:mm:ss'. Wertebereich entspricht DATE.                                                                                                               |
| TIMESTAMP | 4 Bytes        |   -                     | Zeitstempel. Wertebereich: 1.1.1970 bis 19.01.2038. Das Format variiert in den MySQL-Versionen.                                                                                           |
| TIME      | 3 Bytes        |   -                     | Zeit zwischen -838:59:59 und 839:59:59. Ausgabe: hh:mm:ss.                                                                                                                                |
| YEAR      | 1 Byte         |   \[(2\|4)\]             | Jahr zwischen 1901 bis 2155 bei (4) und zwischen 1970 bis 2069 bei (2).                                                                                                                   |
| CHAR      | M Byte(s)      |   (M) \[BINARY\]        | Zeichenkette fester Länge M. Wertebereich für M: 0 bis 255.                                                                                                                               |
| VARCHAR   | L+1 Bytes      |   (M) \[BINARY\]        | Zeichenkette variabler Länge, Maximum ist M. Wertebereich für M: 0 bis 255.                                                                                                               |
| BINARY    | M Bytes        |   (M)                   | Zum Speichern binärer Strings, unabhängig vom Zeichensatz. Wertebereich für M: 0 bis 255. Weiterer Typ: VARBINARY                                                                         |
| BLOB      | L+2 Bytes      |   (M)                   | Binäres Objekt mit variablen Daten. Weitere Typen: TINYBLOB, MEDIUMBLOB und LONGBLOB. M ist ab Version 4.1 definierbar.                                                                   |
| TEXT      | L+2 Bytes      |   (M)                   | Wie BLOB. Ignoriert beim Sortieren & Vergleichen Groß- und Kleinschreibung. Weitere Typen: TINYTEXT, MEDIUMTEXT, LONGTEXT. M ist ab Version 4.1 definierbar.                              |
| SET       | x Bytes        |   ('val1', 'val2', ...) | String-Objekt mit verschiedenen Variablen. 64 sind maximal möglich. Speicherbedarf: x ist 1, 2, 3, 4 oder 8.                                                                              |

Legende:

| Option | Bedeutung
| ------ | --------------------------------------------- |
| ^      | Potenzzeichen                                 |
| \[ \]  | Optionaler Parameter                          |
| BINARY | Attribut für die Sortierung                   |
| D      | Anzahl der Kommastellen bei einer Dezimalzahl |
| L      | Stringlänge (Berechnung Speicherbedarf)       |
| M      | Maximale Anzahl der gezeigten Stellen         |
| Mill.  | Milliarden                                    |
| U      | UNSIGNED (Zahl ohne Vorzeichen)               |
| Z      | Zerofill                                      |

### ENUM

ENUM ist eine Liste von Werten (val1, val2, ...) mit 65.535 möglichen und eineindeutigen Elementen.
Dies kann man super brauchen, wenn man in einer Applikation z.B DropDown Menüs darstellen will. Z.B Attribut Status: ```Entwurf```, ```In Prüfung``` oder ```Freigegeben```. Wird von der Applikation ein Wert mitgegeben, welcher nicht in der ENUM Liste enthalten ist, wird einfach nichts eingefügt. Dies kommt wieder der Daten Konsistenz zu gute.

### JSON                                                                                              |

Es kann reines JSON in einer Zelle eingefügt werden. So muss man z.B nicht 3 Tabellen machen (Benutzer, Rollen, Rollenzuordnungen) sondern kann direkt alle Rollen in einem Feld des Benutzers einfügen. Z.B:

```json
{
  "roles": ["admin", "production"]
}
```

Diese können nun von einer Applikation durch iteriert werden.

## Indexierung

Ab 10'000 Datensätzen in einer Tabelle wird empfohlen die Daten zu indexieren. So kann man die performance erhöhen, wenn man z.B ```WHERE``` oder ```ORDER BY``` nutzt. Durch die Indexierung muss MySQL nicht mehr die ganze Tabelle durchforschen. Um Indizes zu erzeugen gibt es vier Indexierungstypen:

* Der **Primary Key** (meist eine ID) wird für die primäre Indexierung einer Tabelle genutzt. Das heisst, dass gewisse Daten, z.B Vorname, Nachname in der gleichen Zeile zusammengehören.

* **Unique** erlaubt keine doppelten Einträge eines Attributes. Wenn ein Benutzername oder eine Telefonnummer eindeutig sein soll, sollte dieser Typ verwendet werden.

* **Index** ist das Gegenteil von Unique. Es sind gleiche Einträge eines Attributes möglich. (z.B Geburtsdatum)

* **Fulltext** ist sehr aufwendig und jedes Wort wird indiziert. Sollte nur verwendet werden, falls man auch die MySQL Volltextsuche verwendet. (Spezielle SQL-Befehle)

Beim erstellen eines Attributes können diese 4 (natrülich nicht alle beim gleichen Attribut) verwendet werden und alle Einträge (z.B Telefonnummern) werden indiziert. Dazu wird eine eigene Datenstruktur angelegt. Diese weiss dann genau wo die Daten auf der Platte liegen und MySQL ist somit viel performanter als wenn es die ganze Tabelle durchgehen müsste.
