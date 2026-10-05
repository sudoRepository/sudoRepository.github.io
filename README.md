# Horcrux Android Architecture

The Horcrux base separates presentation, pure domain rules, platform services,
and persistence. `horcrux-take-your-seats` uses this structure for its offline game.

## Module Structure

| Module | Responsibility |
| --- | --- |
| `app` | Single Activity, Jetpack Compose, Navigation 3, MVI state and events, repository composition, platform adapters and resource mapping |
| `core:model` | Pure Kotlin domain models, stable identifiers and deterministic gameplay policies |
| `core:storage` | Room entities, DAOs and transactions; DataStore settings; versioned local-save mapping and migration |
| `core:audio` | BGM and sound-effect playback with volume and lifecycle control |
| `core:network` | Optional base networking infrastructure; not connected to this game's runtime |
| `core:crypto` | Optional base cryptography infrastructure; not connected to this game's runtime |
| `build-logic` | Gradle convention plugins for shared Android configuration and dependencies |

The game app depends on `core:model`, `core:storage` and `core:audio`.
Domain rules do not depend on Android UI, Room or network DTOs. Compact and Large
screens consume the same state and event contracts while retaining separate layouts.

## Design Patterns

- **MVI and unidirectional data flow:** Compose emits typed events to the ViewModel.
  A single immutable UI state flows back to the screen; one-shot effects handle
  operations such as opening an external email draft.
- **Repository:** Interfaces separate state orchestration from storage implementations.
  Mappers translate persistence records into application/domain-facing models.
- **Dependency injection:** Hilt provides repositories, storage and platform services
  through constructor injection.
- **Adapter:** Sensor input, audio resource identifiers and Android APIs are isolated
  from pure gameplay policies.
- **Pure state transitions:** Domain policies calculate new state from explicit input,
  making gameplay behavior testable independently of rendering.
- **Transactional, idempotent settlement:** Room transactions persist rewards and
  progress together; durable receipts prevent repeated operations from paying twice.

Typical flow: `Compose -> Event -> ViewModel -> Repository -> Storage/Domain Policy`
and `Committed Data -> UI State -> Compose`.
