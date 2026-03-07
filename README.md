# GG - SS Integration

Integration project for connecting SS (Smartscore) live scoring system with GG (Golf Genius) tournament management platform.

## Project Structure

```
/golf-genius/
├── docs/
│   ├── GG-INTEGRATION-SPEC.md       # English version (for GG sharing)
│   └── GG-INTEGRATION-SPEC-KO.md    # Korean version (internal reference)
├── src/                             # Implementation (TBD)
└── README.md
```

## Documentation

| Document | Language | Purpose |
|----------|----------|---------|
| [GG-INTEGRATION-SPEC.md](docs/GG-INTEGRATION-SPEC.md) | English | Share with GG team (Adrian) |
| [GG-INTEGRATION-SPEC-KO.md](docs/GG-INTEGRATION-SPEC-KO.md) | Korean | Internal team reference |

## Integration Scope

### P0 (Must Have)
- Event synchronization (GG → SS)
- Roster synchronization (GG → SS)
- Pairings synchronization (GG → SS)
- Live score submission (SS → GG)

### P1 (Should Have)
- Leaderboard display
- Roster change detection during round

### Optional
- Golf Course ERP integration (for pre-populated round info)

## Key Design Decisions

### SS Tablet Operation
- **No changes** to existing tablet workflow
- Players/Marshals enter name + phone number as usual
- All GG synchronization handled by SS Server
- Player mapping performed server-side

### Architecture Options
1. **Basic**: GG ↔ SS Server ↔ Tablets
2. **Extended**: GG ↔ SS Server ↔ Golf Course ERP ↔ Tablets

## Environment

| Environment | URL | API Key |
|-------------|-----|---------|
| GG Staging | https://www.ggstest.com/ | `BBC9Dhduwyu1PmGz7xSnQw` |
| GG Production | TBD | TBD |

## Next Steps

1. Share English spec with GG (Adrian via Slack)
2. Schedule technical call to discuss questions
3. Confirm synchronization approach
4. Begin implementation

## Tech Stack (Planned)

- **Language:** Kotlin
- **Framework:** Spring Boot
- **JDK:** 21
- **Build:** Gradle

## Contact

- **GG Technical:** Adrian (Slack)
- **SS Dev:** TBD
