
# Lich Scripting Engine & Proxy - Technical Documentation (GemStone IV Focus)

**Version:** Based on Lich 5.12.0-beta.5 (from provided code)

## Table of Contents
1.  [High-Level Overview & Script Purpose](#1-high-level-overview--script-purpose)
    *   [1.1. Primary Purpose](#11-primary-purpose)
    *   [1.2. Main Mode of Operation](#12-main-mode-of-operation)
    *   [1.3. Initiation and Usage](#13-initiation-and-usage)
    *   [1.4. Key Entry Points](#14-key-entry-points)
2.  [Code Flow & Key Components](#2-code-flow--key-components)
    *   [2.1. Primary Flow of Execution](#21-primary-flow-of-execution)
    *   [2.2. Major Modules, Classes, and Responsibilities](#22-major-modules-classes-and-responsibilities)
    *   [2.3. Component Interaction Diagram](#23-component-interaction-diagram)
3.  [Game Interaction & Automation](#3-game-interaction--automation)
    *   [3.1. Interaction Mechanisms](#31-interaction-mechanisms)
    *   [3.2. Key Game Commands Sent (by Core)](#32-key-game-commands-sent-by-core)
    *   [3.3. Important Game Text Patterns Reacted To (by Core)](#33-important-game-text-patterns-reacted-to-by-core)
    *   [3.4. Automation Loops & State Machines (Core)](#34-automation-loops--state-machines-core)
4.  [Data Storage & Persistence](#4-data-storage--persistence)
    *   [4.1. Lich Variables (UserVars/Vars)](#41-lich-variables-uservarsvars)
    *   [4.2. Configuration Files & Databases](#42-configuration-files--databases)
    *   [4.3. In-Memory Data Structures](#43-in-memory-data-structures)
5.  [Configuration](#5-configuration)
    *   [5.1. Command-Line Arguments](#51-command-line-arguments)
    *   [5.2. Lich Settings (Database)](#52-lich-settings-database)
    *   [5.3. User Variables](#53-user-variables)
    *   [5.4. Script Settings](#54-script-settings)
6.  [Lich Commands & User Interface](#6-lich-commands--user-interface)
    *   [6.1. Built-in Lich Commands](#61-built-in-lich-commands)
    *   [6.2. Graphical User Interface (GUI)](#62-graphical-user-interface-gui)
7.  [Event Handling & Recurring Tasks](#7-event-handling--recurring-tasks)
    *   [7.1. Game Event Handling](#71-game-event-handling)
    *   [7.2. Recurring Tasks](#72-recurring-tasks)
8.  [External Dependencies & Integrations](#8-external-dependencies--integrations)
    *   [8.1. Ruby Gems](#81-ruby-gems)
    *   [8.2. Lich Script Interactions (Core)](#82-lich-script-interactions-core)
    *   [8.3. EAccess Authentication (Simutronics Account Portal)](#83-eaccess-authentication-simutronics-account-portal)
    *   [8.4. Logging into Game Character (Post-EAccess)](#84-logging-into-game-character-post-eaccess)
9.  [Error Handling & Logging](#9-error-handling--logging)
    *   [9.1. Error Handling Patterns](#91-error-handling-patterns)
    *   [9.2. Logging Mechanisms](#92-logging-mechanisms)
10. [Security Considerations](#10-security-considerations)
    *   [10.1. Credential Handling](#101-credential-handling)
    *   [10.2. Script Trust Model (Legacy)](#102-script-trust-model-legacy)
    *   [10.3. File System and Registry Access](#103-file-system-and-registry-access)
11. [Other Important Logic](#11-other-important-logic)
    *   [11.1. XML Parsing (`XMLData`)](#111-xml-parsing-xmldata)
    *   [11.2. Map System (`Map`, `Room`)](#112-map-system-map-room)
    *   [11.3. Game Object Management (`GameObj`)](#113-game-object-management-gameobj)
    *   [11.4. Hooks (`UpstreamHook`, `DownstreamHook`)](#114-hooks-upstreamhook-downstreamhook)
12. [Setup and Installation (Initial Observations)](#12-setup-and-installation-initial-observations)
13. [Assumptions and Areas for Clarification](#13-assumptions-and-areas-for-clarification)

---

## 1. High-Level Overview & Script Purpose

### 1.1. Primary Purpose
Lich is a sophisticated third-party assistant, proxy, and scripting engine specifically tailored for enhancing the gameplay experience of Simutronics' text-based MUDs, with a strong focus on **GemStone IV (GSIV)**. Its primary goals are:
*   **Automation:** To automate repetitive or complex in-game tasks through user-written Ruby scripts.
*   **Information Management:** To parse and interpret the game's data stream (often XML), providing scripts and users with a structured understanding of the game state, character status, environment, and more.
*   **Gameplay Enhancement:** To offer tools and utilities that extend the capabilities of the standard game clients, such as advanced mapping, combat assistance, and custom user interfaces.
*   **Proxy Functionality:** To act as an intermediary between the player's game client (e.g., Stormfront, Wizard) and the Simutronics game servers, allowing it to intercept, analyze, and potentially modify the data flow.

### 1.2. Main Mode of Operation
Lich operates as a **persistent background application** while a user is playing a supported game.
*   It functions as a **local proxy server**. The player configures their game client to connect to Lich, and Lich, in turn, connects to the actual game server.
*   It's **event-driven**, reacting to data received from the game server and commands from the game client.
*   It supports **concurrent script execution**, allowing multiple scripts to run simultaneously, each in its own thread.
*   It can be a **utility library** for other scripts (many of its core modules like `Infomon`, `Effects`, `Map` are used this way) and a **standalone scripting host**.

### 1.3. Initiation and Usage
Lich is typically initiated in one of the following ways:
*   **Direct Execution:** Running the main Lich Ruby script (e.g., `ruby lich.rbw`) from the command line.
*   **GUI Launch:** If GTK is available and no restrictive command-line arguments are given, Lich presents a graphical login and management interface.
*   **Simutronics Game Entry (SGE) / .sal File Association:** On Windows (or WINE), Lich can be configured to launch automatically when a user attempts to start a game from the Simutronics website, by handling `.sal` (Simutronics Authentication Launch) files.
*   **Script Commands:** Users interact with Lich and its scripts by typing commands prefixed with a special character (usually `;` or `,` by default) into their game client.
*   **Automatic Execution:** Some scripts can be configured (via `autostart.lic`) to run automatically upon successful login to the game.

### 1.4. Key Entry Points
*   **`lich.rbw` (Implied Main Script):** The top-level script that bootstraps the Lich environment. The execution begins here.
*   **`main/main.rb` - `@main_thread`:** This is the central thread created after initial setup. It orchestrates the login process (EAccess or SAL file), sets up the client listener and game server connection, and initiates the primary communication loops.
*   **`Lich::Common::gui_login` (`common/gui-login.rb`):** If the GTK GUI is used for login, this function handles the user interface and subsequent EAccess authentication.
*   **`Lich::GameBase::Game.open` (`games.rb`):** Establishes the TCP connection to the Simutronics game server.
*   **`client_thread` (in `main/main.rb`):** The thread dedicated to reading commands from the user's game client.
*   **`Game.thread` (in `games.rb`, started by `Game.open`):** The thread dedicated to reading data from the game server.
*   **`do_client(client_string)` (`global_defs.rb`):** Parses commands received from the game client; directs Lich commands to internal handlers (like `Script.start`) and game commands to the server.

---

## 2. Code Flow & Key Components

### 2.1. Primary Flow of Execution
1.  **Initialization (`lich.rbw` and `init.rb`):**
    *   Parses command-line arguments (`main/argv_options.rb`).
    *   Defines core constants and directory paths (`constants.rb`).
    *   Loads essential Ruby gems (`require`).
    *   Checks for required dependencies like SQLite3 and GTK3, prompting for installation if missing and feasible.
    *   Initializes the main SQLite database (`Lich.db` in `lich.rb`).
    *   Creates an instance of `Lich::Common::XMLParser` as `XMLData`.

2.  **Login and Connection Setup (primarily in `@main_thread` within `main/main.rb`):**
    *   **Authentication/Launch Data:**
        *   If `--login`: Uses `Lich::Common::EAccess.auth` to get launch parameters.
        *   If `.sal` file: Parses the file for launch parameters.
        *   If GUI: `gui_login` is called, which uses `EAccess.auth`.
        *   Resulting launch parameters (including game server host, port, and the crucial `AUTH_KEY`) are stored in `@launch_data`.
    *   **Proxy Initialization:**
        *   A local `TCPServer` is started by Lich.
        *   The actual game client (e.g., Stormfront) is launched, configured to connect to Lich's local server.
        *   Lich waits for the game client to connect (`listener.accept`), creating `$_CLIENT_`.
    *   **Game Server Connection:**
        *   Lich connects to the actual game server specified in `@launch_data` using `Lich::GameBase::Game.open`. This creates `$_SERVER_`.

3.  **Handshake and Session Start:**
    *   The `client_thread` (in `main/main.rb`) is started. It reads the initial handshake data (Auth Key, Client ID string) from `$_CLIENT_` and forwards it to `$_SERVER_` via `Game._puts`.
    *   For GemStone IV with Stormfront/Wizard, it often sends `<c>\r\n` commands after the handshake to signal readiness.
    *   The `Game.thread` (in `games.rb`) is started. It reads data from `$_SERVER_`.

4.  **Main Loop - Data Proxying and Script Interaction:**
    *   **Downstream (Server -> Lich -> Client):**
        1.  `Game.thread` reads raw data from `$_SERVER_`.
        2.  The data (often XML) is passed to `XMLData.tag_start/text/tag_end` for parsing. This updates Lich's internal state (character stats, room info, `GameObj`s, active spells via `Effects`, `Infomon` data).
        3.  `Script.new_downstream_xml` and `Script.new_downstream` make the raw and stripped data available to running scripts.
        4.  `DownstreamHook.run` applies any script-registered modifications to the server string.
        5.  The (potentially modified) string is sent to `$_CLIENT_`.
    *   **Upstream (Client -> Lich -> Server):**
        1.  `client_thread` reads data from `$_CLIENT_`.
        2.  The data is passed to `do_client`.
        3.  If it's a Lich command (e.g., `;#{$lich_char_regex}start_script`), `do_client` handles it.
        4.  If it's a game command, `UpstreamHook.run` applies script modifications.
        5.  The (potentially modified) command is sent to `$_SERVER_` via `Game._puts`.

5.  **Script Execution:**
    *   Scripts run in their own threads, managed by `Lich::Common::Script`.
    *   They interact with the game using Lich's API (`fput`, `waitfor`, `GameObj[]`, `XMLData`, `Stats`, `Skills`, `Spells`, etc.).

6.  **Shutdown:**
    *   When connections close or Lich exits, it attempts to kill running scripts, save settings (`Settings.save`, `Vars.save`), and close sockets.

### 2.2. Major Modules, Classes, and Responsibilities

*   **`Lich` (Module, `lich.rb`):**
    *   Top-level namespace.
    *   Global utilities (`Lich.log`, `Lich.msgbox`), SGE/SAL linking, hosts file management.
    *   Manages `lich.db3`.
    *   Accessor for global Lich display settings (e.g., `Lich.display_lichid`).
*   **`Lich::Common::XMLParser` (Global instance: `XMLData`, `common/xmlparser.rb`):**
    *   **Responsibility:** Parses XML data from the game server using `REXML::StreamListener`.
    *   Maintains the primary in-memory representation of the game state (character status, room details, active effects, inventory, etc.).
    *   Crucial for nearly all script functionality that relies on current game information.
*   **`Lich::Common::Script` (and subclasses `ExecScript`, `WizardScript`) (`common/script.rb`):**
    *   **Responsibility:** Manages the lifecycle of all user scripts.
    *   Handles script loading, execution (label-based for `.lic`/`.wiz`, `eval` for `.rb`), argument passing, and script-specific variables (`@vars`).
    *   Manages script states (running, paused, hidden) and inter-script communication (`die_with_me`).
*   **`Lich::GameBase::Game` (Game-specific: `Lich::Gemstone::Game`) (`games.rb`):**
    *   **Responsibility:** Manages the TCP socket connection to the actual game server.
    *   Contains the `Game.thread` that reads from the server and initiates XML parsing via `XMLData`.
    *   Provides `_puts` (raw send) and `puts` (command-prefixed send) to the game server.
*   **`Lich::Common::GameObj` (`common/gameobj.rb`):**
    *   **Responsibility:** Represents and manages in-game objects (items in room/inventory/hands, NPCs, PCs).
    *   Stores these objects in class-level arrays (e.g., `GameObj.inv`, `GameObj.loot`, `GameObj.npcs`).
    *   Provides methods to query and access these objects.
*   **`Lich::Common::Map` (and `Room`) (`common/map/map_gs.rb`):**
    *   **Responsibility:** Manages game world map data, including room details and pathfinding.
    *   Loads data from map database files.
    *   `Map.current` provides the current room object. `Map.findpath` uses Dijkstra's algorithm.
*   **`Lich::Common::EAccess` (`common/eaccess.rb`):**
    *   **Responsibility:** Handles authentication with Simutronics EAccess servers to obtain game session keys.
*   **`Lich::Common::Settings`, `Lich::Common::CharSettings`, `Lich::Common::GameSettings` (`common/settings/*.rb`):**
    *   **Responsibility:** Provide persistent key-value storage, scoped to scripts, characters, or games, using `lich.db3`.
    *   `SettingsProxy` allows for natural hash/array-like manipulation with auto-saving.
*   **`Lich::Common::Vars` / `Lich::Common::UserVars` (`common/vars.rb`, `common/uservars.rb`):**
    *   **Responsibility:** Global persistent key-value store for user variables, saved per character to `lich.db3`.
*   **`Lich::Gemstone::Infomon` (`gemstone/infomon.rb`):**
    *   **Responsibility (GSIV specific):** Parses detailed character information (stats, skills, experience, society, resources) from `INFO`, `EXP`, `SKILL`, etc., commands and stores it in `infomon.db`. Provides an API for scripts to access this data.
*   **`Lich::Gemstone::Effects` (`gemstone/effects.rb`):**
    *   **Responsibility (GSIV specific):** Tracks active spell effects, buffs, debuffs, and cooldowns based on `XMLData.dialogs`.
*   **`Lich::Common::Spell` (`common/spell.rb`):**
    *   **Responsibility:** Represents individual spells, their properties (cost, duration, messages), and provides methods for casting and checking status. Loads data from `effect-list.xml`.
*   **Hooks (`Lich::Common::UpstreamHook`, `Lich::Common::DownstreamHook`):**
    *   **Responsibility:** Allow scripts to intercept and modify client-server communication.

### 2.3. Component Interaction Diagram

```mermaid
graph TD
    A[User's Game Client (e.g., Stormfront)] <-->|TCP to Lich Port| B(Lich Core);
    B <-->|SSL/TCP| C[Simu EAccess Server];
    B <-->|TCP| D[Simu Game Server (e.g., GSIV)];

    subgraph Lich Core
        E[@main_thread] --> F[Login Logic / GUI];
        F --> G[EAccess Module];
        G --> C;
        E --> H[Game Module];
        H --> D;
        H --> I[XMLData Parser];
        D --> I;
        I --> J[Infomon/Effects/Skills (GSIV)];
        J --> K[infomon.db / lich.db3];
        E --> L[Client Thread];
        A --> L;
        L --> M[do_client handler];
        M --> H; subgraph Game Command
        M --> N[Script Engine]; subgraph Lich Command
        N --> O[Running Scripts];
        O --> P[Lich API (fput, waitfor, XMLData, etc.)];
        P --> M;
        P --> I;
        I --> P;
        O --> Q[Settings/Vars Modules];
        Q --> K;
        O <--> R[Map Module];
        R --> S[Map Database Files];
        P --> T[Hooks (Up/Downstream)];
        I --> T;
        T --> A; 
        H --> T;
    end
```

---

## 3. Game Interaction & Automation

### 3.1. Interaction Mechanisms
*   **Sending Commands:**
    *   `put "command string"`: Sends a command to the game server, prefixed by `$cmd_prefix` (e.g., `<c>`). Output is visible to the user.
    *   `fput "command string"`: Similar to `put`, but attempts to manage roundtime and basic error handling/retries (e.g., if stunned or hands full). It also clears the script's input buffer before sending.
    *   `Game._puts "raw string"`: Lower-level send to the game server, without Lich's command prefix. Used internally and by some advanced scripts.
*   **Matching Game Text:**
    *   `waitfor "text"`, `waitforre /regex/`: Blocks script execution until the specified text or regex is received from the server.
    *   `match "label", "text"`, `matchre "label", /regex/`: Sets up a match condition. When "text" or /regex/ is found, execution jumps to "label". Used with `matchwait`.
    *   `matchtimeout sec, "text"`: Like `waitfor`, but with a timeout.
    *   `get`, `get?`: Reads a line from the script's downstream buffer.
    *   `Lich::Messaging.msg(...)`: Can be used by scripts to send formatted messages to the Lich client window, not directly to the game.
*   **Lich Client Variables/Functions:**
    *   `XMLData`: Access to parsed game state (e.g., `XMLData.room_title`, `XMLData.mana`).
    *   `Infomon` (GSIV): Access to detailed character stats, skills, experience (e.g., `Stats.strength`, `Skills.combat_maneuvers`, `Experience.fame`).
    *   `Effects` (GSIV): Check active buffs, debuffs, cooldowns (`Effects::Spells.active?(101)`).
    *   `Spell`: Query spell properties, cast spells (`Spell[101].cast`).
    *   `GameObj`: Access lists of items, NPCs, PCs (`GameObj.inv`, `GameObj.loot`, `GameObj.npcs`).
    *   `Room` / `Map`: Access current room info and pathfinding (`Room.current.id`, `Map.findpath(...)`).
    *   `Char`: Wrapper for common character attributes (`Char.health`, `Char.stance`).
    *   Status checking functions: `dead?`, `stunned?`, `bleeding?`, `muckled?`, etc.

### 3.2. Key Game Commands Sent (by Core Lich for GSIV)
*   **During EAccess (`Lich::Common::EAccess.auth` to EAccess server):**
    *   `K` (Request hashkey)
    *   `A\t<account>\t<obfuscated_pass>` (Authenticate)
    *   `M` (List games)
    *   `F\t<game_code>` (Frontend info)
    *   `G\t<game_code>` (Game info)
    *   `P\t<game_code>` (Product info)
    *   `C` (Character list)
    *   `L\t<char_code>\t<CLIENT_TYPE>` (Login character, get launch params)
*   **During Game Server Handshake (to game server like `storm.gs4.game.play.net`):**
    *   The `AUTH_KEY` (obtained from EAccess `L` command response).
    *   Client ID string (e.g., `/FE:STORMFRONT /VERSION:1.x /P:platform /XML`).
    *   `<c>` (Often sent once or twice to signal readiness, especially for non-Stormfront frontends or `--without-frontend`).
*   **Internal/Status Commands (often sent by Infomon sync or core utilities):**
    *   `INFO`
    *   `EXP ALL` (or specific skills)
    *   `SKILL ALL` (or specific skills)
    *   `SPELL ALL` (or specific spells)
    *   `CMAN LIST ALL` (and for other PSM types like ARMOR, SHIELD, FEAT, WEAPON, ASCENSION)
    *   `SOCIETY INFO`
    *   `CITIZENSHIP`
    *   `RESOURCE`
    *   `BANK ACCOUNT`
    *   `BOUNTY`
    *   `GROUP`
    *   `INVENTORY (ARMOR|WEAPON|ETC)`
    *   `READY LIST`
    *   `STOW LIST`
    *   `_FLAG DISPLAY DIALOG BOXES 0` (for Wizard/Avalon)
    *   `_INJURY 2` (for Wizard/Avalon to get wound/scar data)
    *   `PROFILE` (for CHE/Society info if GSIV)

### 3.3. Important Game Text Patterns Reacted To (by Core Lich/XMLData/Infomon for GSIV)
*   **XML Tags:** The entire XML stream is parsed. Key tags include:
    *   `<app char="...">`: Character login confirmation, triggers autostart scripts.
    *   `<roundTime value="...">`: Updates roundtime.
    *   `<castTime value="...">`: Updates cast roundtime.
    *   `<spell>...</spell>`: Updates prepared spell.
    *   `<streamWindow id="main" subtitle="...">`: Extracts room title and UID.
    *   `<prompt time="...">...</prompt>`: Updates server time.
    *   `<dialogData id="(Active Spells|Buffs|Debuffs|Cooldowns)" ...>`: Updates `Effects` module.
    *   `<progressBar id="(mana|health|stamina|spirit|pbarStance|mindState|encumlevel|concentration)" ...>`: Updates character vitals and states.
    *   `<indicator id="..." visible="(y|n)">`: Updates status indicators (dead, stunned, poisoned, etc.).
    *   `<nav rm="...">`: Indicates room change, triggers clearing of `GameObj` room lists.
    *   `<component id="room (objs|players|exits|desc)">`: Wraps room content.
    *   `<inv id="...">`, `<a exist="..." noun="...">`, `<rightHand>`, `<leftHand>`: Populate `GameObj` inventory and hands.
    *   `<image id="..." name="...">` (within `<dialogData id="injuries">`): Updates wound/scar info.
*   **Plain Text (parsed by `Infomon::Parser` for GSIV):**
    *   Output of `INFO` command (stats, level, experience).
    *   Output of `EXP` command (experience points, field experience, LTE, deeds).
    *   Output of `SKILL` command (skill ranks and bonuses).
    *   Output of `SPELL` command (spell ranks).
    *   Output of `CMAN LIST ALL` (and other PSM list commands).
    *   Citizenship messages (`You currently have ... citizenship in ...`).
    *   Society messages (`You are a ... in the ...`).
    *   Resource/currency messages (`Essence: ...`, `Voln Favor: ...`, `You have ... silver`).
    *   Spell up/down messages (from `effect-list.xml`, e.g., `Your skin grows hard as stone.`).
    *   Various status effect messages (e.g., "Your mind goes completely blank." for sleep).

### 3.4. Automation Loops & State Machines (Core)
*   **Main Proxy Loops:** The `Game.thread` and `client_thread` are the primary continuous loops for data relay.
*   **Script Execution:** Each script runs in its own thread, often containing its own `loop { ... }` or event-driven logic.
*   **`waitfor`, `matchwait`, `wait_while`, `wait_until`:** These methods create blocking loops within scripts, waiting for specific game states or text.
*   **`Infomon::Parser::State`:** A simple state machine (`:ready`, `:goals`, `:profile`) used by `Infomon` to correctly parse multi-line command outputs like `INFO` or `SKILL`.
*   **`XMLData` Stream/Style Tracking:** `@current_stream` and `@current_style` track the context of XML text to parse it correctly (e.g., text within `<streamWindow id="familiar">` is handled differently).

---

## 4. Data Storage & Persistence

### 4.1. Lich Variables (UserVars/Vars)
*   **Module:** `Lich::Common::Vars` (aliased as `UserVars` for script use).
*   **Storage:** `uservars` table in `DATA_DIR/lich.db3`. Scoped by "GameName:CharacterName".
*   **Mechanism:** Stores a Ruby Hash, serialized using `Marshal`. Saved to DB periodically (every 5 minutes) and on graceful Lich exit.
*   **Purpose:** Allows scripts and users to store and retrieve arbitrary persistent data specific to a character.
*   **Key Variables (Examples set by core or common scripts):**
    *   `UserVars.lootsack`: Name of the container to use for general loot.
    *   `UserVars.skinloot`: Container for skins.
    *   `UserVars.gemloot`: Container for gems.
    *   `UserVars.boxloot`: Container for boxes.
    *   `UserVars.scriptarguments`: Arguments passed to the last run script.
    *   Many scripts define their own `UserVars` for configuration.

### 4.2. Configuration Files & Databases
*   **`DATA_DIR/lich.db3`:**
    *   **`script_auto_settings` table:** Primary storage for `Settings`, `CharSettings`, `GameSettings`. Stores Marshal-dumped Hashes.
    *   **`lich_settings` table:** Global Lich configurations (text values).
    *   **`entry.dat`:** (File, not DB table) Encrypted login profiles from GUI.
*   **`DATA_DIR/infomon.db` (GSIV):**
    *   SQLite database managed by Sequel.
    *   Stores detailed character data (stats, skills, exp, society) in character-specific tables. Values stored as native types where possible.
*   **Map Files (`DATA_DIR/<GameName>/map-*.json` or `.dat`, `.xml`):**
    *   Stores room data, including titles, descriptions, exits, tags, UIDs, etc.
*   **`DATA_DIR/effect-list.xml` (GSIV):**
    *   XML definition for spells, their messages, costs, durations, and effects.
*   **`DATA_DIR/gameobj-data.xml` (GSIV):**
    *   XML definition for item type classification regexes.
*   **Script-Specific Files (`DATA_DIR/<script_name>.<ext>`, `DATA_DIR/<script_name>.db3`):**
    *   Scripts can create their own files (YAML, JSON, text, SQLite) in `DATA_DIR` for custom data persistence.

### 4.3. In-Memory Data Structures (Central to Core State)
*   **`Lich::Common::XMLData` (global instance `XMLData`):**
    *   Holds numerous instance variables representing the live game state (e.g., `@mana`, `@health`, `@room_title`, `@room_exits`, `@active_spells`, `@injuries`, `@indicator`).
    *   `@dialogs`: A hash storing the content of XML `<dialogData>` tags, key for `Effects` module.
*   **`Lich::Common::GameObj`:**
    *   Class variables store arrays of game objects: `@@inv` (inventory), `@@loot` (room items), `@@npcs`, `@@pcs`, `@@contents` (container contents), `@@right_hand`, `@@left_hand`.
*   **`Lich::Common::Map`:**
    *   `@@list`: Array of `Map`/`Room` objects.
    *   `@@uids`: Hash mapping game UIDs to Lich Room IDs.
*   **`Lich::Common::Spell`:**
    *   `@@list`: Array of `Spell` objects, loaded from `effect-list.xml`. Each object stores spell properties and its current active status/timeleft.
*   **`Lich::Gemstone::Infomon` (GSIV):**
    *   `@cache`: An in-memory cache (`Infomon::Cache`) of data read from `infomon.db` to reduce DB lookups.
*   **`Script` Instances:**
    *   Each running script object holds its own state: `@vars` (command-line arguments), `@watchfor` patterns, `@downstream_buffer`, etc.

---

## 5. Configuration

### 5.1. Command-Line Arguments
*   See section [1.3. Initiation and Usage](#13-initiation-and-usage) and `main/argv_options.rb`.
*   Examples: `--login MyChar --gemstone --platinum`, `-g storm.gs4.game.play.net:10124`, `--scripts=/alt/scripts/dir`, `--start-scripts=myscript,another`.

### 5.2. Lich Settings (Database - `lich_settings` table in `lich.db3`)
*   Managed via `Lich.<setting_name> = value` or `;#{$lich_char_regex}set <setting_name> (on|off)`.
*   **Key Settings:**
    *   `display_lichid` (boolean): Show Lich map IDs in room announcements. Default: true for GSIV, false for DR.
    *   `display_uid` (boolean): Show game unique room IDs. Default: true for GSIV, false for DR.
    *   `display_exits` (boolean): Show additional known room exits. Default: false.
    *   `display_stringprocs` (boolean): Show exits that are dynamic `StringProc`s. Default: false.
    *   `track_dark_mode` (boolean): Persists dark mode preference for GUI.
    *   `track_layout_state` (boolean): Persists tabbed layout preference for GUI.
    *   `track_autosort_state` (boolean): Persists auto-sort preference for GUI saved logins.
    *   `debug_messaging` (boolean): Enables/disables certain Lich debug messages to client.
*   Effect: These settings control global Lich behavior, primarily related to information display and client integration.

### 5.3. User Variables (`UserVars` / `Vars`)
*   Managed via `UserVars.var_name = value` in scripts or `;#{$lich_char_regex}vars set var_name value`.
*   Stored in `uservars` table in `lich.db3`, scoped by "GameName:CharacterName".
*   Purpose: Allows users and scripts to define and persist custom variables.
*   Effect: Entirely script-dependent. Examples: `UserVars.lootsack` defines the default container for looting scripts.

### 5.4. Script Settings (`Settings`, `CharSettings`, `GameSettings`)
*   Managed via `Settings[key] = value`, `CharSettings[key] = value`, etc.
*   Stored in `script_auto_settings` table in `lich.db3`.
*   Purpose: Provides a structured and scoped way for scripts to save their own configurations.
    *   `Settings`: Scoped by script name (global to that script across characters/games).
    *   `CharSettings`: Scoped by script name AND "GameName:CharacterName".
    *   `GameSettings`: Scoped by script name AND "GameName".
*   Effect: Entirely script-dependent. Scripts use this to remember user preferences, states, or learned data.

---

## 6. Lich Commands & User Interface

### 6.1. Built-in Lich Commands
*   All commands are prefixed by `;#{$lich_char_regex}` (e.g., `;` or `,`).
*   **Script Management:**
    *   `;#{$lich_char_regex}<script_name> [args]`: Starts `<script_name>` with optional `[args]`.
        *   Example: `;#{$lich_char_regex}go2 town`
    *   `;#{$lich_char_regex}force <script_name> [args]`: Starts script even if already running.
    *   `;#{$lich_char_regex}k[ill] <script_name>` or `;#{$lich_char_regex}stop <script_name>`: Kills a running script.
    *   `;#{$lich_char_regex}k[ill]` or `;#{$lich_char_regex}stop`: Kills the most recently started script.
    *   `;#{$lich_char_regex}p[ause] <script_name>`: Pauses a script.
    *   `;#{$lich_char_regex}p[ause]`: Pauses the most recently started, unpaused script.
    *   `;#{$lich_char_regex}u[npause] <script_name>`: Unpauses a paused script.
    *   `;#{$lich_char_regex}u[npause]`: Unpauses the most recently paused script.
    *   `;#{$lich_char_regex}ka` or `;#{$lich_char_regex}kill all`: Kills all killable scripts.
    *   `;#{$lich_char_regex}pa` or `;#{$lich_char_regex}pause all`: Pauses all pausable scripts.
    *   `;#{$lich_char_regex}ua` or `;#{$lich_char_regex}unpause all`: Unpauses all pausable scripts.
    *   `;#{$lich_char_regex}l[ist]`: Lists currently running, non-hidden scripts.
    *   `;#{$lich_char_regex}la` or `;#{$lich_char_regex}list all`: Lists all running scripts, including hidden ones.
*   **Execution & Debugging:**
    *   `;#{$lich_char_regex}e[xec] <code>`: Executes Ruby `<code>` in a temporary script environment.
    *   `;#{$lich_char_regex}eq[xecq] <code>`: Same as `exec` but quieter (no start/exit messages).
    *   `;#{$lich_char_regex}en[execname] <name> <code>`: Executes `<code>` in a named temporary script.
    *   `;#{$lich_char_regex}send <line>`: Sends `<line>` to all running scripts' downstream buffers.
    *   `;#{$lich_char_regex}send to <script_name> <line>`: Sends `<line>` to a specific script.
    *   `;#{$lich_char_regex}hmr <regex_pattern>`: Hot Module Reload matching files.
*   **Configuration & Info:**
    *   `;#{$lich_char_regex}set <variable_name> (on|off)`: Sets a global Lich boolean setting.
    *   `;#{$lich_char_regex}lich5-update --<subcommand>`: Manages Lich updates (see `;#{$lich_char_regex}lich5-update --help`).
    *   `;#{$lich_char_regex}infomon sync` (GSIV): Resynchronizes `Infomon` data.
    *   `;#{$lich_char_regex}infomon reset` (GSIV): Resets and resynchronizes `Infomon` data.
    *   `;#{$lich_char_regex}infomon show [full]` (GSIV): Displays stored `Infomon` data.
    *   `;#{$lich_char_regex}infomon effects [true|false]` (GSIV): Toggles display of spell effect durations by `magic` command.
    *   `;#{$lich_char_regex}display (lichid|uid|exits|stringprocs) [true|false]`: Toggles various display settings.
    *   `;#{$lich_char_regex}help`: Displays this help information.
*   **Script Trust (Ruby < 2.3):**
    *   `;#{$lich_char_regex}trust <script_name>`
    *   `;#{$lich_char_regex}distrust <script_name>`
    *   `;#{$lich_char_regex}lt` or `;#{$lich_char_regex}list trusted`

### 6.2. Graphical User Interface (GUI)
*   Located in `common/gui-*.rb` files, primarily `gui-login.rb`, `gui-saved-login.rb`, `gui-manual-login.rb`.
*   **Triggered:** If GTK3 is available and Lich is started without arguments that bypass the GUI (like `--login` or `--without-frontend`).
*   **Functionality:**
    *   **Login:**
        *   "Saved Entry" tab: Lists characters from `entry.dat` for quick login. Allows removal of entries.
        *   "Manual Entry" tab: Allows manual input of account ID, password. Connects to EAccess to fetch game/character list, then allows selection and login.
        *   Options to save manual entries, specify frontend (Stormfront, Wizard, Avalon), and use custom launch commands.
    *   **Global GUI Settings:** Toggles for dark mode, tabbed layout, and auto-sort for saved logins.
*   **Interaction:** The GUI runs in the main thread. Actions (like button clicks) trigger Ruby Procs that perform EAccess calls, update `@launch_data`, and eventually close the GUI to proceed with game connection.

---

## 7. Event Handling & Recurring Tasks

### 7.1. Game Event Handling
*   **`Watchfor` Class (`common/watchfor.rb`):**
    *   Allows scripts to register a Regexp pattern and a Proc (code block).
    *   `Script.new_downstream` (called from `Game.thread`) iterates through active `watchfor`s for each script. If a line matches a script's `watchfor` pattern, the associated Proc is executed in a new thread.
    *   Syntax: `watchfor /pattern/, proc { #... code ... }` or `watchfor("string") { #... code ... }`
*   **`match`, `matchre`, `matchwait` (`global_defs.rb`):**
    *   These are script-level blocking methods that pause script execution until a matching line is received.
    *   `match` and `matchre` can also set up a "match stack" (`@match_stack_labels`, `@match_stack_strings`) which `matchwait` (with no arguments) uses to check multiple patterns and `goto` a corresponding label.
*   **XML Parsing (`XMLData`):**
    *   Acts as a large-scale event handler for XML tags. `tag_start`, `text`, and `tag_end` methods are called by the REXML stream parser, triggering updates to internal state variables. Many scripts then poll these `XMLData` variables or rely on game-specific modules (like `Infomon`, `Effects`) that are updated by `XMLData`.

### 7.2. Recurring Tasks
*   **Main I/O Loops:**
    *   `Game.thread` (in `games.rb`): Continuously reads from game server. Interval is network-dependent.
    *   `client_thread` (in `main/main.rb`): Continuously reads from game client. Interval is client-input-dependent.
*   **`UserVars` Auto-Save (`common/vars.rb`):**
    *   A background thread that calls `Vars.save` every 300 seconds (5 minutes).
*   **`ActiveSpell` Update (`gemstone/infomon/activespell.rb` for GSIV):**
    *   `ActiveSpell.watch!` creates a thread that waits on a `Queue`.
    *   `ActiveSpell.request_update` pushes to this queue, typically triggered by `XMLData` when `<dialogData id="Active Spells">` (or Buffs, Debuffs, Cooldowns) is received or cleared.
    *   This updates the active status and remaining time for `Spell` objects.
*   **GTK Idle Loop (`common/gtk.rb`):**
    *   If GTK is active, `GLib::Idle.add` is set up to `sleep 0.01` repeatedly when GTK is idle. This helps keep the Ruby interpreter responsive and allows other threads to process.
*   **Script-Defined Loops:** Individual scripts frequently implement their own `loop { ... sleep X ... }` constructs for recurring checks or actions.

---

## 8. External Dependencies & Integrations

### 8.1. Ruby Gems
*   **`base64`:** Used for encoding/decoding, e.g., in `entry.dat`.
*   **`digest/md5`**, **`digest/sha1`:** Cryptographic hashes, potentially used for data integrity or older security mechanisms. `MD5` is used in `XMLData` for generating a room ID if no game UID is available.
*   **`drb/drb`:** Distributed Ruby. Its core usage isn't prominent in the provided files but might be used by extensions or specific scripts for inter-process communication.
*   **`json`:** For parsing and generating JSON (e.g., map files, Genie session files, `LNet` data).
*   **`monitor`:** Provides `Mutex` and `ConditionVariable` for thread synchronization (though `Mutex` is often used directly).
*   **`net/http`:** For making HTTP requests (e.g., `Lich::Util::Update` fetches updates from GitHub).
*   **`openssl`:** For SSL/TLS connections (crucial for `EAccessClient`) and cryptographic functions.
*   **`ostruct`:** `OpenStruct` is used for creating flexible, hash-like objects (e.g., in `Stats`, `Skills`).
*   **`resolv`:** Network hostname resolution.
*   **`rexml/document`**, **`rexml/streamlistener`:** Core XML parsing libraries used by `XMLData`.
*   **`socket`:** Low-level networking.
*   **`sqlite3`:** Interface to SQLite3 databases (`lich.db3`, `infomon.db`, script-specific DBs).
*   **`stringio`:** In-memory I/O streams.
*   **`terminal-table`:** Used by some scripts/core functions to format output as tables for the client.
*   **`time`:** Standard time/date operations.
*   **`yaml`:** For parsing YAML configuration files (used by many scripts, though not heavily by Lich core for its *own* primary config).
*   **`zlib`:** For handling gzipped script files (`.lic.gz`, `.rb.gz`).
*   **`gtk3` (Optional):** For the graphical user interface.

### 8.2. Lich Script Interactions (Core)
*   **`autostart.lic`:** If present, this script is automatically started by Lich after a successful login. It's typically used to launch other essential or user-preferred scripts.
*   **Repository Scripts (e.g., `repository.lic`, `update.lic`):** Used for managing and updating Lich scripts and core files. `Lich::Util::Update` interacts with these concepts.
*   **`LNet` (`lnet.lic`):** If running, provides inter-Lich communication capabilities, which other scripts might use.
*   Scripts can start other scripts (`Script.start`, `start_script`).
*   Scripts can send messages to other scripts (`send_to_script`, `unique_send_to_script`).

### 8.3. EAccess Authentication (Simutronics Account Portal)
*   **Component:** `Lich::Common::EAccess` (`lib/common/eaccess.rb`).
*   **Pseudo-code:**
    ```pseudo
    FUNCTION EACCESS_AUTH(account_name, password, target_character_name, target_game_code, target_client_type="STORM"):
        _logger.LogInformation("Starting EAccess authentication for {account_name}, char: {target_character_name}, game: {target_game_code}")

        // 1. Establish Secure Connection
        SSL_SOCKET = connect_to_eaccess_server_via_ssl("eaccess.play.net", 7910)
        VERIFY_PEM_CERTIFICATE(SSL_SOCKET) // Ensure server is authentic

        // 2. Request Encryption Hashkey
        SEND "K\n" to SSL_SOCKET
        HASHKEY_RESPONSE = READ from SSL_SOCKET
        HASHKEY = PARSE HASHKEY_RESPONSE (trimming newlines)
        IF HASHKEY is empty:
            THROW EAccessProtocolException("Failed to receive hashkey")
        END IF
        _logger.LogDebug("Received HASHKEY (length: {HASHKEY.length})")

        // 3. Obfuscate Password
        PASSWORD_BYTES = convert_password_to_ascii_bytes(password)
        HASHKEY_BYTES = convert_hashkey_to_ascii_bytes(HASHKEY)
        OBFUSCATED_PASSWORD_BYTES = array of same length as PASSWORD_BYTES
        FOR i from 0 to PASSWORD_BYTES.length - 1:
            OBFUSCATED_PASSWORD_BYTES[i] = ((PASSWORD_BYTES[i] - 32) XOR HASHKEY_BYTES[i]) + 32
        END FOR

        // 4. Send Authentication Request
        AUTH_COMMAND_STRING = "A\t" + account_name + "\t"
        SEND AUTH_COMMAND_STRING + OBFUSCATED_PASSWORD_BYTES (as string) + "\n" to SSL_SOCKET
        AUTH_RESPONSE = READ from SSL_SOCKET
        _logger.LogDebug("Auth Response: {AUTH_RESPONSE}")

        // 5. Check Authentication Success & Extract EAccess Session Key
        MATCH_DATA = REGEX_MATCH(AUTH_RESPONSE, /KEY\t(?<key>.*)\t/)
        IF NOT MATCH_DATA.success:
            ERROR_DETAIL = PARSE_ERROR_FROM_AUTH_RESPONSE(AUTH_RESPONSE)
            THROW EAccessAuthenticationException("Authentication failed: {ERROR_DETAIL}")
        END IF
        EACCESS_SESSION_KEY = MATCH_DATA.group("key")
        _logger.LogInformation("EAccess Session Key obtained.")

        // 6. Get Game List
        SEND "M\n" to SSL_SOCKET
        GAME_LIST_RESPONSE = READ from SSL_SOCKET
        IF NOT GAME_LIST_RESPONSE starts with "M\t":
            THROW EAccessProtocolException("Unexpected response after M command.")
        END IF
        // GAME_LIST_DATA = substring of GAME_LIST_RESPONSE after "M\t"

        // 7. Select Game & Get Character Code (Simplified for a single target game/char)
        // Lich's EAccess.auth in Ruby iterates if needed or directly uses provided game_code.
        // Here, we assume target_game_code is the one we want.

        SEND "F\t" + target_game_code + "\n" to SSL_SOCKET // Frontend/Subscription info
        F_RESPONSE = READ from SSL_SOCKET
        IF NOT F_RESPONSE matches /(NORMAL|PREMIUM|TRIAL|INTERNAL|FREE)/:
             THROW EAccessProtocolException("Invalid subscription status from F command: {F_RESPONSE}")
        END IF

        SEND "G\t" + target_game_code + "\n" to SSL_SOCKET // Game info
        READ from SSL_SOCKET // Discard G_RESPONSE

        SEND "P\t" + target_game_code + "\n" to SSL_SOCKET // Product info
        READ from SSL_SOCKET // Discard P_RESPONSE

        SEND "C\n" to SSL_SOCKET // Character list for the game
        C_RESPONSE = READ from SSL_SOCKET
        CHAR_DATA_STRING = PARSE_CHARACTER_LIST_HEADER(C_RESPONSE) // Remove "C\t<numbers>\t"
        TARGET_CHAR_CODE = FIND_CHAR_CODE(CHAR_DATA_STRING, target_character_name)
        IF TARGET_CHAR_CODE is not found:
            THROW EAccessException("Character '{target_character_name}' not found for game '{target_game_code}'.")
        END IF
        _logger.LogInformation("Found CharCode '{TARGET_CHAR_CODE}' for Character '{target_character_name}'.")

        // 8. Request Launch Parameters (including Game Server Key)
        SEND "L\t" + TARGET_CHAR_CODE + "\t" + target_client_type + "\n" to SSL_SOCKET
        LAUNCH_RESPONSE = READ from SSL_SOCKET
        _logger.LogDebug("Launch Response: {LAUNCH_RESPONSE}")

        MATCH_DATA = REGEX_MATCH(LAUNCH_RESPONSE, /^L\tOK\t(?<params_string>.*)/)
        IF NOT MATCH_DATA.success:
            THROW EAccessProtocolException("Failed to get launch parameters: {LAUNCH_RESPONSE}")
        END IF
        PARAMS_STRING = MATCH_DATA.group("params_string")

        // 9. Parse Launch Parameters
        LAUNCH_PARAMETERS_HASH = empty_hash
        FOR EACH key_value_pair_string in PARAMS_STRING.split_by_tab():
            KEY, VALUE = key_value_pair_string.split_by_equals_sign()
            LAUNCH_PARAMETERS_HASH[KEY.to_uppercase()] = VALUE
        END FOR

        CLOSE SSL_SOCKET
        _logger.LogInformation("EAccess authentication successful. Launch parameters obtained.")
        RETURN LAUNCH_PARAMETERS_HASH
    ENDFUNCTION
    ```

### 8.4. Logging into Game Character (Post-EAccess)
*   **Component:** Logic within `@main_thread` and `client_thread` in `main/main.rb`, and `Game` module in `games.rb`.
*   **Pseudo-code (Lich initiating the game server connection):**
    ```pseudo
    FUNCTION LOGIN_TO_GAME_SERVER(launch_parameters_hash):
        GAME_SERVER_HOST = launch_parameters_hash["GAMEHOST"]
        GAME_SERVER_PORT = launch_parameters_hash["GAMEPORT"].to_integer()
        GAME_SERVER_KEY  = launch_parameters_hash["KEY"] // This is the authKey

        // 1. Connect to Game Server
        GAME_SOCKET = TCP_CONNECT(GAME_SERVER_HOST, GAME_SERVER_PORT)
        _logger.LogInformation("Connected to game server: {GAME_SERVER_HOST}:{GAME_SERVER_PORT}")

        // 2. Send Game Server Key (AuthKey)
        _logger.LogDebug("Sending Game Server Key (first 20 chars): {GAME_SERVER_KEY.substring(0,20)}...")
        SEND GAME_SERVER_KEY + "\r\n" to GAME_SOCKET // Lich's Game._puts ensures \r\n

        // 3. Send Client Identification String
        CLIENT_ID_STRING = "/FE:STORMFRONT /VERSION:1.0.1.26 /P:<platform> /XML" // Or similar
        _logger.LogDebug("Sending Client ID: {CLIENT_ID_STRING}")
        SEND CLIENT_ID_STRING + "\r\n" to GAME_SOCKET // Lich's Game._puts ensures \r\n

        // 4. Send "Ready" commands (simulates client readiness)
        // This step is more explicit in Lich for non-Stormfront frontends or --without-frontend.
        // For direct Stormfront proxying, these commands would come from the actual client.
        // If Lich is connecting directly (e.g., --without-frontend), it sends these:
        READY_COMMAND = "<c>" // Default command prefix
        _logger.LogDebug("Sending Ready Command 1: {READY_COMMAND}")
        SEND READY_COMMAND + "\r\n" to GAME_SOCKET
        SLEEP 0.3 seconds
        _logger.LogDebug("Sending Ready Command 2: {READY_COMMAND}")
        SEND READY_COMMAND + "\r\n" to GAME_SOCKET

        _logger.LogInformation("Handshake and ready commands sent. Character should now be logging in.")
        // At this point, Lich starts its main proxy loops (Game.thread and client_thread)
        // to relay data and listen for server responses indicating successful login.
        // The game server uses the GAME_SERVER_KEY to identify and load the correct character.
    ENDFUNCTION
    ```

---

## 9. Error Handling & Logging

### 9.1. Error Handling Patterns
*   **`begin..rescue StandardError => e ... end`:** Widely used to catch general runtime errors. Offending lines, error messages, and partial backtraces are often sent to the client via `respond` and logged via `Lich.log`.
*   **Specific Rescues:** `SyntaxError`, `LoadError`, `SystemExit`, `SecurityError`, `ThreadError`, `NoMemoryError`, `SQLite3::BusyException` are caught in various places to handle specific failure conditions.
*   **`report_errors { ... }` (`global_defs.rb`):** A common wrapper for blocks of code, especially within script execution contexts, to provide standardized error reporting.
*   **Timeout Logic:** Methods like `dothistimeout` and the EAccess connection include timeout mechanisms to prevent indefinite blocking.

### 9.2. Logging Mechanisms
*   **`Lich.log(message)`:** Primary method for writing to the main Lich debug log file (e.g., `TEMP_DIR/debug-*.log`). Includes timestamps.
*   **`respond(message)` / `_respond(message)`:** Sends messages to the user's game client window. `respond` attempts to format for the frontend, `_respond` is more raw.
*   **`echo(message)`:** A script-facing wrapper for `respond`, prepending `[script_name: ...]`.
*   **`Lich::Messaging` module:** Provides methods like `Lich::Messaging.msg("error", ...)` for sending styled/colored messages to the client (frontend-dependent).
*   **`Lich::Util::Log` module:** Contextual debug logging, can be enabled/filtered for specific modules/scripts.
*   **`$stderr` redirection:** Standard error output is redirected to the main Lich debug log file.

---

## 10. Security Considerations

### 10.1. Credential Handling
*   **EAccess:** Usernames and passwords for Simutronics accounts are handled by `Lich::Common::EAccess`. Passwords are obfuscated (XORed with a server-provided hashkey) before being sent over an SSL connection.
*   **`DATA_DIR/entry.dat`:** If users save login profiles via the GUI, credentials (including the password) are stored in this file. The file content is Marshal-dumped and then Base64 encoded. The code implies the password within this structure is further encrypted/obfuscated, but the specific method isn't detailed in the provided core files (it's assumed to be handled during the Marshal load/dump of the `@entry_data` structure). **[ASSUMPTION: Passwords in `entry.dat` are encrypted/obfuscated beyond simple Marshal/Base64.]**

### 10.2. Script Trust Model (Legacy - Primarily for Ruby < 2.3)
*   **`lich.db3` - `trusted_scripts` table:** Stored names of scripts explicitly trusted by the user.
*   **Purpose:** In older Ruby versions with more effective `$SAFE` levels, trusted scripts could bypass certain security restrictions. In modern Ruby, this is less of a hard sandbox and more of a user acknowledgment.
*   **Commands:** `;#{$lich_char_regex}trust <script>`, `;#{$lich_char_regex}distrust <script>`.

### 10.3. File System and Registry Access
*   **File I/O:** Lich and its scripts read/write to various files within `LICH_DIR` and its subdirectories (`DATA_DIR`, `SCRIPT_DIR`, `LOG_DIR`, `TEMP_DIR`). Scripts using `Script.open_file` are generally confined to `DATA_DIR/<script_name>.<ext>`.
*   **System-Level Changes:**
    *   Modifying Windows/WINE registry keys (for SGE/SAL integration via `Lich.link_to_sge/sal`).
    *   Modifying the system's hosts file (for direct proxy mode via `Lich.modify_hosts`).
    *   These operations typically require elevated (admin/root) privileges. Lich attempts to request these if not already running with them for these specific functions.

---

## 11. Other Important Logic

### 11.1. XML Parsing (`XMLData`)
*   The `Lich::Common::XMLParser` class is instantiated as the global `XMLData` object.
*   It uses `REXML::StreamListener` for efficient, event-based parsing of the continuous XML stream from game servers.
*   **Key Responsibilities:**
    *   Populating character status variables: `@mana`, `@health`, `@spirit`, `@stamina`, `@stance_text`, `@stance_value`, `@mind_text`, `@mind_value`, `@prepared_spell`, `@encumbrance_text`, `@encumbrance_value`, `@roundtime_end`, `@cast_roundtime_end`.
    *   Updating status indicators: `@indicator` hash (e.g., `XMLData.indicator['IconDEAD']`).
    *   Parsing wound and scar data: `@injuries` hash.
    *   Extracting room information: `@room_title`, `@room_name`, `@room_description`, `@room_exits`, `@room_exits_string`, `@room_id`.
    *   Managing `GameObj` lists by clearing them on room changes (`<nav>`) and populating them from `<component id="room objs/players/desc">` and `<inv>` tags.
    *   Handling `<dialogData>` for active spells, buffs, debuffs, and cooldowns, which populates `@dialogs`, used by the `Effects` module in GSIV.
    *   Provides `active_spells` as a consolidated view (for GSIV).

### 11.2. Map System (`Lich::Common::Map`, `Lich::Common::Room`)
*   Manages an in-memory database of game rooms loaded from files.
*   `Map.current` returns the `Room` object for the player's current location, identified by matching title, description, exits, and optionally UIDs or unique room contents against the loaded map data.
*   `Map.findpath(start_room_id, end_room_id)` uses Dijkstra's algorithm (implemented in `Map#dijkstra`) to find the shortest path between rooms based on travel times defined in the map data.
*   Supports multiple data formats for map files: `.dat` (Marshal), `.xml` (custom Lich format), `.json`.
*   Handles UIDs (Unique Room Identifiers from the game) and can map multiple UIDs to a single Lich room ID if rooms are functionally identical but have different UIDs (e.g., due to dynamic instances).

### 11.3. Game Object Management (`Lich::Common::GameObj`)
*   Represents all dynamic entities in the game environment:
    *   Items on the ground (`GameObj.loot`).
    *   Items in inventory (`GameObj.inv`) and containers (`GameObj.containers`).
    *   Items in hands (`GameObj.left_hand`, `GameObj.right_hand`).
    *   NPCs (`GameObj.npcs`).
    *   Player Characters (`GameObj.pcs`).
    *   Static room description items (`GameObj.room_desc`).
*   Objects are identified by a unique `id` (from XML `exist` attribute), `noun`, and `name`.
*   `GameObj.type` method uses regexes from `gameobj-data.xml` to classify items (e.g., "weapon", "armor", "gem", "herb", "skin").
*   Lists are cleared and repopulated based on XML tags like `<nav>` (room change) and `<component id="room objs">`.

### 11.4. Hooks (`Lich::Common::UpstreamHook`, `Lich::Common::DownstreamHook`)
*   Provide a mechanism for scripts to intercept and modify data.
*   **Downstream Hooks:**
    *   Registered via `DownstreamHook.add("hook_name", proc { |server_string| ... })`.
    *   The Proc receives the server string and must return the (potentially modified) string, or `nil` to suppress the string from reaching the client and subsequent hooks.
    *   Called in `Game.thread` after XML parsing but before sending to the client.
*   **Upstream Hooks:**
    *   Registered via `UpstreamHook.add("hook_name", proc { |client_string| ... })`.
    *   The Proc receives the client command string and must return the (potentially modified) string, or `nil` to suppress the command from being sent to the game server.
    *   Called in `do_client` before a game command is sent to the server.

---

## 12. Setup and Installation (Initial Observations)

*   **Required Ruby Version:** `2.6` or higher (as per `REQUIRED_RUBY` in `version.rb`). `3.2` is recommended.
*   **Gem Dependencies:**
    *   `sqlite3`: Essential for database operations. Lich attempts to prompt for install if missing on Windows.
    *   `gtk3` (Optional, for GUI): Lich attempts to prompt for install if missing on Windows and GUI is desired.
    *   Other gems listed in [Section 8.1](#81-ruby-gems) are used by core functionalities and should ideally be present. Lich does not currently have an automated installer for all of them beyond `sqlite3` and `gtk3`.
*   **Location for Script Files:**
    *   Lich itself (core files) typically resides in a user-chosen directory (e.g., `C:\Lich5`, `~/lich`).
    *   User scripts (`.lic`, `.rb`, etc.) are placed in the `scripts` subdirectory and its `scripts/custom` subdirectory.
*   **Initial Configuration Steps:**
    1.  Ensure Ruby and required gems are installed.
    2.  Run Lich (e.g., `ruby lich.rbw`).
    3.  **First-time GUI Login (Recommended):**
        *   Use the "Manual Entry" tab to enter Simutronics account ID and password.
        *   Connect to EAccess to fetch game/character list.
        *   Select character and game instance.
        *   Check "Save this info for quick game entry".
        *   Optionally configure frontend and custom launch commands.
        *   Click "Play".
    4.  **SGE/SAL Integration (Optional, Windows/WINE):**
        *   Run Lich with admin/root privileges.
        *   Use `;#{$lich_char_regex}repo download sge-connector` and follow its instructions, or manually use (now deprecated) `--install` or the newer `Lich.link_to_sge`/`Lich.link_to_sal` if doing so programmatically.
    5.  **Map Database:** A map file for the target game (e.g., `map-*.json` for GSIV) needs to be present in `DATA_DIR/<GameName>/`. This is often downloaded via the `repository` script.
    6.  **Essential Data Files:** `effect-list.xml` and `gameobj-data.xml` should be in `DATA_DIR/`. These can also be fetched via the `repository` script.
    7.  **UserVars:** Set up common `UserVars` like `UserVars.lootsack` for scripts that use them.
    8.  **Autostart:** Configure `autostart.lic` to launch preferred utility scripts on login.

---

## 13. Assumptions and Areas for Clarification

*   **Game Focus:** This documentation primarily focuses on **GemStone IV** interactions, as indicated by the presence of modules like `Lich::Gemstone::Infomon`, `Effects`, `PSMS`, etc. While DragonRealms support exists, the depth of specific DR parsing and state management is less detailed in the provided core files compared to GSIV.
*   **`$SAFE` Levels:** Assumed that the more complex `$SAFE` level manipulations from older Lich versions are largely non-functional or less relevant in modern Ruby (>=2.3), and security relies more on user discretion and Lich's attempts to confine file I/O.
*   **Wizard/Avalon Frontend Emulation:** The code contains logic (`sf_to_wiz`) to translate Stormfront XML to a format presumably understood by the older Wizard/Avalon frontends. The specifics of this format are based on legacy conventions.
*   **`entry.dat` Encryption:** The exact encryption/obfuscation mechanism for passwords within `entry.dat` is not visible in this core codebase snapshot; it's assumed to be handled by `Marshal.load/dump` in conjunction with some transformation not explicitly shown.
*   **DRb Usage:** The `require 'drb/drb'` is noted, but its extensive use within the core loop isn't immediately apparent. It might be for specific utilities or inter-script communication features not central to the basic proxy/scripting operation.
*   **Completeness of PSM/Spell Data:** The hardcoded PSM tables (`Armor`, `CMan`, `Feat`, `Shield`, `Weapon`, `Warcry`) and spell data from `effect-list.xml` are assumed to be reasonably up-to-date for GSIV, but game updates can always introduce changes.
*   **DragonRealms Infomon (`DRInfomon`):** The structure and capabilities of `DRInfomon` are present but may differ significantly in detail and completeness compared to GSIV's `Infomon`. The parsing logic in `drparser.rb` is specific to DR's output.

**Questions for Clarification/Further Investigation:**
1.  What is the precise method used for encrypting/decrypting passwords stored in `DATA_DIR/entry.dat`?
2.  Are there any active core uses of `DRb` that are critical to basic Lich operation or common script interactions?
3.  How is the `gameobj-data.xml` (item type definitions) typically maintained and updated by the Lich project?
4.  What are the current best practices for script security and sandboxing in the context of modern Ruby versions used by Lich?
5.  Are there any specific, known issues or frequently encountered problems related to the interaction between Lich's core XML parsing and recent/ongoing Simutronics XML changes for GemStone IV?
6.  How complete and actively maintained is the DragonRealms-specific parsing and state management (`DRInfomon`, `DRParser`, `DRSkills`, etc.) compared to its GemStone IV counterparts?
