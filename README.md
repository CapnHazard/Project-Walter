# ⚔️ Yudh Sarthi — Rapid Crisis Response

> **Google Solution Challenge 2026** | Category: Rapid Crisis Response — Open Innovation
> 
> A dual-role, offline-first web platform for war-affected regions — connecting verified citizens with government authorities in real time, even on 2G networks.

---

## 🌍 The Problem

In war-affected and disaster-struck regions, civilians lack real-time access to critical safety information — safe zones, emergency resources, and government directives — while authorities lack a unified, secure platform to coordinate crisis response and collect verified ground data.

Existing solutions require stable internet, making them ineffective in conflict zones where 2G is the maximum available network.

---

## 💡 Our Solution

**Yudh Sarthi** (Hindi: *Companion in War*) is a web application that works on 2G networks with offline-first architecture, providing:

- 🗺️ **Citizens** — a live satellite map with government-marked safe, warring, and dangerous zones, emergency resource locations, and government polls
- 🏛️ **Authorities** — a secure dashboard to mark crisis zones, manage emergency resources, publish polls, and monitor activity in real time

---

## 🎯 UN Sustainable Development Goals

| SDG | Relevance |
|-----|-----------|
| **SDG 16** — Peace, Justice & Strong Institutions | Core mission — civilian safety infrastructure in conflict zones |
| **SDG 11** — Sustainable Cities & Communities | Safe zone mapping and emergency resource coordination |
| **SDG 3** — Good Health & Well-being | Medical checkpoint visibility and resource access |

---

## ✨ Features

### For Citizens
- 📱 **Phone OTP Login** — +91 Indian mobile number, no email required
- 🗺️ **Satellite Map** — Mapbox satellite-streets with live green/amber/red zone polygons
- 📍 **Resource Markers** — shelters, medical camps, food supply points with contact info
- 📋 **Government Polls** — respond to authority-published forms and surveys
- 👤 **Verified Profile** — Aadhaar/PAN validated identity with Verhoeff checksum algorithm
- 👨‍👩‍👧 **Family Registration** — add and manage family members under one profile

### For Government Authorities
- 🔐 **Secure Role-Based Login** — email + Firestore role verification (`role: authority`)
- 🗾 **Zone Management** — mark safe, warring, and dangerous zones via map click or location name
- 🏥 **Resource Management** — add shelters, medical camps, food supply points
- 📊 **Poll Creation** — publish citizen-facing polls and registration forms
- 📋 **Live Activity Log** — real-time log of all authority actions stored in Firestore
- 📈 **Live Statistics** — total registered users, verified citizens, poll response counts

---

## 🏗️ Architecture

```
┌─────────────────────┐     ┌──────────────────────┐     ┌─────────────────────┐
│   Frontend (React)  │     │   Firebase Backend    │     │  External Services  │
│                     │     │                       │     │                     │
│ React 19 +          │────▶│ Firebase Auth         │────▶│ Mapbox Geocoding    │
│ TypeScript          │     │ (Phone OTP + Email)   │     │ API                 │
│                     │     │                       │     │                     │
│ Mapbox GL JS        │────▶│ Firestore DB          │     │ Firebase Hosting    │
│ (Satellite Map)     │     │ (Real-time + Offline) │     │ (Deployment)        │
│                     │     │                       │     │                     │
│ Aadhaar/PAN         │     │ Security Rules        │     │ UIDAI API (Planned) │
│ Validation Logic    │     │ (Role-based Access)   │     │                     │
│                     │     │                       │     │ Gemini AI (Planned) │
│ Offline IndexedDB   │     │ onSnapshot Listeners  │     │                     │
│ Persistence         │     │ (Live Data Sync)      │     │                     │
└─────────────────────┘     └──────────────────────┘     └─────────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 19 + TypeScript |
| Build Tool | Vite |
| Styling | Tailwind CSS |
| Maps | Mapbox GL JS (satellite-streets) |
| Auth | Firebase Authentication (Phone OTP + Email/Password) |
| Database | Firebase Firestore (real-time + offline IndexedDB persistence) |
| Security | Firestore Security Rules (role-based) |
| Validation | Verhoeff Algorithm (Aadhaar) + PAN regex |
| Hosting | Firebase Hosting |

---

## 🚀 Getting Started

### Prerequisites
- Node.js v18+
- A Firebase project with Authentication and Firestore enabled
- A Mapbox account (free tier)

### Installation

```bash
git clone https://github.com/YOUR_USERNAME/yudh-sarthi.git
cd yudh-sarthi
npm install
```

### Environment Setup

Create a `.env` file in the project root:

```env
VITE_FIREBASE_API_KEY=your_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=your_app_id
VITE_MAPBOX_TOKEN=your_mapbox_token
```

### Firebase Setup

1. Enable **Phone** and **Email/Password** authentication in Firebase Console
2. Create Firestore database in `asia-south1` region
3. Deploy security rules:

```bash
firebase deploy --only firestore:rules
```

4. Create an authority user in Firebase Console → Authentication → Add User
5. In Firestore, create document `users/{uid}` with field `role: "authority"`

### Run Locally

```bash
npm run dev
```

Open `http://localhost:3000`

### Deploy

```bash
npm run build
firebase deploy --only hosting
```

---

## 🔐 Firestore Security Rules

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth.uid == userId;
    }
    match /zones/{zoneId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null &&
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'authority';
    }
    match /resources/{resourceId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null &&
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'authority';
    }
    match /polls/{pollId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null &&
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'authority';
    }
    match /pollResponses/{responseId} {
      allow read, write: if request.auth != null;
    }
    match /activityLog/{logId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null &&
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'authority';
    }
  }
}
```

---

## 📁 Project Structure

```
yudh-sarthi/
├── src/
│   ├── components/
│   │   ├── Dashboard.tsx      # Main app — citizen + authority views
│   │   └── Map.tsx            # Mapbox satellite map with zones + resources
│   ├── lib/
│   │   ├── firebase.ts        # Firebase init + offline persistence
│   │   ├── validation.ts      # Verhoeff (Aadhaar) + PAN validation
│   │   └── utils.ts           # Utility functions
│   ├── App.tsx                # Auth flows — Phone OTP + Email login
│   ├── types.ts               # TypeScript interfaces
│   └── main.tsx
├── firestore.rules            # Firestore security rules
├── firebase-blueprint.json    # DB schema documentation
└── .env.example               # Environment variable template
```

---

## 🔮 Future Development

- **Gemini AI Integration** — crisis zone summarization, resource optimization, auto-triage of poll data
- **UIDAI AUA API** — real Aadhaar OTP verification (requires government registration)
- **React Native App** — better field usability for soldiers and aid workers
- **Firebase Cloud Messaging** — push notifications for zone status changes
- **SMS Fallback** — critical alerts via SMS for users without smartphones
- **Multi-tier Authority Access** — district / state / national scoped permissions
- **Multi-language Support** — Hindi, Tamil, Telugu, Bengali, Punjabi

---

## 👥 Team

| Name | Role |
|------|------|
| [Team Leader Name] | Lead Developer |
| [Member 2] | Frontend |
| [Member 3] | Backend / Firebase |
| [Member 4] | UI/UX + Documentation |

**Institution:** KIIT University, Bhubaneswar
**Challenge:** Google Solution Challenge 2026 — Rapid Crisis Response: Open Innovation

---

## 📎 Links

- 🌐 **Live App:** https://yudh-sarthi-d3481.web.app
- 🎥 **Demo Video:** [Add link]
- 📊 **Presentation:** [Add link]

---

## 📄 License

MIT License — open for use, modification, and distribution.
