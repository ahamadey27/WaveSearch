oftware Development Plan and Functional Specification: Project "AudioSearch"
This document provides a comprehensive development roadmap and functional specification for the "AudioSearch" web application. It is designed to guide the implementation of a minimalist, search-driven audio library, adhering to the specified technical stack and project constraints.

1. Detailed Implementation Plan
This section provides a granular, step-by-step guide for development, including technical explanations and illustrative code snippets.

Phase 1: Foundational Backend and Admin Interface
Objective: Establish the data persistence layer and a secure method for the administrator to upload content.

1.1.1: Project Initialization: Create a new project directory. Inside, initialize a Node.js project (npm init -y) and a React application (npx create-react-app client). This structure separates the backend and frontend concerns.

1.1.2: Backend Server Setup: In the root of the Node.js project, install Express (npm install express). Create a server file (e.g., index.js) to initialize a basic server. This will be the foundation for all API endpoints.

JavaScript

// index.js
const express = require('express');
const app = express();
const PORT = process.env.PORT |

| 3001;

app.use(express.json()); // Middleware to parse JSON bodies

app.get('/api/health', (req, res) => {
  res.status(200).send({ status: 'UP' });
});

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```
1.1.3: Vercel Serverless Configuration: Create a vercel.json file in the project root to configure build outputs and routing rules. This tells Vercel how to handle the React frontend and the Node.js API endpoints as serverless functions.    

1.2.1: Database Provisioning: Use the Vercel dashboard to create a new Postgres database. Vercel will provide a connection string, which you will add to your project's environment variables.    

1.2.2: Database Schema Definition: Create an SQL file (e.g., schema.sql) to define the structure of your audio_files table. This ensures consistency and provides a source of truth for your data model.

SQL

-- schema.sql
CREATE TABLE audio_files (
  id SERIAL PRIMARY KEY,
  file_url TEXT NOT NULL,
  description TEXT,
  keywords TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
1.2.3: Database Initialization Script: Use a Node.js SQL client library like pg (npm install pg) to connect to the database and run the schema.sql script. This is a one-time setup step.

1.3.1: File Storage Provisioning: In the Vercel dashboard, enable Vercel Blob for your project. This will provide the necessary credentials and SDK for file uploads.    

1.3.2: File Upload Middleware Integration: Install multer, a middleware for handling multipart/form-data, which is used for file uploads.   

JavaScript

// In your Express server file (e.g., index.js)
const multer = require('multer');
const upload = multer({ dest: '/tmp/uploads/' }); // Temporary local storage before moving to Blob
1.3.3: Server-Side Upload Logic: In your upload endpoint, use the Vercel Blob SDK (@vercel/blob) to take the file from the temporary storage location provided by multer and upload it to the Blob store.

1.4.1: API Endpoint for Upload: Create a POST endpoint that uses the multer middleware to process the incoming form data.

JavaScript

// In index.js, using the Vercel Blob and Postgres clients
const { put } = require('@vercel/blob');
const { pool } = require('./db-client'); // Your configured Postgres client

app.post('/api/upload', upload.single('audioFile'), async (req, res) => {
  const { file } = req;
  const { description, keywords } = req.body;

  if (!file) {
    return res.status(400).send('No file uploaded.');
  }

  try {
    // Upload file to Vercel Blob
    const blob = await put(file.originalname, file.path, { access: 'public' });

    // Insert metadata and blob URL into Postgres
    await pool.query(
      'INSERT INTO audio_files (file_url, description, keywords) VALUES ($1, $2, $3)',
      [blob.url, description, keywords]
    );

    res.status(200).json({ message: 'File uploaded successfully', url: blob.url });
  } catch (error) {
    console.error('Upload failed:', error);
    res.status(500).send('Upload failed.');
  }
});
1.4.2: Endpoint Logic: The endpoint orchestrates the process: it receives the file and metadata, uploads the file to Vercel Blob, gets the public URL back, and then writes a new record to the Postgres database with that URL and the associated metadata.

1.4.3: Error Handling: Wrap the upload and database logic in a try...catch block to handle potential failures gracefully and return an appropriate error status to the client.

1.5.1: Admin Page Component: In your React app, create a new component (e.g., AdminUpload.js) for the upload form.

1.5.2: Secure Admin Routing: Use react-router-dom to create a route for the admin page with a long, unguessable path. Do not link to this route from anywhere in the public UI.

JavaScript

// In App.js
<Route path="/upload-admin-a1b2c3d4" element={<AdminUpload />} />
1.6.1: Admin Upload Form UI: Build the form in the AdminUpload.js component with inputs for the file, description, and keywords.

1.6.2: Client-Side Upload Logic: Implement the form submission handler. It will use the FormData API to package the file and text fields for the fetch request.

JavaScript

// In AdminUpload.js
const handleSubmit = async (event) => {
  event.preventDefault();
  const formData = new FormData(event.target);

  const response = await fetch('/api/upload', {
    method: 'POST',
    body: formData,
  });

  if (response.ok) {
    alert('Upload successful!');
  } else {
    alert('Upload failed.');
  }
};
1.6.3: UI Feedback: Use React state to track the upload status (e.g., isUploading, error, success) and display messages or loading indicators to the admin.

Phase 2: Public Search and Results Display
Objective: Create the public-facing interface for users to search for and view audio files.

2.1.1: Public Landing Page Component: Create the main component for the landing page (e.g., SearchPage.js). This component will manage the overall state of the search experience.

2.1.2: Search Bar Component: Build a reusable <SearchBar /> component. Initially, it will be styled to be in the center of the page. After a search is performed, its position will change.

2.2.1: API Endpoint for Search: Create a public GET endpoint in your Express server that accepts a search query parameter.

JavaScript

// In index.js
app.get('/api/search', async (req, res) => {
  const { q } = req.query;
  if (!q) {
    return res.status(400).json({ message: 'Query parameter "q" is required.' });
  }
  //... search logic
});
2.2.2: Backend Search Logic: Use the Postgres client to perform a case-insensitive search using ILIKE against the description and keywords columns.

JavaScript

// Inside the /api/search endpoint
try {
  const searchTerm = `%${q}%`;
  const { rows } = await pool.query(
    'SELECT * FROM audio_files WHERE description ILIKE $1 OR keywords ILIKE $1 ORDER BY created_at DESC',

  );
  res.status(200).json(rows);
} catch (error) {
  console.error('Search failed:', error);
  res.status(500).send('Search failed.');
}
2.2.3: SQL Injection Prevention: Using parameterized queries (like $1 in the example above) with the pg library automatically sanitizes the input, preventing SQL injection attacks.

2.3.1: Frontend State Management: In your SearchPage.js component, use useState hooks to manage the application's state.

JavaScript

// In SearchPage.js
const [query, setQuery] = useState('');
const = useState();
const [isLoading, setIsLoading] = useState(false);
const [error, setError] = useState(null);
const = useState(false);
2.3.2: API Call Function: Implement the function that calls the search API. This function will update the loading state before the fetch and update the results, error, and loading states after the fetch completes.

2.4.1: Search Results List Component: Create a <ResultsList /> component that accepts the results array as a prop.

2.4.2: Mapping Results: Inside <ResultsList />, use the .map() method to iterate over the results array and render a <ResultItem /> component for each entry, passing the item's data as props.

2.5.1: Conditional Rendering for UI States: In SearchPage.js, use JSX to conditionally render different UI elements based on the state variables (isLoading, error, results, hasSearched).

JavaScript

// In SearchPage.js render method
<div>
  <SearchBar... />
  {isLoading && <p>Loading...</p>}
  {error && <p>{error}</p>}
  {hasSearched &&!isLoading && results.length === 0 && <p>No results found.</p>}
  {!isLoading && results.length > 0 && <ResultsList results={results} />}
</div>
Phase 3: Interactive Audio Playback
Objective: Integrate the waveform visualization and implement the critical single-play audio logic.

3.1.1: Install Waveform Library: Add the chosen waveform library to your React project: npm install react-audio-wave-modern.    

3.1.2: Integrate Waveform Component: In your <ResultItem /> component, import and use the ReactWaveform component, passing it the file_url from the search result props.

JavaScript

// In ResultItem.js
import ReactWaveform from 'react-audio-wave-modern'; // [3]

const ResultItem = ({ item }) => {
  const waveformOptions = {
    height: 80,
    waveColor: "#ccc",
    progressColor: "#007bff",
    cursorWidth: 0,
    barWidth: 3,
    barGap: 2,
    mediaControls: false, // Important: we build our own controls
  }; // [3]

  return (
    <div className="result-item">
      <p>{item.description}</p>
      <ReactWaveform audioUrl={item.file_url} options={waveformOptions} />
      {/* Play/Pause button will be added next */}
    </div>
  );
};
3.1.3: Style Waveform: Customize the waveform's appearance using the options prop as shown above to match your site's design.

3.2.1: Global Playback Context: Create a new file for the React Context to manage which audio is currently playing across the entire app.

JavaScript

// PlaybackContext.js
import { createContext } from 'react';
export const PlaybackContext = createContext(null);
3.2.2: Context Provider: In your main App.js or SearchPage.js, wrap the component tree with the PlaybackContext.Provider. The value of the provider will hold the ID of the currently playing track and a reference to its play/pause functions.

3.3.1: Add Play/Pause Button: In <ResultItem />, add a button next to the waveform.

3.3.2: Control Button State: Use the PlaybackContext to determine if the current item is the one playing. If playbackContext.currentlyPlayingId === item.id, show a "Pause" icon; otherwise, show a "Play" icon.

3.3.3: Implement onClick Handler: The button's click handler will be responsible for initiating the single-play logic.

3.4.1: Enforce Single-Play Behavior: The core logic resides in the onClick handler. Before playing a new track, it must first call the pause() function on the currently playing track (if any) stored in the context. You will need to get a reference to the wavesurfer instance from the ReactWaveform component to call its methods. The react-audio-wave-modern library provides ways to get this instance. After pausing the old track, it can play the new one and update the context with the new track's ID and its control functions.

Phase 4: Deployment and Finalization
Objective: Deploy the application to the live hosting environment and perform final configuration.

4.1.1: Vercel Project Creation: Go to the Vercel dashboard, select "Add New... Project," and import your Git repository. Vercel automatically detects the framework and configures the build settings.    

4.1.2: Build Configuration: Verify that Vercel has correctly identified your frontend framework (Create React App) and the root directory. Ensure the "Output Directory" for the frontend is set to client/build and the backend is configured as serverless functions.

4.2.1: Configure Environment Variables: In your Vercel project's settings, navigate to "Environment Variables." Add the POSTGRES_URL provided by Vercel Postgres and the BLOB_READ_WRITE_TOKEN from Vercel Blob.

4.2.2: Add Secret Admin URL: Add another environment variable, e.g., REACT_APP_ADMIN_URL, with the secret path (/upload-admin-a1b2c3d4). Use this variable in your React Router setup so the secret path is not hard-coded in your repository.

4.3.1: Trigger Deployment: Push your code to the main branch of your connected Git repository. Vercel will automatically start a new deployment.

4.3.2: End-to-End Admin Test: Once the deployment is live, navigate to the secret admin URL. Upload a new WAV file with metadata and verify that it appears in your Vercel Postgres database and Vercel Blob storage.

4.4.1: Test Public URL: Access the main URL of your Vercel deployment (e.g., your-project.vercel.app).

4.4.2: End-to-End Public Test: Perform a search for the file you just uploaded. Verify that it appears in the results, the waveform renders correctly, and the single-play audio functionality works as expected.

4.5.1: Final Review: Thoroughly check the deployed application for any visual glitches, console errors, or unexpected behavior across different browsers and screen sizes.

4.5.2: Project Documentation: Create a README.md file in your project's root. Document the project's purpose, the local setup process, and a list of all required environment variables. This is crucial for future maintenance and for anyone else who might work on the project.

2. spec.md
This section provides the functional specification for the AudioSearch application. It details the behavior, user flows, and acceptance criteria for each feature outlined in the roadmap.

Phase 1: Admin Upload
Feature: Admin File Upload and Storage

Description: As the administrator, I need a private, non-public page where I can upload WAV audio files. The interface should allow me to select a file from my local machine and provide associated metadata, specifically a textual description and a list of keywords for searching. When I submit the form, the system must securely store the WAV file in a cloud object store and save its location along with the metadata in a SQL database. This process should be atomic; if any part fails, the entire operation should be rolled back to prevent orphaned files or data. The security for this page will rely on an unguessable URL (security through obscurity), which is sufficient for this project's internal-use-only admin requirement.

Acceptance Criteria:

When I navigate to a pre-configured secret URL (e.g., /upload-admin-a1b2c3d4), the admin upload form is displayed. This URL must not be linked from any public part of the site.

The form contains a file input that accepts only .wav files, a text area for "Description," and a text input for "Keywords" (where multiple keywords can be entered, separated by commas).

Upon selecting a file and filling out the metadata fields, I can click a "Submit" button.

On successful submission, the WAV file is uploaded to the configured Vercel Blob container.   

Simultaneously, a new record is created in the audio_files table in the Vercel Postgres database. This record must contain the file's description, keywords, and the publicly accessible URL of the uploaded file from Vercel Blob.   

After a successful upload, a confirmation message (e.g., "File uploaded successfully") is displayed on the admin page.

If the file upload fails, no record is added to the database, and an error message (e.g., "Error uploading file. Please try again.") is displayed.

Attempting to submit the form without a selected file or with missing metadata results in a validation error message, and no API call is made.

Phase 2: Search UI
Feature: Public Search Interface

Description: A visitor to the website's main URL will be presented with an extremely minimalist interface: a blank page with a single, centrally located search bar. There will be no other content or navigation visible. When the visitor types a query into the search bar and submits it, the application will query the backend. The UI will then dynamically update to display a list of matching audio files below the search bar. The application must provide clear visual feedback to the user during this process, indicating loading, no-results, and error states.

Acceptance Criteria:

On initial page load, the browser viewport shows a blank background with a single search input field and a search button centered horizontally and vertically.

When a user enters a term (e.g., "drums") into the search bar and presses Enter or clicks the search button, an API request is sent to the backend search endpoint (/api/search?q=drums).

While the API request is in progress, the list area (which is not yet visible) displays a loading indicator (e.g., a spinner animation).

If the API returns one or more matching audio files, the search bar moves to the top of the page, and a list of result items is rendered below it.

If the API returns zero results, the search bar moves to the top, and a message "No results found for 'drums'" is displayed below it.

If the API call fails due to a network or server error, a user-friendly error message (e.g., "An error occurred. Please try your search again.") is displayed.

Submitting an empty search query results in no API call and the UI remains in its initial state.

Phase 3: Waveform & Playback
Feature: Waveform Visualization & Single-Play Audio

Description: Each item in the search results list represents a single audio file and must be an interactive component. It will prominently display a visual waveform of the audio, generated directly from the WAV file's data. Adjacent to or overlaid on the waveform will be a simple play/pause control. When a user initiates playback on one track, any other track that might be playing elsewhere on the page must automatically and immediately stop. This ensures a clean, focused listening experience where only one audio source is active at any given time. Playback progress will be visually represented on the waveform itself.

Acceptance Criteria:

Each search result item correctly renders a waveform visualization corresponding to its specific audio file using the react-audio-wave-modern library.   

Each item contains a control button that visually toggles between a "play" icon and a "pause" icon.

Scenario: Single Play Enforcement

GIVEN I see a list of results for Track A, Track B, and Track C.

WHEN I click the "play" button on Track A.

THEN Track A begins to play, its button changes to a "pause" icon, and its waveform shows playback progress (e.g., the played portion changes color).

WHEN I then click the "play" button on Track B.

THEN Track A's audio immediately stops playing.

AND Track A's button reverts to a "play" icon.

AND Track B's audio begins to play, its button changes to a "pause" icon, and its waveform shows playback progress.

Clicking the "pause" button on a currently playing track stops its audio playback and toggles its icon back to "play".

Phase 4: Deployment
Feature: Production Deployment and Automation

Description: The complete full-stack application will be deployed to a public URL using Vercel's hosting platform. The deployment pipeline will be fully automated through Git integration. A push to the main branch of the repository will automatically trigger a new build and deploy it to production with zero downtime. All sensitive information, such as database credentials and the secret admin URL, must be managed as secure environment variables within the Vercel platform and must not be hard-coded in the source code.

Acceptance Criteria:

The project is successfully linked to a Vercel account and configured for automated deployments from a Git repository.   

Pushing a new commit to the main branch automatically triggers a new deployment on Vercel.

The live application is accessible via its Vercel domain (e.g., project-name.vercel.app).

All production environment variables (e.g., DATABASE_URL, BLOB_READ_WRITE_TOKEN, ADMIN_URL_SECRET) are configured in the Vercel project settings and are correctly accessed by the application.

The public-facing search and playback features are fully functional in the deployed production environment.

The admin upload page is accessible and fully functional at its secret URL in the production environment.

A review of the client-side source code in the browser's developer tools confirms that no secret keys or environment variables are exposed.