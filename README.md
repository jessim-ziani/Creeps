# Creeps - Asynchronous AI Swarm & Multi-Agent Engine

> **Showcase Repository:** This repository serves as an architectural showcase. To comply with EPITA's anti-plagiarism rules, the full raw source code is kept private, but core architectural concepts, engine design, and concurrent patterns are documented below.

## 📖 Project Context

**Creeps** is a real-time strategy (RTS) AI project focusing on interacting with a game server via a REST API. The challenge lies in managing multiple units (Citizens, Turrets, Bomber Bots) simultaneously in a persistent world, handling network latency and strict server-side cooldowns while pursuing complex game achievements.

### Key Technical Objectives:
* **Massive Concurrency:** Orchestrating independent unit agents without thread-locking using the `CompletableFuture` API.
* **Network Reliability:** Implementing a non-blocking polling mechanism to synchronize unit actions with server-side reports.
* **Autonomous Decision-Making:** Developing agents capable of scanning the map and prioritizing resource gathering (Wood, Rock, Food) using the `Cartographer`.

## 🏗️ Architectural Overview

The application is built around a non-blocking engine designed to handle network latency and swarm management seamlessly.

### 1. Independent Asynchronous Agents (Actor-like Model)
The engine treats each unit as an independent agent. Each unit receives its own execution loop via `CompletableFuture.runAsync`, allowing multiple citizens to evaluate their surroundings and execute commands in parallel.

### 2. Goal-Oriented Intelligence
Instead of a single monolithic loop, the AI dispatches units toward specific tasks. For instance, one agent can focus on gathering wood while another expands the territory by building roads to stay protected from the server's garbage collector (Hector).

## 🚀 Technical Highlights & Snippets

### Non-blocking Polling Logic
To respect server constraints, the networking layer manages HTTP requests and report tracking asynchronously. The `command` method handles the lifecycle of an action: sending the request and polling for the final report with a managed delay:

```java
// Logic simplified from Server.java
public HttpResponse<JsonNode> command(String opcode, String unitId, Parameter param) {
    // 1. Dispatch action to the server
    HttpResponse<JsonNode> response = Unirest.post(command_url).body(param).asJson();
    String reportId = response.getBody().getObject().getString("reportId");

    // 2. Poll until the action is completed
    while (true) {
        delay(200); // Wait for server ticks
        HttpResponse<JsonNode> reportResp = Unirest.get(report_url + reportId).asJson();
        if (!"noreport".equals(reportResp.getBody().getObject().getString("opcode"))) {
            return reportResp; // Action finished
        }
    }
}
```

### Swarm Orchestration
The main entry point spawns independent threads for each citizen. This allows the AI to scale its operations across the map:

```java
// Logic simplified from Program.java
CompletableFuture<Void> agent1 = CompletableFuture.runAsync(() -> {
    while (active) {
        serv.observer(j1, Tile.Wood); // Self-contained logic for Wood gathering
        serv.build("road", j1);       // Territory expansion
    }
});

CompletableFuture<Void> agent2 = CompletableFuture.runAsync(() -> {
    while (active) {
        serv.observer(j2, Tile.Rock); // Parallel Rock gathering
    }
});

// Group agents to keep the program alive
CompletableFuture.allOf(agent1, agent2).join();
```

## 🎯 Demonstration

* **Server Activity:** Real-time terminal logs showing concurrent units fetching reports and executing orders in parallel.
![Terminal Activity](img/terminal_demo.gif)

* **Game Visualizer:** Autonomous citizens gathering resources and building infrastructure to expand territory.
![Game Visualizer](img/web_viewer_demo.gif)

## 🛠️ Tech Stack

* **Language:** Java 21
* **Concurrency:** CompletableFuture API
* **Networking:** Unirest & Jackson for JSON REST interaction
* **Build System:** Maven

## 👤 Developer

**Jessim Ziani** - Engineering Student at EPITA.

<div align="center">
  <a href="https://github.com/jessim-ziani">
    <img src="https://github.com/jessim-ziani.png" width="60px" style="border-radius: 50%;" alt="Jessim Ziani"/><br />
    <sub><b>Jessim Ziani</b></sub>
  </a>
</div>
