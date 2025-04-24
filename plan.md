detailed, step-by-step blueprint and then break it down into LLM prompts suitable for TDD and incremental implementation.

## I. Project Blueprint & High-Level Plan

This plan prioritizes building the foundational backend pieces first, then the core extension functionality, followed by the webcam enhancement, and finally the auxiliary extension parts. Documentation is integrated alongside backend development.

**Phase 1: Backend API (`POST /api/cards`)**

1.  **Setup:** Ensure existing backend structure (`server.ts`, `state.ts`, `Flashcard` class) is ready.
2.  **Validation Logic & Tests:** Implement and unit-test the validation logic for the `POST /api/cards` request body (`front`, `back` required, non-empty).
3.  **State Update Logic & Tests:** Implement and unit-test the function in `state.ts` to add a new `Flashcard` to Bucket 0.
4.  **API Endpoint Implementation:** Create the `POST /api/cards` route in `server.ts`, integrate validation logic, call the state update function on success, and return the correct success (201) or error (400) responses.
5.  **Integration Tests:** Write integration tests for the `POST /api/cards` endpoint covering success and validation failure cases.
6.  **Documentation (AF/RI/SRE):** Add required TSDoc comments to `Flashcard`, `state.ts`, and `algorithm.ts`.

**Phase 2: Core Extension Functionality (Content Script & Modal)**

1.  **Basic Extension Structure:** Create `manifest.json`, empty `content.js`, `content.css`, `background.js`, `popup.html`, `popup.js`. Ensure basic loading works.
2.  **Text Selection Detection:** Implement logic in `content.js` to detect valid text selections (`mouseup`, length >= 5, not in input). Store selected text locally.
3.  **Inline Button Display:** Implement logic in `content.js` to show/hide/position the "+ add flashcard" button based on selection validity. Basic styling in `content.css`.
4.  **Modal Structure & Display:** Implement logic in `content.js` to create and show a basic modal (HTML structure for fields and buttons) when the inline button is clicked. Pre-fill 'Back' field.
5.  **Modal Styling & Close:** Style the modal using `content.css`. Implement 'Cancel'/'X' button functionality to close the modal.
6.  **Modal Save Logic (Client-Side):** Implement 'Save' button logic in `content.js`: client-side validation, tag parsing.
7.  **Modal Save Logic (API Integration):** Integrate the `fetch` call to `POST /api/cards`.
8.  **Modal Save Logic (Response Handling):** Implement success (show message, close after delay) and error (show message) handling based on the API response.

**Phase 3: Webcam/Gesture Enhancement (Web App)**

1.  **Gesture Classification Logic & Tests:** Create and unit-test the `classifyGesture(landmarks)` function using mock data and simple geometric rules.
2.  **Webcam UI Placeholders:** Add the "Use Webcam" button and overlay `div` to `PracticeView.tsx`. Add basic state management for enabling/disabling.
3.  **Webcam Activation & Permissions:** Implement webcam stream activation (`getUserMedia`), permission handling (denied message), and display the video feed in the overlay.
4.  **TFJS Integration & Prediction:** Integrate TensorFlow.js, load `MediaPipeHands`, and get landmark predictions from the video stream.
5.  **Connect Classification & Timer Logic:** Connect landmark predictions to `classifyGesture`. Implement the 3-second consistent gesture hold logic.
6.  **Trigger Action & Feedback:** Trigger the `submitAnswer` call on successful 3-second hold. Implement the progressive border-fill feedback animation.

**Phase 4: Auxiliary Extension Components & Finalization**

1.  **Background Script:** Implement the minimal `onInstalled` listener in `background.js`.
2.  **Action Popup:** Implement `popup.html` layout and `popup.js` functionality (open practice app button).
3.  **Documentation File:** Create/update `DOCUMENTATION.md` summarizing AF/RI/SRE.
4.  **Manual Testing:** Perform full E2E manual tests for the extension and webcam feature as specified.
5.  **Code Review & Refinement:** Review code, refine styles, check permissions, finalize documentation.

## II. Iterative Steps & LLM Prompts

This breakdown focuses on small, testable, and integrated steps.

---

### Phase 1: Backend API (`POST /api/cards`)

**Step 1.1: Validation Logic & Unit Tests**

```text
Context: We need to implement the backend API endpoint `POST /api/cards`. The first step is to create and test the validation logic for the request body.

Request:
In the backend project structure (`backend/src/`), create a new file, potentially `backend/src/utils/validation.ts`.
Define a function `validateCardPayload(payload: any): { isValid: boolean; errors: string[] }`.
This function should check if the `payload` object contains `front` and `back` properties that are non-empty strings after trimming.
If validation passes, it returns `{ isValid: true, errors: [] }`.
If validation fails, it returns `{ isValid: false, errors: ["Error message detailing missing/empty fields"] }`.

Then, create a corresponding unit test file (e.g., `backend/src/utils/validation.test.ts`) using Jest (`describe`/`it` format).
Write unit tests for `validateCardPayload` covering:
- Valid payload.
- Payload missing 'front'.
- Payload missing 'back'.
- Payload with empty string 'front'.
- Payload with empty string 'back'.
- Payload with 'front'/'back' needing trimming but otherwise valid.
- Null or undefined payload.
```

**Step 1.2: State Update Logic & Unit Tests**

```text
Context: We need a way to add a newly created `Flashcard` to Bucket 0 in our backend state managed by `state.ts`. We assume `state.ts` manages `currentBuckets: BucketMap` (where `BucketMap` is likely `Map<number, Set<Flashcard>>`) and the `Flashcard` class exists in `logic/flashcards.ts`.

Request:
1.  Modify `backend/src/state.ts`:
    - Export a function `addCardToBucket0(card: Flashcard): void`.
    - This function should get the Set for bucket 0 from `currentBuckets`. If bucket 0 doesn't exist, create it (`new Set<Flashcard>()`).
    - Add the provided `card` object to the Set for bucket 0.
    - Ensure `currentBuckets` is updated.
2.  Create/Update a unit test file for `state.ts` (e.g., `backend/src/state.test.ts`) using Jest.
3.  Write unit tests for `addCardToBucket0`:
    - Test adding a card when bucket 0 is initially empty. Verify the card is in `currentBuckets.get(0)`.
    - Test adding a card when bucket 0 already exists and has other cards. Verify the new card is added alongside existing ones.
    - Consider testing with a mock `Flashcard` object.
    - Ensure tests properly set up initial state and check the final state of `currentBuckets`. Remember to reset state between tests if necessary.
```

**Step 1.3: API Endpoint Implementation (Initial Route & Integration)**

```text
Context: We have validation logic (`validateCardPayload`) and state update logic (`addCardToBucket0`). Now we integrate them into the `POST /api/cards` endpoint in `server.ts`. Assume necessary imports (`express`, state functions, validation function, `Flashcard` class) are handled.

Request:
In `backend/src/server.ts`:
1.  Define the route `app.post('/api/cards', (req, res) => { ... });`.
2.  Inside the route handler:
    - Call `validateCardPayload(req.body)`.
    - **If `isValid` is false**: Return a `400 Bad Request` response with a JSON body like `{ message: "Validation failed", errors: validationResult.errors }`.
    - **If `isValid` is true**:
        - Extract `front`, `back`, `hint` (optional), and `tags` (optional, default to empty array if not provided) from `req.body`.
        - Create a new `Flashcard` instance: `const newCard = new Flashcard(front, back, hint, tags);`
        - Call `addCardToBucket0(newCard)`.
        - Return a `201 Created` response with the JSON body `{ message: "Flashcard created successfully" }`.
    - Add basic `try...catch` block around the logic to handle unexpected errors and return a `500 Internal Server Error` response.
```

**Step 1.4: API Integration Tests**

```text
Context: The `POST /api/cards` endpoint is implemented. We need integration tests to verify its behavior using a testing framework like Jest with Supertest.

Request:
Create an integration test file (e.g., `backend/tests/api.test.ts`).
Write integration tests for the `POST /api/cards` endpoint using Supertest to simulate HTTP requests against the running Express app instance.
Test scenarios:
1.  **Success Case:** Send a valid payload (with `front`, `back`, optional `hint`, `tags`).
    - Assert the response status is `201`.
    - Assert the response body matches `{ message: "Flashcard created successfully" }`.
    - (Optional but recommended) Check the backend state (`state.getBuckets()`) to confirm the card was actually added to bucket 0. Requires exporting state accessors if not already done.
2.  **Failure Case (Validation):** Send payloads that should fail validation (missing `front`, empty `back`, etc.).
    - Assert the response status is `400`.
    - Assert the response body contains an appropriate error message and/or error details.
```

**Step 1.5: Documentation (AF/RI/SRE)**

```text
Context: The core backend logic and API endpoint are implemented and tested. We need to add the required documentation comments.

Request:
Add TSDoc/JSDoc comments to the following files, including Abstract Function (AF), Representation Invariant (RI), and notes on Safety from Representation Exposure (SRE) where applicable:
1.  `backend/src/logic/flashcards.ts`: Add comments to the `Flashcard` class definition.
2.  `backend/src/state.ts`: Add comments at the module level explaining the state representation (`currentBuckets`, `practiceHistory`, etc.) and potentially to key accessor/mutator functions like `addCardToBucket0`, `getBuckets`.
3.  `backend/src/logic/algorithm.ts`: Add comments to the main functions responsible for the Leitner algorithm logic (e.g., `practice`, `update`, `computeProgress` if they exist). Focus on their purpose (AF), any assumptions about input state (preconditions related to RI), and how they handle data structures (SRE).
```

---

### Phase 2: Core Extension Functionality (Content Script & Modal)

**Step 2.1: Basic Extension Structure**

```text
Context: We are starting the Chrome Extension implementation.

Request:
Create the basic file structure for the extension:
- Create `manifest.json` with the content specified in the requirements (including version 3, name, description, basic permissions like `storage`, `contextMenus`, `tabs`, host permission `<all_urls>`, action popup definition, background service worker definition, content script definition pointing to `content.js`/`content.css`).
- Create empty files: `content.js`, `content.css`, `background.js`, `popup.html`, `popup.js`.
- Add a simple `<h1>Hello Popup</h1>` to `popup.html` and `console.log("Background script loaded");` to `background.js`. Add `console.log("Content script loaded");` to `content.js`.
- Provide instructions on how to load this unpacked extension in Chrome and verify that the background script logs its message to the service worker console, the content script logs its message to the console of a regular webpage, and clicking the extension icon shows the "Hello Popup" text.
```

**Step 2.2: Text Selection Detection**

```text
Context: The basic extension loads. Now, `content.js` needs to detect user text selections.

Request:
Modify `content.js`:
1.  Add a global variable `let currentSelectedText = null;`.
2.  Add an event listener for the `mouseup` event on the `document`.
3.  Inside the `mouseup` handler:
    - Get the current selection using `window.getSelection()`.
    - Get the selected text using `selection.toString().trim()`.
    - Check if the selection's anchor node's parent element is an INPUT or TEXTAREA (use `selection.anchorNode.parentElement.tagName`).
    - **If** the trimmed text length is >= 5 **and** it's not inside an INPUT/TEXTAREA:
        - Store the text in `currentSelectedText`.
        - `console.log("Valid selection detected:", currentSelectedText);` // Temporary feedback
    - **Else**:
        - Set `currentSelectedText = null;`
        - `console.log("Selection invalid or cleared.");` // Temporary feedback
4.  Provide instructions for testing: Load the extension, select text (>5 chars) on a webpage, check the console. Select text in an input field, check console. Click without selecting, check console.
```

**Step 2.3: Inline Button Display**

```text
Context: `content.js` can detect valid selections. Now, display a button near the selection.

Request:
Modify `content.js`:
1.  Add a function `createOrGetButton()`:
    - Checks if a button element with a specific ID (e.g., `flashcard-ext-add-button`) already exists. If yes, return it.
    - If not, create a `button` element. Set its ID, text content ("+ add flashcard"), and some basic initial styles (e.g., `position: absolute; display: none; z-index: 9999;`).
    - Append the button to the `document.body`.
    - Return the button element.
2.  Modify the `mouseup` handler:
    - Get the button using `createOrGetButton()`.
    - **If** a valid selection is detected:
        - Get the selection range using `selection.getRangeAt(0)`.
        - Get the range's bounding rectangle using `range.getBoundingClientRect()`.
        - Calculate the button's position (e.g., `top: window.scrollY + rect.bottom + 5 + 'px'`, `left: window.scrollX + rect.right + 5 + 'px'`). Handle potential edge cases (e.g., button going off-screen - basic clamping might be needed later).
        - Set the button's `style.top`, `style.left`, and `style.display = 'block'`.
    - **Else** (invalid selection):
        - Set the button's `style.display = 'none'`.
3.  Add a `mousedown` event listener to the `document`:
    - Inside the handler, get the button using `createOrGetButton()` (or check if it exists).
    - If the button exists, set its `style.display = 'none'`.
4.  Add basic styles to `content.css` for the button ID (e.g., background color, border, padding, cursor).
5.  Provide testing instructions: Load, select text, verify button appears near selection. Click elsewhere, verify button disappears. Select invalid text, verify button doesn't appear or disappears.
```

**Step 2.4: Basic Modal Structure & Display**

```text
Context: The inline button appears. Clicking it should open a modal dialog created by `content.js`.

Request:
Modify `content.js`:
1.  Add a function `createModal()`:
    - Creates the DOM structure for the modal:
        - An overlay `div` (e.g., ID `flashcard-ext-overlay`, styles: fixed position, full screen, background semi-transparent, high z-index, initially hidden).
        - A modal container `div` (e.g., ID `flashcard-ext-modal`, styles: fixed position, centered, background white, padding, border, high z-index, initially hidden).
        - Inside modal container: Add placeholders/labels and inputs (`textarea` for Front/Back, `input` for Hint/Tags), and buttons ('Save', 'Cancel', 'X' close button). Assign IDs to inputs and buttons (e.g., `flashcard-ext-front`, `flashcard-ext-save-btn`).
    - Appends the overlay and modal container to `document.body`.
2.  Add a function `showModal()`:
    - Finds the overlay and modal elements.
    - Sets their `display` style to 'block' (or flex/grid as appropriate for centering).
    - Finds the 'Back' textarea (`flashcard-ext-back`).
    - Sets its `value` to the `currentSelectedText` variable (stored from the selection step).
3.  Add a function `hideModal()`:
    - Finds the overlay and modal elements.
    - Sets their `display` style to 'none'.
    - Optionally clear input fields within the modal.
4.  Modify the `createOrGetButton()` function (or add separately): Add a click event listener to the inline "+ add flashcard" button.
    - Inside the listener, call `showModal()`.
5.  Provide testing instructions: Load, select text, click the inline button, verify a basic modal appears with the 'Back' field pre-filled.
```

**Step 2.5: Modal Styling & Close Logic**

```text
Context: The basic modal appears. Now, style it properly and make the Cancel/Close buttons work.

Request:
1.  Modify `content.css`: Add comprehensive styles for `#flashcard-ext-overlay`, `#flashcard-ext-modal`, and the elements within the modal (inputs, labels, buttons) to make it look clean and centered. Use high `z-index` values.
2.  Modify `content.js`:
    - In the `createModal()` function (or add separately), add click event listeners to:
        - The 'Cancel' button (`#flashcard-ext-cancel-btn`).
        - The 'X' close button (give it an ID like `#flashcard-ext-close-btn`).
    - Both listeners should simply call the `hideModal()` function.
3.  Provide testing instructions: Verify modal appearance is improved. Verify clicking Cancel or 'X' closes the modal. Verify clicking the overlay does *not* close it (as per spec).
```

**Step 2.6: Modal Save Logic (Client-Side Validation & Parsing)**

```text
Context: The modal displays and closes. Now, implement the client-side logic for the 'Save' button.

Request:
Modify `content.js`:
1.  Add a function `parseTags(tagsString)`:
    - Takes the comma-separated tag string as input.
    - Splits the string by comma.
    - Trims whitespace from each resulting item.
    - Filters out any empty strings.
    - Returns the array of valid tag strings.
2.  Add a function `showModalError(message)`:
    - Finds/creates an element within the modal (e.g., a `div` with ID `#flashcard-ext-error-message`) to display error messages.
    - Sets its text content to `message` and makes it visible. Hides it if `message` is empty or null.
3.  In `createModal()` (or add separately), add a click event listener to the 'Save' button (`#flashcard-ext-save-btn`).
4.  Inside the 'Save' button listener:
    - Clear any previous error messages by calling `showModalError(null)`.
    - Get the current values from the 'Front' (`#flashcard-ext-front`) and 'Back' (`#flashcard-ext-back`) textareas, trimming whitespace.
    - **Validate**: Check if `frontValue` or `backValue` is empty.
        - If validation fails, call `showModalError("Front and Back fields are required.")` and **return** (stop processing).
    - Get the value from the 'Tags' input (`#flashcard-ext-tags`).
    - Call `parseTags()` on the tags value to get the tags array.
    - Get the value from the 'Hint' input (`#flashcard-ext-hint`).
    - `console.log("Validation passed. Data:", { front: frontValue, back: backValue, hint: hintValue, tags: parsedTags });` // Temporary log
5.  Provide testing instructions: Test saving with empty Front/Back fields (verify error message). Test saving with valid Front/Back and various tag inputs (comma-separated, extra spaces, empty commas) and check the console log for correctly parsed data.
```

**Step 2.7: Modal Save Logic (API Integration)**

```text
Context: Client-side validation and parsing work. Now, call the backend API on save.

Request:
Modify the 'Save' button click listener in `content.js`:
1.  After the `console.log` for validated data (from the previous step):
    - Construct the `payload` object: `{ front: frontValue, back: backValue, hint: hintValue, tags: parsedTags }`. Handle potential undefined `hint`.
    - Use the `fetch` API to make a `POST` request to `http://localhost:3001/api/cards` (ensure the port matches your backend).
    - Set the `method: 'POST'`, `headers: { 'Content-Type': 'application/json' }`, and `body: JSON.stringify(payload)`.
    - Add `.then(response => ...)` and `.catch(error => ...)` to handle the response.
    - **Inside `.then()`**: `console.log("API Response Status:", response.status); return response.json();` (Log status for now).
    - **Inside `.then()` (after `.json()`):** `console.log("API Response Body:", data);`
    - **Inside `.catch()`**: `console.error("API Fetch Error:", error); showModalError("Failed to save card. Network error.");`
    - Add temporary UI feedback like disabling the Save button while the request is pending.
2.  Provide testing instructions: Run the backend server. Load the extension. Create a card via the modal, click Save. Check the browser console for API status/body logs. Check the backend console logs to see if the request was received and processed. Verify the card appears in the backend state (e.g., by checking `state.getBuckets()` if possible, or adding a temporary GET endpoint).
```

**Step 2.8: Modal Save Logic (Response Handling)**

```text
Context: The API call is being made. Now, handle the success and error responses properly in the UI.

Request:
Modify the `fetch` response handling in the 'Save' button listener (`content.js`):
1.  **Inside the first `.then(response => ...)` block:**
    - Check `response.ok` (which is true for statuses 200-299).
    - **If `response.ok` (specifically check `response.status === 201`):**
        - Parse the JSON body: `response.json().then(data => { ... });`
        - Inside this nested `.then()`:
            - Call `showModalError(null)` to clear any previous errors.
            - Display the success message from `data.message` *within the modal* (e.g., replace form content or show in a dedicated status area).
            - Use `setTimeout(() => { hideModal(); /* TODO: Reset modal state if needed */ }, 2000);` to close the modal after 2 seconds.
    - **Else (if not `response.ok`):**
        - Parse the JSON error body: `response.json().then(errorData => { ... }).catch(() => { ... });`
        - Inside the nested `.then(errorData => ...)`: Call `showModalError(\`Error: ${errorData.message || 'Failed to save card.'}\`);`.
        - Inside the nested `.catch()` (if parsing error body fails): Call `showModalError(\`Error: ${response.statusText || 'Failed to save card.'}\`);`.
        - Re-enable the Save button if it was disabled.
2.  **Inside the main `.catch(error => ...)` block (for network errors):**
    - Ensure `showModalError("Failed to save card. Network error or server down.");` is called.
    - Re-enable the Save button.
3.  Provide testing instructions: Test successful save (verify success message, modal closes). Test save with invalid data that *passes client-side* but might fail backend validation (if applicable - e.g., if backend adds more rules later) or trigger a backend error (verify error shown in modal). Stop the backend server and test save (verify network error shown in modal).
```

---

### Phase 3: Webcam/Gesture Enhancement (Web App)

_(Assumption: You have an existing React frontend with a `PracticeView.tsx` component, `submitAnswer` function, etc.)_

**Step 3.1: Gesture Classification Logic & Unit Tests**

```text
Context: We need to classify hand gestures based on landmarks. Start with the core logic and tests. Assume landmarks are [{x, y, z}, ...].

Request:
1.  Create a new file `src/lib/gestureClassifier.ts` (or similar location).
2.  Define and export an enum `GestureLabel { ThumbsUp = 'ThumbsUp', ThumbsDown = 'ThumbsDown', FlatHand = 'FlatHand', None = 'None' }`.
3.  Define and export a function `classifyGesture(landmarks: Landmark[]): GestureLabel`. (Define a simple `Landmark` type `{x: number, y: number, z: number}` if needed).
4.  Implement **very basic** rules inside `classifyGesture` based on expected thumb/finger tip positions relative to wrist/palm for 'ThumbsUp', 'ThumbsDown', 'FlatHand'. Return `GestureLabel.None` otherwise or if `landmarks` is empty/invalid. *(Example rule for ThumbsUp: Thumb tip Y is significantly lower (higher number in image coords) than finger MCP Ys, and thumb tip X is away from index finger MCP X)*.
5.  Create `src/lib/gestureClassifier.test.ts`.
6.  Create mock landmark data arrays (`MOCK_THUMBS_UP_LANDMARKS`, `MOCK_THUMBS_DOWN_LANDMARKS`, `MOCK_FLAT_HAND_LANDMARKS`, `MOCK_FIST_LANDMARKS` for 'None', `MOCK_EMPTY_LANDMARKS`) - **placeholder data initially, or capture real data later**.
7.  Write unit tests using Jest (`describe`/`it`) for `classifyGesture` asserting the correct `GestureLabel` is returned for each mock data input.
8.  Refine the rules in `classifyGesture` just enough to make the initial tests pass.
```

**Step 3.2: Webcam UI Placeholders & State**

```text
Context: Implement the basic UI elements for the webcam feature in the existing `PracticeView.tsx`.

Request:
Modify `src/components/PracticeView.tsx`:
1.  Add state using `useState`: `const [isWebcamEnabled, setIsWebcamEnabled] = useState(false);`
2.  Add a button element with text "Use Webcam hand detection". Add an `onClick` handler that toggles the `isWebcamEnabled` state: `setIsWebcamEnabled(prev => !prev);`.
3.  Add a `div` element that will serve as the webcam overlay container. Style it using inline styles or CSS classes to be fixed position, bottom-right corner, with defined width/height (e.g., 150x120px), border, and background color.
4.  Conditionally render the overlay `div` based on `isWebcamEnabled`.
5.  Provide testing instructions: Verify the button appears. Click it, verify the placeholder overlay div appears/disappears in the bottom-right corner.
```

**Step 3.3: Webcam Activation & Permissions**

````text
Context: Display the actual webcam feed and handle permissions when the feature is enabled.

Request:
Modify `src/components/PracticeView.tsx`:
1.  Add state for the video stream and permission status:
    `const [videoStream, setVideoStream] = useState<MediaStream | null>(null);`
    `const [permissionStatus, setPermissionStatus] = useState<'idle' | 'pending' | 'granted' | 'denied'>('idle');`
2.  Create a `useRef` for the video element: `const videoRef = useRef<HTMLVideoElement>(null);`
3.  Create an asynchronous function `startWebcam()`:
    - Set `permissionStatus` to 'pending'.
    - Use `navigator.mediaDevices.getUserMedia({ video: true })`.
    - **On success (.then(stream => ...))**:
        - Set `setVideoStream(stream)`.
        - Set `permissionStatus` to 'granted'.
    - **On failure (.catch(err => ...))**:
        - Set `setVideoStream(null)`.
        - Set `permissionStatus` to 'denied'.
        - Log the error.
4.  Create a function `stopWebcam()`:
    - If `videoStream` exists, stop its tracks: `videoStream.getTracks().forEach(track => track.stop());`
    - Set `setVideoStream(null)`.
    - Set `permissionStatus` to 'idle'.
5.  Use `useEffect` to manage the webcam based on `isWebcamEnabled`:
    ```typescript
    useEffect(() => {
      if (isWebcamEnabled) {
        startWebcam();
      } else {
        stopWebcam();
      }
      // Cleanup function to stop webcam if component unmounts while enabled
      return () => {
        stopWebcam();
      };
    }, [isWebcamEnabled]);
    ```
6.  Modify the webcam overlay `div`:
    - If `permissionStatus === 'denied'`, display the "webcam permission denied" message on a grey background inside the overlay.
    - If `permissionStatus === 'granted'` and `videoStream` exists, render a `<video>` element inside the overlay. Set its `ref={videoRef}`, `autoPlay`, `playsInline`, `muted`.
7.  Use another `useEffect` to connect the stream to the video element:
    ```typescript
    useEffect(() => {
      if (videoRef.current && videoStream) {
        videoRef.current.srcObject = videoStream;
      }
    }, [videoStream]);
    ```
8.  Modify the "Use Webcam" button's `onClick` handler to only toggle `isWebcamEnabled`. The `useEffect` hook will handle starting/stopping.
9.  Provide testing instructions: Click enable button, grant permission, verify video feed appears. Click again, verify feed stops. Refresh, click enable, deny permission, verify "permission denied" message appears.
````

**Step 3.4: TFJS Integration & Prediction**

```text
Context: Webcam stream is showing. Now integrate TensorFlow.js and the hand pose model.

Request:
Modify `src/components/PracticeView.tsx`:
1.  Install TFJS dependencies: `npm install @tensorflow/tfjs-core @tensorflow/tfjs-converter @tensorflow/tfjs-backend-webgl @tensorflow-models/hand-pose-detection`
2.  Import necessary TFJS/model modules.
3.  Add state for the hand pose detector: `const [detector, setDetector] = useState<handPoseDetection.HandDetector | null>(null);`
4.  Add state for the detected landmarks: `const [landmarks, setLandmarks] = useState<handPoseDetection.Keypoint[] | null>(null);`
5.  Create an async function `loadModel()`:
    - Call `handPoseDetection.createDetector(handPoseDetection.SupportedModels.MediaPipeHands, { runtime: 'tfjs', modelType: 'lite' /* or 'full' */ });`
    - Set the result using `setDetector()`. Log success/error.
6.  Call `loadModel()` in a `useEffect` hook that runs once on component mount: `useEffect(() => { loadModel(); }, []);`
7.  Create an async function `runHandDetection()`:
    - If `detector` and `videoRef.current` exist and the video is ready (`readyState >= 3`):
        - Call `detector.estimateHands(videoRef.current)`.
        - If hands are detected (`hands.length > 0`), set the landmarks of the first hand: `setLandmarks(hands[0].keypoints);`
        - Else, `setLandmarks(null);`
    - Use `requestAnimationFrame(runHandDetection);` to continuously run detection on the next frame.
8.  Start `runHandDetection()` in a `useEffect` hook when the `detector` is loaded and the `videoStream` is active. Stop the loop (`cancelAnimationFrame`) when the stream stops or detector unloads.
9.  Temporarily render the detected landmarks count or `JSON.stringify(landmarks)` somewhere to verify detection is working.
10. Provide testing instructions: Enable webcam, verify model loads (check console), verify landmark data is being output/updated as you move your hand in front of the camera.
```

**Step 3.5: Connect Classification & Timer Logic**

````text
Context: We have landmarks and the classification function. Now connect them and implement the 3-second hold timer.

Request:
Modify `src/components/PracticeView.tsx`:
1.  Import `classifyGesture` and `GestureLabel` from `gestureClassifier.ts`.
2.  Add state for the currently detected gesture, held gesture, and timer:
    `const [currentGesture, setCurrentGesture] = useState<GestureLabel>(GestureLabel.None);`
    `const [heldGesture, setHeldGesture] = useState<GestureLabel>(GestureLabel.None);`
    `const [gestureStartTime, setGestureStartTime] = useState<number | null>(null);`
3.  Modify the `useEffect` hook where `setLandmarks` is called (or create a new one reacting to `landmarks` changes):
    - If `landmarks` exist, call `const gesture = classifyGesture(landmarks);`.
    - Else, `const gesture = GestureLabel.None;`.
    - Set `setCurrentGesture(gesture);`
4.  Add another `useEffect` hook that runs whenever `currentGesture` changes, to manage the timer:
    ```typescript
    useEffect(() => {
      const HOLD_DURATION_MS = 3000;

      if (currentGesture !== GestureLabel.None && currentGesture === heldGesture) {
        // Gesture continues being held
        if (gestureStartTime && Date.now() - gestureStartTime >= HOLD_DURATION_MS) {
          // *** GESTURE CONFIRMED ***
          console.log("Held gesture confirmed:", currentGesture);
          // Trigger action (will be done in next step)
          // Reset timer state
          setHeldGesture(GestureLabel.None);
          setGestureStartTime(null);
        }
        // else, still holding, do nothing yet
      } else if (currentGesture !== GestureLabel.None && currentGesture !== heldGesture) {
        // New valid gesture detected, start timer
        setHeldGesture(currentGesture);
        setGestureStartTime(Date.now());
      } else {
        // No valid gesture, or gesture changed to None
        setHeldGesture(GestureLabel.None);
        setGestureStartTime(null);
      }
    }, [currentGesture]); // Dependency array includes currentGesture, heldGesture, gestureStartTime
    ```
    *(Self-correction: Need to refine the dependency array and logic slightly for robustness, maybe using refs for timers is better than state for triggering effects.)*
    **Alternative approach using refs for timer:**
    ```typescript
    const gestureTimerRef = useRef<NodeJS.Timeout | null>(null);
    const currentHeldGestureRef = useRef<GestureLabel>(GestureLabel.None);

    useEffect(() => {
        const HOLD_DURATION_MS = 3000;

        // Clear previous timer if gesture changes
        if (gestureTimerRef.current && currentGesture !== currentHeldGestureRef.current) {
            clearTimeout(gestureTimerRef.current);
            gestureTimerRef.current = null;
            currentHeldGestureRef.current = GestureLabel.None;
        }

        // Start a new timer if a valid gesture is detected and no timer is running
        if (currentGesture !== GestureLabel.None && !gestureTimerRef.current) {
            currentHeldGestureRef.current = currentGesture; // Track which gesture started the timer
            gestureTimerRef.current = setTimeout(() => {
                console.log("Held gesture confirmed:", currentHeldGestureRef.current); // Log the gesture that finished
                // Trigger Action (next step)
                // Reset timer state implicitly as it has fired
                gestureTimerRef.current = null;
                currentHeldGestureRef.current = GestureLabel.None;
                 // Might need to force reset currentGesture state if landmarks don't update immediately
            }, HOLD_DURATION_MS);
        }

        // Cleanup timer on unmount or when webcam disabled
        return () => {
            if (gestureTimerRef.current) {
                clearTimeout(gestureTimerRef.current);
            }
        };
    }, [currentGesture]);
    ```
5.  Provide testing instructions: Enable webcam, make gestures (Thumbs Up/Down/Flat Hand), hold them. Check console logs to see if the "Held gesture confirmed" message appears after approx. 3 seconds of holding the *same* gesture. Verify changing gesture or removing hand resets the process.
````

**Step 3.6: Trigger Action & Visual Feedback**

```text
Context: The 3-second hold is detected. Now trigger the flashcard action and add visual feedback.

Request:
Modify `src/components/PracticeView.tsx`:
1.  Import `AnswerDifficulty` enum if not already done.
2.  Add state for visual feedback: `const [feedbackColor, setFeedbackColor] = useState<string | null>(null);`
3.  Modify the logic where the `console.log("Held gesture confirmed...")` happens (inside the `useEffect` or timer callback):
    - Determine the `difficulty: AnswerDifficulty` based on the confirmed `heldGesture` (e.g., `ThumbsUp` -> `Easy`, `ThumbsDown` -> `Wrong`, `FlatHand` -> `Hard`).
    - Call the existing `submitAnswer(currentCard.front, currentCard.back, difficulty);` function (ensure `currentCard` is accessible).
    - Determine the feedback border color (`green`, `red`, `orange`) based on the gesture.
    - Set `setFeedbackColor(color);`
    - Use `setTimeout(() => setFeedbackColor(null), 1000);` to clear the feedback color after 1 second. *(Spec adjustment: Use a 1s flash AFTER completion, not the fill animation itself as primary feedback)*.
4.  Modify the webcam overlay `div`: Add dynamic styling (e.g., `style={{ borderColor: feedbackColor || 'initialColor', borderWidth: feedbackColor ? '3px' : '1px' }}`) to show the colored border flash when `feedbackColor` is set.
5.  **(Optional Refinement - Progressive Fill):** Implement the progressive border fill feedback *during* the hold. This is more complex, potentially involving CSS animations triggered by state changes tracking hold progress, or calculating border gradients dynamically within the timer loop. Start with the simple 1s flash first.
6.  Provide testing instructions: Enable webcam, hold a gesture for 3s. Verify the correct difficulty action (`submitAnswer`) is triggered (check app behavior/network requests). Verify the overlay border flashes the correct color (green/red/orange) for 1 second.
```

---

### Phase 4: Auxiliary Extension Components & Finalization

**Step 4.1: Background Script Implementation**

````text
Context: Implement the minimal background script as specified.

Request:
Modify `background.js`:
1.  Ensure it contains the following code:
    ```javascript
    chrome.runtime.onInstalled.addListener(() => {
      console.log("[Background] Flashcard Extension installed/updated.");
      // You could potentially set initial extension state in chrome.storage here if needed later.
    });

    // Add other listeners if needed (e.g., for messages, alarms) - none required by current spec.
    console.log("[Background] Service worker started.");
    ```
2.  Verify it logs correctly when the extension is installed/updated or the service worker starts.
````

**Step 4.2: Action Popup Implementation**

```text
Context: Implement the simple toolbar action popup.

Request:
1.  Modify `popup.html`:
    - Set up basic HTML structure.
    - Add elements to display: `<h1>Flashcard Extension</h1>`, `<p>Version: 1.0</p>`, `<button id="openAppBtn">Open Practice App</button>`.
    - Link to `popup.css` (create if needed for styling) and `popup.js`.
2.  Modify `popup.js`:
    - Add a DOMContentLoaded event listener.
    - Inside, get the button by ID (`openAppBtn`).
    - Add a click event listener to the button.
    - Inside the click listener, use `chrome.tabs.create({ url: 'http://localhost:5173' });` (Replace port 5173 with the actual port of your practice web app).
3.  (Optional) Add basic styling in `popup.css`.
4.  Provide testing instructions: Load the extension. Click the extension icon in the toolbar. Verify the popup shows the title, version, and button. Click the button and verify it opens the practice web app in a new tab.
```

**Step 4.3: Documentation File**

```text
Context: Consolidate AF/RI/SRE documentation into a separate file.

Request:
Create a file named `DOCUMENTATION.md` in the project root (or a `/docs` folder).
Summarize the Abstract Function (AF), Representation Invariant (RI), and Safety from Representation Exposure (SRE) points previously documented in TSDoc comments for:
- `Flashcard` class
- `state.ts` module
- `algorithm.ts` module
Structure the file clearly with headings for each component.
```

**Step 4.4: Manual Testing**

```text
Context: All features are implemented. Perform thorough manual testing based on the specification.

Request:
Execute the detailed manual test plans defined in the specification document (Section 5) for:
1.  The Browser Extension Frontend (E2E workflow).
2.  The Webcam/Gesture Feature.
Document any bugs or deviations found.
```

**Step 4.5: Code Review & Refinement**

```text
Context: Final checks before completion.

Request:
1.  Review all code for clarity, consistency, and adherence to best practices.
2.  Refactor any complex sections if needed.
3.  Review and minimize permissions requested in `manifest.json`.
4.  Ensure all console logs used for debugging are removed or placed behind a development flag.
5.  Finalize all documentation (in-code and `DOCUMENTATION.md`).
6.  Address any bugs found during manual testing.
```

---

This detailed breakdown provides a step-by-step guide, integrating testing and documentation, suitable for generating specific, actionable prompts for a code-generation LLM. Each step builds incrementally on the previous ones.
