# TaskMaster – Directus-Powered Multi-User Task Manager

A production-ready, containerized headless backend built with **Directus 11.15.1** + **PostgreSQL** + **Podman** (rootless). Demonstrates real-world relational schema design, role-based access control, file attachments, automated notifications via Flows, and full schema export for reproducibility.

Perfect portfolio piece for backend / full-stack / DevOps learners.

## Features

### Users & Roles
- **Administrator** — full access via v11 Policies system
- **User** — restricted: only see/edit/delete own tasks (`$CURRENT_USER` ownership filter), auto-assigned as owner on task create

### Collections & Schema
- `categories` — simple lookup table (id, name)
- `tasks` — core entity: title, description, due_date, status (todo/in_progress/done), owner (M2O → directus_users), category (M2O → categories), attachments (M2M → directus_files), tags (M2M → tags), comments (O2M → comments)
- `tags` — reusable labels (M2M with tasks via tasks_tags junction)
- `comments` — per-task comments (O2M from tasks, M2O to directus_users, auto-timestamped)
- `tasks_files` — auto-generated junction for file attachments
- `tasks_tags` — auto-generated junction for tags

### Ownership & Security
- `$CURRENT_USER` filter on read/update/delete for User role
- Preset auto-sets `user` field on create
- v11 Policies system (not the v10 role matrix)

### File Attachments
- Multiple files per task (images, PDFs, docs)
- Stored on disk (`./uploads`) with Directus thumbnail/preview support

### Tags (Many-to-Many)
- Reusable tags across tasks
- Full nested API query support: `?fields=title,tags.tags_id.name`

### Comments (One-to-Many)
- Per-task comment thread with author and timestamp
- Nested query: `?fields=title,comments.body,comments.date_created`

### Flows & Notifications (Directus Automation)
| Flow | Trigger | Behaviour |
|---|---|---|
| **Task Assigned** | Event Hook — `items.update` on tasks, Filter (Blocking) | Creates in-app notification when a user is assigned to a task |
| **Overdue Tasks** | Schedule — `0 8 * * *` (daily 8am) | Finds all overdue non-done tasks with assigned users → batch creates notifications |

### API Filtering
Full filter support with curl (`-g` flag required for bracket escaping):
```bash
# By status
curl -g '...?filter[status][_eq]=todo'

# By tag
curl -g '...?filter[tags][tags_id][name][_eq]=urgent'

# Overdue (dynamic date)
curl -g '...?filter[due_date][_lt]=$NOW'

# Combined AND
curl -g '...?filter[_and][0][status][_eq]=todo&filter[_and][1][tags][tags_id][name][_eq]=urgent'

# Sort + paginate
curl -g '...?sort=due_date&limit=5&page=1'
```

## Tech Stack

- **Directus 11.15.1** — headless CMS + REST API + Flows automation
- **PostgreSQL 15** — relational database
- **Podman + podman-compose** — Docker-compatible, rootless containers

## Quick Start

1. **Clone & enter directory**
   ```bash
   git clone https://github.com/parsar2025/taskmaster.git
   cd taskmaster
   ```

2. **Copy and configure environment**
   ```bash
   cp docker-compose.yml docker-compose.local.yml
   # Edit docker-compose.local.yml — fill in KEY, SECRET, passwords, admin credentials
   ```

3. **Start services**
   ```bash
   podman-compose up -d
   ```

4. **Access Directus**
   - URL: `http://localhost:8055`
   - Admin credentials: as set in your compose env

5. **Apply schema**
   ```bash
   curl -X POST http://localhost:8055/schema/apply \
     -H "Authorization: Bearer YOUR_ADMIN_TOKEN" \
     -H "Content-Type: application/yaml" \
     --data-binary @schema-full-v11-YYYYMMDD-HHMMSS.yaml
   ```

6. **Test the API**
   ```bash
   # Get all tasks with tags and comments
   curl -g 'http://localhost:8055/items/tasks?fields=title,status,tags.tags_id.name,comments.body' \
     -H "Authorization: Bearer YOUR_TOKEN"
   ```


## Project Structure

```
taskmaster/
├── docker-compose.yml                     # Sanitized service definition (fill in secrets)
├── schema-full-v11-YYYYMMDD-HHMMSS.yaml  # Full schema snapshot (collections, fields, relations)
├── permissions.json                       # Permission rules per role
├── roles.json                             # Role definitions
├── policies.json                          # v11 Policies
├── flows.json                             # Automation flows (Task Assigned + Overdue Tasks)
├── .gitignore
└── README.md
```


## Learning Highlights

- Normalized relational design (M2O, M2M, O2M)
- Dynamic ownership via `$CURRENT_USER` + presets
- Rootless Podman + SELinux volume fixes (`:Z,U`)
- Directus v11 Flows: event hooks, scheduled triggers, condition routing, batch notifications
- Full schema + permissions export for reproducibility and CI
- Multi-user testing (admin vs restricted user)


## License

MIT – feel free to fork, learn, adapt.

Made with ❤️ for learning Directus + databases + containers.

Questions? Open an issue or reach out: parsar2025@gmail.com