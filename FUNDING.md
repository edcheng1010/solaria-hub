# Funding

Solaria is an independent open-source project created and maintained by Edward Cheng. There is no corporate backer — development is sustained entirely by community support.

---

## Why Funding Matters

Solaria's hybrid architecture means different types of work have different costs:

- **TYPE 1 firmware** (open-firmware hardware like ESP32, micro:bit™, CyberBrick, mBot2, StackChan, Raspberry Pi) — requires writing SSP firmware that runs directly on the device. Once built, ALL client platforms get support for free.
- **TYPE 2 protocol libraries** (closed-firmware hardware like LEGO® SPIKE™ Prime, Powered Up, Sony® toio™, UBTECH® uGot) — requires reverse-engineering the proprietary protocol and building a translation library. Each client platform then needs a thin wrapper to integrate it.
- **Universal clients** (App Inventor, Python, Web, MicroBlocks, Scratch™, MakeCode®) — requires building the SSP client logic for each programming environment. Once built, all TYPE 1 hardware is automatically supported.

**Common costs across all work:**

- **Hardware** — the physical device (and often accessories/cables/adapters) for development and testing
- **AI-assisted development (vibe coding)** — Claude Code, Manus, and other AI tools are core to the development workflow. Subscriptions and API usage are a real, significant recurring cost.
- **Development time** — protocol research, firmware/library development, client integration, and testing. Each integration represents **100–300 hours** of focused development work.
- **Documentation** — tutorials, guides, API references, sample projects, and translations
- **Infrastructure** — CI/CD, hosting, and tooling

---

## How to Support

### GitHub Sponsors (primary)

The most direct way to support Solaria. Monthly or one-time contributions:

→ [Sponsor Edward Cheng on GitHub](https://github.com/sponsors/edcheng1010)

### Ko-fi (one-time coffee donations)

Quick one-time support if you find the project useful:

→ [Buy me a coffee on Ko-fi](https://ko-fi.com/edcheng1010)

### Sponsor a Specific Integration

Want a particular hardware platform or client supported sooner? You can sponsor it directly to move it up the queue. See below.

---

## Sponsor a Bridge / Client

Want to see a specific integration sooner? You can **sponsor it** to move it up the development queue.

**How it works:**

1. Check the [ROADMAP.md](ROADMAP.md) for the full list of planned integrations with cost estimates.
2. Contact via GitHub Discussions or Sponsors with the integration you'd like prioritised.
3. Sponsorship covers hardware purchase + AI tooling + development time.
4. Progress is tracked publicly in the relevant repo.
5. You're credited as the sponsor in the repo README.

### TYPE 1: Firmware (open-firmware hardware)

You're sponsoring SSP firmware for a device. Once complete, every existing client automatically supports it.

| Complexity | Example | Hardware Cost | Dev Time | AI Tooling | Total Estimate |
|-----------|---------|:---:|:---:|:---:|:---:|
| Low | BBC micro:bit™ (simple BLE, well-documented) | ~$50–100 | ~100–150 hrs | ~$300–500 | **$1,000–$1,500** |
| Medium | CyberBrick / mBot2 (ESP32, open library) | ~$100–200 | ~100–200 hrs | ~$400–700 | **$1,000–$2,000** |
| High | StackChan / Raspberry Pi (complex peripherals) | ~$150–400 | ~150–250 hrs | ~$500–1,000 | **$1,500–$2,500** |

### TYPE 2: Protocol Library (closed-firmware hardware)

You're sponsoring reverse-engineering of a proprietary protocol. The library is built once, but each client needs a wrapper (~$500–$1,200 per client).

| Complexity | Example | Hardware Cost | Dev Time | AI Tooling | Total Estimate (lib only) |
|-----------|---------|:---:|:---:|:---:|:---:|
| Medium | LEGO Powered Up (documented LWP3 protocol) | ~$200–400 | ~150–250 hrs | ~$500–1,000 | **$2,000–$3,000** |
| Medium | Sony® toio™ (published BLE spec) | ~$100–200 | ~100–200 hrs | ~$400–800 | **$1,500–$2,500** |
| High | UBTECH® uGot (undocumented, reverse-eng. needed) | ~$300–600 | ~200–300 hrs | ~$800–1,200 | **$2,500–$4,000** |

**Plus per-client wrappers:** $500–$1,200 each (App Inventor, Python, Web, MicroBlocks, Scratch™, MakeCode®)

### Universal Client

You're sponsoring a new programming environment. Once built, all TYPE 1 hardware is automatically supported.

| Client | Dev Time | AI Tooling | Total Estimate |
|--------|:---:|:---:|:---:|
| Python SDK | ~100–150 hrs | ~$400–700 | **$1,200–$2,000** |
| Web (JavaScript) | ~100–150 hrs | ~$400–700 | **$1,200–$2,000** |
| MicroBlocks | ~80–120 hrs | ~$300–500 | **$1,000–$1,500** |
| Scratch™ | ~120–180 hrs | ~$500–800 | **$1,500–$2,000** |
| MakeCode® | ~100–150 hrs | ~$400–700 | **$1,200–$1,800** |

*Based on actual costs from the SPIKE Prime bridge (Phase 1): $500+ in AI tooling, $400+ in hardware, 100+ hours of development — and that was a single-device MVP with one client.*

---

## Funding Per Repository

Each Solaria repository can receive funding independently:

| Repository | Type | What funding supports |
|-----------|------|----------------------|
| **solaria-hub** (this repo) | Hub | SSP spec development, ecosystem coordination, documentation |
| **appinventor-lego-spike-prime-extension** | TYPE 2 wrapper | SPIKE Prime bridge maintenance, new features, bug fixes |
| *Future `solaria-fw-*` repos* | TYPE 1 | Hardware-specific SSP firmware |
| *Future `solaria-lib-*` repos* | TYPE 2 | Protocol translation libraries |
| *Future `solaria-client-*` repos* | Client | Programming environment integrations |

All repos have a Sponsor button linking to the same GitHub Sponsors profile.

---

## Transparency

- All sponsorship funds are used exclusively for Solaria development
- Hardware purchases and development milestones are reported publicly
- Sponsors are credited in the relevant repo README
- Monthly development updates posted in GitHub Discussions

---

## Future Sustainability: AI Layer SaaS Model

In addition to community sponsorship, Solaria plans a future SaaS model for the AI/Agent layer (Phase 3) that will complement the current funding approach:

- **Free tier (BYOK):** Users bring their own LLM API key. Full AI features, zero cost to Solaria.
- **Hosted paid tier (for schools):** Solaria-managed inference with curated prompts, school dashboard, usage analytics, and curriculum integration. One invoice, no API key friction for teachers.

Sponsorship funds the open-source build (protocol, firmware, clients). The hosted tier is a future revenue stream that sustains ongoing maintenance and expansion without gating the core platform.

---

## For Organizations

If your school, makerspace, or company would like to:
- Sponsor multiple integrations
- Become a sustaining partner
- Commission a custom integration for your hardware
- Bundle Solaria support into a curriculum package

Please open a Discussion thread or reach out directly via GitHub Sponsors.
