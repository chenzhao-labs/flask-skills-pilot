# 01 - Restore per-route Automatic OPTIONS override

**What to build:** When Automatic OPTIONS is disabled globally, a function or
class-based view that explicitly enables Automatic OPTIONS must accept an
OPTIONS request and receive Flask's generated response, without changing
existing explicit opt-out or default route behavior.

**Blocked by:** None - can start immediately.

Status: completed

- [x] A function view that explicitly enables Automatic OPTIONS works when the global default is disabled.
- [x] A class-based view receives the same per-route override behavior.
- [x] The generated response and its advertised methods are verified through public HTTP behavior.
- [x] Existing explicit opt-out, explicitly declared OPTIONS, and global-default behavior remain covered.
