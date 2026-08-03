
# 🤖 Robotics Platform Teil 1 {: #robotics-platform-main }

**Technische Projekt- und Administrationsdokumentation**  
**Version:** 0.2-alpha  
**Systemtyp:** Browserbasierte Robotik-Telemetrie-Plattform  
**Status:** Funktionierender Prototyp



## 1. Projektübersicht

Die Robotics Platform ist eine selbst entwickelte Webplattform zur Verwaltung und Überwachung von Robotern. Ziel ist eine zentrale Oberfläche, über die Roboterdaten verwaltet, gespeichert, angezeigt und später in Echtzeit verarbeitet werden können.

Der aktuelle Prototyp simuliert Roboter und deren Telemetriedaten.

### Zielsetzung

Aufbau eines modularen Robotik-Managementsystems.

| Funktion | Status |
|----------|--------|
| Roboterverwaltung | ✅ |
| Telemetrie-Speicherung | ✅ |
| REST API | ✅ |
| React Dashboard | ✅ |
| Datenbank | ✅ |
| WebSocket-Vorbereitung | ✅ |
| Live-Steuerung | ❌ |
| 3D-Simulation | ❌ |
| Hardware-Anbindung | ❌ |

### Bekannte Einschränkungen

- Kein automatisches Live-Update im Frontend
- WebSocket-Client noch nicht implementiert
- Keine 3D-Darstellung
- Keine Steuerbefehle
- Kein Login oder Benutzerverwaltung
- Health-Check zeigt `database` und `redis` als `"unknown"`
- Telemetrie-DTO fehlt

---

## 2. Systemarchitektur

```
Benutzer
    |
    ↓
React Web Interface (Port 5173)
    |
    ↓
REST API
    |
    ↓
NestJS Backend (Port 3000)
    |
    ↓
Prisma ORM
    |
    ↓
PostgreSQL Datenbank
```

### Gesamt-Datenfluss

```
Browser (React)
    ↓
Axios GET / PATCH
    ↓
NestJS Backend (Port 3000)
    ↓
CORS-Prüfung (origin: '*')
    ↓
Controller
    ↓
Service (RobotsService)
    ↓
PrismaService (PrismaClient + PrismaPg Adapter)
    ↓
PostgreSQL
    ↓
Antwort als JSON
    ↓
Axios response.data
    ↓
React State
    ↓
Browser-Anzeige
```

### WebSocket-Datenfluss (vorbereitet)

```
RobotsService.updateTelemetry()
    ↓
TelemetryGateway.sendTelemetry(data)
    ↓
Socket.IO Server
    ↓
Event: 'robotTelemetry'
    ↓
Alle verbundenen Clients
```

> Der Frontend-Client für WebSocket ist noch nicht implementiert.

---

## 3. Entwicklungsumgebung

| Komponente | Wert |
|------------|------|
| Server-Hostname | `xxxxxxxxx` |
| Betriebssystem | Linux Server |
| Server-IP | `xxx.xxx.xxx.xxx` |
| Projektpfad | `/home/xxxxx/projects/robotics-platform` |

### Ports

| Komponente | Port | URL |
|------------|------|-----|
| React Dev-Server | 5173 | `http://xxx.xxx.xxx.xxx:5173` |
| NestJS Backend | 3000 | `http://xxx.xxx.xxx.xxx:3000` |

### Startbefehle

| Komponente | Befehl |
|------------|--------|
| Backend | `cd ~/projects/robotics-platform/backend/api && npm run start:dev` |
| Frontend | `cd ~/projects/robotics-platform/frontend && npm run dev -- --host 0.0.0.0` |
| Prisma Studio | `npx prisma studio` |
| Prisma Migrate | `npx prisma migrate dev` |
| Prisma Generate | `npx prisma generate` |

---

## 4. Backend-Architektur

### 4.1 Einstiegspunkt – main.ts

**Datei:** `src/main.ts`

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.enableCors({
    origin: '*'
  });

  await app.listen(3000, '0.0.0.0');
}

bootstrap();
```

| Zeile | Bedeutung |
|-------|-----------|
| `NestFactory.create(AppModule)` | Erstellt NestJS-App mit allen Modulen |
| `app.enableCors({origin:'*'})` | Erlaubt Cross-Origin-Requests von allen Hosts |
| `app.listen(3000,'0.0.0.0')` | Lauscht auf Port 3000 an allen Netzwerkinterfaces |

> **Warum CORS?** Frontend (`xxx.xxx.xxx.xxx:5173`) und Backend (`xxx.xxx.xxx.xxx:3000`) laufen auf unterschiedlichen Ports. Ohne CORS würde der Browser API-Anfragen blockieren.

> **Warum `0.0.0.0`?** Damit ist das Backend nicht nur lokal (`localhost`), sondern auch von anderen Geräten im Netzwerk erreichbar.

### 4.2 AppModule

**Datei:** `src/app.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { PrismaModule } from './prisma/prisma.module';
import { RobotsModule } from './robots/robots.module';
import { TelemetryModule } from './telemetry/telemetry.module';
import { HealthController } from './health/health.controller';

@Module({
  imports: [
    PrismaModule,
    TelemetryModule,
    RobotsModule
  ],
  controllers: [
    AppController,
    HealthController
  ],
  providers: [
    AppService
  ]
})
export class AppModule {}
```

| Import | Zweck |
|--------|-------|
| `PrismaModule` | Globale Datenbankverbindung |
| `TelemetryModule` | WebSocket-Gateway |
| `RobotsModule` | REST-API für Roboter und Telemetrie |
| `AppController` | Standard-Controller |
| `HealthController` | Health-Check-Endpunkt |
| `AppService` | Standard-Service |

### 4.3 Modul-Architektur

```
AppModule
│
├── PrismaModule (@Global)
│   └── PrismaService
│       ├── PrismaPg Adapter
│       └── PostgreSQL
│
├── RobotsModule
│   ├── RobotsController
│   └── RobotsService
│       ├── PrismaService (via DI)
│       └── TelemetryGateway
│
└── TelemetryModule
    └── TelemetryGateway
```

### 4.4 Dependency Injection

NestJS erstellt Objekte automatisch. Im `RobotsService` genügt:

```typescript
constructor(
  private prisma: PrismaService,
  private telemetryGateway: TelemetryGateway
) {}
```

Es ist kein `new PrismaService()` notwendig.

**Vorteile:**
- Keine doppelte Instanziierung
- Zentrale Verwaltung
- Einfache Erweiterbarkeit
- Bessere Testbarkeit
- Lose Kopplung der Komponenten

### 4.5 RobotsModule

**Datei:** `src/robots/robots.module.ts`

```typescript
@Module({
  imports: [TelemetryModule],
  controllers: [RobotsController],
  providers: [RobotsService]
})
export class RobotsModule {}
```

`TelemetryModule` wird importiert, damit `RobotsService` auf `TelemetryGateway` zugreifen kann.

### 4.6 Start der Anwendung

```
main.ts
    ↓
NestFactory
    ↓
AppModule
    ↓
PrismaModule → TelemetryModule → RobotsModule
    ↓
Controller → Services
    ↓
Server bereit auf Port 3000
```

---

## 5. Datenbank & Prisma ORM

### 5.1 PrismaModule

**Datei:** `src/prisma/prisma.module.ts`

```typescript
import { Global, Module } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

| Dekorator | Bedeutung |
|-----------|-----------|
| `@Global()` | Modul ist global verfügbar – kein wiederholter Import nötig |
| `providers` | Registriert `PrismaService` im Modul |
| `exports` | Erlaubt anderen Modulen die Nutzung von `PrismaService` |

### 5.2 PrismaService

**Datei:** `src/prisma/prisma.service.ts`

```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';
import { PrismaPg } from '@prisma/adapter-pg';
import { PrismaClient } from '../../generated/prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {

  constructor() {
    const adapter = new PrismaPg({
      connectionString: process.env.DATABASE_URL!,
    });

    super({ adapter });
  }

  async onModuleInit() {
    await this.$connect();
  }
}
```

| Komponente | Beschreibung |
|------------|--------------|
| `PrismaPg` | PostgreSQL-Adapter für Prisma Client |
| `DATABASE_URL` | Umgebungsvariable mit Verbindungsdaten |
| `OnModuleInit` | Lifecycle-Hook – wird beim Modulstart ausgeführt |
| `$connect()` | Baut die Datenbankverbindung auf |

### 5.3 Prisma Schema

**Datei:** `prisma/schema.prisma`

```prisma
generator client {
  provider     = "prisma-client"
  output       = "../generated/prisma"
  moduleFormat = "cjs"
}

datasource db {
  provider = "postgresql"
}

model Robot {
  id        Int      @id @default(autoincrement())
  name      String
  type      String
  weight    Float?
  speed     Float?
  status    String   @default("idle")
  positionX Float    @default(0)
  positionY Float    @default(0)
  positionZ Float    @default(0)
  battery   Int      @default(100)
  createdAt DateTime @default(now())

  telemetry RobotTelemetry[]
}

model RobotTelemetry {
  id        Int      @id @default(autoincrement())
  robotId   Int
  positionX Float?
  positionY Float?
  positionZ Float?
  speed     Float?
  battery   Int?
  status    String?
  timestamp DateTime @default(now())

  robot Robot @relation(fields: [robotId], references: [id], onDelete: Cascade)
}
```

### 5.4 Model Robot

| Feld | Typ | Pflicht? | Standard | Beschreibung |
|------|-----|----------|----------|--------------|
| `id` | `Int @id` | Ja | `autoincrement()` | Eindeutige ID |
| `name` | `String` | Ja | – | Name des Roboters |
| `type` | `String` | Ja | – | Typ (z. B. Humanoid) |
| `weight` | `Float?` | Nein | – | Gewicht in kg |
| `speed` | `Float?` | Nein | – | Geschwindigkeit |
| `status` | `String` | Ja | `"idle"` | Betriebsstatus |
| `positionX` | `Float` | Ja | `0` | Horizontale Position |
| `positionY` | `Float` | Ja | `0` | Seitliche Position |
| `positionZ` | `Float` | Ja | `0` | Höhe |
| `battery` | `Int` | Ja | `100` | Ladestand in % |
| `createdAt` | `DateTime` | Ja | `now()` | Erstellungszeitpunkt |
| `telemetry` | `RobotTelemetry[]` | – | – | 1:n-Beziehung |

### 5.5 Model RobotTelemetry

| Feld | Typ | Pflicht? | Standard | Beschreibung |
|------|-----|----------|----------|--------------|
| `id` | `Int @id` | Ja | `autoincrement()` | Eindeutige ID |
| `robotId` | `Int` | Ja | – | Fremdschlüssel zu Robot |
| `positionX` | `Float?` | Nein | – | Gespeicherte X-Position |
| `positionY` | `Float?` | Nein | – | Gespeicherte Y-Position |
| `positionZ` | `Float?` | Nein | – | Gespeicherte Z-Position |
| `speed` | `Float?` | Nein | – | Gespeicherte Geschwindigkeit |
| `battery` | `Int?` | Nein | – | Gespeicherter Batteriestand |
| `status` | `String?` | Nein | – | Gespeicherter Status |
| `timestamp` | `DateTime` | Ja | `now()` | Zeitpunkt der Aufzeichnung |
| `robot` | `Robot` | Ja | – | Beziehung zum Roboter |

### 5.6 Beziehung

```
Robot (1)
   │
   ├── RobotTelemetry (n)
   ├── RobotTelemetry (n)
   └── RobotTelemetry (n)
```

- Ein Roboter hat **viele** Telemetrie-Einträge.
- Jeder Telemetrie-Eintrag gehört zu **genau einem** Roboter.
- `onDelete: Cascade` – Bei Löschung eines Roboters werden alle Telemetrie-Einträge automatisch gelöscht.

### 5.7 Migrationen

```bash
# Migration erstellen und anwenden
npx prisma migrate dev

# Prisma Client neu generieren
npx prisma generate

# Prisma Studio öffnen (Browser-Editor)
npx prisma studio
```

| Befehl | Zweck |
|--------|-------|
| `migrate dev` | Vergleicht Schema mit DB, generiert SQL, wendet an |
| `generate` | Aktualisiert TypeScript-Typen und Client |
| `studio` | Öffnet visuellen Editor unter `http://localhost:5555` |

### 5.8 Wichtige Prisma-Methoden

| Methode | Verwendungszweck |
|---------|------------------|
| `prisma.robot.findMany()` | Alle Roboter laden |
| `prisma.robot.findUnique()` | Einzelnen Roboter laden |
| `prisma.robot.create()` | Neuen Roboter erstellen |
| `prisma.robot.update()` | Roboter aktualisieren |
| `prisma.robot.delete()` | Roboter löschen |
| `prisma.robotTelemetry.create()` | Telemetrie speichern |
| `prisma.robotTelemetry.findMany()` | Telemetrie-Historie laden |

### 5.9 Besonderheiten

- **Custom Output-Pfad:** Der Prisma Client wird nach `../generated/prisma` generiert (nicht Standard-`node_modules`).
- **PrismaPg-Adapter:** Native PostgreSQL-Verbindung statt Standard-Prisma-Verbindung.
- **Optionale Felder:** Telemetrie kann unvollständig gespeichert werden (z. B. nur Batteriestand).
- **Int für battery:** Ganzzahlige Prozentwerte statt Float.

---

## 6. REST-API, Controller & DTOs

### 6.1 RobotsController

**Datei:** `src/robots/robots.controller.ts`

Der `RobotsController` bildet die REST-Schnittstelle. Er nimmt HTTP-Anfragen entgegen und leitet diese an den `RobotsService` weiter. Der Controller enthält **keine Logik** – nur Routing und Parameter-Extraktion.

#### Endpunkte

| Methode | Route | Funktion |
|---------|-------|----------|
| `GET` | `/robots` | Alle Roboter abrufen |
| `GET` | `/robots/:id` | Einzelnen Roboter laden |
| `GET` | `/robots/:id/telemetry` | Telemetrie-Historie laden |
| `POST` | `/robots` | Neuen Roboter anlegen |
| `PATCH` | `/robots/:id` | Roboter aktualisieren |
| `PATCH` | `/robots/:id/telemetry` | Telemetrie speichern |
| `DELETE` | `/robots/:id` | Roboter löschen |

#### Beispiel: GET /robots

```typescript
@Get()
findAll() {
  return this.robotsService.findAll();
}
```

#### Beispiel: GET /robots/:id

```typescript
@Get(':id')
findOne(@Param('id') id: string) {
  return this.robotsService.findOne(Number(id));
}
```

> Die ID wird als String empfangen und mit `Number(id)` in eine Zahl umgewandelt, da Prisma eine Zahl erwartet.

### 6.2 RobotsService (Logik)

**Datei:** `src/robots/robots.service.ts`

Das Herzstück des Backends. Übernimmt alle Datenbankoperationen und die Telemetrie-Verarbeitung.

#### Methoden

| Methode | Aufgabe |
|---------|---------|
| `findAll()` | Alle Roboter laden (inkl. Telemetrie) |
| `findOne(id)` | Einzelnen Roboter laden |
| `create(data)` | Neuen Roboter speichern |
| `update(id, data)` | Roboter aktualisieren |
| `updateTelemetry(id, data)` | Telemetrie speichern & broadcasten |
| `getTelemetry(id)` | Historische Telemetrie laden |
| `remove(id)` | Roboter löschen |

#### findAll()

```typescript
async findAll() {
  return this.prisma.robot.findMany({
    include: { telemetry: true }
  });
}
```

`include: { telemetry: true }` lädt automatisch alle Telemetrie-Einträge jedes Roboters mit.

#### updateTelemetry() – Wichtigste Methode

Ablauf:
1. Roboter in `Robot`-Tabelle aktualisieren (aktueller Zustand)
2. Neuen Eintrag in `RobotTelemetry` erstellen (Historie)
3. `TelemetryGateway.sendTelemetry()` aufrufen (Echtzeit-Broadcast)
4. Aktualisierten Roboter zurückgeben

```
PATCH /robots/1/telemetry
    ↓
RobotsController
    ↓
RobotsService.updateTelemetry()
    ↓
1. prisma.robot.update()
    ↓
2. prisma.robotTelemetry.create()
    ↓
3. telemetryGateway.sendTelemetry(data)
    ↓
Socket.IO emit 'robotTelemetry'
```

#### Cascade Delete

Im Prisma-Schema ist `onDelete: Cascade` definiert. Beim Löschen eines Roboters werden alle zugehörigen Telemetrie-Einträge automatisch gelöscht.

### 6.3 DTOs

#### CreateRobotDto

**Datei:** `src/robots/dto/create-robot.dto.ts`

```typescript
export class CreateRobotDto {
  name: string;
  type: string;
  weight?: number;
  speed?: number;
}
```

| Feld | Typ | Pflicht? |
|------|-----|----------|
| `name` | `string` | Ja |
| `type` | `string` | Ja |
| `weight` | `number` | Nein |
| `speed` | `number` | Nein |

#### UpdateRobotDto

**Datei:** `src/robots/dto/update-robot.dto.ts`

```typescript
export class UpdateRobotDto {
  name?: string;
  type?: string;
  weight?: number;
  speed?: number;
  status?: string;
}
```

| Feld | Typ | Pflicht? |
|------|-----|----------|
| `name` | `string` | Nein |
| `type` | `string` | Nein |
| `weight` | `number` | Nein |
| `speed` | `number` | Nein |
| `status` | `string` | Nein |

> **Hinweis:** Ein separates DTO für Telemetrie-Updates existiert aktuell nicht. Telemetriedaten werden direkt als Objekt verarbeitet.

---

## 7. WebSocket Gateway

### 7.1 TelemetryGateway

**Datei:** `src/telemetry/telemetry.gateway.ts`

```typescript
import {
  WebSocketGateway,
  WebSocketServer,
  OnGatewayInit
} from '@nestjs/websockets';
import { Server } from 'socket.io';

@WebSocketGateway({
  cors: { origin: '*' }
})
export class TelemetryGateway implements OnGatewayInit {

  @WebSocketServer()
  server: Server;

  afterInit(server: Server) {
    console.log('🚀 WebSocket Gateway gestartet');
  }

  sendTelemetry(data: any) {
    console.log('📡 Sending telemetry:', data);
    this.server.emit('robotTelemetry', data);
  }
}
```

| Dekorator | Bedeutung |
|-----------|-----------|
| `@WebSocketGateway({cors:{origin:'*'}})` | Markiert Klasse als WebSocket-Gateway mit CORS |
| `@WebSocketServer()` | Injectiert die Socket.IO Server-Instanz |
| `OnGatewayInit` | Lifecycle-Interface – `afterInit` nach Initialisierung |

#### Event

- **Event-Name:** `robotTelemetry`
- **Payload:** Komplettes Datenobjekt
- **Empfänger:** Alle verbundenen Socket.IO-Clients

### 7.2 TelemetryModule

**Datei:** `src/telemetry/telemetry.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { TelemetryGateway } from './telemetry.gateway';

@Module({
  providers: [TelemetryGateway],
  exports: [TelemetryGateway]
})
export class TelemetryModule {}
```

Wird in `RobotsModule` importiert, damit `RobotsService` auf `TelemetryGateway` zugreifen kann.

### 7.3 Vollständiger Telemetrie-Ablauf

```
PATCH /robots/1/telemetry
    ↓
RobotsController
    ↓
RobotsService.updateTelemetry()
    ↓
1. prisma.robot.update()          → Aktueller Zustand
    ↓
2. prisma.robotTelemetry.create() → Historie
    ↓
3. telemetryGateway.sendTelemetry(data)
    ↓
Socket.IO Server.emit('robotTelemetry', data)
    ↓
Alle verbundenen Clients
```

> **Hinweis:** Der WebSocket-Client im React-Frontend ist noch nicht implementiert. `socket.io-client` ist zwar installiert, aber noch nicht genutzt.

---

## 8. Frontend

### 8.1 Übersicht

Das Frontend ist eine React-Anwendung auf Basis von Vite.

| Komponente | Funktion |
|------------|----------|
| React | Benutzeroberfläche |
| TypeScript | Programmiersprache |
| Vite | Entwicklungsserver & Build |
| Axios | HTTP-Kommunikation |

### 8.2 Verzeichnisstruktur

```
frontend/
├── index.html
├── package.json
└── src/
    ├── main.tsx
    ├── App.tsx
    ├── index.css
    └── api/
        └── robots.ts
```

### 8.3 index.html

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>frontend</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

### 8.4 main.tsx

```typescript
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App.tsx'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```

| Komponente | Bedeutung |
|------------|-----------|
| `createRoot` | React 18 Rendering-API |
| `StrictMode` | Aktiviert zusätzliche Prüfungen in der Entwicklung |

### 8.5 API-Verbindung – robots.ts

**Datei:** `src/api/robots.ts`

```typescript
import axios from "axios";

const API = "http://xxx.xxx.xxx.xxx:3000";

export async function getRobots() {
    const url = `${API}/robots`;
    console.log("📡 Anfrage:", url);

    const response = await axios.get(url);
    console.log("📦 Antwort:", response.data);

    return response.data;
}
```

> **Wichtig:** Die API-Adresse darf keinen `/api`-Suffix enthalten, da die Backend-Routen direkt unter `/robots` liegen (nicht `/api/robots`).

### 8.6 App.tsx

**Datei:** `src/App.tsx`

```typescript
import { useEffect, useState } from "react";
import { getRobots } from "./api/robots";

function App() {
  const [robots, setRobots] = useState<any[]>([]);

  useEffect(() => {
    getRobots()
      .then(data => {
        console.log("ROBOTS:", data);
        setRobots(data);
      })
      .catch(err => {
        console.error("FEHLER:", err);
      });
  }, []);

  return (
    <div style={{ padding: "30px" }}>
      <h1>🤖 Robotics Platform</h1>
      <h2>Roboter</h2>

      {robots.length === 0 ?
        <p>Keine Roboter geladen...</p>
        :
        robots.map(robot => (
          <div key={robot.id}
            style={{
              border: "1px solid gray",
              padding: "15px",
              margin: "10px"
            }}>
            <h2>🤖 {robot.name}</h2>
            <p>Typ: {robot.type}</p>
            <p>Status: {robot.status}</p>
            <p>
              Position:<br />
              X {robot.positionX}<br />
              Y {robot.positionY}<br />
              Z {robot.positionZ}
            </p>
            <p>Speed: {robot.speed}</p>
            <p>🔋 {robot.battery}%</p>
          </div>
        ))
      }
    </div>
  );
}

export default App;
```

#### State-Verwaltung

| Hook | Zweck |
|------|-------|
| `useState<any[]>([])` | Speichert die geladenen Roboter |
| `useEffect(...)` | Lädt Roboter beim ersten Render |

#### Datenladung

```
React startet
    ↓
useEffect ausführen
    ↓
getRobots()
    ↓
axios GET
    ↓
Backend antwortet
    ↓
setRobots(data)
    ↓
React rendert Karten
```

#### Fehlerbehandlung

Falls das Backend nicht erreichbar ist:
- **Konsole:** `AxiosError` mit Details
- **UI:** `"Keine Roboter geladen..."`

### 8.7 index.css – Design-System

**Datei:** `src/index.css`

Das Stylesheet definiert ein komplettes Design-System mit CSS-Variablen, Light/Dark Mode und responsivem Layout.

#### CSS-Variablen (Light Mode)

| Variable | Wert | Beschreibung |
|----------|------|--------------|
| `--text` | `#6b6375` | Haupttextfarbe |
| `--text-h` | `#08060d` | Überschriftenfarbe |
| `--bg` | `#fff` | Hintergrund |
| `--border` | `#e5e4e7` | Rahmenfarbe |
| `--accent` | `#aa3bff` | Akzentfarbe (Lila) |
| `--shadow` | `rgba(...)` | Schatten |
| `--sans` | `system-ui, ...` | Fließtext-Schrift |
| `--mono` | `ui-monospace, ...` | Code-Schrift |

#### Dark Mode

```css
@media (prefers-color-scheme: dark) {
  :root {
    --text: #9ca3af;
    --text-h: #f3f4f6;
    --bg: #16171d;
    --border: #2e303a;
    --accent: #c084fc;
  }
}
```

Aktivierung basiert auf den **Systemeinstellungen** des Benutzers.

#### Layout

```css
#root {
  width: 1126px;
  max-width: 100%;
  margin: 0 auto;
  text-align: center;
  border-inline: 1px solid var(--border);
  min-height: 100svh;
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
}
```

#### Typografie

| Element | Größe (Desktop) | Größe (Mobile <1024px) |
|---------|-------------------|------------------------|
| `h1` | 56px | 36px |
| `h2` | 24px | 20px |
| `body` | 18px | 16px |
| `code` | 15px | – |

### 8.8 Frontend-Start

```bash
cd ~/projects/robotics-platform/frontend
npm run dev -- --host 0.0.0.0
```

| URL | Beschreibung |
|-----|--------------|
| `http://localhost:5173` | Lokal |
| `http://xxx.xxx.xxx.xxx:5173` | Netzwerk |

> `--host 0.0.0.0` ist notwendig, damit andere Geräte im Netzwerk zugreifen können.

### 8.9 Aktueller Stand Frontend

| Funktion | Status |
|----------|--------|
| React 18 mit Vite | ✅ |
| Axios-API-Verbindung | ✅ |
| Roboter-Anzeige mit Karten | ✅ |
| Light/Dark Mode | ✅ |
| Responsive Design | ✅ |
| WebSocket-Client | ❌ |
| Roboter-Auswahl/Detailansicht | ❌ |
| Live-Update der Telemetrie | ❌ |

---

## 9. Tests & Fehleranalyse

### 9.1 Teststrategie

Vor jeder Änderung wurde geprüft, ob das System grundsätzlich funktioniert.

#### Testreihenfolge

```
Backend gestartet?
    ↓
API erreichbar?
    ↓
Datenbank erreichbar?
    ↓
Roboter vorhanden?
    ↓
Telemetrie speichern?
    ↓
Frontend erreichbar?
    ↓
Frontend lädt Daten?
    ↓
WebSocket vorbereitet?
```

### 9.2 Backend testen

```bash
cd ~/projects/robotics-platform/backend/api
npm run start:dev
```

**Erwartete Ausgabe:**
```
[Nest] Application started
🚀 WebSocket Gateway gestartet
```

✅ **Backend erfolgreich gestartet**

### 9.3 API testen

```bash
curl http://localhost:3000/robots
```

**Erwartete Antwort:**
```json
[
  {
    "id": 1,
    "name": "Atlas"
  }
]
```

✅ **API antwortet korrekt**

### 9.4 Telemetrie testen

```bash
curl -X PATCH http://localhost:3000/robots/1/telemetry \
  -H "Content-Type: application/json" \
  -d '{
    "positionX": 100,
    "positionY": 50,
    "positionZ": 20,
    "speed": 20,
    "battery": 55,
    "status": "autonomous"
  }'
```

**Erfolgreiche Antwort:**
```json
{
  "id": 1,
  "name": "Atlas",
  "status": "autonomous"
}
```

✅ **Daten erfolgreich gespeichert**

### 9.5 Datenbank prüfen

```bash
curl http://localhost:3000/robots
```

Atlas enthält:
```json
{
  "name": "Atlas",
  "battery": 55,
  "status": "autonomous",
  "telemetry": [ ... ]
}
```

Damit bestätigt:
- ✅ Robot aktualisiert
- ✅ Telemetrie gespeichert
- ✅ Datenbank funktioniert

### 9.6 Frontend-Test

```bash
cd ~/projects/robotics-platform/frontend
npm run dev -- --host 0.0.0.0
```

**Anzeige:**
```
Local:    http://localhost:5173
Network:  http://xxx.xxx.xxx.xxx:5173
```

Frontend zeigt:
```
🤖 Robotics Platform

Roboter

Atlas
Rover-X1
Drone-01
```

Nach Auswahl:
```
Status:     autonomous
Position:   100 / 50 / 20
Battery:    55%
```

✅ **Datenübertragung erfolgreich**

### 9.7 Bekannte Fehler & Lösungen

#### Fehler 1: Falscher API-Pfad

**Symptom:**
```
🤖 Robotics Platform
Roboter
Keine Roboter geladen
```

**Browser-Network:**
```
GET http://xxx.xxx.xxx.xxx:3000/api/robots
404 Not Found
```

**Ursache:**
```typescript
// FALSCH:
const API = "http://xxx.xxx.xxx.xxx:3000/api";

// Backend-Log:
RoutesResolver RobotsController {/robots}
```

Die Backend-Route ist `/robots`, nicht `/api/robots`.

**Lösung:**
```typescript
// RICHTIG:
const API = "http://xxx.xxx.xxx.xxx:3000";
```

#### Fehler 2: Falscher WebSocket-Test im Terminal

**Eingabe:**
```bash
ws://localhost:3000
```

**Ergebnis:**
```
-bash: ws://localhost:3000: No such file or directory
```

**Ursache:** WebSocket-URLs sind keine Linux-Befehle.

**Richtig:**
- Socket.IO Client
- Browser / JavaScript
- WebSocket-Testtool

#### Fehler 3: JSON im Terminal

**Eingabe:**
```bash
{
  "id": 1
}
```

**Ergebnis:**
```
id:1 command not found
```

**Ursache:** Die Bash interpretiert jede Zeile als eigenen Befehl.

**Richtig:**
```bash
echo '{"id": 1}'
# oder
curl -d '{"id": 1}' ...
```

#### Fehler 4: Axios-Fehler (404)

**Browser-Konsole:**
```
AxiosError: Request failed with status code 404
```

**Lösung:**
1. Network-Tab öffnen
2. Request URL prüfen
3. Mit Backend-Logs (`docker logs xxxxxxxx-backend | grep Routes`) vergleichen
4. API-Adresse im Frontend korrigieren

### 9.8 Docker-Logs

```bash
docker logs xxxxxxxx-backend
docker logs xxxxxxxx-backend | grep Robots
```

**Erwartete Ausgabe:**
```
RobotsModule dependencies initialized
RoutesResolver RobotsController {/robots}
```

Damit prüfbar:
- ✅ Backend gestartet
- ✅ Controller registriert
- ✅ Route korrekt geladen

### 9.9 Typische Debug-Reihenfolge

Bei zukünftigen Problemen:

1. **Backend läuft?**
2. **Docker-Container aktiv?**
3. **Route existiert?** (`docker logs ... | grep Routes`)
4. **curl-Test erfolgreich?**
5. **Browser-Netzwerk prüfen**
6. **Browser-Konsole prüfen**
7. **Backend-Logs prüfen**
8. **Datenbankinhalt kontrollieren**

### 9.10 Aktueller Teststatus

| Test | Ergebnis |
|------|----------|
| Backend startet | ✅ |
| Prisma verbunden | ✅ |
| Datenbank erreichbar | ✅ |
| Roboter abrufbar | ✅ |
| PATCH Telemetrie | ✅ |
| Telemetrie gespeichert | ✅ |
| Frontend erreichbar | ✅ |
| Roboterauswahl | ✅ |
| Roboterdetails | ✅ |
| WebSocket Gateway startet | ✅ |
| React empfängt WebSocket | ❌ Noch nicht umgesetzt |

### 9.11 Zusammenfassung

Alle Kernkomponenten des Prototyps wurden erfolgreich getestet. Die meisten Probleme entstanden durch:
- **Konfigurationsfehler** (falsche API-Pfade)
- **Missverständnisse beim Testen** (WebSocket im Terminal)
- **Typische Shell-Eingabefehler** (JSON ohne `echo`/`curl`)

Diese ließen sich systematisch und nachvollziehbar beheben.

---

## 10. Dependencies & Umgebung

### 10.1 AppController

**Datei:** `src/app.controller.ts`

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('api')
export class AppController {

  @Get('status')
  getStatus() {
    return {
      system: 'Robotics Platform',
      status: 'online',
      version: '0.1.0-alpha',
      server: 'xxxxxxxxx'
    };
  }
}
```

| Route | Antwort |
|-------|---------|
| `GET /api/status` | Systemstatus |

**Beispiel-Antwort:**
```json
{
  "system": "Robotics Platform",
  "status": "online",
  "version": "0.1.0-alpha",
  "server": "xxxxxxxxx"
}
```

### 10.2 AppService

**Datei:** `src/app.service.ts`

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

> Der Service ist im `AppModule` registriert, wird aber aktuell von keinem Controller direkt genutzt.

### 10.3 HealthController

**Datei:** `src/health/health.controller.ts`

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('api/health')
export class HealthController {

  @Get()
  getHealth() {
    return {
      system: 'Robotics Platform',
      status: 'online',
      version: '0.1.0-alpha',
      server: 'xxxxxxxxx',
      services: {
        backend: 'online',
        database: 'unknown',
        redis: 'unknown'
      },
      timestamp: new Date().toISOString()
    };
  }
}
```

| Route | Antwort |
|-------|---------|
| `GET /api/health` | Gesundheits-Check |

**Beispiel-Antwort:**
```json
{
  "system": "Robotics Platform",
  "status": "online",
  "version": "0.1.0-alpha",
  "server": "xxxxxxxxx",
  "services": {
    "backend": "online",
    "database": "unknown",
    "redis": "unknown"
  },
  "timestamp": "2026-08-01T08:51:00.000Z"
}
```

> Die Felder `database` und `redis` sind Platzhalter für zukünftige Health-Checks.

### 10.4 Backend package.json

**Datei:** `backend/api/package.json`

#### Grunddaten

| Feld | Wert |
|------|------|
| `name` | `api` |
| `version` | `0.0.1` |
| `private` | `true` |

#### Scripts

| Script | Befehl | Zweck |
|--------|--------|-------|
| `build` | `nest build` | Kompiliert TypeScript |
| `format` | `prettier --write ...` | Formatiert Quellcode |
| `start` | `nest start` | Produktivstart |
| `start:dev` | `nest start --watch` | Entwicklung mit Hot-Reload |
| `start:debug` | `nest start --debug --watch` | Debug-Modus |
| `start:prod` | `node dist/main` | Produktivstart aus `dist/` |
| `lint` | `eslint ... --fix` | Linting mit Auto-Fix |
| `test` | `jest` | Unit-Tests |
| `test:watch` | `jest --watch` | Tests im Watch-Modus |
| `test:cov` | `jest --coverage` | Tests mit Coverage |
| `test:e2e` | `jest --config ./test/jest-e2e.json` | End-to-End-Tests |

#### Dependencies

| Paket | Version | Zweck |
|-------|---------|-------|
| `@nestjs/common` | `^11.0.1` | NestJS Kern-Module |
| `@nestjs/config` | `^4.0.4` | Konfigurationsmanagement |
| `@nestjs/core` | `^11.0.1` | NestJS Kern |
| `@nestjs/platform-express` | `^11.1.28` | HTTP-Server (Express) |
| `@nestjs/platform-socket.io` | `^11.1.28` | WebSocket-Server |
| `@nestjs/websockets` | `^11.1.28` | WebSocket-Dekoratoren |
| `@prisma/adapter-pg` | `^7.9.1` | Prisma PostgreSQL-Adapter |
| `@prisma/client` | `^7.9.1` | Prisma Client |
| `dotenv` | `^17.4.2` | `.env`-Dateien laden |
| `ioredis` | `^6.0.0` | Redis-Client |
| `pg` | `^8.22.0` | PostgreSQL-Treiber |
| `prisma` | `^7.9.1` | Prisma CLI |
| `redis` | `^6.1.0` | Redis-Client (alt.) |
| `reflect-metadata` | `^0.2.2` | TypeScript-Metadaten |
| `rxjs` | `^7.8.1` | Reaktive Programmierung |
| `socket.io` | `^4.8.3` | WebSocket-Bibliothek |

#### DevDependencies

| Paket | Version | Zweck |
|-------|---------|-------|
| `@nestjs/cli` | `^11.0.0` | NestJS CLI |
| `@nestjs/testing` | `^11.0.1` | Test-Utilities |
| `eslint` | `^9.18.0` | Linter |
| `jest` | `^30.0.0` | Test-Framework |
| `prettier` | `^3.4.2` | Code-Formatierer |
| `ts-jest` | `^29.2.5` | Jest für TypeScript |
| `ts-node` | `^10.9.2` | TypeScript direkt ausführen |
| `typescript` | `^5.7.3` | TypeScript-Compiler |
| `typescript-eslint` | `^8.20.0` | ESLint für TypeScript |

### 10.5 Frontend package.json

**Datei:** `frontend/package.json`

#### Grunddaten

| Feld | Wert |
|------|------|
| `name` | `frontend` |
| `version` | `0.0.0` |
| `type` | `module` |

#### Scripts

| Script | Befehl | Zweck |
|--------|--------|-------|
| `dev` | `vite` | Entwicklungsserver |
| `build` | `tsc -b && vite build` | Kompilieren und bauen |
| `lint` | `eslint .` | Linting |
| `preview` | `vite preview` | Produktions-Preview |

#### Dependencies

| Paket | Version | Zweck |
|-------|---------|-------|
| `axios` | `^1.19.0` | HTTP-Client |
| `react` | `^19.2.8` | React-Bibliothek |
| `react-dom` | `^19.2.8` | React DOM-Renderer |
| `socket.io-client` | `^4.8.3` | WebSocket-Client |

#### DevDependencies

| Paket | Version | Zweck |
|-------|---------|-------|
| `@vitejs/plugin-react` | `^6.0.4` | Vite-Plugin für React |
| `eslint` | `^10.8.0` | Linter |
| `typescript` | `~6.0.2` | TypeScript-Compiler |
| `typescript-eslint` | `^8.65.0` | ESLint für TypeScript |
| `vite` | `^8.2.0` | Build-Tool und Dev-Server |

### 10.6 Wichtige Versionshinweise

| Komponente | Version | Bemerkung |
|------------|---------|-----------|
| NestJS | 11.x | Aktuelle Major-Version |
| React | 19.2.8 | React 19 (neueste) |
| Vite | 8.2.0 | Entwicklungsserver |
| Prisma | 7.9.1 | ORM mit Custom Generator |
| Socket.IO | 4.8.3 | Backend und Frontend |
| PostgreSQL-Adapter | 8.22.0 | `pg`-Treiber für PrismaPg |

### 10.7 Umgebungsvariablen

**Datei:** `backend/api/.env`

```
DATABASE_URL="postgresql://xxxxxxxx:XXXXXXXX@xxxxxxx:5432/xxxxxxxx_platform"
```

#### Erklärung

| Komponente | Wert | Beschreibung |
|------------|------|--------------|
| Protokoll | `postgresql` | PostgreSQL-Verbindung |
| Benutzer | `xxxxxxxx` | Datenbankbenutzer |
| Passwort | `XXXXXXXX` | Datenbankpasswort |
| Host | `xxxxxxx` | Hostname (Docker-Container oder Server) |
| Port | `5432` | Standard-PostgreSQL-Port |
| Datenbank | `xxxxxxxx_platform` | Name der Datenbank |

> **Hinweis:** Der Host lautet nicht `localhost`, sondern ein separater Container-Name. Das bedeutet, PostgreSQL läuft als separater Container und kommuniziert über Docker-Netzwerk oder interne DNS.

#### Ablauf der Verbindung

```
.env laden
    ↓
process.env.DATABASE_URL
    ↓
PrismaService Konstruktor
    ↓
PrismaPg Adapter
    ↓
PostgreSQL
```

---

## 11. Gesamtübersicht & Index

### 11.1 Vollständige Projektstruktur

```
robotics-platform/
├── backend/
│   └── api/
│       ├── .env
│       ├── package.json
│       ├── prisma/
│       │   └── schema.prisma
│       └── src/
│           ├── main.ts
│           ├── app.module.ts
│           ├── app.controller.ts
│           ├── app.service.ts
│           ├── prisma/
│           │   ├── prisma.module.ts
│           │   └── prisma.service.ts
│           ├── robots/
│           │   ├── dto/
│           │   │   ├── create-robot.dto.ts
│           │   │   └── update-robot.dto.ts
│           │   ├── robots.controller.ts
│           │   ├── robots.module.ts
│           │   └── robots.service.ts
│           ├── telemetry/
│           │   ├── telemetry.gateway.ts
│           │   └── telemetry.module.ts
│           └── health/
│               └── health.controller.ts
│
└── frontend/
    ├── package.json
    ├── index.html
    └── src/
        ├── main.tsx
        ├── App.tsx
        ├── index.css
        └── api/
            └── robots.ts
```

### 11.2 API-Endpunkt-Referenz

| Methode | Route | Funktion | Controller |
|---------|-------|----------|------------|
| `GET` | `/api/status` | Systemstatus | `AppController` |
| `GET` | `/api/health` | Gesundheits-Check | `HealthController` |
| `GET` | `/robots` | Alle Roboter laden | `RobotsController` |
| `GET` | `/robots/:id` | Einzelnen Roboter laden | `RobotsController` |
| `GET` | `/robots/:id/telemetry` | Telemetrie-Historie laden | `RobotsController` |
| `POST` | `/robots` | Neuen Roboter anlegen | `RobotsController` |
| `PATCH` | `/robots/:id` | Roboter aktualisieren | `RobotsController` |
| `PATCH` | `/robots/:id/telemetry` | Telemetrie speichern | `RobotsController` |
| `DELETE` | `/robots/:id` | Roboter löschen | `RobotsController` |

### 11.3 Modul-Übersicht

| Modul | Aufgabe | Exporte |
|-------|---------|---------|
| `PrismaModule` | Globale Datenbankverbindung | `PrismaService` |
| `RobotsModule` | REST-API für Roboter und Telemetrie | `RobotsController`, `RobotsService` |
| `TelemetryModule` | WebSocket-Gateway | `TelemetryGateway` |
| `AppModule` | Zentrale Sammelstelle aller Module | – |

### 11.4 Service-Übersicht

| Service | Aufgabe | Ort |
|---------|---------|-----|
| `PrismaService` | Datenbankverbindung via PrismaPg-Adapter | `prisma/prisma.service.ts` |
| `RobotsService` | Logik: CRUD, Telemetrie, WebSocket-Auslösung | `robots/robots.service.ts` |
| `AppService` | Standard-Service (`Hello World`) | `app.service.ts` |
| `TelemetryGateway` | WebSocket-Broadcast von Telemetrie | `telemetry/telemetry.gateway.ts` |

### 11.5 Controller-Übersicht

| Controller | Aufgabe | Ort |
|------------|---------|-----|
| `AppController` | Systemstatus-Endpunkt | `app.controller.ts` |
| `HealthController` | Gesundheits-Endpunkt | `health/health.controller.ts` |
| `RobotsController` | REST-API für Roboter und Telemetrie | `robots/robots.controller.ts` |

### 11.6 DTO-Übersicht

| DTO | Zweck | Pflichtfelder | Ort |
|-----|-------|---------------|-----|
| `CreateRobotDto` | Neuanlage | `name`, `type` | `robots/dto/create-robot.dto.ts` |
| `UpdateRobotDto` | Teilaktualisierung | Keine (alle optional) | `robots/dto/update-robot.dto.ts` |

> **Hinweis:** Ein separates Telemetrie-DTO existiert nicht. Telemetriedaten werden direkt als Objekt verarbeitet.

### 11.7 Technologie-Stack

| Schicht | Technologie | Version |
|---------|-------------|---------|
| Betriebssystem | Linux | – |
| Datenbank | PostgreSQL | – |
| Backend-Framework | NestJS | 11.x |
| Backend-Sprache | TypeScript | 5.7.3 |
| ORM | Prisma | 7.9.1 |
| PostgreSQL-Adapter | `pg` (PrismaPg) | 8.22.0 |
| WebSocket-Server | Socket.IO | 4.8.3 |
| Frontend-Framework | React | 19.2.8 |
| Frontend-Build | Vite | 8.2.0 |
| Frontend-Sprache | TypeScript | ~6.0.2 |
| WebSocket-Client | Socket.IO Client | 4.8.3 (installiert, nicht genutzt) |

### 11.8 Funktionsumfang

| Funktion | Status |
|----------|--------|
| Roboterverwaltung | ✅ |
| Telemetrie-Speicherung | ✅ |
| REST-API | ✅ |
| React-Dashboard | ✅ |
| Datenbank (PostgreSQL + Prisma) | ✅ |
| WebSocket-Gateway (Backend) | ✅ |
| CORS / Netzwerkfreigabe | ✅ |
| Health-Check | ✅ |
| Systemstatus-Endpunkt | ✅ |
| Automatisches Live-Update | ❌ |
| WebSocket-Client (Frontend) | ❌ |
| 3D-Darstellung | ❌ |
| Steuerbefehle | ❌ |
| Login / Benutzerverwaltung | ❌ |
| Redis-Anbindung | ❌ |

---

## Zusammenfassung

Die **Robotics Platform v0.2-alpha** ist ein funktionierender Prototyp mit folgenden Kernkomponenten:

- NestJS-Backend mit modularer Architektur
- PostgreSQL-Datenbank mit Prisma ORM
- React-Frontend mit Vite
- REST-API für Roboter und Telemetrie
- Vorbereiteter WebSocket-Server für Echtzeitkommunikation
- CORS- und Netzwerkfreigabe für LAN-Zugriff

Alle sensiblen Daten (IP-Adressen, Benutzernamen, Passwörter, Hostnamen, DB-Credentials) wurden in dieser Dokumentation zensiert.
