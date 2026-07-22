# Montesierra Public Reservation Portal

A single-file, GitHub Pages–ready public website that plugs directly into your existing Dormitory Management System (`server.js` + `data.json`). **No new backend, no new database.**

## Files
- `index.html` — the entire public site (drop into any GitHub Pages repo).

## Setup

1. Point the site at your backend. Either edit this line near the top of `index.html`:
   ```js
   BACKEND_URL: (localStorage.getItem('MONTESIERRA_BACKEND_URL') || "http://localhost:3000")
   ```
   …or open the deployed site in a browser, open the console and run:
   ```js
   setBackend("https://your-api.example.com")
   ```
   (Persists in `localStorage`.)

2. Enable CORS on your Express server so the GitHub Pages origin can call it.
   Your `server.js` already uses `cors()`. For production restrict it:
   ```js
   app.use(cors({ origin: ['https://<your-username>.github.io'] }));
   ```

3. Deploy `index.html` to GitHub Pages. Done.

## How it integrates (single source of truth)

- **Read**: `GET /api/data` — fetches the existing full snapshot (rooms, beds, reservations, boarders). Refreshes every 60s.
- **Write**: `POST /api/data` — the same endpoint the admin app uses. The server merges + preserves `emailLogs` / `emailCounter` / `reminderHistory` automatically.
- **Email**: `POST /api/send-reservation-email` `{ kind:'confirmation', reservation }` — uses your existing reservation email pipeline (with retries + logs).

Reservation objects are pushed into `db.reservations` using the exact schema your admin's `saveReservation()` writes (`id`, `type:'bed'`, `roomId`, `bedNo`, `name`, `contact`, `email`, `date`, `time`, `expDate`, `expTime`, `deposit:0`, `method:'Pending'`, `status:'Active'`, `createdAt`) plus a `remarks` field carrying the extra public form data and a `source:'public-portal'` tag. The selected bed is flagged `isReserved:true` with `reservationId` set — identical to the admin flow.

## What's hidden
Dashboard, boarders, billing, reports, inventory, maintenance, calendar, email/audit logs, settings, check-in/out, tenant names, edit/delete/add controls — none of it exists in this file.

The **only** actions are: browse, search, filter, view room, pick a bed, submit reservation.

## Tenant privacy
Occupied beds display `Tenant #001`, `Tenant #002`, … derived from the stable index in `db.boarders`. Real names never leave the server.
