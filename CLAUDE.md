# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ODON Booking is a Flutter hotel management app with a Node.js/Express + MongoDB backend. It handles room bookings, expenses, salaries, inventory, invoices, and AI-driven business insights.

Each dart file is one screen in the app. The app runs on both Android and web (`flutter run -d chrome`).
## Commands

### Flutter (Frontend)

```bash
flutter run                  # Run on connected device/emulator
flutter run -d web           # Run in browser
flutter build apk            # Build Android APK
flutter build ios            # Build iOS
flutter test                 # Run all tests
flutter analyze              # Static analysis
flutter clean                # Clean build artifacts
```

### Backend (`flutter_mongodb_backend/`)

```bash
npm install                  # Install dependencies
npm start                    # Start Express server (port 3000)
```

## Architecture

### Frontend (`lib/`)

Flat structure — all screens live directly in `lib/` with no subdirectories except `lib/models/`.

**Entry point**: `lib/main.dart` → `HomeScreen` (`home_screen.dart`)

**API layer**: `lib/api_service.dart` — all HTTP calls go through this single service. Currently pointing to `localhost:3000` for local dev. Switch back to the Railway URL after rehosting. For a physical Android device use the machine's local IP (e.g. `http://192.168.1.26:3000`), for an emulator use `http://10.0.2.2:3000`.

**State management**: Plain `StatefulWidget` + `setState()` — no Provider, BLoC, or Riverpod.

**Feature areas**:

- Bookings: `room_selection_screen.dart`, `view_bookings_screen.dart`, `edit_booking_screen.dart`, `past_bookings_screen.dart`, `future_bookings_screen.dart`, `selected_day_booking.dart`
- Financials: `calculate_profit_page.dart`, `expenses_screen.dart`, `ViewEditSalariesExpensesScreen.dart`
- Invoices: `generate_invoice_screen.dart`, `invoice.dart`, `price_settings_screen.dart`
- Inventory: `add_inventory_item_screen.dart`, `edit_inventory_item_screen.dart`
- AI insights: `ai_insights_page.dart` + `lib/models/ai_insights_service.dart`
- Auth: `login_screen.dart` (hardcoded credentials, no backend auth)
- Web/mobile PDF saving: `file_saver.dart` (conditional export) → `file_saver_web.dart` (opens PDF in new browser tab) / `file_saver_mobile.dart` (saves to device and opens)

`invoice.dart` — PDF template and generation. Uses `pw.Font.helvetica()` built-in fonts. Do NOT swap to TTF via `rootBundle` — the font files are declared under `fonts:` not `assets:` in pubspec.yaml so rootBundle can't load them.
`generate_invoice_screen.dart` — invoice form UI; fetches room prices from API on load (falls back to hardcoded defaults if API fails); price-change icon in AppBar opens `price_settings_screen.dart`.
`price_settings_screen.dart` — edit all room/package prices and driver room price; saves to DB via `PUT /prices`.
`room_selection_screen.dart` — adds new bookings to the database.

### Backend (`flutter_mongodb_backend/`)

Express.js server with five Mongoose models and their corresponding REST routes:

- **Booking** — room reservations (roomNumber, roomType, package, checkIn, checkOut, total, advance, balanceMethod)
- **Inventory** — hotel inventory items
- **Salary** — employee salary records (with monthly query endpoint `/salaries/month/:year/:month`)
- **Expense** — business expenses with category support (with monthly query `/expenses/month/:year/:month`)
- **PriceConfig** — single-document store for room prices. `GET /prices` seeds defaults on first run; `PUT /prices` updates them. The `packages` field is a Mixed type (map of package → room type → price) so `markModified('packages')` must be called before saving.

Database: MongoDB Atlas (`hotel` database). Connection string is hardcoded in `server.js`.

## Key Dependencies

| Package                               | Purpose                                        |
| ------------------------------------- | ---------------------------------------------- |
| `http`                                | API calls                                      |
| `table_calendar`                      | Calendar date picker                           |
| `pdf` + `path_provider` + `open_file` | Invoice PDF generation and opening             |
| `google_fonts`                        | Typography (outfit font also bundled as asset) |
| `image_picker`                        | Inventory image selection                      |
| `month_picker_dialog`                 | Month selection for financial reports          |
| `intl`                                | Date/currency formatting                       |

## Invoice PDF notes

- Check-in time is fixed at **2:00 PM**, check-out at **11:00 AM** — displayed inline on the PDF.
- Extra hour charge note (LKR 1,000/hr) appears in red below the stay info.
- Guest phone number is optional — shown under guest name in "BILL TO" if provided.
- Fixed notes use `-` instead of `•` bullets because Helvetica has no Unicode bullet support.

## Switching Between Local and Production Backend

In [lib/api_service.dart](lib/api_service.dart), toggle the `baseUrl`. Current dev setup uses `localhost:3000`. Kill stale node processes with `lsof -ti :3000 | xargs kill -9` if port 3000 is already in use.
