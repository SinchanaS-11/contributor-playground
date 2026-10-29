# Database (DBMS)

**Lead:** Sinchana

The database layer owns persistent data storage, relational integrity, query performance, schema migrations, and database-level security where required. PostgreSQL stores user identity metadata, workspace and workflow metadata, run results, pull requests, reflections, and audit events.

The backend API remains responsible for authentication, authorization decisions, and workflow transitions. The database supports these responsibilities through appropriate relationships, constraints, indexes, and, if agreed, Row-Level Security (RLS).

## Stack

- PostgreSQL 15+
- SQL for schema definition, queries, constraints, and migrations
- Migration/ORM tooling: **To be confirmed with the backend lead**
- Local database: Docker Compose, if adopted by the team
- Database client: pgAdmin or DBeaver, optional
- Testing: SQL tests or Vitest with Testcontainers, depending on the agreed setup

## Modules

- **Users:** Store stable user IDs and GitHub identity metadata.
- **Sessions:** Store session metadata only if PostgreSQL is selected as the session store.
- **Issues:** Store seeded issue metadata, status, and assignment relationships.
- **Workspaces:** Store workspace ownership, related issue, and repository/snapshot references.
- **Commits:** Store commit metadata and snapshot identifiers. Git remains the source of truth for repository history.
- **Runs:** Store job status, command/snapshot references, timestamps, and execution results.
- **Hidden tests:** Store protected test metadata and version references separately from learner-accessible data.
- **Pull requests:** Store workspace association, submission details, and workflow status.
- **Reviews:** Store review decisions, comments or feedback, and related PR references as required.
- **Reflections:** Store learner self-assessments associated with pull requests. Reflections do not contribute to readiness scores.
- **Events:** Store append-only audit events for commits, runs, PRs, reviews, and relevant state changes.
- **Migrations and seeds:** Version schema changes and maintain reproducible initial issue data.
- **Maintenance:** Document backups, restore procedures, retention policies, and cleanup requirements.

## Data Relationships

The following relationships are a preliminary design and require confirmation:

- One user may own multiple workspaces.
- Each workspace relates to a seeded issue.
- A workspace may have multiple commits and runs.
- A workspace may have multiple pull requests, subject to workflow rules.
- A pull request may receive multiple reviews.
- A pull request may have one or more reflections, depending on the agreed requirements.
- Issues may have multiple versions of hidden tests.
- Users and workspaces may be associated with audit events.

The final schema must reflect the agreed issue-assignment rules, workspace lifecycle, and PR workflow.

## Integrity and Constraints

- Primary keys identify records consistently.
- Foreign keys enforce valid relationships between entities.
- Unique constraints prevent duplicate values where required.
- `NOT NULL` constraints enforce mandatory fields.
- `CHECK` constraints may enforce valid status values.
- Timestamps support workflow history and maintenance.
- Delete/update behavior must be explicitly chosen for dependent records.
- Audit events must be append-only through the application's permitted database operations.

## Authorization and Security

- Every workspace, run, and PR request must be authorized by the backend against the current session user.
- Database relationships must support ownership checks.
- Foreign keys alone do not enforce requester authorization.
- PostgreSQL RLS may be considered if the team agrees to use it.
- Hidden test source must not be exposed through learner-facing API responses.
- Session tokens must not be stored as plaintext if session data is persisted; the precise storage design must be agreed with the backend lead.
- Database credentials must follow least-privilege access.
- Logs and stored execution results must avoid exposing secrets.
- Readiness and pass status must be set by the backend, not trusted from client-submitted data.

## Indexes and Queries

Potential indexing targets include:

- Unique GitHub identity identifiers.
- Workspace ownership and issue references.
- Runs by workspace and status.
- Pull requests by workspace and status.
- Reviews by pull request.
- Events by timestamp, actor, and related resource.
- Hidden-test versions by issue and version.

Indexes will be selected based on actual API queries and query plans rather than added indiscriminately.

## Migrations, Testing, and Recovery

- Keep schema changes in version-controlled migrations.
- Make local database setup reproducible.
- Seed the initial issue data consistently.
- Test constraints, foreign keys, uniqueness, and workflow transitions.
- Test authorization-related data access paths with the backend.
- Use `pg_dump` for backups if PostgreSQL backups are managed directly.
- Verify that backups can be restored.
- Document retention and cleanup rules for old run records and temporary data.

## Open Questions

- Which ORM or query library and migration tool will the backend use?
- Will sessions be stored in PostgreSQL, Redis, or another session store?
- Can an issue be assigned to multiple learners?
- Can one user have multiple workspaces for the same issue?
- What repository and snapshot metadata must be stored in the database?
- Which run fields and execution results are required by the backend API?
- Can a pull request receive multiple reviews, and how many reflections can it have?
- What is the agreed audit-event format?
- Is PostgreSQL RLS required for the MVP, or will backend authorization checks be used without RLS?
- What are the retention rules for old runs, events, and other records?
- Which database queries must support readiness and progress calculations?

## Deliverables

- ER diagram and relationship documentation.
- PostgreSQL schema and version-controlled migrations.
- Seed data for development.
- Indexes and query improvements based on measured needs.
- Integrity and database integration tests.
- Authorization-related data-access test cases.
- Backup/restore and cleanup documentation.

The schema will be finalized in coordination with the backend lead and updated as API contracts and workflow requirements become clearer.
