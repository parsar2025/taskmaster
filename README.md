# TaskMaster – Directus-Powered Multi-User Task Manager

A production-ready, containerized headless backend built with **Directus 11.15.1** + **PostgreSQL** + **Podman** (rootless). Demonstrates:

- Relational schema design (M2O, M2M)
- Role-based access control with ownership filtering
- File attachments (multiple per task)
- Auto-assignment of task owner on creation
- Real-world container debugging (permissions, SELinux, rootless Podman)
- Full schema + permissions export for reproducibility

Perfect portfolio piece for backend / full-stack / DevOps learners.

## Features

- **Users & Roles**  
  Admin (full access) + custom "User" role (only see/edit own tasks)

- **Collections**  
  - `categories` (lookup)  
  - `tasks` (title, description, due_date, status, owner, category, attachments)  
  - Auto-generated `tasks_files` junction for many-to-many files

- **Ownership & Security**  
  - `$CURRENT_USER` filter on read/update/delete  
  - Preset auto-sets `user` field on create  
  - Permissions on `directus_files` / folders / junction for uploads

- **File Attachments**  
  Multiple files per task (images, PDFs, docs)  
  Stored on disk (`./uploads`) with thumbnails/previews

- **Containerization**  
  Podman rootless + podman-compose  
  Persistent PostgreSQL bind mount (`./postgres-data`)  
  Uploads bind mount with `:Z,U` SELinux fix

## Tech Stack

- Directus 11.15.1 (headless CMS + API)  
- PostgreSQL 15  
- Podman + podman-compose (Docker-compatible, rootless)  
- Fedora host

## Quick Start (for other developers)

1. **Clone & enter directory**
   ```bash
   git clone https://github.com/parsar2025/taskmaster.git
   cd taskmaster
   ```

2. **Start services**
   ```bash
   podman-compose up -d
   ```

3. **Access Directus**
   - URL: http://localhost:8055
   - Email: `admin@example.com` (or from your .env/compose)
   - Password: from compose env or initial setup

4. **(Optional) Apply schema + permissions**
   ```bash
   # Schema only
   curl -X POST http://localhost:8055/schema/apply \
     -H "Authorization: Bearer YOUR_ADMIN_TOKEN" \
     -H "Content-Type: application/yaml" \
     --data-binary @schema-full-v11-YYYYMMDD-HHMMSS.yaml

   # Or manually import permissions JSON via API/UI
   ```

5. **Test users**
   - Create test user → assign "User" role
   - Log in → only see own tasks + attachments

## Project Structure

```
taskmaster/
├── docker-compose.yml          # Full service definition
├── .gitignore
├── schema-full-v11-*.yaml      # Exported schema (collections, fields, relations)
├── permissions-export.json     # User role CRUD rules + filters
├── roles-export.json           # Role definitions
├── README.md                   # This file
└── (ignored)                   postgres-data/   uploads/
```

## Learning Highlights

- Normalized relational design (users ↔ tasks ↔ categories ↔ files)
- Dynamic ownership via `$CURRENT_USER` + presets
- Rootless Podman + SELinux volume fixes (`:Z,U`)
- Full export of schema & permissions for CI/reproducibility
- Multi-user testing (admin vs restricted user)

## License

MIT – feel free to fork, learn, adapt.

Made with ❤️ for learning Directus + databases + containers.

Questions? Open an issue!
```

### Next Actions

1. Create/update `README.md` with the content above (edit YOUR_USERNAME, emails, etc.).
2. Commit everything:
   ```bash
   git add .
   git commit -m "Final phase 1-4 commit: full schema + permissions exports, professional README, clean structure"
   git tag -a v1.0-complete -m "v1.0 – Fully documented multi-user task manager with Directus v11"
   ```

3. Push to GitHub (if remote exists):
   ```bash
   git push origin main
   git push origin --tags
   ```