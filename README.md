# BaseJava Course Project - Resume Database

A training project from the BaseJava course. It is a Java 8 web application that stores and manages resumes with multiple storage backends (arrays, collections, file system, serialization formats, and PostgreSQL) and a simple JSP UI.

## What this project contains

- Core domain model for resumes, contacts, and sections.
- Storage implementations using different data structures and persistence strategies.
- JDBC-based SQL storage with transactional helpers.
- Web UI built with Servlets + JSP + JSTL.
- JUnit 4 tests for storage behavior.
- SQL scripts to initialize and populate the database.

## Key technologies

- Java 8
- Servlets, JSP, JSTL
- PostgreSQL + JDBC
- Gson (JSON), JAXB (XML)
- JUnit 4

## Project structure (high level)

- `src/model/` - Domain model (`Resume`, `ContactType`, `Section` types, `Organization`).
- `src/storage/` - Storage API and implementations.
- `src/storage/serializer/` - Serialization strategies (Object, DataStream, JSON, XML).
- `src/sql/` - JDBC helpers (`SqlHelper`, transactions, exception mapping).
- `src/web/` - `ResumeServlet` (CRUD controller).
- `src/until/` - Utilities (JSON adapter, XML parser, dates, HTML helpers).
- `web/` - JSP views and static assets.
- `test/` - JUnit tests for storage implementations.
- `config/` - DB config and SQL scripts.

## Domain model

- `Resume` has a `uuid`, `fullName`, `contacts` (`ContactType -> String`) and `sections` (`SectionType -> Section`).
- Sections are typed:
  - `TextSection` for objective/personal text.
  - `ListSection` for achievements/qualifications.
  - `OrganizationSection` for experience/education with positions and dates.

## Storage implementations

All storages implement `storage.Storage`:

- Array-based: `ArrayStorage`, `SortedArrayStorage` (fixed size array).
- Collection-based: `ListStorage`, `MapUuidStorage`, `MapResumeStorage`.
- File system: `FileStorage` and `PathStorage` using `StreamSerializer`:
  - `ObjectStreamSerializer` (Java serialization)
  - `DataStreamSerializer` (custom binary format)
  - `JsonStreamSerializer` (Gson)
  - `XmlStreamSerializer` (JAXB)
- Database: `SqlStorage` with JDBC and transactions. Sections are stored as JSON strings.

`storage.Config` reads `config/resumes.properties` and currently wires `SqlStorage` as the active storage.

## Database schema

See `config/init_db.sql`:

- `resume` (uuid, full_name)
- `contact` (resume_uuid, type, value)
- `section` (resume_uuid, type, content)

Optional sample data is in `config/populate.sql`.

## Web layer

- `web.ResumeServlet` handles `GET`/`POST` for list, view, add, edit, delete.
- JSPs are in `web/WEB-INF/jsp/`:
  - `list.jsp`, `view.jsp`, `edit.jsp`
- The servlet is mapped to `/resume` in `web/WEB-INF/web.xml`.

## Configuration

`config/resumes.properties`:

- `storage.dir` - directory for file/path storages (currently a Windows path).
- `db.url`, `db.user`, `db.password` - PostgreSQL connection settings.

Note: `storage.Config` uses `config\\resumes.properties` and a Windows-style path by default. On macOS/Linux, update `storage.dir` and consider changing the config path to `config/resumes.properties` if needed.

## How to run (IDE + Tomcat)

This project does not include Maven/Gradle. The typical workflow is via IntelliJ IDEA with manually added libraries.

1. Configure JDK 8.
2. Add libraries from the root JARs (`gson-2.8.2.jar`, `postgresql-42.2.1.jar`) and the `lib/` folder (servlet/JSP/JSTL APIs).
3. Create the database and schema using `config/init_db.sql`.
4. Update `config/resumes.properties` with your local paths and DB credentials.
5. Set up a Tomcat run configuration and deploy the `web/` folder.
6. Open `http://localhost:8080/resume`.

Example DB setup with `psql`:

```bash
createdb resumes
psql -d resumes -f /Users/platon/IdeaProjects/basejava/config/init_db.sql
psql -d resumes -f /Users/platon/IdeaProjects/basejava/config/populate.sql
```

## Tests

JUnit 4 tests live under `test/`. The main suite is `test/storage/AllStorageTest.java`. Run them from the IDE after adding JUnit 4 to the test classpath.

## Notes

- `src/Main*` classes are small exercises and demos for arrays, collections, dates, reflection, etc.
- There are two `Config` classes: `src/Config.java` (root) and `src/storage/Config.java`. The web app uses `src/storage/Config.java`.
