# Stardew Pets: Codebase Overview

## The big idea

This extension is **two programs that talk to each other by sending messages**:

1. **The "VS Code side"** (backend). It talks to VS Code, handles commands, settings, and saving.
2. **The "game side"** (frontend). It's a tiny website inside a VS Code panel. It draws the pets, moves them around, and handles clicks.

Think of a restaurant:
- The **game side is the dining room**. Customers (pets) walk around and you see everything.
- The **VS Code side is the kitchen and cashier**. It remembers what's saved, takes orders, and writes things down.
- They talk by passing little notes called **messages**, like `{ type: 'spawn_pet', name: 'Mar', specie: 'cat' }`.

---

## The folders

```
Stardew-Pets/
├── package.json        <- The extension's "ID card" and menu/settings definitions
├── src/                <- The code you write (TypeScript)
│   ├── extension.ts    <- VS Code side (the kitchen)
│   └── game/           <- Game side (the dining room)
│       ├── main.ts
│       ├── engine.ts
│       ├── util.ts
│       └── entities/
│           ├── characters.ts
│           ├── pets.ts
│           ├── monsters.ts
│           ├── decoration.ts
│           └── states.ts
├── media/              <- What the user sees
│   ├── main.html       <- The page's structure
│   ├── main.css        <- The page's looks
│   ├── sprites/        <- All the images
│   ├── fonts/          <- The Stardew font
│   └── icons/          <- Little button icons
├── out/ and media/main.js   <- GENERATED. Never edit these.
```

---

## What each file does

### `package.json`: the ID card
Not code. It tells VS Code:
- the extension's name, version, and description
- **Settings** the user can change: background, scale (size), monsters on/off
- **Commands** like "Add pet" and "Remove pet", plus their buttons and icons
- **Where the panel appears**: in the Explorer sidebar
- **Build scripts** (`build-all`, and so on)

If you want a new setting or a new button, start here.

### `src/extension.ts`: the kitchen (VS Code side)
- **Lists of pets**: `PetSpecies` has every pet and its color variants. `Names` is the list of random name suggestions.
- **Saving and loading**: writes pets, money, and decorations to a file called `save.json`, so your pets are still there after you restart VS Code.
- **Commands**: "Add pet" asks for species, variant, and name. "Remove pet" lets you pick one to delete.
- **Settings listener**: when you change a setting, it tells the game.
- **`WebViewProvider`**: loads `main.html` into the panel and receives messages from the game, for example "player got money, save it".

### `media/main.html`: the skeleton
The page structure: the background, the canvas (where pets are drawn), the menus (actions, store, money), and the decor mode buttons. Plain HTML, so there's no logic here.

### `media/main.css`: the looks
Colors, sizes, fonts, menu borders, and backgrounds (grass, sand, snow, wood...). The text color is `--text: #551215`, near the top.

### `src/game/main.ts`: the game's front door
- **Receives messages** from the VS Code side (`spawn_pet`, `background`, `scale`...) and acts on them.
- Handles **mouse clicks**.
- Builds the **store menu** (the buy buttons).
- Starts the game at the bottom (`Game.start(...)`).

### `src/game/engine.ts`: the game's motor
- **`Game`**: runs the game loop, which updates and redraws everything **30 times per second**. It holds the money, the lists of pets/monsters/decor, and the current action.
- **`GameObject`**: the base for *everything* you see (position, size, sprite, animation, "was I clicked?").
- **`Animation`**: flips through sprite frames (walking, sleeping...).
- **`Menus`**, **`DecorMode`**, **`Cursor`**: the UI helpers.
- **`Ball`**: the ball you throw for the pets.

### `src/game/util.ts`: the toolbox
Small helpers like `Vec2` (an x/y position), random number functions, and a timeout helper.

### `src/game/entities/`: the creatures and objects
- **`characters.ts`**: the **AI** (the little brain). Every pet and monster follows the same loop: **idle -> (maybe sleep) -> walk somewhere random -> idle...**
- **`pets.ts`**: every pet's animations and its **moods** (the emoji bubbles). Each species is a small class (`Cat`, `Dog`, `Junimo`...) that says "my size is X, and each color sits at this spot in the image".
- **`monsters.ts`**: the same idea for slimes, bugs, crabs, and golems. Clicking one kills it and gives you 40-80 gold.
- **`decoration.ts`**: a giant list of furniture (name, size, price, sprite position), plus the `Decoration` class (dragging and selling).
- **`states.ts`**: a few AI state names (`idle`, `move`, `special`).

---

## The flow, with an example: "Add a cat"

1. You click the **+** button. VS Code runs the command `stardew-pets.addPet`. It is declared in `package.json` and handled in `extension.ts`.
2. `extension.ts` asks you for species, variant, and name.
3. It saves the pet to `save.json`.
4. It sends the message `spawn_pet` to the game.
5. `main.ts` receives it and runs `new Cat(name, color)`.
6. The cat appears in the panel and starts wandering around.

---

## "I want to change something. Where do I go?"

- **Change the size of the pets**: in `package.json`, find `stardew-pets.scale`. The actual numbers (1, 2, 3) are in `main.ts`, in `case 'scale'`.
- **Add a new background**: add a name in the `enum` list in `package.json`. Add a CSS block in `main.css` (copy the `#background[background="sand"]` block). Put the image in `media/sprites/backgrounds/`.
- **Change the colors or look of the menus**: edit `media/main.css`.
- **Change the text color**: edit `--text` at the top of `main.css`.
- **Change menu text and buttons**: edit `media/main.html`.
- **Add a new color variant for an existing pet**: add the name in `PetSpecies` in `extension.ts`. Then add a `case` with the image offset in that pet's class in `pets.ts`.
- **Add a whole new pet**: add it in `PetSpecies` in `extension.ts`. Add its class in `pets.ts`, copying `Cat`, plus its animations. Register it in the `spawn_pet` switch in `main.ts`. Add its sprite sheet in `media/sprites/pets/`.
- **Change money from monsters**: `monsters.ts`, in `click()`, the line `Game.addMoney(40 + 5 * ...)`.
- **Change how often monsters appear**: the `30 * 1000` values (30 seconds). They're in `main.ts` (`case 'monsters'`) and in `monsters.ts`.
- **Change how fast pets walk or sleep**: `characters.ts`, in the `AI` class. The durations are at the top (`idleDuration`, `sleepDuration`...).
- **Change pet walking animation speed**: `PetAnimations` in `pets.ts`. The `5` after the frames is the speed.
- **Change decoration prices or add furniture**: `decoration.ts`, in `DecorationPresets`.
- **Change the pet name suggestions**: `Names` in `extension.ts`.
- **Change how long the heart mood stays**: `pets.ts`, `10 * 60 * 1000` (10 minutes).
- **Change the messages that pop up**, like "Say hi to...": `extension.ts`.

---

## Important: building and running

- `media/main.js` and `out/` are **generated** from the TypeScript, and git ignores them. They may be missing or outdated.
- After changing code, run the build in a terminal:
  ```
  npm install
  npm run build-all
  ```
  `npm install` is only needed once. `npm run build-all` turns `src/` into runnable files.
- Then press **F5** in VS Code to test. This starts a second VS Code window with your extension loaded. If F5 doesn't work, check `.vscode/launch.json`.
- `build.bat` packages the extension into an installable `.vsix` file, using the tool `vsce`.

---

## Where to start

Start with something small:
- Change a color in `main.css` and see it update.
- Change a gold amount in `monsters.ts`.
- Add a name to `Names` in `extension.ts`.

After each change, run `npm run build-all` and then press F5 to see the result.
