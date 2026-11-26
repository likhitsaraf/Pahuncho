# Gramin Go — Product Requirements (v1)

## Vision
A distinctive, hyperlocal ride-booking app for tier-2/3 Indian cities (starting Bankura, WB). Warm, tactile claymorphism UI, deep-indigo + terracotta + mint palette; deliberately different from Uber/Ola/Rapido.

## Rider MVP (this iteration)
- **Splash** with animated stylized town + brand mark (auto-advances to Home).
- **Home** with an SVG isometric mini-city (moving auto-rickshaw + bus), live demand strip, EN / हि / বাং language toggle, voice search for pickup / drop, quick-place chips.
- **Vehicle Selection** with a circular floating orbit of 5 vehicles (e-rickshaw, shared auto, bike taxi, car rental, local bus), ETA / fare / seats / availability stats.
- **Fare** screen with tactile drag slider tying price ↔ wait time and floating bubble stats.
- **Booking** with press-&-hold confirmation ring, progressive haptics, backend ride creation.
- **Tracking** with real react-native-maps on iOS/Android (dashed indigo route line, Bankura coords) and an SVG illustrated fallback on Web; live driver card + call button.
- **Completion** with celebratory bounce, 5-star rating and tip chips (₹0 / 10 / 20 / 50), rating persists via `/api/rides/{id}/rate`.

## Backend
FastAPI (`/api` prefix), MongoDB.
- `GET /api/vehicles` seed catalog
- `GET /api/demand` playful mock counts
- `POST /api/fare/estimate` min/recommended/max fare + ETA
- `POST /api/rides`, `GET /api/rides/{id}`, `POST /api/rides/{id}/rate`
- `POST /api/stt/transcribe` — voice booking through Emergent LLM key + OpenAI Whisper (`emergentintegrations.llm.openai.speech_to_text.OpenAISpeechToText`).

## Integrations
- **Emergent LLM key** ➜ OpenAI Whisper for STT (Hindi/Bengali/English).
- **react-native-maps** for live tracking (native only).
- **expo-audio** for hold-to-record voice input.
- **expo-haptics** for tap-hold booking, star rating, mic feedback.

## Design tokens (see `/app/design_guidelines.json`, mirrored in `frontend/src/theme.ts`)
Deep Indigo `#3B1F85`, Terracotta `#D84315`, Mint `#00BFA5`, Off-white `#F6F4EF`. Radius tokens 12 / 20 / 32 / pill. Custom claymorphism shadow presets.

## Growth hook / smart business enhancement
The **Smart Fare Slider** doubles as a demand-priced marketplace: riders self-select their price vs. wait tolerance, allowing the platform to charge a small transparent surcharge on "instant" rides (right of slider) and route slow rides to shared autos — a hyperlocal earnings booster with no algorithmic surge blame.

## Changelog
- **Expo SDK 57 → 58 upgrade:** expo 58, react-native 0.88.1, react 19.3.0. All Expo/RN packages aligned with `expo install --fix`, and the `expo-asset` and `@expo/log-box` peers were added. In `src/theme.ts`, the removed `StyleSheet.NamedStyles` type was swapped for a local type (needed for RN 0.88's strict TypeScript API). expo-doctor passes 20/20 and tsc is clean. No UI, feature, backend or database changes.

## Backlog
- Use `router.dismissTo('/home')` after completion, make touch targets at least 44pt, fix the overlapping home hint, show an error state when the fare request fails, and validate rating and fare inputs on the backend.
- Migrate the fare slider to the gesture-handler v3 hook API (the legacy API still works but is deprecated).
