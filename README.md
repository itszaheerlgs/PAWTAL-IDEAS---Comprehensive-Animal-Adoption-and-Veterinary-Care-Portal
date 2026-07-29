# PAWTAL — Comprehensive Animal Adoption and Veterinary Care Portal

PAWTAL is a web-based platform that unifies two functions normally handled by separate systems: **animal adoption matching** and **veterinary care management**. Instead of a shelter using a simple listing page and a clinic keeping paper or spreadsheet records, PAWTAL tracks an animal's full journey — from intake, to adoption, to ongoing medical care — in a single system.

The platform also introduces two features that address real gaps in typical adoption systems:

1. **Identity verification of adopters** using recognized Philippine government-issued IDs, to reduce fraudulent or bad-faith adoption applications.
2. **A geolocation-based veterinary facility finder**, so that once an animal is adopted, the new owner can immediately locate and reach the nearest verified veterinary clinic for continued care.

The system can serve a single shelter or clinic, or scale to a network of multiple shelters and clinics operating under one shared platform (for example, coordinated by a city or provincial veterinary/animal welfare office).

---

## Table of Contents

- [Objectives](#objectives)
- [Scope and Limitations](#scope-and-limitations)
- [User Roles](#user-roles)
- [System Modules](#system-modules)
  - [Adopter Registration and ID Verification](#1-adopter-registration-and-id-verification-module)
  - [Animal Adoption Module](#2-animal-adoption-module)
  - [Veterinary Care Module](#3-veterinary-care-module)
  - [Veterinary Facility Map Module](#4-veterinary-facility-map-module)
  - [Post-Adoption Follow-up Module](#5-post-adoption-follow-up-module)
  - [Community and Content Module](#6-community-and-content-module)
  - [Admin and Analytics Module](#7-admin-and-analytics-module)
- [Core Data Model](#core-data-model)
- [Technology Stack](#technology-stack)
- [Significance](#significance)
- [Roadmap](#roadmap)
- [License](#license)

---

## Objectives

### General Objective

To design and develop a web-based portal that streamlines animal adoption and veterinary care management, while ensuring that adopters are verified individuals and that adopted animals have continued access to nearby veterinary care.

### Specific Objectives

- Develop an adopter registration module with automated government ID validation to reduce fraudulent applications.
- Develop an adoption management module covering animal intake, listing, application review, and approval workflows.
- Develop a veterinary care module for scheduling appointments and maintaining digital medical records per animal.
- Develop a map-based veterinary facility locator that helps adopters find the nearest verified vet clinic.
- Implement post-adoption follow-up tracking to monitor animal welfare after placement.
- Provide administrative dashboards and reports for shelters, clinics, and system-wide oversight.

---

## Scope and Limitations

### Scope

- Online adopter registration with ID upload and automated validation checks.
- Animal profile creation and adoption listing management by shelter staff.
- Adoption application submission, review, and approval/rejection workflow.
- Veterinary clinic registration, verification, and map-based display.
- Appointment booking and digital medical record-keeping.
- Vaccination and follow-up reminder notifications.
- Role-based access for Admin, Shelter Staff, Veterinarian, and Adopter/Owner accounts.

### Limitations

- The system validates ID format, structure, and document authenticity indicators (such as signs of tampering); it does **not** perform real-time verification directly against PSA, DFA, LTO, or PRC government databases, since these are not publicly accessible to third-party developers. Full database-level verification would require formal integration through an accredited eKYC provider or a government data-sharing agreement. This is outside the current scope but is supported by the architecture for future integration.
- Payment processing (adoption fees, vet service fees) is designed at the schema level but is not connected to a live payment gateway in the initial version.
- The system assumes participating shelters and clinics manually confirm their own accreditation documents during registration approval.

---

## User Roles

| Role | Description | Key Permissions |
|---|---|---|
| Guest / Public | Unregistered visitor | Browse adoptable animals, read articles, view the vet map |
| Adopter / Pet Owner | Registered and ID-verified individual | Apply for adoption, book vet appointments, view own pet's medical records |
| Shelter Staff | Manages a shelter's animals and applications | Add/edit animal listings, review applications, manage intake |
| Veterinarian / Vet Staff | Manages clinical operations | Manage appointments, medical records, prescriptions |
| Facility Admin | Manages a registered vet clinic's profile | Update clinic info, services, hours, staff accounts |
| System Admin | Oversees the whole platform | Approve shelters/clinics, manage users, view system-wide reports |

---

## System Modules

### 1. Adopter Registration and ID Verification Module

Handles account creation for adopters and validates identity using one of the following recognized Philippine government-issued IDs:

- PhilSys National ID (PhilID / ePhilID)
- Philippine Passport (DFA)
- Driver's License (LTO)
- PRC ID
- UMID Card (SSS/GSIS)
- Postal ID (PHLPost)
- Voter's ID / Voter's Certification (COMELEC)
- Senior Citizen ID (OSCA)
- PWD ID
- TIN ID (BIR)
- PhilHealth ID
- OWWA ID / iDOLE Card

**Verification approach.** Since real-time verification against government databases is not publicly available to third-party developers, the system applies a layered approach:

| Layer | Method | What It Catches |
|---|---|---|
| 1. Format validation | OCR extraction of ID number, name, birthdate, expiry; format/checksum rules per ID type | Wrong ID type, malformed ID numbers, obvious typos |
| 2. Document forensics | Image analysis for tampering signs: font mismatches, blur/edit artifacts, recompression signatures, missing expected security features | Edited/photoshopped IDs, screenshots of other people's IDs |
| 3. Face match | Live selfie compared against the ID photo using a face-matching library or API | Stolen or borrowed IDs used by someone else |
| 4. Manual staff review | Shelter/clinic staff performs a final check before granting verified status | Anything the automated checks miss or flag as uncertain |

The architecture also leaves room for future integration with a licensed eKYC provider for direct PhilSys/PSA-backed verification, without requiring a redesign of the core system.

**Registration workflow**

1. Adopter creates an account and uploads a front (and back, if applicable) image of a valid ID plus a live selfie.
2. System runs OCR extraction and automated format/tamper checks.
3. System runs face-match comparison between the selfie and the ID photo.
4. Application is tagged: Auto-Passed, Needs Manual Review, or Rejected.
5. Shelter/clinic staff reviews and gives final approval before the account becomes "Verified."
6. Duplicate ID numbers are flagged automatically to prevent one person from registering multiple accounts.

### 2. Animal Adoption Module

- Animal intake and profile creation (species, breed, age, temperament, medical flags, photos).
- Public search and filter by species, size, age, location, and special needs.
- Adoption application submission, tied to a verified adopter account.
- Application review workflow: Submitted → Under Review → Approved/Rejected → Adopted.
- Optional home-check step for shelters that require it.
- Adoption status tracking: Available, Pending, Adopted, Fostered, Returned.

### 3. Veterinary Care Module

- Appointment scheduling with slot-based or walk-in queue booking.
- Digital medical records per animal: vaccinations, surgeries, allergies, weight history, diagnoses.
- Prescription and treatment plan tracking.
- Automated vaccination and deworming reminder notifications.
- Emergency/urgent-case flagging for faster triage.

### 4. Veterinary Facility Map Module

Allows registered veterinary clinics to appear on an interactive map so adopters and pet owners can quickly find nearby care.

**Clinic registration workflow**

1. Clinic submits registration: name, address, accreditation/license number, services offered, operating hours, and a copy of their business/veterinary license.
2. Address is geocoded into latitude/longitude coordinates.
3. System Admin reviews the submitted license and approves or rejects the clinic.
4. Once approved, the clinic pin appears live on the public map.

**Adopter-facing map features**

- "Find Nearest Vet" using the user's device location or a manually entered address.
- Distance-sorted list of clinics with map pins.
- Filtering by service type (emergency care, vaccination, grooming, boarding, etc.).
- Clinic profile page: hours, contact details, services, ratings, and a direct "Book Appointment" button.

### 5. Post-Adoption Follow-up Module

- Automated check-ins at 7, 30, and 90 days after adoption.
- Adopter-submitted behavior or health issue reports.
- Return-to-shelter workflow for adoptions that do not work out.

### 6. Community and Content Module

- Pet care articles and breed guides.
- Lost & found pet board.
- Donation and sponsorship listings for shelter animals.

### 7. Admin and Analytics Module

- Adoption rate statistics and average time-to-adoption.
- Veterinary caseload and common diagnosis reports.
- Revenue tracking (adoption fees, donations, paid vet services).
- Shelter and clinic approval/oversight dashboard.

---

## Core Data Model

Modeled as Firestore collections (with subcollections where data is naturally nested under a parent document):

| Collection | Purpose |
|---|---|
| `users` | Base accounts and role assignment across the platform (role stored as a custom claim + mirrored field) |
| `adopters` | Adopter profile and verification status |
| `adopters/{adopterId}/idDocuments` | Uploaded ID image references (Storage paths), OCR data, verification results |
| `animals` | Animal profiles, medical flags, adoption status |
| `adoptionApplications` | Application workflow and decision tracking, referencing an `adopterId` and `animalId` |
| `shelters` | Shelter organization records |
| `vetFacilities` | Registered veterinary clinics, including a `location` GeoPoint field and `geohash` for proximity queries |
| `vetFacilities/{facilityId}/services` | Services offered per clinic |
| `appointments` | Scheduled vet visits, referencing a `facilityId` and `animalId` |
| `animals/{animalId}/medicalRecords` | Per-animal clinical history |
| `animals/{animalId}/vaccinations` | Vaccination/deworming history and reminder schedule |
| `notifications` | System-generated alerts and reminders, keyed by `userId` |

Storage paths (Firebase Storage) are referenced by document fields rather than storing binary data in Firestore, for example: `id-documents/{adopterId}/{docId}-front.jpg`, `animal-photos/{animalId}/{photoId}.jpg`, `vet-licenses/{facilityId}/license.pdf`.

---

## Technology Stack

PAWTAL can be built on **Firebase** as the primary backend platform, which removes the need to host and maintain a separate server for auth, database, and file storage.

| Layer | Firebase Service | Notes |
|---|---|---|
| Frontend | React (or Vue) + Vite, hosted on **Firebase Hosting** | SPA or PWA; works for both web and an installable mobile-like experience |
| Authentication | **Firebase Authentication** (Email/Password, plus Phone OTP for PH mobile numbers) | Handles Adopter, Staff, Vet, and Admin accounts; custom claims used for role-based access |
| Database | **Cloud Firestore** | Stores Adopters, Animals, AdoptionApplications, VetFacilities, Appointments, MedicalRecords, Notifications, etc. as collections/subcollections |
| File Storage | **Firebase Storage** | Stores uploaded ID images (front/back), live selfies, animal photos, and vet clinic license documents |
| Server Logic | **Cloud Functions for Firebase** (Node.js) | Runs OCR triggers, face-match calls, duplicate-ID checks, notification dispatch, and admin approval workflows on document create/update |
| Notifications | **Firebase Cloud Messaging (FCM)** | Push notifications for vaccination reminders, application status updates, and appointment confirmations |
| Map & Geolocation | Leaflet.js + OpenStreetMap, or Google Maps JS API (Firebase has no built-in mapping service) | Clinic coordinates stored as a Firestore GeoPoint field |
| Geocoding | Nominatim (free) or Google Geocoding API, called from a Cloud Function | Converts a clinic's submitted address into latitude/longitude on registration |
| Geo Queries ("nearest vet") | Firestore GeoPoint + a geohashing library (e.g., `geofire-common`) inside a Cloud Function | Firestore has no native radius query, so geohashing is used to query clinics within a bounding area, then sorted by real distance |
| OCR / ID Processing | Google Cloud Vision API (Document Text Detection), called from a Cloud Function | Extracts ID number, name, birthdate, and expiry from uploaded ID images |
| Face Matching | Cloud Function calling a face-comparison API (e.g., AWS Rekognition CompareFaces, Azure Face API, or a self-hosted `face_recognition` microservice) | Firebase has no built-in face-match service, so this is called from within a Function |
| Security Rules | **Firestore Security Rules** + **Storage Security Rules** | Enforces role-based read/write access (e.g., only the owning Adopter and assigned Vet Staff can read a given MedicalRecord) |
| Analytics/Reporting | **Firebase Analytics**, plus BigQuery export from Firestore for admin dashboards and reports | Powers the Admin and Analytics Module |

**Why Firebase fits this project**

- Authentication and role management (Adopter, Shelter Staff, Vet Staff, Facility Admin, System Admin) are handled out of the box via custom claims, instead of building a login system from scratch.
- Firestore's real-time listeners are a natural fit for live status updates — for example, an adopter sees their application move from "Under Review" to "Approved" instantly, and a shelter's available-animal list updates live without a page refresh.
- Firebase Storage combined with Security Rules gives a straightforward way to keep uploaded ID documents and medical records private per user, which matters for both privacy and trust.
- Cloud Functions keep sensitive logic (OCR calls, face-match calls, duplicate-ID checks) server-side rather than exposing API keys in client code.
- Firebase Hosting and FCM make it simple to ship a fast, installable PWA without managing separate hosting or notification infrastructure.

**Trade-off to note:** Firestore is a NoSQL document database, so relational-style joins (for example, an Animal's full Adoption + Medical + Vaccination history) need to be handled through denormalized data structures or multiple queries composed in the client or a Cloud Function, rather than SQL joins. This is a deliberate design shift from an earlier SQL Server-based approach and should be planned for carefully at the data-modeling stage.

---

## Significance

- **For Shelters:** reduces manual paperwork, centralizes animal records, and improves adoption tracking.
- **For Adopters:** provides a trustworthy, verified adoption process and easy access to nearby veterinary care after adoption.
- **For Veterinary Clinics:** streamlines appointment scheduling and medical record-keeping, and increases visibility to new pet owners.
- **For Local Government/Animal Welfare Offices:** provides consolidated data on adoption rates and animal welfare outcomes across multiple shelters.

---

## Roadmap

- [ ] Confirm platform scope: single shelter/clinic deployment vs. multi-shelter/multi-clinic network.
- [ ] Finalize technology stack based on team familiarity and deployment environment.
- [ ] Produce a full Entity-Relationship Diagram (ERD) and wireframes for each module.
- [ ] Identify an eKYC or OCR provider for ID verification proof-of-concept.
- [ ] Define a phased development plan (Adoption Module first, then Vet Care Module, then Map/Verification features).

---

## License

License to be determined by the project owner.
