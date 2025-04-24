specification document for the "Flashcard Extension" and the associated webcam feature integration.

---

## Project Specification: Flashcard Extension & Webcam Practice Enhancement

**1. Overview**

This project involves two main parts:

- Creating a Chrome browser extension ("Flashcard Extension") that allows users to quickly create new flashcards by selecting text on any webpage.
- Enhancing an existing web application (running locally for flashcard practice) to allow users to rate card difficulty ("Wrong", "Hard", "Easy") using webcam-based hand gestures.

**2. Core Flashcard Creation (Browser Extension)**

This feature is implemented as a Chrome Extension.

**2.1. Manifest (`manifest.json`)**

```json
{
  "manifest_version": 3,
  "name": "Flashcard Extension",
  "version": "1.0",
  "description": "add new flashcards by selecting any text from any webpage",
  "permissions": [
    // "activeTab", // Use scripting instead for content script injection based on our design
    "storage", // Potentially useful for future settings, keep for now
    "contextMenus" // Keep for now, though primary interaction is inline button
    // "tabs" // Needed for popup.js to open new tab
  ],
  "host_permissions": [
    "<all_urls>" // Needed for content script and API calls to localhost
  ],
  "action": {
    "default_popup": "popup.html"
  },
  "background": {
    "service_worker": "background.js"
    // "type": "module" // Add if using ES modules in background.js
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"],
      "css": ["content.css"],
      "run_at": "document_idle" // Ensures page is mostly loaded
    }
  ],
  "web_accessible_resources": [
    // Needed if modal injects resources like images/fonts
    {
      "resources": ["images/*.png"], // Example if modal uses images
      "matches": ["<all_urls>"]
    }
  ]
}
```

_(Note: Final permissions should be reviewed and minimized based on actual usage. `scripting` might be more appropriate than `activeTab` depending on how the content script is ultimately injected if not statically loaded as above. Added `tabs` for popup functionality. Added `web_accessible_resources` as a potential need for the modal.)_

**2.2. Content Script (`content.js` & `content.css`)**

- **Selection Detection (`content.js`):**
  - Listen for the `mouseup` event on the document.
  - On `mouseup`, use `window.getSelection()` to get the selected text.
  - Validate the selection:
    - Trimmed text length must be >= 5 characters.
    - The selection's origin must not be within an `<input>` or `<textarea>` element.
  - Store the valid selected text in a local variable within `content.js`.
- **Inline Button ("+ add flashcard"):**
  - If a selection is valid, display a small button (icon and/or text "+ add flashcard") near the end of the selection.
  - Positioning should use `selection.getRangeAt(0).getBoundingClientRect()`.
  - Styling is defined in `content.css` (ensure it has a high `z-index` and avoids conflicts with page styles).
  - The button is hidden if:
    - The selection becomes invalid (e.g., text cleared, length < 5).
    - The user triggers a `mousedown` event anywhere else on the document.
- **Modal Dialog (Triggered by clicking "+ add flashcard" button):**
  - Managed entirely by `content.js`.
  - **Appearance:** Displayed directly on the current webpage as a modal dialog (e.g., a centered `div` with its own styling). Includes a semi-transparent overlay behind it to dim the page content. Has a visible 'X' close button. Styling defined in `content.css` or dynamically applied.
  - **Content:**
    - **Front:** `<textarea>` input, labeled, required.
    - **Back:** `<textarea>` input, labeled, required, pre-filled with the selected text stored in the `content.js` variable.
    - **Hint:** `<input type="text">`, labeled, optional.
    - **Tags:** `<input type="text">`, labeled, optional. User enters tags as a single comma-separated string (e.g., `"tag1, tag2, tag3"`).
    - **Save Button:** Primary action button.
    - **Cancel Button:** Secondary action button.
  - **Behavior:**
    - **Save Click:**
      1.  Validate that 'Front' and 'Back' fields are not empty. Show an error message _within the modal_ if validation fails.
      2.  Parse the 'Tags' input string: split by comma, trim whitespace from each part, filter out empty strings to create an array (e.g., `"tag1, tag2 ,, tag3 "` -> `["tag1", "tag2", "tag3"]`).
      3.  Send a `fetch` request to the backend: `POST /api/cards` with a JSON body: `{ "front": ..., "back": ..., "hint": ..., "tags": [...] }`.
      4.  **On Success (Backend responds `201 Created`):**
          - Temporarily display the success message (from the response body) within the modal.
          - After a 2-second delay, automatically close the modal.
      5.  **On Error (Backend responds non-201):**
          - Display an error message within the modal (using the backend error message if available).
          - The modal remains open for the user to correct or cancel.
    - **Cancel Click / 'X' Button Click:** Immediately close the modal, discarding inputs.
    - **Overlay Click:** Clicking the overlay behind the modal does **not** close it.

**2.3. Background Script (`background.js`)**

- Minimal initial role.
- Include a `chrome.runtime.onInstalled` listener that logs a message to the console (e.g., "Flashcard Extension installed/updated.") to confirm loading.

**2.4. Action Popup (`popup.html` & `popup.js`)**

- This UI appears when the user clicks the extension icon in the Chrome toolbar.
- **Content (`popup.html`):**
  - Display Title: "Flashcard Extension".
  - Display Version: "v1.0".
  - Display Button: Text like "Open Practice App".
- **Functionality (`popup.js`):**
  - Add a click listener to the "Open Practice App" button.
  - On click, execute `chrome.tabs.create({ url: 'YOUR_LOCALHOST_PRACTICE_APP_URL' });`, replacing the placeholder URL with the actual URL of the locally running web application (e.g., 'http://localhost:5173').

**3. Backend API Endpoint**

- **Endpoint:** `POST /api/cards`
- **Method:** `POST`
- **Request Body JSON:**
  ```json
  {
    "front": "string",
    "back": "string",
    "hint": "string" // Optional
    "tags": ["string", "string", ...] // Optional, expects an array parsed by frontend
  }
  ```
- **Validation:**
  - `front` field must exist and be a non-empty string.
  - `back` field must exist and be a non-empty string.
  - Return `400 Bad Request` if validation fails.
- **Success Action:**
  - Create a new `Flashcard` object using the provided data.
  - Add the new `Flashcard` object to Bucket 0 in the application state (`state.ts`).
- **Success Response:**
  - **Status Code:** `201 Created`
  - **Body:** `{ "message": "Flashcard created successfully" }`
- **Error Response:** Return appropriate 4xx or 5xx status codes with informative JSON error messages.

**4. Flashcard Practice Enhancement (Web Application)**

These changes apply to the existing frontend web application, specifically within the `PracticeView.tsx` component (or equivalent).

- **Activation Button:** Add a button labeled "Use Webcam hand detection".
- **Functionality:**
  - Clicking the button enables/disables the webcam feature.
  - The original difficulty buttons ("Wrong", "Hard", "Easy") remain visible and functional at all times.
- **Webcam Display:**
  - When enabled, display the live webcam feed in a small, fixed overlay element in the **bottom-right** corner of the practice view area.
- **Permissions:**
  - On first activation, the browser will prompt for webcam permission.
  - If permission is **denied**:
    - The webcam overlay area shows a **grey background** with the text **"webcam permission denied"**.
    - The "Use Webcam hand detection" button remains clickable, allowing the user to try initiating permission again.
- **Gesture Detection:**
  - **Technology:** Use TensorFlow.js (`tfjs`) with the `@tensorflow-models/hand-pose-detection` package, specifically using the `MediaPipeHands` model.
  - **Detected Gestures & Mapping:**
    - **Thumbs Up:** -> "Easy"
    - **Thumbs Down:** -> "Wrong"
    - **Flat Hand (palm facing camera or side):** -> "Hard" _(Rule needs precise definition)_
  - **Trigger Logic:** A detected gesture (Thumbs Up/Down/Flat Hand) only triggers an action if it is held **consistently for 3 seconds**.
    - The system continuously monitors the gesture.
    - If a target gesture is detected, a 3-second timer starts/continues.
    - If the _same_ gesture remains detected for 3 consecutive seconds, the action is triggered.
    - If the gesture changes, the hand disappears, or a non-target gesture is shown before 3 seconds, the timer resets.
  - **Feedback (During Hold):** While a target gesture is being held, the border of the webcam overlay progressively fills with a color over the 3 seconds:
    - Thumbs Up: **Green**
    - Thumbs Down: **Red**
    - Flat Hand: **Orange**
  - **Feedback (On Trigger):** The completion of the border fill animation serves as the primary feedback that the gesture was registered. The border fill then immediately resets.
  - **Action:** When a gesture is successfully registered after the 3-second hold, the system immediately calls the existing `submitAnswer` function (or equivalent) with the corresponding difficulty level ('Easy', 'Wrong', or 'Hard') for the currently displayed flashcard.
- **Gesture Classification Logic:**
  - Implement using simple geometric rules based on the landmark coordinates provided by the `MediaPipeHands` model (e.g., relative positions/angles of thumb tip, finger tips, wrist).

**5. Testing Requirements**

- **Backend (`POST /api/cards`):**
  - **Unit Tests:** Cover validation logic for request body fields (`describe`/`it` format, e.g., using Jest/Mocha).
  - **Integration Tests:** Test the `POST /api/cards` endpoint directly, verifying:
    - Successful creation (201 status, response message, state updated correctly).
    - Failure cases (400 status for missing/empty required fields).
- **Browser Extension Frontend:**
  - **Manual E2E Test Plan:** Define detailed steps to manually test the entire workflow:
    - Select text on various websites.
    - Verify button appearance/disappearance.
    - Click button, verify modal opens with correct pre-filled 'Back' text.
    - Test form input and validation within the modal.
    - Test 'Save' -> success message -> auto-close.
    - Test 'Save' -> error message -> modal stays open.
    - Test 'Cancel'/'X' button.
    - Verify card correctly appears in the main practice application afterwards.
- **Webcam/Gesture Feature (Web Application):**
  - **Unit Tests:** (`describe`/`it` format)
    - Test the specific function(s) that perform gesture classification.
    - Input: Mock hand landmark data arrays representing various gestures (Thumbs Up, Down, Flat Hand, None, edge cases like empty data).
    - Assertion: Verify the function returns the correct string label ('ThumbsUp', 'ThumbsDown', 'FlatHand', 'None').
  - **Manual Test Plan:** Define detailed steps to manually test:
    - Activation/Deactivation of the feature.
    - Permission request and denial handling (overlay message).
    - Recognition of each target gesture (Thumbs Up/Down/Flat Hand) after the 3-second hold.
    - Correctness of the 3-second border fill feedback (color, progression).
    - Correct triggering of Easy/Wrong/Hard actions on the flashcard.
    - Resetting of timer/feedback if gesture changes or disappears before 3 seconds.
    - Handling of non-target gestures.
    - Functionality alongside fallback ('Wrong'/'Hard'/'Easy') buttons.
    - Robustness under reasonable variations (lighting, background).
    - Interaction when card hint is displayed.
    - Overall performance impact.

**6. Documentation Requirements**

- **Format:**
  - In-code comments using TSDoc/JSDoc format above relevant classes and functions.
  - A separate markdown file (e.g., `DOCUMENTATION.md`) summarizing key components.
- **Content:** For specified components, document:
  - **Abstract Function (AF):** What the component represents from the client's perspective.
  - **Representation Invariant (RI):** Properties that must always be true about the internal data structure.
  - **Safety from Representation Exposure (SRE):** How encapsulation is maintained (or where risks exist).
- **Scope:** Provide AF/RI/SRE documentation for:
  - `Flashcard` class (`logic/flashcards.ts`)
  - State management module (`backend/src/state.ts`)
  - Core algorithm logic functions/module (`logic/algorithm.ts`)

---
