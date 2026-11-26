# Pahuncho — Frontend ↔ Backend Integration Contracts

These are **frontend integration contracts only**. The backend (Python + FastAPI, PostgreSQL on Neon, hosted on Railway) is built separately. Paths and fields must be checked against the backend's OpenAPI spec. Any change only needs to be made in `frontend/src/services/live.ts` and `frontend/src/types.ts`.

## Configuration (`frontend/.env`, public `EXPO_PUBLIC_*` values only, never secrets)
| Variable | Purpose |
|---|---|
| `EXPO_PUBLIC_PAHUNCHO_API_URL` | REST base, e.g. `https://api.pahuncho.com` |
| `EXPO_PUBLIC_PAHUNCHO_WS_URL` | WebSocket base, e.g. `wss://api.pahuncho.com/ws` |
| `EXPO_PUBLIC_DEMO_MODE` | `true` = clearly labelled sample data from `src/mocks/demo.ts`; set `false` for the real backend |
| `EXPO_PUBLIC_VOICE_ENABLED` | Shows the voice-booking mic once `/v1/voice/transcribe` exists |
| `EXPO_PUBLIC_GOOGLE_MAPS_API_KEY` | Public, platform-restricted Maps/Places key (optional until provided) |

## Auth
- All authenticated calls send `Authorization: Bearer <access_token>`.
- On a `401`, the client calls `POST /v1/auth/refresh {refresh_token}` once, then retries the request.
- Tokens are stored in Keychain/Keystore (`storage.secure*`).

| Method & path | Request | Response |
|---|---|---|
| POST `/v1/auth/otp/request` | `{phone:"+91XXXXXXXXXX", role:"passenger"\|"driver"}` | `{request_id, resend_after_sec}` |
| POST `/v1/auth/otp/verify` | `{phone, code, request_id, role}` | `{access_token, refresh_token, user:User, is_new_user}` |
| POST `/v1/auth/refresh` | `{refresh_token}` | `{access_token, refresh_token?}` |
| POST `/v1/auth/logout` | – | 204 |
| GET `/v1/me` | – | `User {id, role, phone, name?, email?, referral_code?}` |
| POST `/v1/me/register` | `{name, email?, referral_code?}` | `User` |
| PATCH `/v1/me` | `{name?, email?}` | `User` |

## Catalog, places and service area
| Method & path | Request | Response |
|---|---|---|
| GET `/v1/vehicle-categories` | – | `VehicleCategory[]` `{id, code:"toto"\|"shared_auto"\|"car"\|"bike"\|"bus", name, capacity, enabled, modes:[{mode:"private"\|"shared", enabled}]}`. Disabled categories and modes are shown as "Coming soon" and can't be booked. |
| GET `/v1/places/nearby?lat&lng` | – | `Place[] {id, name, address?, lat, lng, kind?}` |
| GET `/v1/places/search?q&lat&lng` | – | `Place[]` (backend proxies Places Autocomplete) |
| POST `/v1/service-area/check` | `{lat, lng}` | `{serviceable, city?, message?}` (Kolkata excluded) |
| GET `/v1/rentals/services` | – | `RentalService[] {id, type:"car"\|"truck"\|"bike", title, description, enabled}` |
| POST `/v1/rentals/inquiries` | `{service_id, pickup, start_at, duration, notes?}` | `{inquiry_id, message}` |
| GET `/v1/support` | – | `{phone?, whatsapp?, email?, faqs:[{q,a}]}` |
| GET `/v1/content/{privacy\|terms}` | – | `{title, updated_at, body}` |

## Rides (passenger)
| Method & path | Request | Response |
|---|---|---|
| POST `/v1/fares/estimate` | `{pickup:Place, drop:Place, category_id, mode, scheduled_for?}` | `FareEstimate {estimate_id, distance_km, duration_min, pickup_eta_min, fare, fare_min?, fare_max?, currency, expires_at}` |
| POST `/v1/rides` | `{estimate_id, pickup, drop, category_id, mode, scheduled_for?, coupon_code?, payment_method:"cash", selected_fare?}` | `Ride` (`status:"searching"` or `"scheduled"`) |
| GET `/v1/rides/{id}` | – | `Ride` |
| GET `/v1/rides?bucket=current\|upcoming\|completed\|cancelled` | – | `Ride[]` |
| GET `/v1/rides/cancel-reasons` | – | `[{id,label}]` |
| GET `/v1/rides/{id}/cancel-preview` | – | `{eligible, charge, message?}` |
| POST `/v1/rides/{id}/cancel` | `{reason_id, note?}` | `Ride` |
| POST `/v1/rides/{id}/rating` | `{stars, comment?, tip?}` | `{ok}` |
| GET `/v1/rides/{id}/receipt` | – | `{booking_code, lines:[{label,amount}], total, payment_status, issued_at}` |

`Ride` = `{id, booking_code, status, pickup, drop, category_id, category_name?, mode, scheduled_for?, estimated_fare, final_fare?, fare_breakdown?, payment_status:"pending"|"paid"|"failed"|"refunded", payment_method, driver?:{id,name,phone,whatsapp?,vehicle_number,vehicle_model?,rating?,eta_min?,location?}, pickup_otp?, distance_km?, created_at, cancel_reason?, cancellation_charge?}`

**Ride status flow:** `searching → driver_assigned → driver_arriving → driver_arrived (pickup_otp visible to passenger) → in_progress (after the driver's OTP is verified) → completed`. A ride can also end as `cancelled` or `no_driver_found`; scheduled rides start as `scheduled`.

## Real-time
- Passenger: `WS {WS_URL}/rides/{ride_id}?token=<access>` sends JSON events:
  - `{"type":"ride.status","ride":Ride}`
  - `{"type":"driver.location","location":{lat,lng,heading?,updated_at}}`

  If the socket fails, the client polls `GET /v1/rides/{id}` every 5 s.
- Driver: `WS {WS_URL}/driver/location?token=<access>`. The client sends `{lat,lng,at}` while online or on a trip.

## Promotions
| Method & path | Request | Response |
|---|---|---|
| GET `/v1/offers` | – | `Offer[] {id, code, title, description, discount_text, valid_until, terms?}` |
| POST `/v1/coupons/validate` | `{code, estimate_id?}` | `{valid, discount, message}` |
| GET `/v1/referrals/me` | – | `{code, share_message, total_rewards, history:[{id,name,status,reward,date}]}` |

## Driver
| Method & path | Request | Response |
|---|---|---|
| GET `/v1/driver/me` | – | `DriverProfile {id, name?, phone, status:"not_registered"\|"pending_documents"\|"under_review"\|"approved"\|"rejected"\|"suspended", rejection_reason?, city?, vehicle?, documents:[{type,status,url?,reason?}], is_online, rating?}` |
| POST `/v1/driver/register` | `{name, city, address, emergency_contact, vehicle:{type, number, model?, color?}}` | `DriverProfile` |
| POST `/v1/driver/documents` (multipart) | `file`, `doc_type` = `driving_license\|aadhaar\|vehicle_rc\|insurance\|permit\|profile_photo` | `DriverDocument` |
| POST `/v1/driver/kyc/submit` | – | `DriverProfile` (`under_review`) |
| POST `/v1/driver/availability` | `{online, lat?, lng?}` | `DriverProfile`. Only `approved` drivers may go online. |
| GET `/v1/driver/ride-requests` | – | `RideRequest[] {id, ride_id, pickup, drop, distance_km, estimated_fare, pickup_distance_km, passenger_name, expires_at}` |
| POST `/v1/driver/ride-requests/{id}/accept` / `/reject` | – | `DriverRide` / `{ok}` |
| GET `/v1/driver/rides/active` | – | `DriverRide \| null` |
| POST `/v1/driver/rides/{id}/arrived` | – | `DriverRide` |
| POST `/v1/driver/rides/{id}/start` | `{otp}` | `DriverRide` (`in_progress`), or 400 if the OTP is wrong |
| POST `/v1/driver/rides/{id}/complete` | – | `DriverRide` with `final_fare` |
| GET `/v1/driver/rides` | – | `DriverRide[]` |
| GET `/v1/driver/earnings` | – | `{today, week, month, trips_today, commission_rate, commission_amount, net_payable, payouts:[{id,amount,status,date,reference?}]}` |

`DriverRide` = `Ride` + `passenger:{name, phone, whatsapp?}`

## Voice (optional)
- POST `/v1/voice/transcribe`, multipart `file`, `field`, `language` → `{transcript}`

## Errors
- Non-2xx responses should return `{detail: string}`. The app shows it to the user as is.
