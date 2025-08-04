Software Development Plan and Functional Specification: Project "AudioSearch"
This document provides a comprehensive development roadmap and functional specification for the "AudioSearch" web application. It is designed to guide the implementation of a minimalist, search-driven audio library, adhering to the specified technical stack and project constraints.

1. Roadmap (Phases, Steps & Sub-steps)
The development process is structured into four distinct phases, each with concrete steps and sub-steps. This phased approach ensures a logical progression from foundational backend setup to final deployment, allowing for iterative development and testing.

Phase 1: Foundational Backend and Admin Interface
Objective: Establish the data persistence layer and a secure method for the administrator to upload content.

Step 1.1: Project Initialization & Backend Server Setup

Sub-step 1.1.1: Initialize a monorepo or two separate repositories for the frontend (React) and backend (Node.js).

Sub-step 1.1.2: Set up a basic Node.js server using the Express framework.

Sub-step 1.1.3: Configure the project for Vercel's Serverless Functions environment.

Step 1.2: Database Provisioning & Schema Definition

Sub-step 1.2.1: Provision a new Vercel Postgres database through the Vercel dashboard.   

Sub-step 1.2.2: Define the SQL schema for the audio_files table, including columns for ID, file identifiers, and searchable metadata.

Sub-step 1.2.3: Create an SQL script to initialize the database schema.

Step 1.3: File Storage Provisioning & Upload Logic

Sub-step 1.3.1: Enable and configure Vercel Blob for the project to handle WAV file storage.   

Sub-step 1.3.2: Integrate the multer middleware into the Express server to process multipart/form-data requests.   

Sub-step 1.3.3: Implement the server-side logic to stream uploaded files to Vercel Blob.

Step 1.4: Create API Endpoint for File Upload & Metadata Ingestion

Sub-step 1.4.1: Define a secure POST endpoint (e.g., /api/upload).

Sub-step 1.4.2: Implement the endpoint logic to orchestrate the file upload to Vercel Blob and the subsequent metadata insertion into Vercel Postgres.

Sub-step 1.4.3: Implement robust error handling for failed uploads or database writes.

Step 1.5: Build Secure Admin Page (Frontend)

Sub-step 1.5.1: Create a new, unlinked React component for the admin page.

Sub-step 1.5.2: Implement routing to make the page accessible only via a non-discoverable, hard-coded URL.

Step 1.6: Implement Admin Upload Form (Frontend)

Sub-step 1.6.1: Build the UI form with inputs for file selection, description, and keywords.

Sub-step 1.6.2: Implement the client-side logic to construct the FormData object and submit it to the backend API endpoint.

Sub-step 1.6.3: Add UI feedback for success, failure, and in-progress states.

Phase 2: Public Search and Results Display
Objective: Create the public-facing interface for users to search for and view audio files.

Step 2.1: Build Public Landing Page & Search Bar Component

Sub-step 2.1.1: Create the main React component for the public landing page.

Sub-step 2.1.2: Design and implement a reusable, centrally positioned search bar component.

Step 2.2: Create API Endpoint for Search

Sub-step 2.2.1: Define a public GET endpoint (e.g., /api/search).

Sub-step 2.2.2: Implement the backend logic to query the Vercel Postgres database using a case-insensitive LIKE clause on metadata fields.

Sub-step 2.2.3: Ensure the endpoint sanitizes user input to prevent SQL injection vulnerabilities.

Step 2.3: Implement Frontend State Management for Search

Sub-step 2.3.1: Set up state variables in the main React component to manage the search query, results, loading status, and error messages.

Sub-step 2.3.2: Implement the function to call the search API and update the component's state with the response.

Step 2.4: Build Search Results List Component

Sub-step 2.4.1: Create a component that takes an array of search results as a prop.

Sub-step 2.4.2: Implement the logic to map over the results array and render a distinct result item for each audio file.

Step 2.5: Implement UI States (Loading, No Results, Error)

Sub-step 2.5.1: Use conditional rendering to display a loading indicator while the API call is pending.

Sub-step 2.5.2: Implement conditional rendering to display a "No results found" message when the API returns an empty array.

Sub-step 2.5.3: Implement an error boundary or conditional rendering to show a user-friendly error message on API failure.

Phase 3: Interactive Audio Playback
Objective: Integrate the waveform visualization and implement the critical single-play audio logic.

Step 3.1: Integrate Waveform Component into Search Results

Sub-step 3.1.1: Install and import the react-audio-wave-modern library.   

Sub-step 3.1.2: Modify the search result item component to include the waveform component, passing it the public URL of the WAV file.

Sub-step 3.1.3: Style the waveform to match the application's minimalist aesthetic, configuring properties like color and bar height.   

Step 3.2: Implement Global Playback State Management

Sub-step 3.2.1: Create a React Context provider to manage the global playback state (e.g., currentlyPlayingId, isPlaying).

Sub-step 3.2.2: Wrap the main application component with this provider to make the state accessible to all child components.

Step 3.3: Implement Play/Pause Controls

Sub-step 3.3.1: Add a button to each search result item.

Sub-step 3.3.2: The button's appearance (e.g., play or pause icon) should be determined by the global playback state.

Sub-step 3.3.3: Implement the onClick handler to trigger playback or pause via the waveform library's API and update the global state.

Step 3.4: Enforce Single-Play Behavior

Sub-step 3.4.1: Before playing a new track, the onClick handler must first check the global state for any currently playing track.

Sub-step 3.4.2: If another track is playing, the handler will programmatically pause it using its reference from the waveform library.

Sub-step 3.4.3: After ensuring all other tracks are paused, it will proceed to play the selected track and update the global state accordingly.

Phase 4: Deployment and Finalization
Objective: Deploy the application to the live hosting environment and perform final configuration.

Step 4.1: Vercel Project Configuration & Git Integration

Sub-step 4.1.1: Create a new project on Vercel and link it to the application's Git repository (GitHub, GitLab, or Bitbucket).   

Sub-step 4.1.2: Configure the build settings for the Node.js backend and the React frontend.

Step 4.2: Configure Production Environment Variables

Sub-step 4.2.1: In the Vercel project settings, add the connection string for the Vercel Postgres database.

Sub-step 4.2.2: Add environment variables for Vercel Blob storage credentials.

Sub-step 4.2.3: Add an environment variable for the secret admin URL path to avoid committing it to the repository.

Step 4.3: Deploy and Test Admin Functionality

Sub-step 4.3.1: Push the code to the main branch to trigger a production deployment on Vercel.

Sub-step 4.3.2: Access the deployed application at the secret admin URL and perform an end-to-end test of the file upload functionality.

Step 4.4: Deploy and Test Public Search Functionality

Sub-step 4.4.1: Access the public URL of the deployed application.

Sub-step 4.4.2: Perform end-to-end tests of the search and playback features, verifying all UI states and the single-play constraint.

Step 4.5: Final Review and Documentation

Sub-step 4.5.1: Review the deployed application for any bugs or styling issues.

Sub-step 4.5.2: Create a README.md file documenting the project setup, environment variable requirements, and the secret admin URL path for future reference.

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