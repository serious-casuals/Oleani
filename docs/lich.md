# Lich Legacy Ruby Client - Technical Documentation

## 1. High-Level Overview

### 1.1. Purpose of the Application
Lich is a third-party proxy client and scripting engine primarily designed for Simutronics text-based MUDs (Multi-User Dungeons), most notably GemStone IV and DragonRealms. Its core purposes are:
1.  **Proxying Connections:** It sits between the official game client (like Stormfront/Wrayth or Wizard) and the game servers, intercepting and potentially modifying data streams in both directions.
2.  **Scripting Engine:** It allows users to write and run Ruby scripts to automate tasks, parse game information, react to game events, and enhance gameplay.
3.  **Information Management:** It parses game data (often XML sent by the server) to maintain an internal representation of game state, character status, room details, and more, making this information accessible to scripts.
4.  **Frontend Abstraction:** It provides a layer of abstraction to work with different official and third-party game frontends.

### 1.2. Main Workflow
The application's main workflow is event-driven and depends on how it's launched and used:

1.  **Initialization:**
    *   Lich starts, processes command-line arguments.
    *   Sets up necessary directory structures and constants.
    *   Loads core libraries, including networking, XML parsing, database, GUI (if available), and game-specific modules.

2.  **Login/Connection Phase:**
    *   **EAccess Authentication (if Lich initiates login):** If started with `--login` or through its GUI's manual/saved entry, Lich connects to Simutronics' EAccess servers to authenticate the user's account and character, retrieving a session key and game server details.
    *   **.SAL File Processing:** If started with a `.sal` file, Lich reads the pre-authenticated session key and server details from this file.
    *   **Proxy Setup:**
        *   Lich starts a local TCP server.
        *   It then launches the user's chosen game client (e.g., Stormfront, Wizard), directing it to connect to Lich's local server.
        *   Lich establishes its own connection to the actual Simutronics game server using the obtained session key and server details.
    *   **Direct Proxy (less common for Simu games):** If specific host/port are given via command line without EAccess, Lich acts as a simpler proxy, expecting the client to provide any necessary authentication.

3.  **Session Active - Proxying and Scripting:**
    *   **Data Interception:**
        *   **Downstream (Server to Client):** Lich reads data from the game server, parses XML, updates its internal game state (via `XMLData`, `Infomon`, etc.), runs downstream hooks (allowing scripts to modify data or react), and forwards the (potentially modified) data to the user's game client.
        *   **Upstream (Client to Server):** Lich reads commands from the user's game client. If a command is a Lich-specific command (e.g., starting with `;` or `,`), Lich processes it internally (e.g., starting/stopping/pausing a script). If it's a game command, Lich runs upstream hooks and forwards the (potentially modified) command to the game server.
    *   **Script Execution:** Users can start, stop, and manage scripts. Running scripts interact with the game by sending commands, reading game data, and reacting to events.
    *   **GUI Interaction (if active):** If the GTK GUI is used, it runs in the main thread, handling user interactions for login, script management, etc.

4.  **Termination:**
    *   User exits the game client, or Lich is quit.
    *   Lich attempts a graceful shutdown: saves settings, executes script exit procedures, closes network connections.

### 1.3. Key Entry Points
*   **Main Script Execution:** The primary entry point is the execution of the main Lich Ruby script itself (implied to be `lich.rbw` or a similar top-level file). The execution flow begins from the top of this file.
*   **`@main_thread` (`main/main.rb`):** This thread, once initialized, becomes the central orchestrator for login, client/server connection setup, and managing the core communication loops.
*   **`gui_login` (`common/gui-login.rb`, called from `@main_thread`):** If the GTK interface is used for login, this function handles the GUI presentation and EAccess calls.
*   **`Game.open` and `Game.thread` (`games.rb`):** Initiates the connection to the game server and starts the thread responsible for reading data from the server.
*   **`client_thread` (`main/main.rb`):** The thread responsible for reading data from the connected game client.
*   **Script Commands (e.g., `;scriptname`):** Processed by `do_client` in `global_defs.rb`, leading to `Script.start`.

## 2. Code Flow & Architecture

### 2.1. Primary Flow of Execution
1.  **Startup & Initialization (`lich.rbw` / top-level script):**
    *   Argument Parsing (`main/argv_options.rb`): Command-line arguments determine initial behavior (login details, paths, frontend mode, etc.).
    *   Constants & Paths (`constants.rb`): Define core directory locations.
    *   Core Requires: Loads essential Ruby libraries and Lich's own core modules.
        *   `lich.rb`: Lich module utilities, DB initialization.
        *   `init.rb`: Early setup, platform checks, GTK/SQLite3 checks.
        *   `common/*`: Various shared utilities, class extensions, GUI components, settings management.
        *   `gameobj.rb`, `xmlparser.rb`, `games.rb`: Core data handling and game communication.
    *   Global `XMLData = Lich::Common::XMLParser.new` instance is created.
    *   The main logic resides in `@main_thread` defined in `main/main.rb`.

2.  **`@main_thread` in `main/main.rb`:**
    *   **Login/Launch Data Acquisition:**
        *   If `--login <char>`: Reads `entry.dat`, calls `EAccess.auth` (from `common/eaccess.rb`).
        *   If `.sal` file provided: Parses the `.sal` file.
        *   If GUI: Calls `gui_login` (from `common/gui-login.rb`), which handles UI and can call `EAccess.auth`.
        *   The result is stored in `@launch_data`.
    *   **Proxy Setup (if `@launch_data` is available or direct connection params are set):**
        *   Determines game (GS/DR) and includes game-specific modules.
        *   Starts a `TCPServer` on a local port for the game client to connect to.
        *   Launches the user's game client (e.g., Stormfront, Wizard) using system commands, pointing it to Lich's local server. The launch command often includes the session key (`AUTH_KEY` from `@launch_data`).
        *   Waits for the game client to connect to Lich (`$_CLIENT_`).
        *   Lich connects to the actual game server (`Game.open` from `games.rb`).
    *   **Direct Proxy (if `-g host:port` is used):**
        *   Lich listens on the specified port, modifies the system's hosts file to redirect game traffic for the specified host to `127.0.0.1`.
        *   Waits for the game client to connect.
        *   Connects to the *actual* game server.
        *   Restores the hosts file.
    *   **Communication Loop Startup:**
        *   `client_thread` is started: Reads from `$_CLIENT_` (the game client), processes via `do_client` (in `global_defs.rb`), and sends to `Game` (the game server). This thread handles the initial handshake (auth key, client ID) between the proxied client and the game server.
        *   `Game.thread` is started (implicitly by `Game.open`): Reads from `$_SERVER_` (the game server), passes data to `XMLData` for parsing, runs `DownstreamHook`s, and writes to `$_CLIENT_`.
        *   `detachable_client_thread` may be started for secondary client connections.

3.  **Ongoing Session:**
    *   `Game.thread` continuously receives data from the game server. It uses `XMLData.tag_start`, `XMLData.text`, `XMLData.tag_end` (from `common/xmlparser.rb`) to parse the XML stream and update game state variables (`XMLData.mana`, `XMLData.room_title`, `GameObj` instances, etc.). `DownstreamHook.run` allows scripts to modify this data.
    *   `client_thread` continuously receives data from the user's game client.
        *   If it's a Lich command (e.g., `;#{$lich_char_regex}somecommand`), `do_client` processes it (e.g., `Script.start`, `Script.kill`).
        *   If it's a game command, `UpstreamHook.run` processes it, and then `Game._puts` sends it to the game server.
    *   Scripts run in their own threads, interacting via methods like `put`, `fput`, `get`, `waitfor`, `match`, and by accessing shared state like `XMLData`, `GameObj`, `Char`, `Room`, etc.

4.  **Shutdown:**
    *   Triggered by the game client disconnecting, the game server disconnecting, user exiting Lich GUI, or an explicit exit command.
    *   `Game.thread.join` waits for the server connection loop to finish.
    *   Scripts are killed (`Script.running.each(&:kill)`).
    *   Settings are saved (`Settings.save`, `Vars.save`).
    *   Connections are closed.
    *   If GTK is running, `Gtk.main_quit` is called.

### 2.2. Significant Classes and Responsibilities
*   **`Lich` (Module, `lich.rb`):**
    *   Top-level namespace.
    *   Provides global utility functions (`Lich.log`, `Lich.msgbox`, SGE/SAL linking, hosts file management).
    *   Manages the main SQLite database (`Lich.db`).
    *   Access point for global Lich settings (e.g., `Lich.display_lichid`).
*   **`Lich::Common::XMLParser` (Instance: `XMLData`, `common/xmlparser.rb`):**
    *   The primary engine for parsing XML data received from Simutronics game servers.
    *   Implements `REXML::StreamListener` to process XML tags and text content.
    *   Maintains a rich set of instance variables representing the current game state (character stats, room details, active spells, inventory, NPCs, etc.).
    *   Provides methods for scripts to query this game state (e.g., `XMLData.mana`, `XMLData.room_title`).
*   **`Lich::Common::Script` / `Lich::Common::ExecScript` / `Lich::Common::WizardScript` (`common/script.rb`):**
    *   Manages the lifecycle of user scripts.
    *   Provides methods to start, stop, pause, and list scripts.
    *   Handles script arguments, variables (`script.vars`), and label-based execution for `.lic` and Wizard scripts.
    *   Provides a sandboxed binding for untrusted scripts (in older Ruby versions).
    *   Manages script-specific data buffers and `watchfor` patterns.
*   **`Lich::GameBase::Game` (and game-specific `Lich::Gemstone::Game`, `Lich::DragonRealms::Game`) (`games.rb`):**
    *   Manages the TCP socket connection to the game server.
    *   Contains the main loop (`@thread`) for reading data from the server.
    *   Provides `_puts` for sending raw data and `puts` for sending game commands (with prefixing).
    *   Handles game-specific XML cleaning and initial processing before passing to `XMLData`.
*   **`Lich::Common::GameObj` (`common/gameobj.rb`):**
    *   Represents in-game entities: items (loot, inventory, hands), NPCs, PCs, room objects.
    *   Stores objects in class-level arrays (e.g., `@@loot`, `@@npcs`, `@@inv`).
    *   Provides methods to query these collections (e.g., `GameObj.loot`, `GameObj.npcs`, `GameObj[]`).
    *   Determines object `type` (e.g., "weapon", "armor") based on `gameobj-data.xml`.
*   **`Lich::Common::Map` / `Lich::Common::Room` (`common/map/map_gs.rb`, `common/map/map_dr.rb`):**
    *   Manages room data, including titles, descriptions, paths, and UIDs.
    *   Loads map data from `.dat`, `.xml`, or `.json` files.
    *   Provides methods to get the current room (`Map.current`), navigate (`Map.findpath`), and find rooms by tags.
*   **`Lich::Common::EAccess` (`common/eaccess.rb`):**
    *   Handles authentication with Simutronics' EAccess servers.
    *   Obtains session keys and character launch parameters.
*   **`Lich::Common::Settings`, `Lich::Common::CharSettings`, `Lich::Common::GameSettings` (`common/settings/*.rb`):**
    *   Provide a persistent key-value storage mechanism for Lich and scripts.
    *   Uses an SQLite database (`lich.db3`) with `Marshal` for serialization.
    *   Scoped settings allow data to be specific to a script, character, or game.
    *   `SettingsProxy` enables hash-like and array-like access to nested settings with automatic saving.
*   **`Lich::Common::Vars` / `Lich::Common::UserVars` (`common/vars.rb`, `common/uservars.rb`):**
    *   Provides a global, persistent key-value store for user-defined variables, saved per character.
*   **`Lich::Common::DownstreamHook` / `Lich::Common::UpstreamHook` (`common/downstreamhook.rb`, `common/upstreamhook.rb`):**
    *   Allow scripts to register Procs that can intercept and modify data flowing from the server to the client (downstream) or from the client to the server (upstream).
*   **Game-Specific Modules (e.g., `Lich::Gemstone::Infomon`, `Lich::Gemstone::Effects`, `Lich::Gemstone::Skills`, `Lich::DragonRealms::DRInfomon`):**
    *   Handle parsing and state management for game-specific XML data and status information.
    *   Provide APIs for scripts to access this structured game data (e.g., `Stats.strength`, `Spells.known`, `Effects::Buffs.active?`).

### 2.3. System Architecture
Lich primarily functions as a **Man-in-the-Middle Proxy** with an embedded **Scripting Engine**.

**Simplified Component Interaction:**

**Key Architectural Points:**

*   **Proxy Model:** Lich intercepts all network traffic.
*   **Event-Driven:** Much of Lich's core and script logic is reactive to incoming data from the server or client.
*   **Stateful:** `XMLData` and game-specific modules maintain a comprehensive state of the game world and character.
*   **Modular:** Core functionalities (XML parsing, scripting, settings, game communication) are in separate modules/classes. Game-specific logic is further modularized.
*   **Extensible:** Hooks and the scripting engine allow users to heavily customize and extend behavior.
*   **Data Persistence:** Settings, map data, and some game state are persisted in SQLite databases and flat files.

## 3. Database Connections & Models

### 3.1. Main Lich Database (`DATA_DIR/lich.db3`)
*   **Connection:** `SQLite3::Database.new("#{DATA_DIR}/lich.db3")`, accessed via `Lich.db`.
*   **Tables & Models (Implicit via `Marshal`):**

    *   **`script_setting`** (Potentially older/less used)
        *   `script` (TEXT, NOT NULL, PK): Name of the script.
        *   `name` (TEXT, NOT NULL, PK): Setting name within the script.
        *   `value` (BLOB): Marshal-dumped Ruby object.
        *   Purpose: Store individual settings for scripts.
    *   **`script_auto_settings`** (Main settings table)
        *   `script` (TEXT, NOT NULL, PK): Name of the script (or global scope like "GSIV" or "GSIV:CharacterName").
        *   `scope` (TEXT, PK): The scope of the settings (e.g., ":", "GSIV", "GSIV:CharacterName").
        *   `hash` (BLOB): Marshal-dumped Ruby Hash containing all settings for that script/scope.
        *   Purpose: Primary storage for `Settings`, `CharSettings`, and `GameSettings`.
    *   **`lich_settings`**
        *   `name` (TEXT, NOT NULL, PK): Global Lich setting name (e.g., "display_lichid", "win32_launch_method").
        *   `value` (TEXT): Value of the setting.
        *   Purpose: Store global configuration options for Lich itself.
    *   **`uservars`**
        *   `scope` (TEXT, NOT NULL, PK): Scope of the user variables, typically "Game:CharacterName".
        *   `hash` (BLOB): Marshal-dumped Ruby Hash containing user-defined variables (via `Vars` / `UserVars`).
        *   Purpose: Store variables created and managed by users through scripts.
    *   **`trusted_scripts`** (Only if Ruby version is < 2.3)
        *   `name` (TEXT, NOT NULL): Name of the script that has been trusted by the user.
        *   Purpose: Maintain a list of scripts allowed to perform potentially unsafe operations.
    *   **`simu_game_entry`**
        *   `character` (TEXT, NOT NULL, PK): Character name.
        *   `game_code` (TEXT, NOT NULL, PK): Game code (e.g., "GS3", "DR").
        *   `data` (BLOB): Marshal-dumped data, likely cached EAccess responses or character-specific launch info.
        *   Purpose: Cache EAccess login information for characters.
    *   **`enable_inventory_boxes`**
        *   `player_id` (INTEGER, NOT NULL, PK): The player's unique ID.
        *   Purpose: Stores a boolean-like flag (presence of row indicates true) for whether to use the Stormfront inventory box display.

*   **Queries:**
    *   Primarily direct SQL executed via `Lich.db.execute(...)` and `Lich.db.get_first_value(...)`.
    *   Uses `INSERT OR REPLACE INTO ...` for upserting settings.
    *   Uses `SELECT ... FROM ... WHERE ...` for retrieving settings.
    *   Uses `DELETE FROM ... WHERE ...` for removing settings.
    *   Data in `BLOB` columns is serialized using `Marshal.dump` and deserialized using `Marshal.load`.

### 3.2. Infomon Database (`DATA_DIR/infomon.db`)
*   **Connection:** `Sequel.sqlite(File.join(DATA_DIR, "infomon.db"))`, accessed via `Infomon.db`.
*   **Tables & Models (via Sequel):**
    *   Dynamic table per character: `"<GameName>_<CharacterName>"` (e.g., `GSIV_MyCharacter`, `DR_MyOtherChar`).
        *   `key` (TEXT, PK): The name of the infomon variable (e.g., "stat.strength", "experience.fame").
        *   `value` (ANY): The value of the infomon variable, stored as its native Ruby type where possible by Sequel (Integer, String, Boolean).
        *   Purpose: Store detailed, parsed character status, skills, experience, society info, etc., specific to each game and character.
*   **Queries (via Sequel ORM):**
    *   `Infomon.table.insert_conflict(:replace).insert(...)`: For upserting data.
    *   `Infomon.table[key: key]`: For selecting a specific key.
    *   `Infomon.table.map(:key).zip(Infomon.table.map(:value))`: For fetching all data for caching.
    *   Asynchronous SQL queue (`Infomon.queue`) is used to batch database writes to improve performance.

### 3.3. Script-Specific Databases (`DATA_DIR/<script_name>.db3`)
*   **Connection:** `SQLite3::Database.new("#{DATA_DIR}/#{script.name.gsub(/\/|\\/, '_')}.db3")`, accessed via `Script.db`.
*   **Schema:** Defined by the individual scripts themselves. Lich core does not impose a schema.
*   **Purpose:** Allows individual scripts to have their own private SQLite database for persistent storage.
*   **Queries:** Handled by the individual scripts, typically using raw SQL via the `SQLite3::Database` object.

## 4. File I/O

*   **`DATA_DIR/lich.db3`:**
    *   Path: `DATA_DIR/lich.db3`
    *   Type: SQLite3 Database.
    *   Purpose: Main persistent storage for Lich settings, user variables, trusted scripts, cached EAccess entries.
    *   Operations: Read/Write.
*   **`DATA_DIR/infomon.db`:**
    *   Path: `DATA_DIR/infomon.db`
    *   Type: SQLite3 Database (managed by Sequel).
    *   Purpose: Stores detailed character information parsed from game XML (stats, skills, experience, etc.) for `Infomon` and `DRInfomon`.
    *   Operations: Read/Write.
*   **`DATA_DIR/<script_name>.db3`:**
    *   Path: `DATA_DIR/<SanitizedScriptName>.db3`
    *   Type: SQLite3 Database.
    *   Purpose: Script-specific persistent storage.
    *   Operations: Read/Write (by individual scripts).
*   **`DATA_DIR/entry.dat`:**
    *   Path: `DATA_DIR/entry.dat`
    *   Type: Binary file containing Base64 encoded, Marshal-dumped array of Hashes.
    *   Purpose: Stores saved login credentials (username, encrypted password, character, game, frontend) from the GUI.
    *   Operations: Read (on startup for GUI), Write (when saving new entries in GUI).
*   **`DATA_DIR/simu.pem`:**
    *   Path: `DATA_DIR/simu.pem`
    *   Type: PEM-formatted SSL certificate.
    *   Purpose: Used to verify the SSL certificate of Simutronics' EAccess servers.
    *   Operations: Read (by `EAccessClient`). Written by `EAccess.download_pem` if missing or mismatched.
*   **Map Files (`DATA_DIR/<GameName>/map-*.dat`, `map-*.xml`, `map-*.json`):**
    *   Path: Example `DATA_DIR/GSIV/map-1678886400.json`
    *   Type: Marshal-dumped Ruby array (`.dat`), XML (`.xml`), or JSON (`.json`).
    *   Purpose: Stores game room information (ID, title, description, paths, exits, tags, etc.).
    *   Operations: Read (by `Map.load`), Write (by `Map.save`, `Map.save_xml`, `Map.save_json`). Backups (`.bak`) are created on save.
*   **`DATA_DIR/<GameName>/spell-ranks.dat`:**
    *   Path: Example `DATA_DIR/GSIV/spell-ranks.dat`
    *   Type: Marshal-dumped array.
    *   Purpose: Stores spell rank information for characters other than the current one (used by `SpellRanks` class, typically for multi-character scripting or information sharing).
    *   Operations: Read/Write.
*   **`DATA_DIR/effect-list.xml`:**
    *   Path: `DATA_DIR/effect-list.xml` (Also checked in `DATA_DIR/<GameName>/effect-list.xml` by some scripts)
    *   Type: XML file.
    *   Purpose: Defines spells, their properties (duration, cost, messages), and bonuses. Used by the `Spell` class.
    *   Operations: Read. Can be downloaded/updated by the `repository` script or if missing.
*   **`DATA_DIR/gameobj-data.xml`:**
    *   Path: `DATA_DIR/gameobj-data.xml`
    *   Type: XML file.
    *   Purpose: Defines patterns (regexes) for classifying in-game items by type (e.g., "weapon", "armor", "gem"). Used by `GameObj.type`.
    *   Operations: Read.
*   **`DATA_DIR/gameobj-custom/gameobj-data.xml`:**
    *   Path: `DATA_DIR/gameobj-custom/gameobj-data.xml`
    *   Type: XML file.
    *   Purpose: Allows users to add custom item type definitions, merging with the base `gameobj-data.xml`.
    *   Operations: Read.
*   **Script Files (`SCRIPT_DIR/...` and `SCRIPT_DIR/custom/...`):**
    *   Path: `SCRIPT_DIR/<script_name>.(lic|rb|cmd|wiz)[.gz]`
    *   Type: Ruby (`.rb`), Lich Scripting Language (`.lic`), Wizard Frontend Script (`.cmd`, `.wiz`). Can be gzipped.
    *   Purpose: User-created scripts that automate gameplay.
    *   Operations: Read (by `Script.start` and related methods).
*   **Script Data Files (`DATA_DIR/<SanitizedScriptName>.<ext>`):**
    *   Path: `DATA_DIR/<SanitizedScriptName>.<user_defined_extension>`
    *   Type: User-defined (text, YAML, JSON, binary).
    *   Purpose: Persistent storage for individual scripts, managed by the scripts themselves via `Script.open_file`.
    *   Operations: Read/Write (by individual scripts).
*   **Lich Debug Logs (`TEMP_DIR/debug-*.log`):**
    *   Path: `TEMP_DIR/debug-YYYY-MM-DD-HH-MM-SS-MS.log`
    *   Type: Plain text log file.
    *   Purpose: Stores debug output from Lich core and scripts. $stderr is redirected here.
    *   Operations: Write. Old logs are periodically pruned.
*   **Script Logs (`LOG_DIR/<script_name>.log`):**
    *   Path: `LOG_DIR/<script_name>.log`
    *   Type: Plain text log file.
    *   Purpose: Allows individual scripts to create their own dedicated log files via `Script.log`.
    *   Operations: Write (by individual scripts).
*   **Temporary SAL Files (`TEMP_DIR/lich*.sal`):**
    *   Path: `TEMP_DIR/lich<random_number>.sal`
    *   Type: Plain text Simutronics Authentication Launch file.
    *   Purpose: Created by Lich when it performs EAccess authentication and needs to launch an external game client. Contains session key and server details.
    *   Operations: Write, then typically Delete after client launch.
*   **Genie Session Files (`/tmp/simutronics/sessions/<CharName>.session` on Linux/macOS):**
    *   Path: `<SystemTempDir>/simutronics/sessions/<CharacterName>.session`
    *   Type: JSON file.
    *   Purpose: Used for integration with the Genie game client, providing connection details (host/port Lich is listening on).
    *   Operations: Write (by `Frontend.create_session_file`), Delete (by `Frontend.cleanup_session_file`).
*   **System Hosts File (OS-dependent):**
    *   Path: e.g., `/etc/hosts` (Linux/macOS), `C:\Windows\System32\drivers\etc\hosts` (Windows).
    *   Type: Plain text.
    *   Purpose: Modified by Lich (if run with sufficient privileges) when using the direct proxy mode (`-g host:port`) to redirect game hostnames to `127.0.0.1`.
    *   Operations: Read, Write, Backup (`.bak`), Restore.
*   **Windows/WINE Registry:**
    *   Path: Various keys under `HKEY_LOCAL_MACHINE\Software\Simutronics` and `HKEY_LOCAL_MACHINE\Software\Classes\Simutronics.Autolaunch`.
    *   Type: Windows Registry / WINE emulated registry.
    *   Purpose: Used for Simutronics Game Entry (SGE) integration (launching Lich when clicking game links on the website) and `.sal` file association.
    *   Operations: Read/Write (by `Lich.link_to_sge`, `Lich.link_to_sal` and their `unlink` counterparts). Requires appropriate permissions (often admin/root).

## 5. Configuration

*   **Command-Line Arguments:**
    *   Parsed by `main/argv_options.rb`.
    *   Control various startup behaviors:
        *   `--login <charname>`: Initiate login for a specific character.
        *   `<game>.sal`: Launch using a Simutronics Authentication Launch file.
        *   `--gemstone`, `--dragonrealms`, `--platinum`, `--test`, etc.: Specify game and instance.
        *   `--frontend <name>`: Specify frontend (e.g., `stormfront`, `wizard`, `genie`).
        *   `--without-frontend`: Run Lich without launching an external game client (Lich connects directly).
        *   `-g host:port`: Direct proxy mode, bypassing EAccess.
        *   `--home`, `--scripts`, `--data`, `--logs`, `--temp`, `--backup`, `--lib`: Override default directory paths.
        *   `--wine=...`, `--wine-prefix=...`: WINE configuration.
        *   `--install`, `--uninstall`: (Deprecated) Registry integration for SGE/SAL.
        *   `--start-scripts=script1,script2`: Automatically start specified scripts after login.
        *   `--detachable-client=host:port`: Enable a secondary, detachable client interface.
        *   `--dark-mode=(true|false)`: Force dark mode for GUI elements.
*   **`lich.db3` - `lich_settings` table:**
    *   Managed by `Lich` module methods (e.g., `Lich.display_lichid=`).
    *   Stores global Lich behavior toggles:
        *   `display_lichid`: Whether to show Lich's internal map ID in room titles.
        *   `display_uid`: Whether to show the game's unique room ID (UID) in room titles.
        *   `hide_uid_flag`: (DR specific) Whether to hide the UID part of the room title even if the game client's "ShowRoomID" flag is on.
        *   `display_exits`: Whether to display non-obvious room exits known to Lich.
        *   `display_stringprocs`: Whether to display exits that are `StringProc` (dynamic) based.
        *   `win32_launch_method`: (Windows) Cached successful method for launching game clients.
        *   `track_autosort_state`, `track_dark_mode`, `track_layout_state`: GUI preference persistence.
        *   `core_updated_with_lich_version`: Tracks the Lich version with which core scripts/data were last updated.
        *   `debug_messaging`: Enables/disables verbose debug messages for certain internal Lich operations.
*   **`lich.db3` - `uservars` table:**
    *   Managed by `Lich::Common::Vars` and `Lich::Common::UserVars` modules.
    *   Stores key-value pairs defined by users or scripts (e.g., `UserVars.lootsack = "my backpack"`).
    *   Scoped by "GameName:CharacterName".
*   **`lich.db3` - `script_auto_settings` table:**
    *   Managed by `Lich::Common::Settings`, `Lich::Common::CharSettings`, `Lich::Common::GameSettings`.
    *   Stores script-specific settings as Marshal-dumped Hashes.
    *   Scoped by script name and a further scope string (e.g., ":", "GameName", "GameName:CharacterName").
*   **`DATA_DIR/entry.dat`:**
    *   Stores saved login profiles from the GUI (account, encrypted password, character, game, frontend preferences).
*   **Environment Variables:**
    *   `WINEPREFIX`, `WINEBIN` (or `wine` in path): For WINE integration.
    *   `DISPLAY` (Linux/macOS): For GTK GUI availability.
    *   `LICH_DIR`, `TEMP_DIR`, etc.: Can be set externally to override compiled-in defaults if argument parsing doesn't override them first.

## 6. Recurring Tasks

*   **Main Game Server Read Loop (`Game.thread` in `games.rb`):**
    *   **Trigger:** Active TCP connection to the game server. Runs continuously.
    *   **Interval:** Driven by incoming data from the server (blocking read).
    *   **Action:**
        1.  Reads data from the game server socket.
        2.  Updates `$_SERVERBUFFER_`.
        3.  Performs game-specific cleaning (`@game_instance.clean_serverstring`).
        4.  Passes the XML string to `XMLData` (instance of `Lich::Common::XMLParser`) for parsing and state updates.
        5.  Calls `Script.new_downstream_xml` to make raw XML available to scripts.
        6.  Strips XML to get plain text, then calls `Script.new_downstream` to make plain text available and triggers `watchfor` patterns.
        7.  Runs the (potentially modified by scripts) string through `DownstreamHook.run`.
        8.  Sends the final string to the connected game client (`$_CLIENT_` and `$_DETACHABLE_CLIENT_`).
*   **Main Game Client Read Loop (`client_thread` in `main/main.rb`):**
    *   **Trigger:** Active TCP connection from the user's game client. Runs continuously.
    *   **Interval:** Driven by incoming data from the client (blocking read).
    *   **Action:**
        1.  Reads commands from the game client.
        2.  Passes the command to `do_client` (in `global_defs.rb`).
        3.  `do_client` processes Lich commands (e.g., starting scripts) or forwards game commands to the game server via `Game._puts` after running them through `UpstreamHook.run`.
*   **Detachable Client Read Loop (`detachable_client_thread` in `main/main.rb`, if enabled):**
    *   **Trigger:** When `--detachable-client` is used and a client connects.
    *   **Interval:** Driven by incoming data from the detachable client.
    *   **Action:** Similar to `client_thread`, reads commands and processes them via `do_client`. Also sends initial state information (health, mana, indicators) to the connecting detachable client.
*   **UserVars Save Thread (`Vars` module in `common/vars.rb`):**
    *   **Trigger:** Runs in a background thread.
    *   **Interval:** Every 300 seconds (5 minutes).
    *   **Action:** Checks if `UserVars` have changed since the last save and, if so, writes the current `UserVars` (Marshal-dumped Hash) to the `uservars` table in `lich.db3`.
*   **ActiveSpell Update Thread (`ActiveSpell.watch!` in `gemstone/infomon/activespell.rb`):**
    *   **Trigger:** Runs in a background thread after Infomon is loaded for GemStone. Waits for `ActiveSpell.request_update` to be called (which pushes to a queue).
    *   **Interval:** Event-driven (when spell effects change or are explicitly requested to update).
    *   **Action:**
        1.  Retrieves current active spell durations from `XMLData.active_spells` (which is populated by `XMLData` from `<dialogData id="Active Spells">`).
        2.  Updates the `timeleft` and `active` status of `Spell` objects in `Spell.list`.
        3.  Can optionally announce spell duration changes to the client.
*   **GTK Idle/Timeout Tasks (`common/gtk.rb`):**
    *   **Trigger:** When the GTK main loop is running.
    *   **Interval:** `GLib::Idle.add` runs when GTK is idle. `GLib::Timeout.add` runs after a specified delay (here, 1ms for `Gtk.queue`).
    *   **Action:** `Gtk.queue` executes a block of code within the main GTK thread, ensuring thread safety for GUI operations. `GLib::Idle.add` in this codebase just sleeps for 0.01s repeatedly.

## 7. Other Important Logic

### 7.1. Complex or Business-Critical Logic
*   **XML Parsing (`XMLData`):** This is central to Lich's understanding of the game state. It handles a wide variety of XML tags from Simutronics servers, including character status (health, mana, spirit, stamina, encumbrance, stance, mind), room information (title, description, exits, objects, NPCs, PCs), active spell effects, roundtime, inventory, and more. It populates `GameObj` instances. The robustness of this parser is critical for almost all script functionality.
*   **EAccess Authentication (`Lich::Common::EAccess`):** Securely logs into Simutronics accounts to retrieve session keys. This involves an SSL connection and a specific multi-step protocol with Simu's auth servers.
*   **Script Execution and Sandboxing (`Lich::Common::Script`):** Manages multiple concurrent user scripts. For `.lic` and Wizard scripts, it parses labels and provides a specific execution environment. For Ruby `.rb` scripts, it uses `eval` within a binding. Older Ruby versions had more explicit sandboxing attempts.
*   **Proxy and Handshake Logic (`main/main.rb`, `games.rb`, `global_defs.rb`):** Correctly handling the initial connection sequence and handshake messages (auth key, client ID, ready commands) for different frontends (Stormfront, Wizard, Avalon, --without-frontend) is crucial for establishing a game session.
*   **Map System (`Lich::Common::Map`):** Provides room data storage and pathfinding (`dijkstra` algorithm) capabilities, essential for navigation scripts.
*   **Settings Persistence (`Lich::Common::Settings`, `Lich::Common::Vars`):** Ensures that script configurations and user variables are saved and loaded across Lich sessions. The `SettingsProxy` allows for more natural manipulation of nested hash/array settings.

### 7.2. Error Handling, Logging, and Retry Mechanisms
*   **General Error Handling:** Many critical operations are wrapped in `begin...rescue...end` blocks.
    *   `report_errors { ... }` in `global_defs.rb` is a common wrapper for script code execution blocks to catch and log/respond to exceptions.
    *   Specific rescue blocks handle `SocketError`, `IOError`, `SystemExit`, `SyntaxError`, `LoadError`, `SecurityError`, `ThreadError`, `NoMemoryError`, `SQLite3::BusyException`, etc.
*   **Logging:**
    *   `Lich.log(message)`: Writes to the main Lich debug log file in the `TEMP_DIR`. This is the primary mechanism for internal logging.
    *   `respond(message)` / `_respond(message)`: Sends messages to the user's game client. `respond` formats for the active frontend, `_respond` sends more raw data. Used for user feedback and script output.
    *   `echo(message)`: A script-facing wrapper around `respond`, prepending the script name.
    *   `Lich::Messaging` module: Provides formatted messaging (bold, colors for some frontends).
    *   `Lich::Util::Log`: Contextual logging for debugging specific modules/scripts, controlled by `Log.on?` and filters.
*   **Retry Mechanisms:**
    *   `SQLite3::BusyException`: Common database operations have a `retry` mechanism with a short sleep if the database is busy.
    *   `move` function (`global_defs.rb`): Contains logic to retry movement if common obstacles occur (e.g., needing to stand, hands full, door closed, RT). It has timeout and line count limits for retries.
    *   `dothis`, `dothistimeout` (`global_defs.rb`): These functions repeatedly execute a command until a success condition is met or a timeout occurs, implicitly handling retries for common game states (stunned, webbed, busy).
    *   Some EAccess operations or network reads might be retried by scripts, but the core `EAccessClient` focuses more on failing on protocol errors.

### 7.3. Security-Sensitive Operations
*   **EAccess Authentication (`Lich::Common::EAccess`):** Handles user account credentials (username, password). Passwords are obfuscated before being sent over SSL using a server-provided hashkey, but the initial password entry/storage is sensitive.
*   **`DATA_DIR/entry.dat`:** Stores encrypted login credentials. The encryption method is not detailed here but is a key security point.
*   **`Script.trust` (for Ruby < 2.3):** Allows designated scripts to bypass some of Lich's security restrictions (which were more relevant and stringent in older Ruby versions due to `$SAFE` levels). Modifying the trusted scripts list has security implications.
*   **File I/O (`Script.open_file`, map saving, etc.):** Scripts with file write access could potentially write anywhere if not properly sandboxed or if Lich is run with excessive permissions. Lich attempts to confine script file operations to subdirectories of `DATA_DIR`.
*   **Registry Modification (Windows/WINE):** `Lich.link_to_sge`, `Lich.link_to_sal` modify system registry settings, requiring admin/root privileges.
*   **Hosts File Modification:** `Lich.modify_hosts` changes a system-level network configuration file, also requiring elevated privileges.
*   **`eval` usage:** Used extensively for script execution. While scripts are somewhat contained, malicious script code could still pose a risk, especially if trusted or if sandbox mechanisms are bypassed.

### 7.4. Known Limitations / Areas for Improvement (Inferred from Code)
*   **Sandboxing (Ruby >= 2.3):** The original `$SAFE` level sandboxing is largely ineffective in modern Ruby. While Lich has some protections, a truly malicious Ruby script could still cause harm. The "trust" system is more of a legacy feature in this context.
*   **XML Parsing Robustness:** While `XMLData` handles many quirks, Simutronics XML can be notoriously inconsistent. The parser relies on specific tag structures and may break if the game significantly changes its XML output. The error handling for XML parsing involves retrying with fixes or resetting state, which can sometimes lead to temporary loss of game state information.
*   **Frontend-Specific Code:** There's a lot of conditional logic based on `$frontend`. Simplifying or further abstracting frontend differences could be beneficial. The `sf_to_wiz` and `fb_to_sf` functions are examples of this translation layer.
*   **Global State:** Heavy reliance on global variables (`$SEND_CHARACTER`, `$_CLIENT_`, `$_SERVER_`, `XMLData` instance, `GameObj` class variables) can make state management complex and debugging harder.
*   **Thread Safety:** While `Mutex` is used in several places (e.g., `Lich.db_mutex`, `Game.@mutex`, `SharedBuffer.@buffer_mutex`), ensuring complete thread safety across all shared resources and script interactions is a complex ongoing challenge in a multi-threaded application like Lich.
*   **Deprecated Code:** The codebase contains numerous functions and aliases marked or known to be deprecated, indicating areas where refactoring or removal might be needed.
*   **WINE/Platform Abstractions:** While efforts are made, maintaining perfect compatibility and feature parity across Windows, Linux (with/without WINE), and macOS can be challenging, especially for system-level interactions like registry and process launching.
*   **DRb Usage:** `require 'drb/drb'` is present, but its active use in the core paths isn't immediately obvious from the provided files. If it's used for inter-Lich communication or plugins, its specific implementation details would need further investigation. (Self-correction: It's likely not a major core component for basic operation but might be used by some advanced scripts or utilities not in this provided set).

This documentation provides a comprehensive overview based on the provided Ruby code. Onboarding new developers would involve them understanding these core components and then diving into specific modules or game-specific logic as needed.
