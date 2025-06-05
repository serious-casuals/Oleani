Okay, consider this the formal response from your contracted AI Analysis Assistant. Based on the provided Ruby codebase snapshot (`code.txt`) and adhering strictly to the "Analysis & Documentation Mandate," here is the redevelopment blueprint:

---

### **Blueprint Report Structure**

### 0. Reverse Engineering Methodology

*   **Methodology:** Insights were derived through static analysis of the provided Ruby source files (`armor.rb`, `cman.rb`, `constants.rb`, `eaccess.rb`, `feat.rb`, `front-end.rb`, `global_defs.rb`, `gtk.rb`, `gui-login.rb`, `gui-manual-login.rb`, `gui-saved-login.rb`, `init.rb`, `lich.rb`, `map.rb`, `map_dr.rb`, `map_gs.rb`, `messaging.rb`, `shield.rb`, `spell.rb`, `stash.rb`, `update.rb`, `util.rb`, `version.rb`, `weapon.rb`, `xmlparser.rb`, `lich.rbw`). This involved examining class structures, method definitions, variable usage, comments, and inter-file dependencies (`require` statements).
*   **Assumptions:**
    *   Class names like `Armor`, `CMan` (Combat Maneuvers), `Feat`, `Spell`, `Shield`, `Weapon` represent specific game mechanics or character abilities within a MUD (Multi-User Dungeon) or similar online role-playing game.
    *   Class variables (e.g., `@@berserk` in `CMan`) store ranks or states of these abilities.
    *   `XMLData` is assumed to be a global object or module, not defined in the provided files, that holds parsed real-time game state information received from the game server via an XML stream. Many functions in `global_defs.rb` rely on it.
    *   `GameObj` is a similar global accessor for game objects (NPCs, PCs, items).
    *   "Lich" itself is a game assistance tool or botting framework.
    *   Files like `eaccess.rb` and `gui-login.rb` suggest direct interaction with game authentication and account management services provided by Simutronics (`eaccess.play.net`).
*   **Ambiguous Areas / Limited Information:**
    *   The overall entry point and main execution flow of the "Lich" application are not present in this snapshot.
    *   The exact business purpose and context of many game-specific abilities (e.g., "Adamantine Bulwark" in `shield.rb`) beyond their mechanical representation (rank, cost) are inferred.
    *   No database schemas or direct database interaction code (e.g., ActiveRecord models, SQL queries) are present in these files.
    *   No direct COM SDK interaction code is present in these files. `win32ole` is not explicitly required, though the context of Simutronics games often involves Windows.
    *   The `gtk.rb` file suggests optional GTK+-based GUI components, but their full integration and usage are not detailed.
*   **Confidence Justification:** See end of each major section.

*   _Confidence: High. Reason: This section describes the direct analytical approach to the provided codebase._

---

### 1. Introduction & Purpose

**1.1 Primary Functionality:**
The provided codebase forms part of "Lich," an auxiliary application designed to interact with and likely automate or assist gameplay in text-based MUDs, specifically Simutronics games (Gemstone IV, DragonRealms). Key functionalities evident are:
*   Authentication with game servers (`eaccess.rb`).
*   Management and tracking of character abilities, spells, and combat maneuvers (`armor.rb`, `cman.rb`, `feat.rb`, `spell.rb`, `shield.rb`, `weapon.rb`).
*   Parsing and utilizing game state information, likely from an XML stream (`xmlparser.rb`, `global_defs.rb`'s reliance on `XMLData`).
*   Session management for game clients (`front-end.rb`).
*   Providing a set of global utility functions for scripting game interactions (`global_defs.rb`).
*   Optional GUI components for login and settings management (`gui-*.rb`, `gtk.rb`).
*   A system for updating Lich itself (`update.rb`).

**1.2 Key Interactions (Inferred):**
*   **Database(s):**
    *   Technology: No direct database interaction (e.g., SQL, ActiveRecord) is evident in the provided files.
    *   Connection: Data persistence for Lich settings, script settings, and potentially game data (like map files or user variables) appears to be handled via SQLite (`Lich.db` in `lich.rb`, `init.rb` creates tables).
    *   Connection Strings: Not applicable for SQLite in this context; database file paths are constructed (e.g., `DATA_DIR}/lich.db3`).
*   **COM SDK:** No direct COM SDK usage (e.g., via `win32ole`) is evident in the provided files.
*   **File I/O:**
    *   `eaccess.rb`: Reads/writes `simu.pem` (SSL certificate) from `DATA_DIR`.
    *   `constants.rb`: Defines various directory paths (`LICH_DIR`, `DATA_DIR`, `SCRIPT_DIR`, etc.) using `File.dirname` and `File.expand_path`.
    *   `front-end.rb`: Creates and deletes JSON-based session files in a temporary directory (`Dir.tmpdir`). Uses `FileUtils.mkdir_p`, `File.open`, `File.delete`.
    *   `init.rb`: Checks for file existence, creates directories.
    *   `gui-login.rb`, `gui-saved-login.rb`: Read/write `entry.dat` (Marshal-dumped login credentials).
    *   `update.rb`: Downloads zip files, moves files, creates snapshot directories.
    *   `map*.rb`: Loads map data from `.dat`, `.xml`, or `.json` files.
    *   `lich.rb`: SQLite database interaction.
*   **Network Communication:**
    *   `eaccess.rb`: Establishes SSL TCP socket connections to `eaccess.play.net:7910` for authentication. Uses `OpenSSL::SSL::SSLSocket`.
    *   `lich.rbw` (main application file): Establishes TCP connections to game servers and listens for client frontend connections.

**1.3 CLI Nature:**
While `lich.rbw` processes command-line arguments for configuration (game server, frontend type, connection details), the provided library files (`armor.rb`, `eaccess.rb`, etc.) do not directly parse command-line arguments themselves. They are components used by the larger Lich application.
Example CLI invocation (inferred from `lich.rbw`'s argument processing):
`ruby lich.rbw --gemstone --stormfront --login MyCharName`
`ruby lich.rbw -g gs3.simutronics.net:4900 -w`

*   _Confidence: Medium. Reason: High for technical interactions found in code (SSL, file I/O, SQLite). Lower for overall "CLI nature" as the entry point logic is in `lich.rbw` rather than the libraries, and the exact behavior is an interplay of these components with the main application loop._

---

### 2. General Codebase Structure

*   **Key Directories/Modules and Roles:**
    *   `lib/`: Contains core library files providing specific functionalities (e.g., `eaccess.rb` for authentication, `spell.rb` for spell definitions, `map.rb` for game world mapping).
    *   `data/`: Implied by `constants.rb` and used by `eaccess.rb` for `simu.pem` and `lich.rb` for `lich.db3`. Stores persistent data.
    *   `scripts/`: Implied by `constants.rb`. Likely stores user-created or Lich-provided game scripts.
    *   `Lich` (module in `lich.rb`): Acts as a central namespace for global settings, database access, and utility functions.
    *   `Games::Gemstone::Spell`, `Games::Gemstone::GameObj`, etc. (in `lich.rbw` and related files): Namespaces for game-specific data structures and logic.
    *   `XMLData` (instance of `XMLParser`): Central object for accessing parsed game state.
*   **Architectural Patterns:**
    *   **Global Registries/Static-like Classes:** Classes like `Armor`, `CMan`, `Feat`, `Shield`, `Weapon`, and `Spell` use class variables (`@@variable_name`) and class methods extensively to store and provide access to game ability ranks, costs, and definitions. They function similarly to static classes or global registries in other languages.
    *   **Modular Design:** Functionality is somewhat separated into modules (`EAccess`, `Frontend`, `Lich::Util`, `Lich::Messaging`, `Lich::Stash`).
    *   **Procedural within Classes/Modules:** Much of the logic within methods is procedural.
    *   **Dynamic Dispatch:** Heavy use of `send` in ability classes (`Armor.[](name)`) for dynamic method invocation based on string inputs. `method_missing` is used as a fallback.
    *   **Global Helper Functions:** `global_defs.rb` defines a large number of global functions available to Lich scripts (e.g., `checkhealth`, `move`, `waitfor`).
    *   **GUI (Optional):** `gtk.rb` and `gui-*.rb` files indicate an optional GTK-based graphical user interface for login and settings.
*   **Indicators of Legacy or Idiomatic Ruby:**
    *   **Class Variables as Global State:** `@@variable_name ||= 0` pattern for initializing class variables that hold game-wide state.
    *   **Class-Level Accessors:** Defining class methods like `def CMan.berserk; @@berserk; end` and `def CMan.berserk=(val); @@berserk=val; end` for every ability.
    *   **`method_missing`:** Used for handling undefined ability lookups.
    *   **`send` for Dynamic Calls:** `Armor.send(name_string)` to call methods dynamically.
    *   **Global Variables/Constants:** Defined in `constants.rb` (e.g., `LICH_DIR`, `$lich_dir`).
    *   **Monkey Patching (potential):** While not directly evident in *these* files as modifications to core Ruby classes, the overall style suggests a system where such practices might occur in other parts of Lich or user scripts. The `gtk.rb` file itself contains polyfills/deprecations for Gtk methods.
    *   `||=` for memoization or default initialization of class variables.

*   _Confidence: High. Reason: Directly inferable from the file structure, Ruby syntax, and common Ruby patterns observed in the codebase._

---

### 3. Important Technical Details

*   **Database Access:**
    *   The primary database interaction is with SQLite via the `sqlite3` gem, managed within the `Lich` module (`lich.rb`).
    *   `Lich.init_db` creates tables like `script_setting`, `lich_settings`, `uservars`.
    *   Data is stored and retrieved using `Lich.db.execute` and `Lich.db.get_first_value`.
    *   Settings and user variables are marshalled/unmarshalled for storage (`Marshal.dump`, `Marshal.load`).
*   **COM SDK Interaction:** No direct COM SDK interaction (e.g., via `win32ole`) is present in these files.
*   **File I/O:**
    *   **Certificate Management (`eaccess.rb`):**
        *   Reads and writes `simu.pem` using `File.read` and `File.write`.
        *   Checks for file existence with `File.exist?`.
    *   **Session Management (`front-end.rb`):**
        *   Creates temporary session files (JSON format) using `File.open` in write mode.
        *   Uses `FileUtils.mkdir_p` to ensure directory existence.
        *   Deletes session files with `File.delete`.
    *   **Directory Constants (`constants.rb`):**
        *   Uses `File.dirname(File.expand_path($PROGRAM_NAME))` to establish base directories.
    *   **Update Mechanism (`update.rb`):**
        *   Downloads zip files using `open-uri` and `File.open` in binary write mode.
        *   Uses `Dir.mkdir`, `FileUtils.cp_r`, `FileUtils.remove_dir`, `FileUtils.mv` for managing update packages and snapshots.
    *   **Map Loading (`map*.rb`):**
        *   Loads map data from `.dat` (Marshal), `.xml` (REXML), or `.json` (JSON gem) files.
*   **Logging & Telemetry:**
    *   `Lich.log` (in `lich.rb`): Writes timestamped messages to `$stderr`, which is redirected to a debug log file in `init.rb`.
    *   `echo`, `_echo`, `respond`, `_respond` (in `global_defs.rb`): Custom methods for outputting messages, often prefixed with the script name, to the game client or Lich console. `silence_me` can suppress this output.
    *   `eaccess.rb` uses `Lich.log` for SSL certificate verification errors.
*   **Error Handling:**
    *   `eaccess.rb`: Uses `fail Exception` for critical authentication errors.
    *   `global_defs.rb`: `report_errors` block in `lich.rbw` wraps script execution to catch and log various exception types. Specific error messages are sent to the user via `respond`.
    *   `gtk.rb`: `Gtk.queue` block includes extensive rescue clauses for various error types, logging them via `Lich.log` and `respond`.
    *   `move` function in `global_defs.rb`: Contains extensive pattern matching for game server responses indicating failure or specific conditions, with retry logic or specific return values.
*   **Concurrency:**
    *   `global_defs.rb`:
        *   `watchhealth`: Creates a new `Thread` to monitor health and execute a block.
        *   `selectput`: Creates a `Thread` for timeout handling.
    *   `gtk.rb`: Uses `GLib::Timeout.add` for scheduling Gtk operations on the main Gtk thread.
    *   `lich.rbw` (main application file): Uses multiple threads for handling server communication (`Game.open` creates a thread), client communication, and detachable client connections. Mutexes (`Mutex.new`) are used for synchronizing access to shared resources like `@@cast_lock` in `Spell`, `@@lich_db` in `Lich`, and buffers in `SharedBuffer`.
*   **Network Communication (`eaccess.rb`):**
    *   Uses `TCPSocket.open` to connect to the authentication server.
    *   Wraps the TCP socket with `OpenSSL::SSL::SSLSocket` for SSL/TLS encryption.
    *   Requires `openssl` and `socket` gems.
    *   Performs certificate verification against a local `simu.pem` file.
    *   Communicates using a simple line-based protocol (e.g., sending "K", "A", "M" commands).
*   **GUI Handling (`gtk.rb`, `gui-*.rb`):**
    *   Conditionally requires `gtk3` gem.
    *   Provides shims and deprecation warnings for Gtk2 to Gtk3 API changes.
    *   Uses `Gtk.queue` to ensure Gtk operations run on the main Gtk thread.
    *   Defines UI elements programmatically (e.g., `Gtk::Window.new`, `Gtk::Button.new`).

*   _Confidence: High. Reason: These details are directly extracted from the provided Ruby code, focusing on libraries used and patterns implemented._

---

### 4. Pointers for Code Understanding (for a Newcomer to C# Migration)

*   **Critical Code Paths to Study First:**
    *   **`lich.rbw` (Main Application Loop):** Understand how it initializes, handles arguments, starts the game connection, client connection, and manages the main threads. This is the entry point.
    *   **`eaccess.rb` (`EAccess.auth`):** This is crucial for understanding the game authentication flow. Pay close attention to the SSL setup, command sequence, and password hashing.
    *   **`xmlparser.rb` (`XMLParser` class):** This class is central to how Lich receives and interprets game state. Understand how it parses the XML stream and updates global state objects (like `XMLData` itself, `GameObj`, `Spell`, `Stats`).
    *   **`global_defs.rb`:** This file contains a vast number of helper functions used by Lich scripts. Understanding their purpose is key to understanding script logic. The `move` function is a good example of game interaction logic.
    *   **Ability Classes (`Armor.rb`, `CMan.rb`, `Feat.rb`, `Shield.rb`, `Weapon.rb`, `Spell.rb`):** Analyze how these classes store ability ranks and costs (using class variables `@@`) and how they provide access via dynamic dispatch (`[]`, `[]=`, `send`). The `.known?` and `.affordable?` methods are common patterns.
    *   **`init.rb`:** Understand how Lich initializes its environment, checks for dependencies (SQLite3, GTK3), and sets up directories.
    *   **`Lich` module (`lich.rb`):** Core for settings, DB access, and global utilities.
*   **Complex or Risky Code Areas for Migration:**
    *   **Dynamic Dispatch in Ability Classes:** The use of `send` and string manipulation (e.g., `name.to_s.gsub(/[\s\-]/, '_').gsub("'", "").downcase`) to call accessor methods in `Armor`, `CMan`, etc., will need a more C#-idiomatic replacement (e.g., Dictionaries, Enums, or dedicated classes per ability).
    *   **Global State (`XMLData`, `GameObj`, `Stats`, `Skills`, `Spells`, etc.):** Ruby's flexibility allows these to be accessed globally. In C#, this will likely require a combination of static classes, singleton services, and dependency injection to manage state access in a more structured way.
    *   **`method_missing`:** Used in ability classes. This dynamic behavior needs to be explicitly handled or designed out in C#.
    *   **Extensive Global Functions (`global_defs.rb`):** These will need to be refactored into utility classes or services in C#. Maintaining them as global static methods in C# is possible but generally discouraged for large sets.
    *   **Thread Management and Mutexes:** While Ruby threads are present, their mapping to .NET threads and synchronization primitives (`Mutex` to `System.Threading.Mutex` or `lock`) needs care, especially considering Ruby's GIL vs .NET's true parallelism.
    *   **Error Handling (`report_errors` in `lich.rbw`, various `rescue` blocks):** The broad `rescue Exception` blocks should be refined to catch more specific exceptions in C#.
    *   **String-based Proc Evaluation (`StringProc` class, eval in `Spell` methods):** This is a security risk and difficult to maintain. This logic will need tobe rewritten in C# directly.
    *   **GTK GUI Components:** If the GUI is to be retained, migrating from Ruby/GTK3 to a .NET GUI framework (e.g., WPF, WinForms, MAUI, AvaloniaUI) will be a significant effort. The current GTK code is procedural UI construction.
*   **Notable TODO/FIXME/HACK Comments:**
    *   `armor.rb`: "## breakout for Armor released with PSM3", "## new code for 5.0.16" - Indicates evolution and potential for older, unrefactored code elsewhere or in history. Similar comments exist in `cman.rb`, `feat.rb`, `shield.rb`, `weapon.rb`.
    *   `global_defs.rb`: (In `move` function) `# avoid stomping the room for the entire session due to a transient failure` - Highlights a workaround for a specific issue.
    *   `global_defs.rb`: (In `sf_to_wiz`) `## The following should be deprecated with the direct-frontend-launch-method`, `## TODO: remove as part of chore/Remove unnecessary Win32 calls`
    *   `lich.rbw`: (In `Game.open`) `# Fixes invalid XML with nested single quotes...`, `# Fixes invalid XML with nested double quotes...` - Indicates workarounds for malformed server data.
    *   `spell.rb`: `# fixme: deal with them dirty bards!`, `# fixme: find multicast in target and check mana for it`
    *   `xmlparser.rb`: (In `initialize`) `# psm 3.0 dialogdata updates`, `# real id updates` - Shows areas of recent change.
    *   `gtk.rb`: Contains many `define_deprecated_singleton_method` and `respond "'Gtk::..."` calls, indicating it's a compatibility layer.

*   _Confidence: High. Reason: Identifies critical components and known Ruby patterns that pose challenges for a direct C# migration, based on the code itself._

---

### 5. Detailed Database Schema & Queries

**5.1 Database Schema Overview:**
The provided code interacts with an SQLite database, likely named `lich.db3`, located in the `DATA_DIR`. The schema is defined programmatically in `lich.rb` within the `Lich.init_db` method.

| Table Name             | Column Name | Type    | Constraints        | Description                                                                 |
| :--------------------- | :---------- | :------ | :----------------- | :-------------------------------------------------------------------------- |
| `script_setting`       | `script`    | `TEXT`  | `NOT NULL`         | Name of the Lich script these settings belong to.                           |
|                        | `name`      | `TEXT`  | `NOT NULL`         | Name of the specific setting.                                               |
|                        | `value`     | `BLOB`  |                    | Value of the setting, stored as a Marshalled Ruby object.                   |
|                        |             |         | `PRIMARY KEY(script, name)` |                                                                             |
| `script_auto_settings` | `script`    | `TEXT`  | `NOT NULL`         | Name of the Lich script.                                                    |
|                        | `scope`     | `TEXT`  |                    | Scope of the settings (e.g., global, game-specific, character-specific).    |
|                        | `hash`      | `BLOB`  |                    | Settings stored as a Marshalled Ruby Hash.                                  |
|                        |             |         | `PRIMARY KEY(script, scope)` |                                                                             |
| `lich_settings`        | `name`      | `TEXT`  | `NOT NULL`         | Name of the global Lich setting.                                            |
|                        | `value`     | `TEXT`  |                    | Value of the setting.                                                       |
|                        |             |         | `PRIMARY KEY(name)`  |                                                                             |
| `uservars`             | `scope`     | `TEXT`  | `NOT NULL`         | Scope for user-defined variables (likely character-specific).               |
|                        | `hash`      | `BLOB`  |                    | User variables stored as a Marshalled Ruby Hash.                            |
|                        |             |         | `PRIMARY KEY(scope)` |                                                                             |
| `trusted_scripts`      | `name`      | `TEXT`  | `NOT NULL`         | Name of a script that is trusted to run with higher privileges (Ruby <2.3). |
| `simu_game_entry`      | `character` | `TEXT`  | `NOT NULL`         | Character name for saved game login entry.                                  |
|                        | `game_code` | `TEXT`  | `NOT NULL`         | Game code (e.g., GS3, DR, GSX) for the saved entry.                         |
|                        | `data`      | `BLOB`  |                    | Login data (account, password, frontend) as a Marshalled Ruby object.     |
|                        |             |         | `PRIMARY KEY(character, game_code)` |                                                                             |
| `enable_inventory_boxes`| `player_id`| `INTEGER`| `NOT NULL`        | Player ID for whom inventory boxes are enabled.                             |
|                        |             |         | `PRIMARY KEY(player_id)` |                                                                             |

No views, stored procedures, or triggers are evident from the provided code for this SQLite database.

**5.2 LINQ Queries & SQL Equivalents:**
Not applicable. The database used is SQLite, and interactions are through direct SQL commands executed via the `sqlite3` Ruby gem, not LINQ.

*Example SQL from `lich.rb` (within `Setting.load` proc):*
```ruby
# Ruby code:
# Lich.db.get_first_value('SELECT value FROM script_setting WHERE script=? AND name=?;', script.name.encode('UTF-8'), setting.encode('UTF-8'))

# Equivalent SQL (conceptual, as '?' are placeholders):
# SELECT value FROM script_setting WHERE script='script_name_value' AND name='setting_name_value';
```

*Example SQL from `lich.rb` (within `Lich.win32_launch_method=`):*
```ruby
# Ruby code:
# Lich.db.execute("INSERT OR REPLACE INTO lich_settings(name,value) values('win32_launch_method',?);", val.to_s.encode('UTF-8'))

# Equivalent SQL:
# INSERT OR REPLACE INTO lich_settings(name,value) values('win32_launch_method', 'some_value');
```

**5.3 Data Access Patterns:**
*   **Direct SQL Execution:** Data access is primarily through direct execution of SQL statements using the `sqlite3` gem's `execute` and `get_first_value` methods.
*   **Data Marshalling:** Complex Ruby objects (Hashes, Arrays) are serialized using `Marshal.dump` before being stored in `BLOB` columns and deserialized using `Marshal.load` upon retrieval. This is common for `script_setting`, `script_auto_settings`, `uservars`, and `simu_game_entry` tables.
*   **Settings Management Abstraction:** Modules like `Setting`, `GameSetting`, `CharSetting`, and `Vars` provide an abstraction layer over the direct SQLite calls for managing script and user configurations. These modules handle the marshalling/unmarshalling and construct the appropriate SQL queries.
*   **No ORM:** An Object-Relational Mapper (like ActiveRecord or Entity Framework) is not used.
*   **Transaction Management:** In `Setting.save`, if multiple settings are being saved, it wraps the database operations in a `BEGIN` and `END` transaction.
*   **Caching:** Simple in-memory caching for settings is implemented in the `Settings` module (the `@@settings` hash). Data is loaded from the DB on first access per script/scope and then served from memory. Changes are periodically written back by a background thread or on explicit save.
*   **No Batching:** No explicit batching of database operations is evident beyond the single transaction in `Setting.save`.

*   _Confidence: High. Reason: The database schema and access patterns are explicitly defined or used in `lich.rb` and `init.rb`._

---

### 🎯 **Focus Area: Section 6 — Domain Model & Business Logic Description**

### 6.1 Domain Entities & Concepts

The primary domain revolves around a character in a MUD-style game, their abilities, game state, and interaction with the game client/server.

| Entity/Concept        | Key Properties/Attributes (Ruby Class Variables/Instance Vars)                                 | Data Types (Inferred) | Relationships                                                                | Purpose in Business Domain                                                                                                                                        |
| :-------------------- | :--------------------------------------------------------------------------------------------- | :-------------------- | :--------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Game Ability** (e.g., `CMan`, `Armor`, `Feat`, `Spell`, `Shield`, `Weapon` classes) | `@@<ability_name>` (e.g., `@@berserk`), `@@cost_hash` (for `CMan`, `Feat`, `Weapon`)         | Integer (ranks), Hash (costs) | None explicit between these classes; they are standalone registries.         | Represents a specific character ability, skill, feat, or spell. Stores its current rank/level and, for some, associated resource costs (e.g., stamina).      |
| **Spell Definition** (`Spell` class instances) | `num`, `name`, `type`, `circle`, `availability`, `cost` (Hash), `duration` (Hash), `msgup`, `msgdn` | Integer, String, Hash | Part of a list of all known spells (`@@list` in `Spell`).                | Defines a specific spell in the game, including its number, name, type, casting messages, cost, duration, and how to calculate these based on character skills. |
| **Lich Script** (`Script` class instances) | `name`, `vars` (arguments), `downstream_buffer`, `upstream_buffer`, `labels`, `current_label`, `paused`, `hidden` | String, Array, Hash   | Managed by Lich core (`lich.rbw`).                                       | Represents an executing user script or internal Lich task, managing its state, input/output, and execution flow through labels.                           |
| **Game Session/Connection** (`EAccess` module, `lich.rbw` logic) | `account`, `password`, `character`, `game_code`, `host`, `port`, `key` (session key)      | String, Integer       | Connects to Simutronics services.                                            | Manages authentication and the network connection to the game server.                                                                                             |
| **Client Session** (`Frontend` module) | `name`, `host`, `port` (for Lich's local server)                                              | String, Integer       | Used by game frontends to connect to Lich.                                 | Manages information for local game clients (e.g., Stormfront, Wizard) to connect to the Lich proxy.                                                           |
| **Game State** (`XMLData` object, implicitly) | `mana`, `health`, `stamina`, `room_title`, `room_description`, `indicator` (Hash), `injuries` (Hash), `prepared_spell`, etc. | Integer, String, Hash | Singleton-like global state.                                                 | Holds the current, parsed state of the player character and their environment as received from the game server's XML stream.                                    |
| **Game Object** (`GameObj` class and instances) | `id`, `noun`, `name`, `status` (for NPCs/PCs)                                                  | String, Hash          | `GameObj.inv`, `GameObj.loot`, `GameObj.npcs`, `GameObj.pcs` are collections. | Represents an entity in the game world (item, character, monster).                                                                                                  |
| **Map Room** (`Map` class instances in `map*.rb`) | `id`, `title`, `description`, `paths` (exits), `uid` (game's room ID), `location`, `wayto` (connections), `timeto` (travel time) | Integer, Array, Hash, String | Connected to other `Map` instances via `wayto`.                           | Represents a single room or location in the game world, including its description, exits, and connections to other rooms for navigation.                        |
| **Stashed Item** (`Lich::Stash` logic) | (Implicitly) reference to `GameObj` items, target container (e.g. `UserVars.lootsack`)        | `GameObj`             | Manages items in hands.                                                      | Facilitates temporarily stowing items from the character's hands into a container and retrieving them.                                                        |

**Computed Properties/Flags/Derived Fields:**
*   **Ability Classes (`CMan`, `Feat`, etc.):**
    *   `known?(name)`: Returns `true` if `AbilityClass[name] > 0`.
    *   `affordable?(name)`: Calculates if the character has enough resources (e.g., `XMLData.stamina`) to use the ability based on `@@cost_hash`.
    *   `available?(name)`: Combines `known?`, `affordable?`, and checks against game cooldowns/debuffs (via `Lich::Util.normalize_lookup`).
*   **`Spell` Class:**
    *   `timeleft`: Calculates remaining duration, decrementing based on `Time.now - @timestamp`.
    *   `active?`: Returns `true` if `timeleft > 0` and `@active` is true.
    *   `known?`: Determined based on character's spell circle ranks (`Spells.minorspiritual`, etc.) and `XMLData.level` vs spell number.
    *   `affordable?`: Checks mana, spirit, stamina costs against character's current values from `XMLData`.
    *   Cost methods (e.g., `mana_cost`, `spirit_cost`): Evaluate formulas defined in `spell-list.xml` (now part of the `Spell` class's internal data after loading) which often depend on `Skills` or `Stats`.
*   **`global_defs.rb` Functions:**
    *   Many `check<status>` functions (e.g., `checkpoison`, `checksitting`, `checkstunned`) derive boolean state from `XMLData.indicator` hash.
    *   `percent<stat>` functions (e.g., `percentmana`, `percenthealth`) calculate percentages based on current/max values from `XMLData`.
    *   `muckled?`: A composite status flag derived from `checkwebbed`, `checkdead`, `checkstunned`, `checksleeping`, `checkbound`.

**Entity Relationship Diagram (Conceptual):**
Due to the dynamic and global nature of many Ruby objects, a strict ERD is less applicable. However, a conceptual model:

```mermaid
graph TD
    A[Lich Application] --> B(Lich Script);
    B --> C{Game Interaction Logic \n (global_defs.rb)};
    A --> D(Lich Core Services \n Lich, XMLParser, EAccess, Frontend);
    D --> E(SQLite Database \n settings, uservars);
    C --> F(Ability Registries \n CMan, Armor, Spell);
    C --> G(Game State Access \n XMLData, GameObj);
    G --> H(Game Server \n via Network);
    D -- Manages Auth --> H;
    A --> I(Map Data \n map.rb);
    I --> E;
    C --> J(Stash Logic \n stash.rb);
    J --> G;

    subgraph GameMechanics
        F
    end
    subgraph GameWorld
        G
        I
    end
```
_Justification: A component diagram or high-level data flow is more suitable than a strict ERD to show how these parts interact, given the Ruby code's nature._

*   _Confidence: Medium-High. Reason: Core entities and their properties are clear from class definitions. Relationships are somewhat loose due to Ruby's dynamic typing and global access patterns but can be inferred. The purpose is derived from method names and comments._

---

### 6.2 Business Process Flows (Enhanced for Causality, Traceability, and Implementation)

#### **Process 1: Authenticate to Simutronics Game Service**

#### 6.2.1 Process Overview

*   **Process Name:** Game Account Authentication
*   **Trigger:** Invocation of `EAccess.auth(password:, account:, character: nil, game_code: nil, legacy: false)`
*   **Business Purpose:** To securely log a player into their Simutronics game account and select a character to play.

#### 6.2.2 Step-by-Step Logical Flow

```
Step 1: [Establish Secure Connection]
*   Actor: EAccess.socket()
*   Input: hostname ("eaccess.play.net"), port (7910)
*   Processing:
    1. Checks if "simu.pem" SSL certificate exists (`EAccess.pem_exist?`).
    2. If not, downloads it via `EAccess.download_pem` (plain TCP, then SSL handshake to get cert).
    3. Opens a TCPSocket to the host and port.
    4. Creates an OpenSSL::SSL::SSLContext, configures it with the PEM file for peer verification.
    5. Wraps the TCPSocket in an OpenSSL::SSLSocket.
    6. Initiates SSL connection (`ssl_socket.connect`).
    7. Verifies the connected peer's certificate against the local "simu.pem" (`EAccess.verify_pem`). If mismatch, re-downloads PEM.
*   Output: Connected and verified `ssl_socket`.
*   Where: eaccess.rb (methods: `socket`, `pem_exist?`, `download_pem`, `verify_pem`)
*   Why: To ensure a secure, encrypted channel for authentication and to verify the identity of the remote server.
```
```
Step 2: [Request Hashkey]
*   Actor: EAccess.auth (within the method, after `EAccess.socket` call)
*   Input: Connected `ssl_socket` from Step 1.
*   Processing:
    1. Sends "K\n" (Key request command) to the server.
    2. Reads the server's response (`EAccess.read`), which contains the hashkey.
*   Output: `hashkey` (string).
*   Where: eaccess.rb (method: `auth`, `read`)
*   Why: The hashkey is used in a client-side password obfuscation step.
```
```
Step 3: [Obfuscate Password]
*   Actor: EAccess.auth
*   Input: User-provided `password`, `hashkey` from Step 2.
*   Processing:
    1. Converts password and hashkey characters to their byte values.
    2. Iterates through password bytes, XORing `(password_byte - 32)` with `hashkey_byte`, then adds 32 back.
    3. Converts modified bytes back to characters and joins them into the obfuscated password.
*   Output: Obfuscated `password`.
*   Where: eaccess.rb (method: `auth`)
*   Why: To avoid sending the plaintext password over the wire, even over SSL, as an additional security measure or legacy requirement.
```
```
Step 4: [Send Authentication Credentials]
*   Actor: EAccess.auth
*   Input: User `account` name, obfuscated `password` from Step 3.
*   Processing:
    1. Sends "A\t#{account}\t#{obfuscated_password}\n" (Account authentication command) to the server.
    2. Reads server response (`EAccess.read`).
    3. Parses the response. If it doesn't match `/KEY\t(?<key>.*)\t/`, authentication failed; returns an error.
*   Output: Session `key` if successful, or an error string.
*   Where: eaccess.rb (method: `auth`, `read`)
*   Why: To submit the user's credentials for verification by the server.
```
```
Step 5: [List Game Subscriptions/Characters (Modern Flow)]
*   Actor: EAccess.auth
*   Input: `game_code`, `character` name (if not legacy flow).
*   Processing:
    1. Sends "M\n" (List game master info/menu). Reads response.
    2. If not `legacy`:
        a. Sends "F\t#{game_code}\n" (Select frontend type). Reads response.
        b. Sends "G\t#{game_code}\n" (Select game). Reads response.
        c. Sends "P\t#{game_code}\n" (Select payment type). Reads response.
        d. Sends "C\n" (List characters). Reads response.
        e. Finds the `char_code` corresponding to the provided `character` name from the character list response.
        f. Sends "L\t#{char_code}\tSTORM\n" (Login character command). Reads response.
        g. Parses the final login information (game host, port, session key etc.) into `login_info` hash.
*   Output: `login_info` (Hash containing connection details and session key).
*   Where: eaccess.rb (method: `auth`)
*   Why: To navigate the server's menu system to select the specific game and character the user wants to play.
```
```
Step 6: [List Game Subscriptions/Characters (Legacy Flow)]
*   Actor: EAccess.auth
*   Input: (Iterates through available games from "M" response).
*   Processing:
    1. Sends "M\n" (List game master info/menu). Reads response.
    2. If `legacy`:
        a. Iterates through each game returned by the "M" command.
        b. For each game:
            i.   Sends "N\t#{game_code}\n" (Navigate to game). Reads response.
            ii.  If response indicates "STORM" (StormFront compatible):
                1.  Sends "F\t#{game_code}\n". Reads response.
                2.  If response indicates valid subscription type:
                    a.  Sends "G\t#{game_code}\n". Reads response.
                    b.  Sends "P\t#{game_code}\n". Reads response.
                    c.  Sends "C\n". Reads response.
                    d.  Parses character list, creating a hash for each character (`game_code`, `game_name`, `char_code`, `char_name`) and adds to `login_info` array.
*   Output: `login_info` (Array of Hashes, each representing a playable character across all subscribed games).
*   Where: eaccess.rb (method: `auth`)
*   Why: To provide a list of all available characters for selection if a specific character wasn't pre-selected (used by GUI login).
```
```
Step 7: [Cleanup]
*   Actor: EAccess.auth
*   Input: Connected `ssl_socket`.
*   Processing: Closes the SSL socket (`conn.close unless conn.closed?`).
*   Output: None.
*   Where: eaccess.rb (method: `auth`)
*   Why: To release network resources.
```

#### 6.2.3 State Transitions & Data Flow

1.  **State Transition Table:** This process primarily involves transitions in the connection state and authentication status rather than persistent domain entities.

| Entity (Conceptual) | Field/Flag (Conceptual) | From State      | To State        | Triggering Step | Reason/Business Meaning                                    |
| :------------------ | :---------------------- | :-------------- | :-------------- | :-------------- | :--------------------------------------------------------- |
| Connection          | SSL Handshake           | Not Connected   | SSL Established | Step 1          | Secure channel ready for authentication.                   |
| Authentication      | User Credentials        | Unauthenticated | Authenticated   | Step 4          | User identity verified, session key (`KEY`) obtained.      |
| Character Selection | Character Login         | No Character    | Character Logged In | Step 5f/6b.ii.2d | Specific character selected and login parameters obtained. |

2.  **Process Visualization (As Appropriate):**
    *   **Diagram Type Chosen:** Sequence Diagram (Mermaid Syntax)
    *   _Justification: A Sequence Diagram is ideal for visualizing the ordered exchange of messages between the Lich client (`EAccess` module) and the Simutronics Authentication Server (`eaccess.play.net`). It clearly shows the request-response nature of the protocol._

    ```mermaid
    sequenceDiagram
        participant C as Lich Client (EAccess)
        participant S as Auth Server (eaccess.play.net)

        C->>S: TCP Connect
        S-->>C: TCP ACK
        C->>S: SSL Hello (Initiate SSL Handshake, send/verify certs)
        S-->>C: SSL Handshake Response
        Note over C,S: Secure SSL Tunnel Established

        C->>S: Send "K" (Request Hashkey)
        S-->>C: Send Hashkey
        C->>C: Obfuscate Password using Hashkey
        C->>S: Send "A" (Account, ObfuscatedPassword)
        S-->>C: Send "KEY" (Session Key) or Error

        alt Modern Flow (Specific Character)
            C->>S: Send "M" (Menu)
            S-->>C: Menu Response
            C->>S: Send "F" (Frontend Type)
            S-->>C: Frontend ACK
            C->>S: Send "G" (Game Code)
            S-->>C: Game ACK
            C->>S: Send "P" (Payment Type)
            S-->>C: Payment ACK
            C->>S: Send "C" (Character List)
            S-->>C: Character List Response
            C->>C: Parse character_code for target character
            C->>S: Send "L" (Login: char_code, STORM)
            S-->>C: Login Success (gamehost, gameport, KEY, etc.)
        else Legacy Flow (List All Characters)
            C->>S: Send "M" (Menu)
            S-->>C: Menu Response (List of Games)
            loop For Each Game
                C->>S: Send "N" (Navigate to Game)
                S-->>C: Game Navigation Response
                opt If Game is STORM compatible
                    C->>S: Send "F" (Frontend Type)
                    S-->>C: Frontend ACK
                    opt If Subscription Valid
                        C->>S: Send "G" (Game Code)
                        S-->>C: Game ACK
                        C->>S: Send "P" (Payment Type)
                        S-->>C: Payment ACK
                        C->>S: Send "C" (Character List)
                        S-->>C: Character List Response
                        C->>C: Add character details to login_info array
                    end
                end
            end
        end
        C->>S: Close Connection
    ```

3.  **State Model Notes:**
    *   The `EAccess` process itself is stateless between calls to `auth`. Each call establishes a new connection.
    *   The server-side maintains session state associated with the obtained `KEY`.

#### 6.2.4 Business Rules & Conditional Logic

*   **PEM Certificate:** The local `simu.pem` must exist, or it will be downloaded. If it exists, it must match the certificate presented by the server; otherwise, it's re-downloaded. This ensures Lich is connecting to a trusted Simutronics server.
*   **Password Obfuscation:** The specific XOR-based obfuscation algorithm must be correctly implemented.
*   **Command Sequence:** The sequence of commands ("K", "A", "M", "F", "G", "P", "C", "L") must be followed correctly for the server to process the login.
*   **Response Parsing:** Responses from the server must be parsed correctly to extract necessary information (hashkey, session key, character codes, login parameters). Failure to parse correctly (e.g., no `KEY` in auth response) results in login failure.
*   **Legacy vs. Modern Flow:** The `legacy` boolean flag in `EAccess.auth` dictates a different path for character/game enumeration, affecting what `login_info` is returned.

#### 6.2.5 External Dependencies & Effects

*   **Network I/O:** Communication with `eaccess.play.net` on port `7910` via TCP/SSL.
*   **File I/O:**
    *   Reads `DATA_DIR/simu.pem`.
    *   Writes `DATA_DIR/simu.pem` if it needs to be downloaded/updated.
*   **Libraries:** `openssl` for SSL/TLS, `socket` for TCP communication.

---

#### **Process 2: Query/Update Game Ability Rank (e.g., `CMan['berserk']` or `CMan['berserk'] = 1`)**

#### 6.2.1 Process Overview

*   **Process Name:** Game Ability Rank Access/Modification
*   **Trigger:** Call to `AbilityClass[ability_name]` (getter) or `AbilityClass[ability_name] = value` (setter), where `AbilityClass` is one of `Armor`, `CMan`, `Feat`, `Shield`, `Weapon`.
*   **Business Purpose:** To retrieve the current rank of a known game ability for the character, or to update this rank (likely as a result of game events or script actions).

#### 6.2.2 Step-by-Step Logical Flow

```
Step 1: [Normalize Ability Name]
*   Actor: AbilityClass.[] or AbilityClass.[]= (e.g., CMan.[] or CMan.[]=)
*   Input: `name` (String, the ability name, e.g., "Bull Rush" or "bull_rush").
*   Processing: Converts the input `name` to a standardized format:
    1. Convert to string (`to_s`).
    2. Replace spaces (`\s`) and hyphens (`\-`) with underscores (`_`).
    3. Remove apostrophes (`'`).
    4. Convert to lowercase (`downcase`).
*   Output: `normalized_name` (String, e.g., "bull_rush").
*   Where: armor.rb, cman.rb, feat.rb, shield.rb, weapon.rb (within `[]` and `[]=` methods)
*   Why: To ensure consistent lookup regardless of minor variations in input string format.
```
```
Step 2: [Dynamic Method Invocation (Getter)]
*   Actor: AbilityClass.[] (e.g., CMan.[])
*   Input: `normalized_name` from Step 1.
*   Processing: Uses Ruby's `send` method to dynamically call the class method corresponding to `normalized_name`.
    *   Example: If `normalized_name` is "bull_rush", `CMan.send("bull_rush")` is called, which executes `def CMan.bull_rush; @@bull_rush; end`.
*   Output: The integer value of the corresponding class variable (e.g., `@@bull_rush`).
*   Where: armor.rb, cman.rb, feat.rb, shield.rb, weapon.rb (within `[]` method)
*   Why: To retrieve the stored rank of the ability.
```
```
Step 3: [Dynamic Method Invocation (Setter)]
*   Actor: AbilityClass.[]= (e.g., CMan.[]=)
*   Input: `normalized_name` from Step 1, `val` (Integer, the new rank).
*   Processing: Uses Ruby's `send` method to dynamically call the class setter method.
    *   The method name is constructed by appending `=` to `normalized_name` (e.g., "bull_rush=").
    *   Example: If `normalized_name` is "bull_rush", `CMan.send("bull_rush=", val.to_i)` is called, which executes `def CMan.bull_rush=(val); @@bull_rush=val; end`.
*   Output: The `val` assigned to the class variable.
*   Where: armor.rb, cman.rb, feat.rb, shield.rb, weapon.rb (within `[]=` method)
*   Why: To update the stored rank of the ability.
```
```
Step 4: [Handle Unknown Ability (method_missing)]
*   Actor: AbilityClass.method_missing (e.g., CMan.method_missing)
*   Input: `arg1` (Symbol, the missing method name), `arg2` (optional arguments).
*   Processing: If `send` in Step 2 or 3 attempts to call a method that doesn't exist (because the ability name was invalid after normalization), `method_missing` is invoked.
    *   It outputs an error message using the `echo` function (e.g., "#{arg1} is not a defined CMan type...").
*   Output: Nil (implicitly, and an error message via `echo`).
*   Where: armor.rb, cman.rb, feat.rb, shield.rb, weapon.rb
*   Why: To provide feedback when an invalid ability name is used.
```

#### 6.2.3 State Transitions & Data Flow

1.  **State Transition Table:**

| Entity          | Field/Flag        | From State (Old Rank) | To State (New Rank) | Triggering Step       | Reason/Business Meaning            |
| :-------------- | :---------------- | :-------------------- | :------------------ | :-------------------- | :--------------------------------- |
| `CMan`          | `@@bull_rush`     | `X`                   | `Y`                 | `CMan["Bull Rush"]=Y` | Character's rank in Bull Rush updated. |
| `Armor`         | `@@armor_support` | `A`                   | `B`                 | `Armor["Armor Support"]=B` | Character's rank in Armor Support updated. |

2.  **Process Visualization (As Appropriate):**
    *   **Diagram Type Chosen:** Flowchart (Mermaid Syntax)
    *   _Justification: A Flowchart clearly illustrates the conditional logic and dynamic dispatch involved in accessing or updating an ability's rank._
    ```mermaid
    graph TD
        A[Start: AbilityClass[name] or AbilityClass[name]=val] --> B{Input: name, (optional) val};
        B --> C[Normalize name string \n to_s.gsub.downcase];
        C --> D{Is it a Getter or Setter?};
        D -- Getter --> E[Call AbilityClass.send(normalized_name)];
        E --> F{Method Exists?};
        F -- Yes --> G[Return @@ability_rank];
        F -- No --> H[Call method_missing];
        H --> I[echo "Error: ability not defined"];
        I --> Z[End];
        D -- Setter --> J[Call AbilityClass.send(normalized_name + \"=\", val.to_i)];
        J --> K{Method Exists?};
        K -- Yes --> L[Set @@ability_rank = val];
        L --> M[Return val];
        K -- No --> H;
        G --> Z;
        M --> Z;
    ```

3.  **State Model Notes:**
    *   Each ability (e.g., `CMan.berserk`, `Armor.armor_blessing`) is an independent integer state (rank).
    *   There are no defined invalid transitions other than attempting to access a non-existent ability, which `method_missing` handles.

#### 6.2.4 Business Rules & Conditional Logic

*   **Name Normalization:** Input ability names are consistently normalized before use.
*   **Integer Conversion for Setters:** The value provided to a setter is converted to an integer (`val.to_i`).
*   **`known?(name)`:** An ability is "known" if its rank (the value of its class variable) is greater than 0.
*   **`affordable?(name)` (`CMan`, `Feat`, `Weapon`):**
    *   Looks up the stamina cost in `@@cost_hash` using the normalized name.
    *   Compares this cost to `XMLData.stamina` (current character stamina).
    *   Returns `true` if cost < current stamina.
*   **`affordable?(name)` (`Armor`, `Shield`):**
    *   `Armor`: Always returns `true` (comment indicates armor abilities have no cost).
    *   `Shield`: Uses `@@shield_techniques` hash, similar to `CMan`'s `@@cost_hash`.
*   **`available?(name)` (`CMan`, `Feat`, `Shield`, `Weapon`):**
    *   Combines `known?(name)` AND `affordable?(name)`.
    *   AND NOT on cooldown (checks `Lich::Util.normalize_lookup('Cooldowns', name)`).
    *   AND NOT affected by "Strained Muscles" debuff (checks `Lich::Util.normalize_lookup('Debuffs', 'Strained Muscles')`).

#### 6.2.5 External Dependencies & Effects

*   **`XMLData.stamina`:** Used by `affordable?` methods to check current character stamina. This is an external dependency, presumably updated by Lich's core XML parsing.
*   **`Lich::Util.normalize_lookup`:** Used by `available?` methods to check cooldowns and debuffs. This is an external utility module within Lich.
*   **`echo` function:** Used by `method_missing` to output error messages.

---

### 6.3 Domain-Specific Terminology

*   **LICH_DIR, TEMP_DIR, DATA_DIR, SCRIPT_DIR, LIB_DIR, MAP_DIR, LOG_DIR, BACKUP_DIR:** Standardized directory names within the Lich application structure, defining where different types of files (core libraries, scripts, data, logs, backups) are stored.
*   **PSM3:** Acronym for "Post-Spell Mastery 3," referring to a significant update or version of the game's spell and combat systems. Files like `armor.rb` are noted as "breakout for Armor released with PSM3."
*   **Ability:** A general term for a character's skill, spell, combat maneuver, feat, or armor/shield/weapon proficiency (e.g., `berserk` is a CMan ability, `armor_blessing` is an Armor ability).
*   **Rank:** The numerical level or proficiency a character has in a specific ability. Typically stored as an integer in the class variables of `Armor`, `CMan`, etc.
*   **`known?` (method):** A common method in ability classes (`Armor`, `CMan`, etc.) to check if a character has at least one rank in an ability, meaning they "know" it.
*   **`affordable?` (method):** A common method in ability classes to check if a character has sufficient resources (usually stamina) to use an ability.
*   **`available?` (method):** A common method in ability classes that combines checks for `known?`, `affordable?`, and game-specific conditions like cooldowns or debuffs.
*   **`XMLData`:** A presumed global object within Lich that holds real-time game state information parsed from the XML stream sent by the Simutronics game server (e.g., `XMLData.stamina`, `XMLData.indicator`).
*   **`GameObj`:** A presumed global object or set of classes within Lich for accessing information about in-game objects like items, NPCs, and PCs.
*   **`Spell` (class):** Manages definitions and states of game spells.
*   **`Script.current`:** A Lich core object representing the currently executing script, providing access to its context and methods.
*   **`echo`, `_respond`, `fput`, `put`:** Custom Lich functions for script output and sending commands to the game.
*   **Roundtime (`rt`), Cast Roundtime (`castrt`):** Game mechanics representing recovery time after performing actions or casting spells. Functions like `waitrt`, `checkrt` manage these.
*   **PEM (Privacy Enhanced Mail):** Refers to the `simu.pem` file, an SSL certificate used by `eaccess.rb` to verify the authenticity of the Simutronics authentication server.
*   **Frontend:** The game client software used by the player (e.g., Wizard, Stormfront, Genie). Lich acts as a proxy between the frontend and the game server.
*   **`win32ole`:** A Ruby library for interacting with COM objects on Windows. Mentioned in context, though not directly used in the provided files, it's relevant for Simutronics games which often have Windows-based clients or SDKs.
*   **SGE (Simutronics Game Entry):** A launcher application for Simutronics games. `lich.rb` includes functions to link/unlink Lich with SGE.
*   **SAL (Simutronics Autolaunch):** `.sal` files are launch files for Simutronics games, containing connection parameters.
*   **Class Variables (`@@var_name`):** Used extensively in `Armor`, `CMan`, etc., to store global-like state for ability ranks and costs.

*   _Confidence: High. Reason: Terms are directly extracted from code comments, variable names, method names, and known Lich/Simutronics game terminology._

---

### 7. External Dependencies & Environment Details

*   **Required Ruby Gems:**
    *   `openssl`: Used by `eaccess.rb` for SSL/TLS communication during authentication.
    *   `socket`: Standard Ruby library used by `eaccess.rb` for TCP socket connections.
    *   `sqlite3`: Used by `lich.rb` and `init.rb` for database storage.
    *   `rexml/document`, `rexml/streamlistener`: Used by `xmlparser.rb` and `map*.rb` for parsing XML data from the game or map files.
    *   `json`: Used by `front-end.rb` for session files and `map*.rb` for JSON map files.
    *   `gtk3` (Optional): Used by `gtk.rb` and `gui-*.rb` files for a graphical user interface. If not present, Lich likely runs in a command-line or headless mode.
    *   `fileutils`: Standard Ruby library used by `front-end.rb` and `update.rb` for file system operations.
    *   `tempfile`: Standard Ruby library, likely used by `front-end.rb` (though `Dir.tmpdir` is directly used).
    *   `open-uri`: Used by `update.rb` for downloading updates.
    *   `zlib`: Used by `Script` class in `lich.rbw` if scripts are gzipped.
    *   `fiddle`: Used by `init.rb` (conditionally on Windows) for interacting with Windows API (e.g., MessageBox).
    *   `win32/registry` (Windows only): Used by `init.rb` for reading Windows registry to find game client installations.
*   **Database Versions and Editions:**
    *   SQLite 3: The `sqlite3` gem is used. The specific version of SQLite itself would depend on the gem's bundled library or the system's SQLite installation.
*   **COM SDK Vendor, Version, Documentation:** No COM SDK is directly used in the provided files.
*   **Sample File Formats:**
    *   **`simu.pem` (`DATA_DIR/`):** Standard X.509 PEM-encoded SSL certificate file.
    *   **Session Files (`Dir.tmpdir/simutronics/sessions/*.session`):** JSON files created by `front-end.rb`, containing:
        ```json
        {
          "name": "CharacterNameOrAlias",
          "host": "127.0.0.1", // Lich's local listening host
          "port": 12345       // Lich's local listening port
        }
        ```
    *   **`entry.dat` (`DATA_DIR/`):** Binary file containing Marshal-dumped array of Hashes for saved login credentials (used by `gui-login.rb`, `gui-saved-login.rb`). Each hash likely contains: `{:char_name, :game_code, :game_name, :user_id, :password, :frontend, :custom_launch, :custom_launch_dir}`.
    *   **Map Files (`DATA_DIR/<game>/map-*.dat` or `.xml` or `.json`):**
        *   `.dat`: Marshal-dumped Ruby array of `Map` objects.
        *   `.xml`: XML format (see `Map.load_xml` and `Map.save_xml` for structure).
        *   `.json`: JSON array of room objects (see `Map.to_json` for structure).
*   **External Service APIs Integrated:**
    *   **Simutronics Authentication Service:**
        *   Endpoint: `eaccess.play.net:7910`
        *   Protocol: Custom TCP-based protocol over SSL/TLS.
        *   Authentication: Proprietary key-request and password-obfuscation mechanism detailed in `eaccess.rb`.
    *   **GitHub API (for updates in `update.rb`):**
        *   Endpoint: `https://api.github.com/repos/elanthia-online/lich-5/releases/latest` (to get release info).
        *   Endpoint: Zipball URL from the release info (e.g., `https://github.com/elanthia-online/lich-5/archive/refs/tags/vX.Y.Z.zip`) for downloading updates.
        *   Protocol: HTTPS.
        *   Authentication: None (public repository access).
*   **Operating System & Environment:**
    *   The codebase contains conditional logic for Windows (`RUBY_PLATFORM =~ /mingw|win/i`), Linux/macOS (implied elsewhere), and WINE (`defined?(Wine)`).
    *   Expects a file system structure as defined in `constants.rb`.
    *   May run with or without a GUI (GTK3).
    *   Relies on environment variables like `HOME` or `WINEPREFIX` when running under WINE.

*   _Confidence: Medium-High. Reason: Dependencies like gems and network endpoints are explicit. File formats are inferable from their creation/parsing logic. Specific OS details are based on conditional compilation/checks in the code._

---

### 8. Known Limitations & Risks

*   **Global State Management:** The heavy reliance on class variables (`@@var`) for game ability ranks (`Armor`, `CMan`, etc.) and global-like access to `XMLData`, `GameObj` makes state management complex and tightly coupled. This can lead to difficulties in testing, debugging, and parallel execution if true parallelism were introduced (Ruby's GIL mitigates some of this for now). Migration to C# would require a significant redesign for state handling (e.g., using dependency injection, dedicated state services, proper encapsulation).
*   **Dynamic Ruby Features:** The extensive use of `send` for dynamic method calls and `method_missing` for fallback logic in ability classes makes static analysis harder and direct translation to C# challenging. This dynamic behavior would need to be mapped to more explicit C# constructs (e.g., dictionaries, strategy pattern, or code generation if ability sets are large and fixed).
*   **Reliance on External `XMLData` Structure:** Many global functions and class methods depend on the specific structure and availability of data within the `XMLData` object (e.g., `XMLData.stamina`, `XMLData.indicator`). The definition and population of `XMLData` are external to these files (handled by `xmlparser.rb` and the main game loop). Any changes to the game's XML stream format could break numerous parts of Lich.
*   **Error Handling Granularity:** Some error handling is broad (e.g., `rescue Exception` in `lich.rbw`). In a C# migration, these should be refined to catch more specific exception types for better diagnostics and recovery.
*   **String-Based Code Evaluation:** The `StringProc` class and `eval` usage (e.g., in `Spell` cost/duration calculations) introduce potential security risks if the strings can be influenced by external untrusted sources, and make debugging difficult. This logic should be rewritten natively in C#.
*   **Performance of `method_missing` and `send`:** While idiomatic in Ruby, heavy reliance on these for core game mechanic lookups might introduce performance overhead compared to direct method calls or dictionary lookups, especially if called very frequently.
*   **File I/O for State:** Marshalling entire data structures (like the map database or login entries) can be inefficient for large datasets and prone to corruption if the process is interrupted. A more robust database-centric approach or structured file format might be better for larger data.
*   **Lack of Formal Database Schema for Game Data:** While SQLite is used for settings, core game data like abilities and their costs are hardcoded into Ruby classes. This makes updates to game mechanics require code changes.
*   **GUI Migration Complexity:** If the GTK-based GUI (`gui-*.rb` files) is a required feature, migrating this to a .NET GUI framework will be a substantial sub-project, as the UI is built programmatically and tied to Ruby/GTK bindings.
*   **Security of Stored Credentials:** `gui-saved-login.rb` implies passwords might be stored (albeit Marshal-dumped). Secure storage of credentials (e.g., using OS credential manager) would be critical in a rewrite. The `eaccess.rb` password obfuscation is client-side only and not a replacement for secure storage at rest.
*   **Single Point of Failure for `XMLData` Parsing:** If `xmlparser.rb` fails or the XML stream from the game is malformed in an unexpected way, the `XMLData` object could become inconsistent, leading to widespread errors in dependent modules.

*   _Confidence: Medium. Reason: Technical limitations are evident from the Ruby code's patterns. Risks are assessed based on common challenges when migrating dynamic languages to statically-typed ones and general software engineering best practices._

---

### Appendix A: Traceability & Migration Matrix (Example)

| Ruby Component / Pattern                                   | Purpose                                                                  | Key Gems / Code (`*.rb` file, Class/Module::method)                                                                 | Proposed C# / .NET 9 Equivalent & Strategy                                                                                                                                                                                                                            |
| :--------------------------------------------------------- | :----------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Ability Classes** (`Armor`, `CMan`, `Feat`, `Spell`, etc.) | Store & manage character ability ranks, costs, availability.             | `armor.rb`, `cman.rb`, etc. (e.g., `CMan.[]`, `CMan.known?`, `CMan.affordable?`, `Spell.cast`)                         | **C# Static Classes or Services with Dictionaries/Enums:** Abilities as `enum` members. Ranks and costs stored in `Dictionary<AbilityEnum, int>` or `Dictionary<AbilityEnum, CostObject>`. Methods like `IsKnown(AbilityEnum)`, `IsAffordable(AbilityEnum)` become static methods or service instance methods. `Spell` class becomes a regular C# class, spell definitions could be loaded from JSON/XML into a list of `Spell` objects. |
| **`EAccess` Module**                                       | Authenticate to Simutronics game servers.                                | `eaccess.rb::auth`, `eaccess.rb::socket` <br/> Gems: `openssl`, `socket`                                               | **C# `SimuAuthService` Class:** Use `System.Net.Http.HttpClient` (if applicable, or `System.Net.Sockets.TcpClient` with `System.Net.Security.SslStream`). SSL/TLS handling via .NET libraries. Password obfuscation logic replicated. PEM cert handling via `X509Certificate2`. |
| **`XMLData` (Implicit via `xmlparser.rb`)**                  | Global access to parsed real-time game state.                            | `xmlparser.rb`, `global_defs.rb` (extensive usage like `XMLData.stamina`, `XMLData.indicator`)                      | **C# `GameStateService` (Singleton or DI):** A class holding game state properties. Updated by an XML parsing component. Properties could raise events on change for reactive UI/logic. Consider `INotifyPropertyChanged`.                                                  |
| **`GameObj` (Implicit)**                                     | Global access to parsed in-game objects (items, NPCs, PCs).              | `lich.rbw` (GameObj class definition), `xmlparser.rb` (populates it)                                                | **C# `GameObjectManagerService` / `Entity` Classes:** A service to manage collections of `Entity` objects (e.g., `Player`, `NPC`, `Item` classes).                                                                                                                 |
| **Global Functions (`global_defs.rb`)**                      | Provide utility functions for Lich scripts.                                | `global_defs.rb` (e.g., `checkhealth`, `move`, `waitfor`, `fput`)                                                     | **C# Utility Classes/Services:** Group related functions into static utility classes (e.g., `GameHelper`, `ScriptHelper`) or instance-based services accessible via DI. For example, `move` becomes `GameInteractionService.MoveAsync()`.                                      |
| **SQLite Database Access (`Lich` module)**                 | Store Lich settings, script settings, user variables.                    | `lich.rb::db`, `lich.rb::init_db`, `Setting` module <br/> Gem: `sqlite3`                                              | **C# Data Access Layer using Entity Framework Core (SQLite Provider) or Dapper:** Define C# model classes for tables. Use EF Core DbContext or Dapper for CRUD operations. Marshalling of BLOBs needs replacement (e.g., JSON serialization).                           |
| **Map System (`map*.rb`)**                                   | Load, store, and query game world map data.                              | `map.rb`, `map_dr.rb`, `map_gs.rb`                                                                                    | **C# `MapService` and `Room` Class:** Load map data (JSON/XML preferred over Marshal) into a collection of `Room` objects. Dijkstra's algorithm can be implemented as a method in `MapService`.                                                                   |
| **Lich Script Engine (`Script`, `ExecScript` classes)**      | Load and execute user scripts.                                           | `lich.rbw` (class definitions)                                                                                      | **C# Scripting Engine (Optional):** If scripting is to be retained, consider Roslyn for C# scripting or a dedicated scripting language integration (e.g., Lua, Python via IronPython). Otherwise, features become native C# modules.                                 |
| **GUI Components (`gui-*.rb`, `gtk.rb`)**                    | Provide a graphical login and settings interface.                        | `gui-login.rb`, `gui-manual-login.rb`, `gui-saved-login.rb`, `gtk.rb` <br/> Gem: `gtk3`                             | **.NET GUI Framework (e.g., WPF, MAUI, AvaloniaUI):** Rebuild UI using XAML or C# code-behind. Separate UI logic from business logic (MVVM or similar pattern).                                                                                                  |
| **File System Constants (`constants.rb`)**                   | Define standard directory paths for Lich.                                  | `constants.rb`                                                                                                      | **C# Configuration/Path Management:** Use `appsettings.json` for configurable paths or a static `PathHelper` class using `System.IO.Path` and `Environment.GetFolderPath`.                                                                                          |
| **Update Mechanism (`update.rb`)**                         | Download and apply updates to Lich.                                      | `update.rb` <br/> Gem: `open-uri`                                                                                     | **C# Update Service:** Use `HttpClient` to fetch release info and download update packages. Use `System.IO.Compression.ZipFile` for unzipping. File operations via `System.IO`.                                                                                    |
| **Frontend Session Mgmt (`front-end.rb`)**                 | Create session files for client FEs.                                     | `front-end.rb`                                                                                                      | **C# `FrontendSessionService`:** Manage session file creation/deletion.                                                                                                                                                                                             |
| **Stash Logic (`stash.rb`)**                               | Stash/retrieve items from hands.                                         | `stash.rb` (`Lich::Stash::stash_hands`, `Lich::Stash::equip_hands`)                                                 | **C# `InventoryService` or `StashService`:** Encapsulate logic for stashing/equipping items, interacting with the C# `GameStateService` / `GameObjectManagerService`.                                                                                            |
| **Messaging (`messaging.rb`)**                             | Format and send messages to client/stream windows.                       | `messaging.rb` (`Lich::Messaging.msg`, `Lich::Messaging.stream_window`)                                               | **C# `NotificationService` or `OutputFormatter`:** Handle message formatting (e.g., color codes, XML tags for specific frontends) and sending output to the appropriate client interface.                                                                          |

---
### **End of Mandate**
