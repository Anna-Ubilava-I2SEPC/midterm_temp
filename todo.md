Okay, here is a detailed `todo.md` checklist based on the step-by-step implementation plan. You can copy this into a `todo.md` file in your project repository and check items off as you complete them.

```markdown
# Project TODO Checklist: Flashcard Extension & Webcam Practice

This checklist follows the iterative implementation plan. Mark items as complete using `- [x]`.

## Phase 1: Backend API (`POST /api/cards`)

- [ ] **Step 1.1:** Implement `validateCardPayload` function in `backend/src/utils/validation.ts`.
- [ ] **Step 1.1:** Write unit tests for `validateCardPayload` in `validation.test.ts`.
- [ ] **Step 1.2:** Implement `addCardToBucket0` function in `backend/src/state.ts`.
- [ ] **Step 1.2:** Write unit tests for `addCardToBucket0` in `state.test.ts`.
- [ ] **Step 1.3:** Implement `POST /api/cards` route handler in `server.ts` (integrating validation and state update).
- [ ] **Step 1.4:** Write integration tests for `POST /api/cards` endpoint (`api.test.ts`).
- [ ] **Step 1.5:** Add TSDoc (AF/RI/SRE) comments to `Flashcard` class (`flashcards.ts`).
- [ ] **Step 1.5:** Add TSDoc (AF/RI/SRE) comments to `state.ts` module.
- [ ] **Step 1.5:** Add TSDoc (AF/RI/SRE) comments to `algorithm.ts` module.

## Phase 2: Core Extension Functionality (Content Script & Modal)

- [ ] **Step 2.1:** Create basic extension file structure (`manifest.json`, empty scripts/HTML).
- [ ] **Step 2.1:** Verify basic extension loading in Chrome.
- [ ] **Step 2.2:** Implement text selection detection logic in `content.js` (check length, not in input, store text).
- [ ] **Step 2.3:** Implement inline "+ add flashcard" button creation, positioning, and show/hide logic in `content.js`.
- [ ] **Step 2.3:** Add basic button styling in `content.css`.
- [ ] **Step 2.4:** Implement modal DOM structure creation in `content.js`.
- [ ] **Step 2.4:** Implement `showModal` function (pre-filling 'Back' field) in `content.js`.
- [ ] **Step 2.5:** Add comprehensive modal styling in `content.css`.
- [ ] **Step 2.5:** Implement 'Cancel' and 'X' button close logic for the modal in `content.js`.
- [ ] **Step 2.6:** Implement client-side 'Save' logic: validation (Front/Back required) in `content.js`.
- [ ] **Step 2.6:** Implement tag parsing logic (`parseTags`) in `content.js`.
- [ ] **Step 2.6:** Implement modal error display function (`showModalError`) in `content.js`.
- [ ] **Step 2.7:** Integrate `fetch` call to `POST /api/cards` on modal save in `content.js`.
- [ ] **Step 2.8:** Implement API success response handling (show message, auto-close modal after 2s) in `content.js`.
- [ ] **Step 2.8:** Implement API error response handling (show error message in modal) in `content.js`.

## Phase 3: Webcam/Gesture Enhancement (Web App)

- [ ] **Step 3.1:** Implement `classifyGesture` function with basic rules in `src/lib/gestureClassifier.ts`.
- [ ] **Step 3.1:** Create mock landmark data for testing gestures.
- [ ] **Step 3.1:** Write unit tests for `classifyGesture` (`gestureClassifier.test.ts`).
- [ ] **Step 3.2:** Add "Use Webcam" button and overlay placeholder `div` to `PracticeView.tsx`.
- [ ] **Step 3.2:** Add basic React state management (`useState`) for enabling/disabling webcam UI.
- [ ] **Step 3.3:** Implement webcam activation (`getUserMedia`) and stream display in the video element (`PracticeView.tsx`).
- [ ] **Step 3.3:** Implement permission handling (denied state/message) for webcam (`PracticeView.tsx`).
- [ ] **Step 3.3:** Implement webcam stop/cleanup logic using `useEffect` (`PracticeView.tsx`).
- [ ] **Step 3.4:** Install TensorFlow.js and hand-pose-detection dependencies.
- [ ] **Step 3.4:** Implement TFJS model loading (`MediaPipeHands`) in `PracticeView.tsx`.
- [ ] **Step 3.4:** Implement hand landmark prediction loop using `requestAnimationFrame` (`PracticeView.tsx`).
- [ ] **Step 3.5:** Connect landmark predictions to `classifyGesture` function (`PracticeView.tsx`).
- [ ] **Step 3.5:** Implement 3-second consistent gesture hold detection logic/timer (`PracticeView.tsx`).
- [ ] **Step 3.6:** Trigger `submitAnswer` action based on confirmed held gesture (`PracticeView.tsx`).
- [ ] **Step 3.6:** Implement visual feedback (1-second border flash) on gesture confirmation (`PracticeView.tsx`).

## Phase 4: Auxiliary Extension Components & Finalization

- [ ] **Step 4.1:** Implement minimal `background.js` structure and `onInstalled` listener.
- [ ] **Step 4.2:** Implement `popup.html` layout (title, version, button).
- [ ] **Step 4.2:** Implement `popup.js` functionality (button click opens practice app tab).
- [ ] **Step 4.3:** Create/Update `DOCUMENTATION.md` summarizing AF/RI/SRE from code comments.
- [ ] **Step 4.4:** Execute full Manual E2E Test Plan for the Browser Extension.
- [ ] **Step 4.4:** Execute full Manual Test Plan for the Webcam/Gesture Feature.
- [ ] **Step 4.4:** Document any bugs found during manual testing.
- [ ] **Step 4.5:** Perform final code review and refactoring.
- [ ] **Step 4.5:** Review and minimize extension permissions in `manifest.json`.
- [ ] **Step 4.5:** Remove debugging console logs (or place behind dev flags).
- [ ] **Step 4.5:** Finalize all documentation (in-code and `DOCUMENTATION.md`).
- [ ] **Step 4.5:** Fix any outstanding bugs found during testing.
```
