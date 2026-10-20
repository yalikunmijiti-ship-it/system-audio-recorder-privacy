# System Audio Recorder

**System Audio Recorder** ist eine plattformübergreifende Audioaufnahme-App auf
Basis von .NET MAUI. Unter Android kann sie Systemaudio, das
Smartphone-Mikrofon oder beide Quellen gleichzeitig aufnehmen. Unter Windows
zeichnet sie den wiedergegebenen Systemton per WASAPI-Loopback auf – ohne
Mikrofon und ohne physischen „Stereo-Mix“-Eingang der Soundkarte.

Aktuelle Version: **1.1.1**  
Android `versionCode`: **3**  
Paket-ID: `de.yalikun.systemaudiorecorder`

## Funktionen

### Android

- Drei wählbare Aufnahmequellen:
  - **Systemaudio**
  - **Smartphone-Mikrofon**
  - **Systemaudio + Mikrofon**
- WAV-Aufnahme mit PCM 16 Bit und 48 kHz.
- Mikrofonaufnahme in Stereo, falls das Gerät dies unterstützt.
- Automatischer Rückfall auf Mono bei nicht unterstützter Stereoaufnahme.
- Lokale Mischung der Systemaudio- und Mikrofonspur zu einer Stereo-WAV-Datei.
- Aufnahmedauer und Lautstärkepegel in Vollansicht und Overlay.
- Verschiebbares Overlay mit:
  - Start
  - Pause
  - Fortsetzen
  - Stopp
  - Rückkehr zur Vollansicht
  - Schließen
- Öffentliche Speicherung über Android MediaStore.
- Benutzeroberfläche auf Deutsch, Englisch und vereinfachtem Chinesisch.
- Getestet auf Samsung One UI und Xiaomi HyperOS.

### Windows

- Systemaudioaufnahme über `WasapiLoopbackCapture` von NAudio.
- Kein physischer „Stereo-Mix“-Eingang erforderlich.
- Auswahl des Audioausgabegeräts.
- Auswahl verschiedener WAV-Qualitäten:
  - ursprüngliches 32-Bit-Gleitkommaformat
  - PCM 24 Bit
  - PCM 16 Bit
- Auswahl eines eigenen Speicherordners.
- Aufnahmedauer, Status- und Lautstärkeanzeige.

## Android-Aufnahmeablauf

In Version 1.1.1 beginnt die Aufnahme im Mikrofon- und kombinierten Modus
bewusst **nicht** unmittelbar in der Vollansicht:

1. Aufnahmequelle auswählen.
2. In der Vollansicht **Aufnahme starten** antippen.
3. Die erforderlichen Android-Berechtigungen bestätigen.
4. Die App startet den benötigten Foreground-Service, solange die Activity noch
   sichtbar ist.
5. Das Overlay erscheint im Zustand **Bereit**.
6. Erst durch **START im Overlay** werden `AudioRecord` und gegebenenfalls
   `MediaProjection` aktiviert.
7. Aufnahmedauer und Pegelanzeige beginnen gleichzeitig mit der echten
   Aufnahme.

Der vorbereitete Foreground-Service allein nimmt noch keine Audiodaten auf.

## Kombinierte Aufnahme

Im Modus **Systemaudio + Mikrofon** werden zwei getrennte PCM-Spuren erfasst.
Nach dem Stoppen führt die App beide Spuren lokal zu einer Stereo-WAV-Datei
zusammen.

- Systemaudio: Stereo, PCM 16 Bit, 48 kHz
- Mikrofon: Stereo, falls unterstützt; andernfalls Mono
- Systemaudio-Mischpegel: 70 %
- Mikrofon-Mischpegel: 80 %
- Begrenzung gegen numerisches Übersteuern
- Berücksichtigung eines möglichen Startversatzes zwischen beiden Recordern
- Mono-Mikrofonsignal wird mittig auf den linken und rechten Kanal gelegt

> **Praxistipp:** Bei Gesang oder Sprache über abgespielter Musik sollten
> Kopfhörer verwendet werden. Bei Wiedergabe über die Smartphone-Lautsprecher
> nimmt das Mikrofon die Musik zusätzlich mit Raumhall und Zeitversatz auf; das
> kann zu Echo oder Kammfiltereffekten führen.

## Speicherung

### Android

Fertige Aufnahmen werden über MediaStore öffentlich gespeichert:

```text
Interner Speicher/Music/SystemAudioRecorder
```

Typische Dateinamen:

```text
Systemaudio_JJJJ-MM-TT_HH-mm-ss.wav
Mikrofonaufnahme_JJJJ-MM-TT_HH-mm-ss.wav
Gemischte_Aufnahme_JJJJ-MM-TT_HH-mm-ss.wav
```

Während der Aufnahme verwendete temporäre Dateien liegen im privaten
App-Cache und werden nach der Verarbeitung entfernt.

### Windows

Standardordner:

```text
Dokumente/PC-Ton-Aufnahmen
```

Der Ordner kann in der Anwendung geändert werden.

## Android-Berechtigungen

Die App deklariert folgende Berechtigungen:

| Berechtigung | Zweck |
|---|---|
| `RECORD_AUDIO` | Mikrofonaufnahme und Android AudioPlaybackCapture |
| `FOREGROUND_SERVICE` | zuverlässiger Aufnahmedienst im Vordergrund |
| `FOREGROUND_SERVICE_MICROPHONE` | Mikrofonaufnahme über den Foreground-Service |
| `FOREGROUND_SERVICE_MEDIA_PROJECTION` | Systemaudioaufnahme über MediaProjection |
| `POST_NOTIFICATIONS` | Statusbenachrichtigung des Aufnahmedienstes |
| `SYSTEM_ALERT_WINDOW` | verschiebbares Bedien-Overlay über anderen Apps |
| `ACCESS_NETWORK_STATE` | durch das Android-/MAUI-Projekt deklarierter Netzwerkstatus |
| `INTERNET` | durch das Android-/MAUI-Projekt deklarierter Netzwerkzugriff; die aktuelle App lädt keine Audioaufnahmen hoch |

Die Systemaudioaufnahme mit Android AudioPlaybackCapture benötigt Android 10
(API 29) oder neuer. Nicht jede Android-App erlaubt, dass ihr Audio durch eine
andere App aufgenommen wird. Geschützte Medien oder Audioquellen mit gesperrter
Wiedergabeerfassung können deshalb stumm bleiben.

## Datenschutz

- Audioaufnahmen werden lokal auf dem Gerät verarbeitet.
- Die App lädt keine Aufnahmen auf einen Server hoch.
- Es werden keine Werbe-, Analyse- oder Tracking-SDKs verwendet.
- Die App benötigt kein Benutzerkonto.
- Die gewählte Sprache wird lokal gespeichert.

Datenschutzerklärung: **https://github.com/yalikunmijiti-ship-it/system-audio-recorder-privacy/blob/main/index.html**

## Technischer Aufbau

- .NET 10
- .NET MAUI Single Project
- C# und XAML
- MVVM mit `CommunityToolkit.Mvvm`
- Dependency Injection über `Microsoft.Extensions.DependencyInjection`
- NAudio/WASAPI unter Windows
- Android `AudioRecord`, `AudioPlaybackCapture`, `MediaProjection` und
  `MediaStore`
- Android Foreground Services und eigenes Overlay-Fenster

Wichtige Projektbereiche:

```text
SystemAudioRecorder/
├── Models/
├── Services/
├── ViewModels/
├── Views/
├── Localization/
├── Platforms/
│   ├── Android/
│   ├── Windows/
│   ├── iOS/
│   └── MacCatalyst/
└── Resources/
```

## Voraussetzungen

- Windows 11 oder ein unterstütztes Entwicklungsbetriebssystem
- Visual Studio mit installierter .NET-MAUI-Workload
- .NET 10 SDK
- Android SDK und ADB für Android-Builds
- Android 10 oder neuer für Systemaudioaufnahme
- Ein echtes Android-Gerät wird für AudioPlaybackCapture empfohlen

Die Projektdatei definiert folgende Zielframeworks:

```text
net10.0-android
net10.0-windows10.0.19041.0
net10.0-ios
net10.0-maccatalyst
```

Android und Windows enthalten die vollständigen Aufnahmeimplementierungen. Die
vorhandenen iOS- und MacCatalyst-Ziele bedeuten nicht, dass dort bereits alle
Aufnahmefunktionen implementiert sind.

## Projekt öffnen und erstellen

Projektmappe öffnen:

```text
SystemAudioRecorder.slnx
```

Pakete und Workloads wiederherstellen:

```powershell
dotnet workload restore .\SystemAudioRecorder\SystemAudioRecorder.csproj
dotnet restore .\SystemAudioRecorder\SystemAudioRecorder.csproj
```

Android-Debugbuild:

```powershell
dotnet build .\SystemAudioRecorder\SystemAudioRecorder.csproj `
  -f net10.0-android `
  -c Debug
```

Windows-Debugbuild:

```powershell
dotnet build .\SystemAudioRecorder\SystemAudioRecorder.csproj `
  -f net10.0-windows10.0.19041.0 `
  -c Debug
```

Bei hartnäckigen Buildfehlern Visual Studio schließen und nur die erzeugten
Ordner `.vs`, `bin` und `obj` löschen. Danach NuGet-Pakete und Workloads erneut
wiederherstellen.

## Android testen

Empfohlene Reihenfolge:

1. **Smartphone-Mikrofon** auswählen.
2. Berechtigungen bestätigen und die Aufnahme im Overlay starten.
3. Pause, Fortsetzen und Stopp prüfen.
4. Die WAV-Datei unter `Music/SystemAudioRecorder` abspielen.
5. **Systemaudio** auswählen und den MediaProjection-Dialog bestätigen.
6. Musik wiedergeben und die Systemaudioaufnahme testen.
7. **Systemaudio + Mikrofon** auswählen.
8. Musik abspielen und gleichzeitig sprechen oder singen.
9. Die erzeugte gemischte WAV-Datei kontrollieren.

## Bekannte Einschränkungen

- Android entscheidet, welche Apps ihr Wiedergabeaudio für
  AudioPlaybackCapture freigeben.
- DRM-geschützte Inhalte können nicht oder nur stumm aufgenommen werden.
- Mikrofon-Stereo hängt vom Audiotreiber und der Hardware des Geräts ab.
- Im kombinierten Modus kann die Wiedergabe über Lautsprecher Echo verursachen.
- Herstelleranpassungen wie Samsung One UI oder Xiaomi HyperOS können
  zusätzliche Berechtigungs- oder Energiespareinstellungen verlangen.
- Das Overlay benötigt die Android-Sonderberechtigung „Über anderen Apps
  einblenden“.

## Version 1.1.1

- Mikrofon- und kombinierte Aufnahme beginnen erst nach **START im Overlay**.
- Foreground-Service wird bei sichtbarer Vollansicht sicher vorbereitet.
- Während der Vorbereitung werden noch keine Audiodaten erfasst.
- Stereo-/Mono-Erkennung des Mikrofons bleibt erhalten.
- Einheitliche Paketversion `1.1.1` für alle Zielframeworks.

## Kontakt

Entwickler: **Yalikun Mijiti**  
E-Mail: **yalikun.mijiti@gmail.com**

## Hinweis

Tonaufnahmen dürfen nur unter Beachtung der geltenden Gesetze und der Rechte
anderer Personen erstellt werden. Der Nutzer ist dafür verantwortlich, vor der
Aufnahme erforderliche Einwilligungen einzuholen und urheberrechtliche sowie
persönlichkeitsrechtliche Vorgaben zu beachten.
