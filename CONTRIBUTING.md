# Contributing to Solaria

Thank you for your interest in contributing. Solaria grows through community effort — whether you're a hardware maker, app developer, educator, or student, there's a way to help.

---

## Ways to Contribute

### 1. Propose a New Hardware Integration

Have a robotics platform you'd like to see supported?

1. Check the [ROADMAP.md](ROADMAP.md) to see if it's already listed.
2. If not, open a Discussion thread with:
   - Hardware name and link
   - Communication protocol (BLE, WiFi, Serial, etc.)
   - Whether the protocol is documented/open
   - Target audience (age range, skill level)
3. The community will vote on proposals during the next prioritisation cycle.

### 2. Build a Hardware Bridge

You're a firmware developer and want to add support for a specific robot:

1. Read the [SSP v0.6 specification](spec/SSP-v0.6.md).
2. Read the [Architecture guide](ARCHITECTURE.md#building-a-new-bridge).
3. Fork and create a new repo: `solaria-bridge-<hardware>`.
4. Implement the bridge following SSP conventions.
5. Test against the reference App Inventor client.
6. Open a PR to this hub repo to add your bridge to the README table.

### 3. Build a Client Extension

You want Solaria to work in a new programming environment:

1. Read the [SSP v0.6 specification](spec/SSP-v0.6.md).
2. Read the [Architecture guide](ARCHITECTURE.md#building-a-new-client).
3. Fork and create a new repo: `solaria-client-<platform>`.
4. Implement SSP message generation and sensor event handling.
5. Test against the reference SPIKE Prime bridge.
6. Open a PR to this hub repo to add your client to the README.

### 4. Improve Documentation

- Fix typos, clarify explanations, add examples
- Translate documentation into other languages
- Write tutorials or guides for specific use cases

### 5. Report Issues

- Found a bug in a bridge or client? Open an issue in the relevant repo.
- Found a problem with the SSP spec itself? Open an issue here.

---

## Pull Request Process

1. Fork the relevant repository.
2. Create a feature branch (`feature/my-contribution`).
3. Make your changes with clear commit messages.
4. Ensure your code follows existing style conventions.
5. Open a PR with a description of what you've done and why.
6. Respond to review feedback.

---

## Code of Conduct

All contributors are expected to follow the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). Be respectful, constructive, and welcoming.

---

## Questions?

Open a Discussion thread. We're happy to help you get started.
