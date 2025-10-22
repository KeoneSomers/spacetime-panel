# Spacetime Panel

Spacetime Panel is a lightweight, web-based control panel designed to interact with your Spacetime server. It provides
an intuitive interface to manage your game’s data, view server state, and test functionality in real-time. Built with
TypeScript and Node.js, it integrates seamlessly with Spacetime’s TypeScript bindings, making it easy to inspect and
manipulate your database tables directly from the browser.

![Screenshot Image](https://i.postimg.cc/T1L579wp/Screenshot-2025-10-22-110023.png)

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

Generate your server bindings:

```bash
spacetime generate --lang typescript --out-dir C:/Users/keone/spacetime-panel/src/spacetime_module_bindings --project-path C:\Users\keone\Elegon\server-csharp
```

Start the local site:

```bash
npm run dev
```

Done! Now you can view Spacetime Panel in your web browser at: http://localhost:5173/

---

## Current Features:

(TODO)

 ---

## Upcoming Features

(TODO)
