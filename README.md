# Tanga Technical Secondary School — Academic Results Portal

A Firebase-backed results workspace for Tanga Technical Secondary School. It supports two protected staff experiences: **Teacher** (self-registration, approval-gated mark entry and correction) and **Academic Office** (student registration, teacher oversight, and a live all-marks register).

The visual language follows the school identity and the reference site at [tangaschool.sc.tz](https://tangaschool.sc.tz/): Tanga Technical Secondary School, Kange, Tanga; Forms 1–6; and A-Level combinations **PCM, PMC, and PCB**.

## What is implemented

- Firebase Email/Password sign-in and password reset.
- Teacher self-registration with subject, class, and optional A-Level combination.
- Approval-gated teacher access: new registrations remain `active: false` until the academic office approves them.
- Teacher markbook for entering and correcting scores for assigned classes, with automatic grade calculation and live Firestore writes.
- Academic workspace for registering students, reviewing teacher registrations, and viewing all submitted marks.
- Responsive desktop and mobile layouts with live status indicators and empty/loading/error states.
- Firestore rules that prevent teachers from creating admin accounts or reading another teacher's marks.

## Run locally

```bash
npm install
npm run dev
```

Then open the Vite URL printed in the terminal. The Firebase web-app configuration is read from `.env.local`; do not add a service-account key to this frontend project.

## Firebase bootstrap

1. In the Firebase project, enable **Authentication → Sign-in method → Email/Password**.
2. Create a Firestore database in production mode.
3. Create the first academic account under **Authentication → Users**.
4. In Firestore, create `users/<UID>` for that account with the following fields:

```json
{
  "uid": "same-as-document-id",
  "email": "academic@example.com",
  "displayName": "Academic Master",
  "role": "ACADEMIC_MASTER",
  "active": true
}
```

5. Deploy the rules and indexes from this project:

```bash
npx -y firebase-tools@latest use tanga-school-portal
npx -y firebase-tools@latest deploy --only firestore:rules,firestore:indexes,hosting
```

The academic account can then sign in, register students, and review teacher self-registrations. To approve a teacher, update that teacher's `users/<UID>` document so `active` is `true`. The teacher will see the markbook on their next session refresh.

## Collections

- `users/{uid}` — staff profile and role. Roles supported by the rules are `ADMIN`, `ACADEMIC_MASTER`, `EXAM_OFFICER`, `TEACHER`, and `STUDENT`.
- `students/{id}` — academic-office-owned student register.
- `results/{id}` — one student/subject/assessment result, written by the assigned teacher and read by academic staff.

## Security model

Teacher self-registration is deliberately constrained in `firestore.rules`: a signed-in user may create only their own profile, only with `role: "TEACHER"`, and only with `active: false`. Only academic roles can modify profiles, register students, and read the complete result register. Teachers can read and write only result documents whose `teacherId` is their own UID.

This is a browser-only Firebase Hosting/Spark implementation. It does not expose Firebase console credentials or service-account keys. Official approval/publication workflows should be reviewed by the school before production use; if a trusted server-side approval or bulk-import process is required, add a backend on an appropriate Firebase plan.

## Production build

```bash
npm run build
```

The current production build completes successfully with TypeScript checking enabled.
