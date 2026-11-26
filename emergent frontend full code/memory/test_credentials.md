# Test Credentials — Pahuncho frontend (DEMO mode)

The app runs with `EXPO_PUBLIC_DEMO_MODE=true`, so all data is sample data from `frontend/src/mocks/demo.ts`. There is no real backend yet.

- **Passenger login:** any valid Indian mobile number, e.g. `9876543210`. The OTP is **123456**. A new number is sent to the register screen (enter a name).
- **Driver login:** Welcome → "Become a Toto Driver" → Register as driver. Use any number, e.g. `9123456780`, with OTP **123456**. Then:
  1. Fill in registration. The vehicle number format is `WB 67 A 1234`.
  2. Upload all 6 documents.
  3. Submit, then tap "DEMO ONLY · simulate admin approval" to reach the dashboard.
- **Driver trip:**
  1. Go online.
  2. A sample request appears after about 4 s. Accept it.
  3. Tap "I've arrived".
  4. Enter the rider OTP **4821**, then tap Complete.
- **Passenger live ride (demo script):**
  1. Searching takes about 3.5 s, then the driver is assigned.
  2. The driver moves for about 9 s and arrives; the pickup OTP 4821 is shown.
  3. After about 6 s the ride starts (in_progress).
  4. After about 9 s it is completed and the app redirects to the completion screen.
- **Coupons:** `PAHUNCHO10`, `GREEN5`
- **Out-of-area test:** search "Kolkata" or "Howrah". Selecting it shows the not-serviceable message.
