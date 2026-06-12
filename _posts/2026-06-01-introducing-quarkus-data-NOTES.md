# Blog Post Revision Notes — from Quarkus Insight (2026-06-08)

Source: Stephane Epardaud's Quarkus Insight on Quarkus Data Hibernate (https://www.youtube.com/watch?v=E_5rSjAD-k0), with Holly Cummins. These notes inform the next revision of the blog post and track open items.

---

## 1. Discoverability on quarkus.io

The rename from "Panache" to "Quarkus Data Hibernate" was motivated partly by discoverability. In the Insight, Stephane acknowledged that while existing Panache users could find the extension easily, newcomers who landed on quarkus.io and searched for "hibernate" or "data" — the terms they'd naturally reach for — got no results pointing to Panache. The name was opaque to anyone outside the community.

The blog post already demonstrates this well with the CLI search screenshots (lines 23–39: `quarkus extension add jpa` returning 16 results, `database` returning even more). That's the strongest part of the intro — it *shows* the problem rather than just describing it. Stephane's Insight validated this exact framing.

But the discoverability problem extends beyond the CLI to quarkus.io itself — search, navigation, landing pages. The rename helps in the CLI, but a user on the website still needs to find their way. The blog post doesn't mention website-level discoverability at all.

**Status**: Not started. Needs a plan for quarkus.io improvements (search indexing, guide navigation, landing page placement) so that "hibernate", "data", "ORM" queries all surface Quarkus Data Hibernate.

## 2. The "yet another extension" transition cost

Stephane's opening slide listed four existing extensions (ORM, ORM+Panache, Reactive, Reactive+Panache) that are largely incompatible. Quarkus Data is supposed to unify them, but right now it's a *fifth* entry in the list. Until the older ones are deprecated, the fragmentation is worse before it gets better.

The blog post opens with exactly this problem (lines 18–39: "Should a user that needs database access use the Hibernate extension, the Hibernate Reactive extension, Panache, or something else?") but the conclusion doesn't close the loop. The "What about Panache 1?" section (lines 549–555) says Panache 1 "is not going away" and Quarkus Data is "where all new development is happening," but this is too gentle. A newcomer who read the intro expecting a clear answer arrives at the end and still doesn't get a strong recommendation.

**Action**: The conclusion should directly answer the question the intro raises. Something like: "For new projects, start with Quarkus Data Hibernate. It replaces the need to choose between the four existing extensions." The intro sets up the punch; the conclusion needs to land it.

## 3. ORM + Reactive compatibility history

Stephane presented ORM/Reactive incompatibility as a hard unsolved problem. Holly corrected him on-stream: Luca had already implemented ORM/Reactive compatibility at the Quarkus infrastructure level; the Panache layer simply never picked it up.

For the blog post, the practical point is: most users won't mix ORM and Reactive in the same codebase. If they need Reactive, they're already committed to it. The value of supporting both in one extension is letting ORM users *experiment* with Reactive without a new module — a single gateway, not a mixing bowl.

The blog post's Reactive section (lines 429–488) handles this reasonably well — it introduces Reactive as something "you don't need to worry about upfront" and says "switch to reactive when you need it." But it doesn't mention that the reactive dependency is opt-in (`quarkus-hibernate-reactive` must be added explicitly). Adding one sentence about this would clarify that including reactive support doesn't bloat the default setup.

Also, line 539 claims "The same repository can also switch between managed and stateless sessions, or between blocking and reactive, at runtime" — this needs verification. The Insight couldn't find a real use case for runtime mixing, and Luca flagged this claim as unconfirmed.

## 4. `@Query` HQL fragments

The blog post already shows this at line 221 with the `cheaperThan` example and the inline comment "No need for 'select ... from Book' as Jakarta Data infers it from the return type." This is clear and effective.

This fragment approach — writing partial HQL clauses that restrict a generated query rather than writing the full query — originated in Panache 1 and was carried forward. It's different from Spring Data's convention of deriving queries from method names: both reduce boilerplate, but HQL fragments give more explicit control while still being checked at compile time.

The blog post mentions Panache's pioneering role at line 55 ("With Jakarta Data now standardizing what Panache pioneered, it was natural for the Quarkus team to build on it") but doesn't specifically say that the fragment approach influenced the spec. Worth adding a brief note positioning Quarkus not just as a consumer of Jakarta Data but as a contributor to its design.

## 5. IDE completion for `@Query` — known gap

Holly asked during the Insight whether users get IDE completion when writing `@Query` expressions (similar to Spring Data). The answer is no, not yet. There's a plan to discuss this with the LSP team, but any solution must work across both VS Code (via LSP) and JetBrains (which has its own plugin model).

The blog post emphasizes compile-time checking (lines 226–235: the `findByIsbn(String i)` error example, and "Other frameworks traditionally validate queries at startup... or worse, in production"). This is already a strong selling point. Compile-time validation covers the most critical case — correctness — even without IDE completion. The two are complementary: validation catches errors, completion prevents them.

**Status**: Needs investigation. No clear implementation approach. Don't promise it in the blog post, but the compile-time story is strong enough to stand on its own.

## 6. Blog post structure: lead with Active Record — Alan Kay framing

Feedback from Yoann, Stephane, and this Insight converge: the tutorial should lead with the Active Record pattern as the simplest entry point, then introduce repositories as the pattern you grow into.

The current blog post structure is:
1. Raw SQL (line 117) — simplest technically, no entities needed
2. Repositories with entities (line 183) — the workhorse pattern
3. Managed session (line 286) — stateful persistence context
4. Hibernate Reactive (line 429) — non-blocking variant
5. Active Record (line 489) — simplest for the user, but buried last

The progression is logical as a technology tour (build up from primitives), but wrong as a user guide (where do I start?). A newcomer has to read 500 lines before reaching the simplest option. The Active Record section itself is thin (~45 lines) compared to repositories (~100) and managed session (~100), which further undermines its position as *the* recommended entry point.

Additionally, the Active Record section still uses `PanacheEntity`/`PanacheRepository` names (line 497) with a NOTE at line 542 saying "this will be renamed." In a blog post titled "Introducing Quarkus Data," seeing the old Panache names as the first thing in the recommended starting point is jarring.

**Framing**: Alan Kay's principle — "Simple things should be simple, complex things should be possible."
- **Active Record** = simple things simple (extend the entity, get CRUD, minimal boilerplate)
- **Repository** = complex things possible (testability via `@InjectMock`, separation of concerns, flexible session strategies)

This connects to the expression problem (Philip Wadler): single inheritance makes adding operations easy (methods on the entity) but makes changing underlying behavior hard (locked into the hierarchy). Active Record is a type-system cage — comfortable inside, constraining when you outgrow it. The repository pattern is the escape hatch. Present Active Record honestly: great for simple CRUD, and you *can* leave when you need to.

**TODO**: Restructure the blog post to lead with Active Record, then repositories, then raw SQL/managed session as advanced topics. Or at minimum, add a "Where to start" section early that clearly says: start with Active Record, graduate to repositories. Expand the Active Record section to be a complete example (entity → endpoint → running app), not just code snippets.

## 7. Stateful vs stateless: tell a story, not a matrix

The Insight demo showed all combinations (Managed/Record x Stateful/Stateless) as equally valid, which confused viewers. A live chat user called mixing them "cursed." Stephane and Holly tried on-stream to find a legitimate mixing use case and couldn't.

The demo exposed a dangerous gotcha: switching from stateful (`ManagedEntity`) to stateless (`RecordEntity`) silently breaks code that relied on implicit dirty-checking and auto-flush. Entity modifications just don't persist, with no compile-time error. Holly called it "a subtle bug."

The blog post currently follows a good progression: stateless repositories first (line 281: "Jakarta Data repositories use Hibernate's StatelessSession under the covers... Every operation is immediate, so you don't need to think about entity lifecycle states"), then managed session as the next step (line 286). This is the right order.

But the transition between the two sections has no warning. The managed session section (lines 316–327) shows the `setAuthor` example where `book.author = session.find(...)` is enough — Hibernate detects the change and flushes automatically. This is exactly the pattern that would silently break if someone later switched to stateless. A short admonition box here would help:

> If you later switch this code to use a `StatelessSession`, the assignment alone won't persist — you must call `save()` explicitly. This is by design: stateless sessions don't track entity state.

The blog post also has TODO comments at lines 387–389 expressing uncertainty about whether to include the managed session comparison. The answer from the Insight is clear: include it, but tell a story:
- **New to Hibernate?** Start with stateless. Everything is explicit, no surprises.
- **Already know Hibernate?** Managed session gives you the convenience you're used to.

Don't present them as a matrix of four combinations. Present them as a progression.

## 8. Holly's static check idea

Holly suggested build-time static analysis: if an entity is modified in a stateless session context but never passed to `.save()`, flag it at compile time. Would directly prevent the subtle bug from #7.

The blog post already requires the Hibernate annotation processor (lines 96–113). This processor could potentially be extended to detect the "modified but never saved" pattern in stateless contexts.

**Status**: Good idea, unclear how to implement. Worth opening an issue to explore.

## 9. Join + pagination — open question

A viewer (sarabwt) asked: "Is the page size guaranteed to return N *entities* when you join one-to-many?" This is a well-known problem: SQL `LIMIT` applies to *rows*, and a one-to-many join multiplies rows per entity. `LIMIT 10` might return 10 rows but only 3 distinct entities.

Stephane didn't know the answer. The blog post doesn't currently have a paging section. Stephane's earlier blog post (`main/quarkus-website/_posts/2026-05-28-panache-next-renamed-to-quarkus-data.asciidoc`, lines 59–91) has a detailed paging section showing offset, cursor, and limit APIs, but this blog post intentionally covers different ground.

If/when paging is added to this blog post, the join edge case must be addressed. Hibernate ORM has mechanisms for this (fetch profiles, `DISTINCT` handling, window functions) — the blog post should explain what Quarkus Data does under the hood.

**Status**: Needs investigation and documentation.

## 10. Remaining TODOs in the blog post

The blog post has several unresolved TODO comments that need attention before publication:

- **Line 94**: `// TODO Add the jdbc driver as well or H2 will be used as default?` — Decision needed on whether to show JDBC driver in the getting-started snippet.
- **Line 98**: `// TODO LUCA Add the link to the issue` — Missing link to the annotation processor auto-config issue (likely https://github.com/quarkusio/quarkus/pull/53901).
- **Line 175**: `// TODO Luca check batching (Stateless disables batching)` — Needs verification of StatelessSession batching behavior.
- **Line 330**: `// TODO this needs verification as Jakarta Data supports also Stateful` — Verify whether Jakarta Data repositories can be backed by a stateful session, not just StatelessSession.
- **Lines 387–389**: `TODO => I'm not sure whether to include this` / `TODO => Entity Manager has other advantages...` — Decision on whether to include the managed session comparison. The Insight feedback says: yes include it, but as a progression story, not a neutral comparison (see #7).
- **Line 439**: `// TODO This is probably a bug and should be addressed elsewhere` — The requirement to declare a `Mutiny.StatelessSession` accessor for reactive repositories. Verify if this is a known bug or a design decision.

---

## Live chat exchanges (Luca's replies)

- **Native SQL support**: Confirmed Quarkus Data supports native SQL queries (`@NativeQuery` / `@SQL`), so complex queries like subqueries and EXISTS are fully supported. The blog post covers this well in the "Simple SQL query" section (lines 117–180).
- **Rename timing**: Confirmed "Panache Next" is now "Quarkus Data Hibernate" on main; artifact rename appears starting from the next Quarkus release. The blog post should mention this explicitly in the "Try it today" section (line 559).
- **Index usage**: Clarified that index usage is a database-level concern. Neither Hibernate nor Quarkus manages indexes at runtime. Users should enable query logging and check execution plans. Not directly relevant to the blog post, but could be a FAQ item.

## Reference

- Video: https://www.youtube.com/watch?v=E_5rSjAD-k0
- Presenters: Stephane Epardaud, Holly Cummins
- Blog post: `2026-06-01-introducing-quarkus-data.adoc`
- Earlier blog post (Stephane's): `2026-05-28-panache-next-renamed-to-quarkus-data.asciidoc` (in `main/`)
- Naming discussion: https://github.com/quarkusio/quarkus/issues/53145
- Annotation processor auto-config: https://github.com/quarkusio/quarkus/pull/53901
