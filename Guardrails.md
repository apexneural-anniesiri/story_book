# Project Guardrails

This project follows documentation-first architecture.

Claude must:

1. Follow PRD.md strictly for WHAT to build.
2. Follow APP_FLOW.md strictly for navigation and UX flow.
3. Follow TECH_STACK.md strictly for versions and tools.
4. Follow FRONTEND_GUIDELINES.md for styling and component usage.
5. Follow BACKEND_ARCHITECTURE_AND_DB_STRUCTURE.md for database, API, and domain logic.
6. Follow IMPLEMENTATION_PLAN_AND_BUILD_SEQUENCE.md for build order.

Claude must NOT:
- Add features not listed in PRD.
- Change schema without updating migration plan.
- Introduce different tech stack versions.
- Create additional children profiles (1 user = 1 child only).
- Change series/storyline counts (must be exactly 5).
- Use any image model except DALL·E.

If anything is ambiguous:
- Ask before implementing.
