# Remission

A mobile app for tracking Inflammatory Bowel Disease (IBD) symptoms, meals, and medications, built with [Expo](https://expo.dev) (React Native + TypeScript).

## Status

This is a basic starter scaffold: project structure, navigation, and a single placeholder screen. Core tracking features (symptom/flare log, meal log, medication log, history view) are not yet implemented.

## Getting started

Requires [Node.js](https://nodejs.org) and npm.

```bash
npm install
npm start
```

This opens the Expo dev tools. From there you can:

- Press `i` to open in the iOS simulator (macOS only)
- Press `a` to open in an Android emulator
- Press `w` to open in a web browser
- Scan the QR code with the [Expo Go](https://expo.dev/go) app on your phone

## Project structure

```
App.tsx                    # App entry point, renders the root navigator
src/
  navigation/
    RootNavigator.tsx       # Stack navigator and route definitions
  screens/
    HomeScreen.tsx          # Placeholder home screen
```

## Tech stack

- [Expo](https://docs.expo.dev/) (managed workflow)
- React Native + TypeScript
- [React Navigation](https://reactnavigation.org/) (native stack)

## Next steps

- Design the data model for symptom/flare, meal, and medication entries
- Add local persistence (e.g. AsyncStorage or SQLite via `expo-sqlite`)
- Build out logging screens and a history/trends view
- Decide on cloud sync (optional) if data needs to be available across devices
