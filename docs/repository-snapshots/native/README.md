<!-- Historical reference copied from https://github.com/RagueL-HigaBase/higa_systems_native/blob/main/README.md. Not an implementation authority. -->

# Higa Systems Native

Candidate-facing React Native application for Higa Systems.

## Initial scope

- CV onboarding and upload flow
- AI-extracted candidate profile review/editing
- Vacancy list with filters
- Simple recruiter chat
- Candidate profile/settings

## Architecture

Cross-project architecture:
- [HigaBase System Architecture](https://github.com/RagueL-HigaBase/HigaBase_Plans/blob/main/SYSTEM_ARCHITECTURE.md)

This repository contains only the mobile client. Candidate authentication remains isolated from the main Higa Systems business SystemUser model; the core platform receives candidate domain data through backend APIs.

Repository-local mobile implementation must not redefine the central business identity, Organization/Location or invitation invariants.

## Stack

- React Native
- Expo
- Expo Router
- TypeScript

## Start

```bash
npm install
npm start
```

Use Expo Go during early UI work. Native builds can be introduced later when device-specific capabilities require them.

## Current state

Foundation shell only. No backend integration or business logic is implemented yet.
