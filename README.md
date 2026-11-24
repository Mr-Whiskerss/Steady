<h1 align="center">🪨 Steady</h1>

<p align="center">
  <strong>A calm, self-hosted place for your to-do lists and projects.</strong>
</p>

<p align="center">
  <img alt="Python" src="https://img.shields.io/badge/python-3.11%2B-34558B">
  <img alt="Flask" src="https://img.shields.io/badge/flask-3.1-34558B">
  <img alt="SQLite" src="https://img.shields.io/badge/storage-SQLite-34558B">
  <img alt="Proxmox LXC" src="https://img.shields.io/badge/proxmox-LXC-D9A92C">
  <img alt="Docker" src="https://img.shields.io/badge/docker-ready-D9A92C">
  <img alt="Google Keep import" src="https://img.shields.io/badge/import-Google%20Keep-D9A92C">
</p>

Steady is for people with a lot of lists on the go who want to see where everything is up to
without feeling buried by it. Every project shows its progress as a row of blocks, one per task,
so you can see at a glance that the garage clear-out is one of four and the homelab is three of six.

Pick a few things for today, focus on one at a time, and break anything big into smaller steps.
Bring your existing Google Keep notes across in one go.

It runs on your own server, needs no account, and keeps everything in a single SQLite file.

## Features

**See where you're up to**
- Every project shows a progress bar made of one block per task
- A Progress page with tasks finished this week, a four-week chart, projects that are nearly
  done, and everything you've finished recently
- No streaks, no red overdue warnings, nothing that tells you off

**Decide what to do**
- **Today**: tap the sun on any task to bring it into today's list. Tasks stay there until you
  finish them or take them off, so nothing disappears overnight.
- **Ideas for today** suggests a few tasks if you're not sure where to start
- **Focus on one thing** shows a single task full screen, with its steps, a Done button and a
  "not now, next one" button
- A gentle note if you've picked more for one day than is realistic

**Keep things small**
- Break any task into steps, and see "2 of 5 steps" at a glance
- **Brain dump**: paste or type a messy list into the inbox, one thing per line. Indented lines
  become steps, and bullets and checkboxes are tidied away.
- Sort things into projects later, or straight away

**Organise without fuss**
- Projects can be Active, Someday, Finished or Archived
- Colours, labels and pinning
- Notes on projects and tasks, plus optional due dates
- Undo for anything you delete

**Comfortable to use**
- Set in Atkinson Hyperlegible, a typeface designed for readability
- Larger text option, light and dark mode, and respect for reduced-motion settings
- Works on phones, tablets and desktops

## Import from Google Keep

Google doesn't offer a way to connect to Keep directly, but Google Takeout gives you a copy of
all your notes, and Steady reads that file as-is.

1. Go to [takeout.google.com](https://takeout.google.com), press **Deselect all**, and tick only **Keep**.
2. Create the export and download the .zip file when Google emails you.
3. In Steady, open **Import and settings** and choose the .zip. There's no need to unzip it.
4. Choose which notes to bring in, and whether each one is Active, Someday, Finished or Archived.

How notes come across:

| In Google Keep | In Steady |
| --- | --- |
| A checklist | A project, with each item as a task. Ticked items are already done. |
| A text note with bullet points or checkboxes | A project, with each bullet as a task and indented bullets as steps |
| A plain text note | A project with the text in its notes |
| Colour, labels, pinned | Kept |
| Archived notes | Left unticked by default, and imported as Archived if you include them |
| Notes in the bin | Left out |
| Links attached to a note | Added to the project's notes |

Importing the same Takeout file again skips notes you've already brought in. Images and drawings
attached to notes aren't imported.

## Installation

Pick whichever suits your setup. All three store data in a single SQLite database.

### Option 1: Proxmox LXC container

Run this in your **Proxmox host's** shell. It creates an unprivileged Debian container, installs
Steady as a systemd service, and prints the address to open.

```bash
curl -fsSL https://github.com/Mrwhiskerss/steady/archive/refs/heads/main.tar.gz | tar xz
cd steady-main
bash proxmox/create-lxc.sh
```

The defaults are 1 core, 512 MB RAM, a 4 GB disk, DHCP on `vmbr0`, port 8081, and start on boot.
Change any of them by setting it in front of the command:

```bash
CTID=151 STORAGE=local-zfs IP=192.168.1.51/24 GATEWAY=192.168.1.1 bash proxmox/create-lxc.sh
```

| Setting | Default | What it does |
| --- | --- | --- |
| `CTID` | next free ID | Container ID |
| `CT_HOSTNAME` | `steady` | Container hostname |
| `STORAGE` | `local-lvm` | Storage for the container's disk |
| `TEMPLATE_STORAGE` | `local` | Storage that holds LXC templates |
| `DISK` | `4` | Disk size in GB |
| `MEMORY` | `512` | RAM in MB |
| `CORES` | `1` | CPU cores |
| `BRIDGE` | `vmbr0` | Network bridge |
| `IP` | `dhcp` | Or a static address such as `192.168.1.51/24` |
| `GATEWAY` | | Required with a static IP |
| `PORT` | `8081` | Port Steady listens on |
| `ROOT_PASSWORD` | | Sets a root password (otherwise use `pct enter <id>`) |

**Already have a Debian or Ubuntu container?** Copy this repository into it and run
`bash proxmox/install.sh` as root.

### Option 2: Docker

```bash
git clone https://github.com/Mrwhiskerss/steady.git
cd steady
docker compose up -d --build
```

Then open `http://<your-server>:8081`. Data is stored in `./data`.

### Option 3: Plain Python

Needs Python 3.11 or newer.

```bash
git clone https://github.com/Mrwhiskerss/steady.git
cd steady
python3 -m venv .venv && . .venv/bin/activate
pip install -r requirements.txt
python app.py
```

Then open `http://localhost:8081`.

## Configuration

| Variable | Default | Description |
| --- | --- | --- |
| `STEADY_DB` | `./data/steady.db` | Path to the SQLite database |
| `PORT` | `8081` | Port when running with `python app.py` |

## Updating

**Proxmox LXC:** download the new version on the host and run the updater with your container ID.
It backs up the database first and keeps all your projects and tasks.

```bash
curl -fsSL https://github.com/Mrwhiskerss/steady/archive/refs/heads/main.tar.gz | tar xz
cd steady-main
bash proxmox/update-lxc.sh <container-id>
```

**Docker:** `git pull && docker compose up -d --build`

## Backups

| Install | Database location |
| --- | --- |
| Proxmox LXC | `/var/lib/steady/steady.db` inside the container (included in Proxmox container backups) |
| Docker | `./data/steady.db` |
| Plain Python | `./data/steady.db`, or wherever `STEADY_DB` points |

You can also download a portable backup from **Import and settings → Download backup**, and
restore it on any Steady install from the same place. Deleted items can be undone for 30 days,
after which they're removed for good.

## Security

Steady has **no login**. Run it on your home network, or put it behind something that handles
access, such as a VPN (Tailscale, WireGuard) or a reverse proxy with authentication (Authelia,
Authentik, Caddy with basic auth). Don't expose it directly to the internet.

## Project structure

```
steady/
├── app.py               Flask app: API, brain dump parser, Google Keep import, progress stats
├── static/index.html    The whole web interface (vanilla JS, no build step)
├── proxmox/
│   ├── create-lxc.sh    Creates a container and installs Steady (run on the Proxmox host)
│   ├── update-lxc.sh    Updates an existing container (run on the Proxmox host)
│   └── install.sh       Installs or updates Steady inside any Debian/Ubuntu system
├── Dockerfile
├── docker-compose.yml
└── requirements.txt     Flask and Gunicorn
```

<details>
<summary><strong>API reference</strong></summary>

All endpoints take and return JSON.

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/api/state` | All projects and tasks |
| `POST` | `/api/projects` | Create a project |
| `PUT` | `/api/projects/<id>` | Update `title`, `notes`, `color`, `status`, `pinned` or `labels` |
| `DELETE` | `/api/projects/<id>` | Delete a project (can be restored for 30 days) |
| `POST` | `/api/projects/<id>/restore` | Undo a project delete |
| `POST` | `/api/tasks` | Create a task: `title`, plus `project_id` or `parent_id` for a step |
| `PUT` | `/api/tasks/<id>` | Update `title`, `notes`, `done`, `today`, `due` or `project_id` |
| `DELETE` | `/api/tasks/<id>` | Delete a task and its steps |
| `POST` | `/api/tasks/<id>/restore` | Undo a task delete |
| `POST` | `/api/tasks/reorder` | Set order with `{"ids": [...]}` |
| `POST` | `/api/braindump` | Turn `text` into tasks, optionally in `project_id` |
| `POST` | `/api/import/keep/preview` | Upload a Takeout .zip or Keep .json files (multipart, field `files`) |
| `POST` | `/api/import/keep/commit` | Import the chosen notes from a preview |
| `GET` | `/api/export` | Download a full backup |
| `POST` | `/api/import/backup` | Restore a backup |
| `GET` | `/api/stats` | Progress figures for the last four weeks |
| `GET` | `/healthz` | Health check |

</details>

## Contributing

Issues and pull requests are welcome. The app is deliberately small: one Python file, one HTML
file, and two dependencies. Please keep it that way where you can, and keep the interface calm.

## License

[MIT](LICENSE)
