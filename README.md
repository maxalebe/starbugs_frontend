My Star Project (Maxime Beck)

Dieses Projekt visualisiert Sterne in einem 3D-Raum und bietet eine API, um Sternendaten abzurufen und Wikipedia-Zusammenfassungen zu Sternen anzuzeigen. Es besteht aus einem Backend, das mit Node.js und Express erstellt wurde, und einem Frontend, das mit Svelte und Three.js entwickelt wurde.

Inhaltsverzeichnis

- [Einführung](#einführung)
- [Installation](#installation)
  - [Backend](#backend)
  - [Frontend](#frontend)
- [Verwendung](#verwendung)
- [API-Endpunkte](#api-endpunkte)
  - [GET /star-data](#get-star-data)
  - [GET /wiki-summary](#get-wiki-summary)
  - [POST /api/wiki-summary](#post-apiwiki-summary)
- [Docker](#docker)
- [Technologien](#technologien)

Einführung

Dieses Projekt visualisiert Sterne in einem 3D-Raum und bietet Benutzern die Möglichkeit, Informationen über die Sterne abzurufen. Das Backend stellt eine API bereit, die Sternendaten aus einer MongoDB-Datenbank abruft und Wikipedia-Zusammenfassungen zu den Sternen liefert. Das Frontend visualisiert die Sterne mit Three.js und ermöglicht es dem Benutzer, mit ihnen zu interagieren.

Installation

Backend

1. Repository klonen:

   ```bash
   git clone https://github.com/dein-benutzername/my-star-project.git
   cd my-star-project/backend
   ```

2. Umgebungsvariablen konfigurieren:**

   Erstelle eine `.env`-Datei im `backend`-Verzeichnis und füge die folgenden Variablen hinzu:

   ```plaintext
   PORT=3000
   MONGO_URI=your_mongo_uri
   MONGO_COLLECTION=your_collection_name
   ```

3. Abhängigkeiten installieren:

   ```bash
   npm install
   ```

4. Backend starten:

   ```bash
   npm start
   ```

Frontend

1. Zum Frontend-Verzeichnis wechseln:

   ```bash
   cd ../frontend
   ```

2. Abhängigkeiten installieren:**

   ```bash
   npm install
   ```

3. Frontend starten:**

   ```bash
   npm run dev
   ```

Verwendung

Öffne deinen Browser und navigiere zu `http://localhost:5000`, um das Frontend der Anwendung anzuzeigen. Du kannst mit den Sternen im 3D-Raum interagieren und auf Informationen über sie zugreifen.

API-Endpunkte

GET /star-data

Ruft eine Liste von Sternen ab, deren Helligkeit (`mag`) kleiner als 7 ist.

- URL: `/star-data`
- Methode: `GET`
- Antwort: JSON-Array von Sternen

GET /wiki-summary

Ruft eine Wikipedia-Zusammenfassung basierend auf einer gegebenen Wikipedia-URL ab.

- URL: `/wiki-summary`
- Methode: `GET`
- Query-Parameter: `url` - Die Wikipedia-URL des Artikels
- Antwort: JSON-Objekt mit der Zusammenfassung

### POST /api/wiki-summary

Ruft eine Wikipedia-Zusammenfassung basierend auf einer gegebenen Wikipedia-URL ab (über POST-Request).

- URL: `/api/wiki-summary`
- Methode: `POST`
- Body: JSON-Objekt mit der `wikiUrl`
- Antwort: JSON-Objekt mit der Zusammenfassung

Docker

Das Backend kann auch in einem Docker-Container ausgeführt werden. Führe die folgenden Befehle im `backend`-Verzeichnis aus:

1. Docker-Image bauen:

   ```bash
   docker build -t my-star-backend .
   ```

2. Docker-Container starten:

   ```bash
   docker run -p 3000:3000 my-star-backend
   ```

Technologien

Backend:
  - Node.js
  - Express
  - Mongoose
  - Axios

Frontend:
  - Svelte
  - Three.js
  - Axios



Stelle sicher, dass du deine MongoDB-URI und andere vertrauliche Daten sicher behandelst und nicht in öffentlichen Repositories teilst. Füge sie zu deiner `.gitignore`-Datei hinzu, falls erforderlich.
