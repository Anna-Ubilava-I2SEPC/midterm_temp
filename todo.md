# Project TODO Checklist: Flashcard Extension & Webcam Practice

This checklist follows the iterative implementation plan, emphasizing writing tests before implementation where specified. Mark items as complete using `- [x]`.

## Phase 1: Backend API (`POST /api/cards`)

- [ ] **Step 1.1 (Validation - Test First):** Define function signature/stub for `validateCardPayload` in `backend/src/utils/validation.ts`.
- [ ] **Step 1.1 (Validation - Test):** Write unit tests for `validateCardPayload` in `validation.test.ts` (expect tests to fail).
- [ ] **Step 1.1 (Validation - Implement):** Implement logic inside `validateCardPayload` to make unit tests pass.
- [ ] **Step 1.2 (State Update - Test First):** Define function signature/stub for `addCardToBucket0` in `backend/src/state.ts`.
- [ ] **Step 1.2 (State Update - Test):** Write unit tests for `addCardToBucket0` in `state.test.ts` (expect tests to fail).
- [ ] **Step 1.2 (State Update - Implement):** Implement logic inside `addCardToBucket0` to make unit tests pass.
- [ ] **Step 1.3 & 1.4 (API Endpoint - Integration Tests First):** Write integration tests for `POST /api/cards` (`api.test.ts`), covering both success (201) and validation failure (400) cases (expect tests to fail).
- [ ] **Step 1.3 & 1.4 (API Endpoint - Implement):** Implement the `POST /api/cards` route handler in `server.ts`, integrating the already-tested `validateCardPayload` and `addCardToBucket0` functions, to make the integration tests pass. Implement necessary error handling (400, 500) and success response (201).
- [ ] **Step 1.5 (Documentation):** Add TSDoc (AF/RI/SRE) comments to `Flashcard` class (`flashcards.ts`).
- [ ] **Step 1.5 (Documentation):** Add TSDoc (AF/RI/SRE) comments to `state.ts` module.
- [ ] **Step 1.5 (Documentation):** Add TSDoc (AF/RI/SRE) comments to `algorithm.ts` module.

## Phase 2: Core Extension Functionality (Content Script & Modal)

- [ ] **Step 2.1:** Create basic extension file structure (`manifest.json`, empty scripts/HTML).
- [ ] **Step 2.1:** Manually verify basic extension loading in Chrome.
- [ ] **Step 2.2:** Implement text selection detection logic in `content.js` (check length, not in input, store text).
- [ ] **Step 2.2:** Manually verify selection detection via console logs.
- [ ] **Step 2.3:** Implement inline "+ add flashcard" button creation, positioning, and show/hide logic in `content.js`.
- [ ] **Step 2.3:** Add basic button styling in `content.css`.
- [ ] **Step 2.3:** Manually verify button appearance/disappearance based on selection.
- [ ] **Step 2.4:** Implement modal DOM structure creation in `content.js`.
- [ ] **Step 2.4:** Implement `showModal` function (pre-filling 'Back' field) in `content.js`.
- [ ] **Step 2.4:** Manually verify modal appears with 'Back' pre-filled on button click.
- [ ] **Step 2.5:** Add comprehensive modal styling in `content.css`.
- [ ] **Step 2.5:** Implement 'Cancel' and 'X' button close logic for the modal in `content.js`.
- [ ] **Step 2.5:** Manually verify modal styling and close functionality.
- [ ] **Step 2.6 (Tag Parsing - TDD):** Define function signature/stub for `parseTags` in `content.js`.
- [ ] **Step 2.6 (Tag Parsing - TDD):** Write unit tests for `parseTags` (if feasible to unit test content script logic, otherwise manually define test cases).
- [ ] **Step 2.6 (Tag Parsing - TDD):** Implement `parseTags` logic to pass tests/meet test cases.
- [ ] **Step 2.6 (Modal Save Logic):** Implement modal error display function (`showModalError`) in `content.js`.
- [ ] **Step 2.6 (Modal Save Logic):** Implement client-side 'Save' logic: validation (Front/Back required) using `showModalError` in `content.js`.
- [ ] **Step 2.6 (Modal Save Logic):** Manually verify client-side validation and error display. Manually verify tag parsing output via logs.
- [ ] **Step 2.7 (API Integration):** Integrate `fetch` call to `POST /api/cards` on modal save in `content.js`.
- [ ] **Step 2.7 (API Integration):** Manually verify API call is made correctly (check network tab, backend logs).
- [ ] **Step 2.8 (Response Handling):** Implement API success response handling (show message, auto-close modal after 2s) in `content.js`.
- [ ] **Step 2.8 (Response Handling):** Implement API error response handling (show error message in modal) in `content.js`.
- [ ] **Step 2.8 (Response Handling):** Manually verify success and error handling UI behavior.

## Phase 3: Webcam/Gesture Enhancement (Web App)

- [ ] **Step 3.1 (Gesture Classifier - TDD):** Define `classifyGesture` stub & `GestureLabel` enum in `src/lib/gestureClassifier.ts`.
- [ ] **Step 3.1 (Gesture Classifier - TDD):** Create mock landmark data for testing gestures.
- [ ] **Step 3.1 (Gesture Classifier - TDD):** Write unit tests for `classifyGesture` (`gestureClassifier.test.ts`) (expect tests to fail).
- [ ] **Step 3.1 (Gesture Classifier - TDD):** Implement `classifyGesture` logic to make unit tests pass.
- [ ] **Step 3.2 (UI Placeholders):** Add "Use Webcam" button and overlay placeholder `div` to `PracticeView.tsx`.
- [ ] **Step 3.2 (UI Placeholders):** Add basic React state management (`useState`) for enabling/disabling webcam UI.
- [ ] **Step 3.2 (UI Placeholders):** Manually verify button and overlay appearance/disappearance.
- [ ] **Step 3.3 (Webcam Activation):** Implement webcam activation (`getUserMedia`) and stream display in the video element (`PracticeView.tsx`).
- [ ] **Step 3.3 (Webcam Activation):** Implement permission handling (denied state/message) for webcam (`PracticeView.tsx`).
- [ ] **Step 3.3 (Webcam Activation):** Implement webcam stop/cleanup logic using `useEffect` (`PracticeView.tsx`).
- [ ] **Step 3.3 (Webcam Activation):** Manually verify webcam feed display and permission denial message.
- [ ] **Step 3.4 (TFJS Integration):** Install TensorFlow.js and hand-pose-detection dependencies.
- [ ] **Step 3.4 (TFJS Integration):** Implement TFJS model loading (`MediaPipeHands`) in `PracticeView.tsx`.
- [ ] **Step 3.4 (TFJS Integration):** Implement hand landmark prediction loop using `requestAnimationFrame` (`PracticeView.tsx`).
- [ ] **Step 3.4 (TFJS Integration):** Manually verify model loads and landmark data is outputted via console logs.
- [ ] **Step 3.5 (Timer Logic):** Connect landmark predictions to the tested `classifyGesture` function (`PracticeView.tsx`).
- [ ] **Step 3.5 (Timer Logic):** Implement 3-second consistent gesture hold detection logic/timer (`PracticeView.tsx`).
- [ ] **Step 3.5 (Timer Logic):** Manually verify gesture confirmation logs appear after 3-second hold via console logs.
- [ ] **Step 3.6 (Action & Feedback):** Trigger `submitAnswer` action based on confirmed held gesture (`PracticeView.tsx`).
- [ ] **Step 3.6 (Action & Feedback):** Implement visual feedback (1-second border flash) on gesture confirmation (`PracticeView.tsx`).
- [ ] **Step 3.6 (Action & Feedback):** Manually verify flashcard action is triggered and visual feedback occurs correctly.

## Phase 4: Auxiliary Extension Components & Finalization

- [ ] **Step 4.1 (Background Script):** Implement minimal `background.js` structure and `onInstalled` listener.
- [ ] **Step 4.1 (Background Script):** Manually verify background script logs on install/update.
- [ ] **Step 4.2 (Action Popup):** Implement `popup.html` layout (title, version, button).
- [ ] **Step 4.2 (Action Popup):** Implement `popup.js` functionality (button click opens practice app tab).
- [ ] **Step 4.2 (Action Popup):** Manually verify popup UI and button functionality.
- [ ] **Step 4.3 (Documentation File):** Create/Update `DOCUMENTATION.md` summarizing AF/RI/SRE from code comments.
- [ ] **Step 4.4 (Manual Testing):** Execute full Manual E2E Test Plan for the Browser Extension.
- [ ] **Step 4.4 (Manual Testing):** Execute full Manual Test Plan for the Webcam/Gesture Feature.
- [ ] **Step 4.4 (Manual Testing):** Document any bugs found during manual testing.
- [ ] **Step 4.5 (Review & Refinement):** Perform final code review and refactoring.
- [ ] **Step 4.5 (Review & Refinement):** Review and minimize extension permissions in `manifest.json`.
- [ ] **Step 4.5 (Review & Refinement):** Remove debugging console logs (or place behind dev flags).
- [ ] **Step 4.5 (Review & Refinement):** Finalize all documentation (in-code and `DOCUMENTATION.md`).
- [ ] **Step 4.5 (Review & Refinement):** Fix any outstanding bugs found during testing.
