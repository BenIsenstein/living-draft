# 03: Supervise external artifact sessions

**What to build:** A user or agent can open, inspect, diagnose, and stop Living Draft sessions for artifacts in arbitrary projects without manually coordinating Studio and artifact servers.

**Blocked by:** 01: Open a static Living Draft artifact; 02: Open an interactive artifact as one island

**Status:** ready-for-agent

- [ ] Open starts or reuses healthy Studio and artifact processes with dynamic ports.
- [ ] Sessions are identified by canonical artifact path and support external projects.
- [ ] Stop and doctor report useful process, URL, log, and health information.
- [ ] Idle sessions shut down cleanly and every filesystem mutation is reported.
