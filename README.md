# 🔮 Oleani 🔮

## Project Description

Oleani is a **proof-of-concept and learning experience** 🎓 focused on exploring the capabilities and limitations of various AI models 🤖 in assisting with the migration of an application from GoLang to .NET 9. The primary goal is to observe the "good, the bad, and the weird" aspects of AI-assisted code generation and translation, acknowledging that significant manual correction and oversight are often required.

This solution encompasses several components designed to interact with online gaming environments and communication platforms:

1.  🌐 **LNet Library:** A C# library for connecting to and communicating over the LNet protocol, specifically tailored for Gemstone IV's LNet.
2.  💎 **Gemstone IV Library:** A C# library for connecting directly to Gemstone IV game instances. It includes parsing capabilities for specific game events like ESP (Extra Sensory Perception) thoughts and character deaths.
3.  💬 **Discord Library (Future):** A planned C# library to bridge communications, shuttling messages between LNet, Gemstone IV, and a Discord server.
4.  ⚙️ **Service Application:** The core application that integrates these libraries, orchestrating the flow of information and managing connections.

This project serves as a practical testbed for AI-driven development workflows, highlighting both the potential efficiencies and the common pitfalls encountered.

## 📜 Table of Contents

*   [Project Description](#project-description)
*   [Technologies Used](#technologies-used)
*   [Project Goals & Philosophy](#project-goals--philosophy)
*   [Current Status](#current-status)
*   [Libraries](#libraries)
    *   [LNet Library](#lnet-library)
    *   [Gemstone IV Library](#gemstone-iv-library)
    *   [Discord Library (Future)](#discord-library-future)
*   [Disclaimer](#disclaimer)

## 🛠️ Technologies Used

Oleani leverages a modern .NET stack and focuses on high-performance and asynchronous operations:

*   🚀 **.NET 9 (Preview/Target):** Utilizing the latest features and performance improvements from the .NET platform.
*   **C#:** The primary programming language.
*   ⚡ **Asynchronous Programming (`async`/`await`):** Heavily used for non-blocking I/O and responsive applications.
*   🌊 **`System.IO.Pipelines` / `Channels`:** For efficient, low-allocation stream processing and message passing.
*   🧠 **`Span<T>` and `Memory<T>`:** For high-performance, allocation-free memory manipulation.
*   📝 **ZLogger:** A high-performance, structured logging library for .NET.
*   🤖 **AI Code Generation Tools:** Various AI models (e.g., GPT-*, Claude 4, Gemini 2.5 Pro) were used as aids during the GoLang to .NET migration process, with subsequent manual review and refinement.

## 🎯 Project Goals & Philosophy

*   **AI Exploration:** To understand the practical application of AI in software migration and development, including its strengths and weaknesses.
*   **Learning .NET 9:** To gain hands-on experience with the latest advancements in the .NET ecosystem.
*   **Performance:** To build efficient and responsive components, taking advantage of modern .NET performance features.
*   **Modularity:** To create reusable libraries for interacting with LNet and Gemstone IV.
*   **Proof of Concept:** This is not intended to be a production-ready, flawless application from the outset. It's an experiment. Expect rough edges and areas where AI suggestions led to suboptimal or incorrect code that required manual intervention.

## 📊 Current Status

*Will update as this project grows*

*   ✅ The LNet library is partially functional, capable of connecting, sending, and receiving basic messages.
*   🛠️ The Gemstone IV library has initial connection logic and stubs for parsing specific game events.
*   🏗️ The core service application is in its early stages of development, focusing on integrating the LNet library.
*   ⏳ The Discord library is planned but not yet started.

## 📚 Libraries

### 🌐 LNet Library

*   **Purpose:** Connect to LNet (Lich KTS Network), primarily used by Gemstone IV players.
*   **Features:**
    *   Connection management (SSL/TLS)
    *   Sending and receiving XML-based LNet messages
    *   Keepalive handling
    *   Message parsing
    *   Channel-based asynchronous message processing

### 💎 Gemstone IV Library (Future)

*   **Purpose:** Connect directly to Gemstone IV game instances and parse game-specific data.
*   **Features (Planned/In-Progress):**
    *   Telnet/SSL connection to game servers
    *   Parsing of ESP (Extra Sensory Perception) "thoughts"
    *   Parsing of character death announcements
    *   (Potentially other game event parsing)

### 💬 Discord Library (Future)

*   **Purpose:** Act as a bridge to relay messages and notifications to/from a Discord server.
*   **Features (Planned):**
    *   Connect to Discord API
    *   Send messages to specified channels
    *   Receive commands from Discord users

## ⚠️ Disclaimer

This project is a personal endeavor and a proof of concept. It is not officially affiliated with or endorsed by Simutronics Corp (the creators of Gemstone IV) or Lich (the LNet software). Use at your own discretion.
