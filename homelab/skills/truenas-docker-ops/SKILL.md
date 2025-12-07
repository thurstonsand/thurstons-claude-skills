---
name: truenas-docker-ops
description: Playbook for anything on the TrueNAS host — consult before touching services, data, or containers. Covers SSH entry, container interaction, and data/layout notes so you can operate safely on TrueNAS.
---

# TrueNAS Docker Operations

## Essentials

- Connect: `ssh truenas` (auth pre-configured)
- Paths: compose `/mnt/performance/home/admin/Develop/nixonomicon/nas/stacks/<container>/compose.yaml`; config `/mnt/performance/docker/<container>/`; large data `/mnt/capacity/watch/<container>/`
- List containers: `ssh truenas docker ps` | filter `... | grep <container>`
- Inspect compose: `ssh truenas cat /mnt/performance/home/admin/Develop/nixonomicon/nas/stacks/<container>/compose.yaml`
- Logs: `ssh truenas docker logs <container>`
- Exec simple command: `ssh truenas docker exec -i <container> <command>`
- Inspect details: `ssh truenas docker inspect <container>`

## Anypod Data Layout

- Host data root: `/mnt/capacity/watch/anypod/data` (binds to `/data` in container)
  - Media downloads live in `media/<feed_id>/` (one directory per feed)
  - Transcripts stored in `transcripts/<feed_id>/`
  - DB at `/data/db/anypod.db`
- DB tables (anypod.db):
  - `feed` (PK `id`; feed metadata plus `source_type`, `source_url`, `title/subtitle/description`, `language`, `author`, `podcast_type`, `category`, `explicit`, `image_ext`, `transcript_*`, counters like `total_downloads`, `consecutive_failures`, timestamps `created_at/updated_at/last_successful_sync/last_failed_sync/last_rss_generation`)
  - `download` (PK `feed_id`, `id`; per-episode info: `source_url`, `title`, `published`, `ext`, `mime_type`, `filesize`, `duration`, `status`, timestamps `discovered_at/updated_at/downloaded_at`, optional `remote_thumbnail_url/thumbnail_ext`, `description`, `quality_info`, retry/error fields, `playlist_index`, `download_logs`, transcript fields `transcript_ext/transcript_lang/transcript_source`)
  - `appstate` (PK `id`; `last_yt_dlp_update`)
  - `alembic_version` (schema version)

## Helper Scripts

- Python (handles nested heredoc/quoting): `scripts/docker_exec_python.sh <container> '<python_code>'`

  - Example:

  ```bash
  scripts/docker_exec_python.sh anypod '
  import sqlite3, json
  conn = sqlite3.connect("/data/db/anypod.db")
  conn.row_factory = sqlite3.Row
  rows = conn.execute("SELECT * FROM feed").fetchall()
  print(json.dumps([dict(r) for r in rows], indent=2))
  conn.close()
  '
  ```

  - Tip: avoid dict indexing inside f-strings; pull to vars first to avoid quote escaping issues.

- SQLite: `scripts/docker_exec_sqlite.sh <container> <db_path> '<sql_query>'`
  - Example:
  ```bash
  scripts/docker_exec_sqlite.sh anypod /data/db/anypod.db '
  SELECT feed_id, status, COUNT(*) AS count
  FROM download
  GROUP BY feed_id, status;
  '
  ```

## When to Use Scripts vs Direct

- Use scripts for multi-line Python/SQL or anything with tricky quoting.
- Use direct `docker exec` for simple commands, listing, logs, or basic inspection.

## Quick Examples

- Is `anypod` running? `ssh truenas docker ps | grep anypod`
- View `anypod` compose: `ssh truenas cat /mnt/performance/home/admin/Develop/nixonomicon/nas/stacks/anypod/compose.yaml`
- Query `anypod` DB: `scripts/docker_exec_sqlite.sh anypod /data/db/anypod.db "SELECT * FROM feed;"`
