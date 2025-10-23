# Spacetime Panel

Spacetime Panel is a lightweight, web-based control panel designed to interact with your Spacetime server. It provides
an intuitive interface to manage your game’s data, view server state, and test functionality in real-time. Built with
TypeScript and Node.js, it integrates seamlessly with Spacetime’s TypeScript bindings, making it easy to inspect and
manipulate your database tables directly from the browser.

![Screenshot Image](https://github.com/KeoneSomers/spacetime-panel/blob/main/public/screenshot.png?raw=true)

---

## Getting Started:

Clone this repositry onto your computer:

```HTTPS
https://github.com/KeoneSomers/spacetime-panel.git
```

Insatall the projects dependencies (Requires Node JS):

```bash
npm install
```

Start your local Spacetime server:

```Spacetime CLI
spacetime start
```

Navigate to your Spacetime Server local directory and run this to generate your server bindings:

```bash
spacetime generate --lang typescript --out-dir <BasePathToSpacetimePanel>/spacetime-panel/src/spacetime_module_bindings
```

Start the local site:

```bash
npm run dev
```

Done! Now you can view Spacetime Panel in your web browser at: http://localhost:5173/

---

## Current Features:

- Connect to your SpacetimeDB Server
- View Tables in realtime

 ---

## Upcoming Features

- View Reducers
- Table types
- Table PK tag
- Login with token
- Show CCU count
