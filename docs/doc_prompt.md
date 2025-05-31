You are an experienced software engineer and technical writer, familiar with Ruby and scripting for game clients like Lich. Your task is to analyze the following Ruby Lich script/library codebase and generate clear, comprehensive documentation in Markdown format. The codebase has no existing documentation and is considered legacy. Documentation should be professional, well-structured, and suitable for onboarding new scripters or understanding its functionality.

This is for Gemstone IV

**Core Task:**
Analyze the provided Ruby Lich script/library and generate comprehensive documentation.

**Documentation Requirements:**

1.  **High-Level Overview & Script Purpose**
    *   Summarize the primary purpose of the script/library. What problem does it solve for the player? What game does it appear to target?
    *   Describe its main mode of operation: Is it a utility library called by other scripts? A standalone automated script? A set of commands for the player?
    *   How is the script typically initiated or used (e.g., Lich command, automatic execution on game events, ` Lich::start ` call from another script)?
    *   Identify key entry points or primary script files if it's a multi-file library.

2.  **Code Flow & Key Components**
    *   Describe the primary flow of execution for its main features.
    *   Outline major Ruby modules, classes, or significant standalone methods and their responsibilities.
    *   How do these components interact? (e.g., event-driven, direct method calls).
    *   Provide a simple diagram (using Mermaid.js syntax if possible, e.g., ` ```mermaid graph TD; A-->B; ``` `) or a structured bullet-list representing the script's functional architecture.

3.  **Game Interaction & Automation**
    *   Identify how the script interacts with the game client (e.g., sending commands via `put`, `fput`; matching game text via `match`, `waitfor`, `Lich::Messaging`; using Lich client variables or functions).
    *   List key game commands the script sends.
    *   List important game text patterns the script reacts to.
    *   Describe any significant automation loops or state machines.

4.  **Data Storage & Persistence**
    *   Identify how the script stores data:
        *   Lich variables (`vars`, `gvars`, `cvars`): List key variables and their purpose.
        *   Configuration files (e.g., YAML, JSON, plain text): Describe their structure and purpose.
        *   In-memory data structures (hashes, arrays) if they are central to long-term state.
    *   For any structured data (e.g., classes representing game items, character stats):
        *   Describe its fields/attributes and their purpose.

5.  **Configuration**
    *   List all ways the script can be configured:
        *   Lich variables that users are expected to set.
        *   Configuration files (e.g., `settings.yaml`, custom files) and their key settings.
        *   Any in-script constants that a user might need to modify (though this is less ideal).
    *   Explain the effect of key configuration options on the script's behavior.

6.  **Lich Commands & User Interface**
    *   If the script provides commands for the user to type into Lich, list them.
    *   For each command:
        *   Syntax (including arguments).
        *   Purpose and what it does.
        *   Example usage.
    *   Describe any other ways the user interacts with the script (e.g., through game aliases set up by the script).

7.  **Event Handling & Recurring Tasks**
    *   Describe how the script handles game events (e.g., `matchadd`, `matchwait`, custom event systems).
    *   Detail any recurring tasks managed by ` Lich::queue `, ` Lich::loop `, or other timers. What do these tasks do?

8.  **External Dependencies & Integrations (Lich and Ruby Gems)**
    *   List any Ruby gems the script `require`s (beyond standard Ruby libraries) and their likely purpose.
    *   Identify any other Lich scripts it explicitly interacts with or depends on (if discernible).
    *   **Key Game System Interactions (Provide Pseudo-code where possible):**
        *   **Authentication to EAccess (or similar game account portal):** If the script handles this, provide pseudo-code for the logic.
            *   Example Pseudo-code Structure:
                ```pseudo
                FUNCTION authenticate_eaccess(username, password):
                  // Check for existing valid session/token
                  // If not valid or expired:
                  //   PREPARE eaccess_login_request (e.g., HTTP POST to specific URL with credentials)
                  //   SEND request
                  //   RECEIVE response
                  //   PARSE response for success/failure and session token
                  //   STORE session token (e.g., in a Lich variable or temp file)
                  //   RETURN success/failure and token
                ENDFUNCTION
                ```
        *   **Logging into the Game Character:** If the script automates this, provide pseudo-code.
            *   Example Pseudo-code Structure:
                ```pseudo
                FUNCTION login_to_game_character(character_name, account_credentials_or_token):
                  // INITIATE connection to game (if not already connected)
                  // NAVIGATE pre-login screens (if any, e.g., sending 'ENTER', specific commands)
                  // IF eaccess_token_needed_and_available:
                  //   USE eaccess_token_for_login_step
                  // ELSE:
                  //   SEND username/password_if_prompted_directly
                  // SELECT character_name from list (e.g., send number or name)
                  // HANDLE any post-character-selection prompts (e.g., password)
                  // WAITFOR game_entry_confirmation_text
                  // RETURN success/failure
                ENDFUNCTION
                ```
        *   Identify any other significant external system interactions (e.g., fetching data from a web API).

9.  **Error Handling & Logging**
    *   Describe common error handling patterns (e.g., `begin..rescue..end` blocks).
    *   How does the script log information or errors? (e.g., `echo`, `Lich::log`, `debug_log`, writing to files).

10. **Security Considerations**
    *   Highlight handling of sensitive information like game credentials (username, password, eaccess details).
    *   Note any use of hardcoded secrets (recommend remediation).
    *   Are credentials stored? If so, how and where?

11. **Other Important Logic**
    *   Summarize any particularly complex algorithms, parsing logic, or game-specific strategies implemented by the script.
    *   Note any known limitations if discernible from comments or code structure.

12. **Setup and Installation (Initial Observations)**
    *   Based on the code, what are the likely steps to install and set up this script for use with Lich?
        *   Required Ruby version (if discernible).
        *   Any gem dependencies to install.
        *   Location for script files (e.g., Lich `scripts` directory).
        *   Initial configuration steps (e.g., setting Lich variables, editing a config file).

13. **Assumptions and Areas for Clarification**
    *   Clearly list any significant assumptions you made during your analysis (e.g., "Assuming `handle_combat_round` is the main combat logic loop due to its name and typical Lich patterns").
    *   Identify areas where purpose or functionality is unclear and would benefit from human clarification. Frame these as specific questions.

**Output Format:**
*   Generate all documentation in a single, well-structured Markdown file.
*   Use headings (H1, H2, H3), bullet points, numbered lists, and code blocks (` ```ruby ... ``` ` for Ruby code, ` ```text ... ``` ` for game text examples) for clarity and readability.
*   Include a Table of Contents at the beginning.
*   Structure the documentation to allow quick onboarding for new scripters and efficient troubleshooting.
*   Where appropriate, include placeholders like `[TODO: Verify X with an experienced user]` or `[ASSUMPTION: Y]` if you are inferring something critical.

**Guiding Principles for AI:**
*   **Focus on Lich Context:** Interpret code patterns through the lens of Lich scripting conventions.
*   **Infer Intelligently:** Make educated guesses based on naming conventions, common Lich functions, and game mechanics, but clearly label them as assumptions.
*   **Be Practical:** The goal is useful, actionable documentation for users of the script.
