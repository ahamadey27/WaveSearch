# Project: AudioSearch

**Goal:** To develop a minimalist, search-driven audio library web application. The application will allow an administrator to upload audio files with associated metadata via a secure, unlisted page. Public users can search for these audio files and play them back with a waveform visualization, with a strict rule that only one audio file can be playing at any given time.

---

# Components

## Environment/Hosting
- **Local Development:** Node.js for the backend, Create React App for the frontend.
- **Version Control:** Git
- **Cloud Hosting:** Vercel (hosting frontend, serverless functions, database, and file storage).

## Software Components

### Web Application Backend & Frontend
- **Backend Framework:** Node.js with Express
- **Frontend Framework:** React
- **Language:** JavaScript
- **Key Libraries:**
    - `multer`: Middleware for handling file uploads.
    - `pg`: Node.js client for PostgreSQL.
    - `@vercel/blob`: SDK for interacting with Vercel Blob storage.
    - `react-router-dom`: For frontend routing, including the secret admin route.
    - `react-audio-wave-modern`: For displaying audio waveforms.

### Core Logic Services & Endpoints
- **Upload Service (`/api/upload`):** Handles file uploads and metadata persistence.
- **Search Service (`/api/search`):** Handles public search queries.
- **Playback Context (`PlaybackContext.js`):** Manages global state for single-play audio functionality.

### Platform Services
- **Database:** Vercel Postgres
- **File Storage:** Vercel Blob

---

# Core Services and Data Structures

## Upload Service (`/api/upload`)
- **Responsibilities:**
    - Accepts `multipart/form-data` requests containing an audio file and text metadata (description, keywords).
    - Uses `multer` middleware to process the incoming file.
    - Uploads the audio file to Vercel Blob storage.
    - Inserts a new record into the `audio_files` table in Vercel Postgres, storing the public file URL from Blob storage and the associated metadata.
    - Provides graceful error handling for failed uploads or database insertions.
- **Key Methods (Conceptual):**
    - `POST /api/upload`: The Express route handler that orchestrates the upload and database insertion logic.
- **Implied Data Models:**
    - `AudioFile` (SQL Table):
        - `id`: SERIAL PRIMARY KEY
        - `file_url`: TEXT NOT NULL
        - `description`: TEXT
        - `keywords`: TEXT
        - `created_at`: TIMESTAMP WITH TIME ZONE

## Search Service (`/api/search`)
- **Responsibilities:**
    - Accepts a GET request with a query parameter `q`.
    - Constructs and executes a case-insensitive SQL query (`ILIKE`) against the `description` and `keywords` columns of the `audio_files` table.
    - Uses parameterized queries to prevent SQL injection attacks.
    - Returns a JSON array of matching `AudioFile` records, ordered by creation date.
- **Key Methods (Conceptual):**
    - `GET /api/search?q=<searchTerm>`: The Express route handler for executing searches.
- **Implied Data Models:**
    - `AudioFile` (JSON Object): A representation of a row from the `audio_files` table.

## Playback Context (`PlaybackContext.js`)
- **Responsibilities:**
    - Creates a React Context to be shared across the component tree.
    - Manages global state to track the ID of the currently playing audio track.
    - Provides a mechanism for a component to signal that it is about to play, which first pauses any other currently playing track.
    - Enforces the "single-play" requirement across the entire application.
- **Implied Data Models:**
    - `PlaybackContextValue`: An object containing `currentlyPlayingId` and functions to update the context state.

## Configuration (Vercel Environment Variables)
- **`POSTGRES_URL`:** Connection string for the Vercel Postgres database.
- **`BLOB_READ_WRITE_TOKEN`:** Access token for the Vercel Blob storage.
- **`REACT_APP_ADMIN_URL`:** The secret, unguessable path for the admin upload page (e.g., `/upload-admin-a1b2c3d4`).

---

# Development Plan

## Phase 1: Foundational Backend and Admin Interface
+ [x] **Step 1.1: Project Initialization:** Set up a monorepo structure with a Node.js backend and a `create-react-app` frontend.
+ [x] **Step 1.2: Backend Server & Vercel Config:** Initialize a basic Express server and create a `vercel.json` file to configure serverless function routing.
+ [ ] **Step 1.3: Database Provisioning & Schema:** Provision a Vercel Postgres database and define the `audio_files` table schema in an SQL file.
+ [ ] **Step 1.4: File Storage Provisioning:** Enable Vercel Blob for the project.
+ [ ] **Step 1.5: Implement Upload API Endpoint:** Create the `POST /api/upload` endpoint, integrating `multer`, the Vercel Blob SDK, and the Postgres client to handle file uploads and metadata insertion.
+ [ ] **Step 1.6: Create Secure Admin Page:** In React, create an `AdminUpload` component and use `react-router-dom` to place it on a secret, unguessable route defined by an environment variable.
+ [ ] **Step 1.7: Build Admin Upload Form:** Implement the frontend form with file, description, and keyword inputs, along with the client-side logic to send the `FormData` to the API.

## Phase 2: Public Search and Results Display
- [ ] **Step 2.1: Create Public UI Components:** Build the main `SearchPage` component and a reusable `SearchBar` component.
- [ ] **Step 2.2: Implement Search API Endpoint:** Create the `GET /api/search` endpoint with logic to query the database based on a search term.
- [ ] **Step 2.3: Frontend State Management:** Use React `useState` hooks in `SearchPage` to manage the search query, results, loading status, and error states.
- [ ] **Step 2.4: Build Results Display:** Create `ResultsList` and `ResultItem` components to map over the search results and display them.
- [ ] **Step 2.5: Implement Conditional Rendering:** Use the component's state to conditionally show loading indicators, error messages, a "No results found" message, or the results list.

## Phase 3: Interactive Audio Playback
- [ ] **Step 3.1: Integrate Waveform Component:** Add `react-audio-wave-modern` to the `ResultItem` component and configure its appearance.
- [ ] **Step 3.2: Create Global Playback Context:** Define and provide a `PlaybackContext` to the application to manage global playback state.
- [ ] **Step 3.3: Add Play/Pause Controls:** Add a button to each `ResultItem` that visually changes based on whether that item is the currently playing track (tracked via context).
- [ ] **Step 3.4: Implement Single-Play Logic:** In the button's `onClick` handler, implement the logic to pause the currently playing track (if any) before playing the new one, and update the global context accordingly.

## Phase 4: Deployment and Finalization
- [ ] **Step 4.1: Vercel Project Configuration:** Connect the Git repository to a Vercel project and verify the build settings for the frontend and backend.
- [ ] **Step 4.2: Configure Environment Variables:** Add `POSTGRES_URL`, `BLOB_READ_WRITE_TOKEN`, and `REACT_APP_ADMIN_URL` to the Vercel project settings.
- [ ] **Step 4.3: End-to-End Admin Test:** Deploy the application and perform a test by navigating to the secret admin URL and uploading a new audio file. Verify its presence in the database and blob storage.
- [ ] **Step 4.4: End-to-End Public Test:** Access the public URL and search for the newly uploaded file. Verify that it appears in the results, the waveform renders, and the single-play audio functionality works correctly.
- [ ] **Step 4.5: Final Review & Documentation:** Conduct a final check for bugs or visual issues. Create a comprehensive `README.md` file detailing the project setup, purpose, and environment variables.