# 🐝 BHive – Concurrent Beehive Simulation

BHive is a Java-based, multithreaded simulation of a beehive ecosystem.
It was loosely inspired by Conway’s Game of Life and focuses on how
different entities (bees, bacteria, and food) interact over time.

The project was developed during university mainly as a way to practice:
- Java concurrency
- Thread coordination
- Event-driven systems
- Simple simulation modeling


## 📌 Overview

The simulation represents a beehive where many entities act independently
but still depend on shared resources such as food and time.

Each living entity runs in its own thread and performs actions once per
simulated “day”. To keep the simulation logic separate from monitoring
and statistics, events are published through RabbitMQ instead of being
handled directly inside the simulation.


## 🧬 Living Entities

All entities inherit from the abstract `LivingThing` class, which provides:
- A unique ID
- Alive/dead state handling
- Runnable behavior
- Abstract lifecycle methods

### Bees
Bees are split into different roles, each with its own behavior:
- QueenBee – responsible for reproduction
- WorkerBee – collects food from the wild
- MaleBee – participates in mating

### Bacteria
Bacteria consume food stored in the hive:
- They reproduce after several consecutive days of eating
- They die after being starved for too long
- When they die, they turn into wild food


## 🌱 Hive Environment

`HiveEnvironment` represents the shared state of the simulation:
- Food stored in the hive
- Food available in the wild
- Progression of simulation days

All shared values are handled using atomic variables to keep the
environment thread-safe.


## 🕒 Simulation Flow

1. The simulation starts with an initial population:
   - 1 Queen Bee
   - Multiple Worker Bees
   - Multiple Bacteria
   - An initial amount of food

2. Each simulated day:
   - The amount of wild food is randomized
   - A “new day” signal is sent
   - All living entities perform their daily actions

3. During a day, entities may:
   - Eat food
   - Starve
   - Reproduce
   - Die

4. Lifecycle and food-related events are published via RabbitMQ


## Concurrency Model

- Each `LivingThing` runs in its own thread
- Threads are managed using an `ExecutorService`
- Thread safety is achieved using:
  - `AtomicInteger`
  - `AtomicBoolean`
  - Thread-safe collections

The simulation avoids explicit locking wherever possible and instead
relies on atomic operations.


## Event System (RabbitMQ)

Events are used to observe the simulation without tightly coupling
monitoring logic to the simulation itself.

### Published Events
- birth
- death
- food
- mating

### EventPublisher
Responsible for:
- Serializing entity state
- Publishing events to a RabbitMQ exchange

### EventReader
- Listens to simulation events
- Keeps track of:
  - Total living entities
  - Bees by type
  - Bacteria count
  - Food changes
  - Mating queue size


## HiveReader (CLI Monitor)

`HiveReader` is a simple command-line tool for inspecting the simulation
while it is running.

Available commands:
- p – print current counters
- r – reset all counters
- q – quit the reader

This makes it possible to observe the simulation in real time without
affecting its behavior.


## ▶️ How to Run

### Requirements
- Java 8 or higher
- RabbitMQ running locally

### Steps
1. Start RabbitMQ
2. Run `HiveSimulation` to start the simulation
3. (Optional) Run `HiveReader` in a separate process to monitor events


## Limitations

- RabbitMQ configuration is hardcoded
- No persistence between runs
- No graphical visualization
- Simulation parameters are fixed


## Possible Improvements

- Proper Maven or Gradle project structure
- Configurable simulation parameters
- Graphical or web-based visualization
- Smarter decision-making logic
- Containerized setup
