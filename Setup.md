# PureNote App Setup Guide

PureNote is a modern, premium PWA (Progressive Web App) for note-taking, built with Vue 3, Vite, and Bootstrap 5.

## 🚀 Getting Started

### Prerequisites
- [Node.js](https://nodejs.org/) (Version 16 or higher recommended)
- [npm](https://www.npmjs.com/) (comes with Node.js)

### 1. Installation
First, clone or download the project, then navigate to the project directory and install the dependencies:

```bash
npm install
```

### 2. Development Mode
To run the app locally for development with hot-reload:

```bash
npm run dev
```
The app will be available at `http://localhost:5173`.

### 3. Build for Production
To create a production-ready build (including PWA assets):

```bash
npm run build
```
The output will be in the `dist/` folder.

### 4. Preview Production Build
To test the production build locally:

```bash
npm run preview
```

## ✨ Key Features
- **PWA Ready**: Can be installed on mobile/desktop and works offline.
- **Glassmorphism UI**: Premium, modern design with smooth animations.
- **LocalStorage Persistence**: Your notes stay on your device even after refreshing.
- **Image Attachments**: Support for adding visual memories to your notes.
- **Organization**: Filter by Pinned/Starred status and Categories (Tags).
- **Responsive**: Fully optimized for both Mobile and Desktop screens.

## 🛠️ Built With
- **Framework**: Vue 3 (Composition API)
- **Styling**: Bootstrap 5 + Bootstrap Icons
- **Build Tool**: Vite
- **PWA Support**: Vite Plugin PWA
- **Icons**: [Bootstrap Icons](https://icons.getbootstrap.com/)

---
*Created with PureNote App - Capture your mind, perfectly organized.*
