# Lifeboat: the house without the internet

Goal: if the internet goes away for days or weeks (a cyberattack, a long outage),
the things a computer is normally used for still work at home, and the most
important ones still work with no power at all.

Three tiers, built from the most durable up:

| Tier | Survives | Holds |
| --- | --- | --- |
| **Paper** | everything | road atlas, printed routes, contacts, medical list, how-tos, this network's recovery steps |
| **Phone and laptop** | days without grid power (power bank, solar) | offline maps with GPS routing, Kiwix with the key books, a small local model, encrypted documents |
| **Home servers** | the internet, while there is power | the full library, maps server, the AI box's models, mirrors, the start page |

## Where things are

| What | Address | Runs on |
| --- | --- | --- |
| Start page | http://10.27.28.202 | cluster, `apps/lifeboat/start-page.yaml` |
| Offline library | http://10.27.28.203 | cluster, `apps/lifeboat/library.yaml` (books listed in its `zims.txt`) |
| Offline Wikipedia (text) | http://10.27.27.10:8888 | AI box, kiwix-serve container (the same file is also in the cluster library) |
| Offline AI chat | http://10.27.27.10 | AI box, Open WebUI + Ollama |
| Read-aloud page | http://10.27.27.12:8765 | laptop, `C:\dev\read-aloud\follow` |

Phones on the home Wi-Fi reach the cluster and the AI box on port 80 through the
existing firewall rules; nothing new had to be opened for the cluster.

## Phases

**Phase 1: library and start page (started 2026-11-07)**
- [x] `apps/lifeboat`: Kiwix on the USB disks, a downloader that keeps `zims.txt` current
      with sha256 checks, and the start page.
- [x] Watch the first download finish: done 2026-11-07, 19 books, 16.6 GB
      (`kubectl -n lifeboat logs deploy/kiwix -c fetch -f`).
- [x] Decide on the big books (2026-11-08). All of Wikipedia's text (no pictures, 53 GB) was
      copied in from the AI box over the home network instead of downloaded. Project
      Gutenberg (221 GB) downloads once. Wikipedia with pictures (127 GB) is skipped: it
      mostly repeats the text, and the medical set already has its pictures. Big books are
      marked `once` in `zims.txt`, so a new edition never re-downloads by itself.
- [x] AI chat reads the library (2026-11-10): an "Offline Library" tool in the AI box's
      Open WebUI, attached to the three Home Assistant presets. It searches the cluster
      library by shelf (medical, repair, survival, food, garden, radio, travel, books), reads
      the best page and gives its link; medical questions prefer the medical encyclopedia.
      "Offline Wikipedia" still reads the AI box's own copy, so it works with the cluster off.

**Phase 2: maps and directions**
- [x] Phones: Organic Maps with the home state and its neighbours downloaded (2026-11-10).
- [ ] Phones: Kiwix app with `zimgit-post-disaster` and `wikipedia_en_medicine_maxi`, so first
      aid and survival guides work with no power at home.
- [ ] Cluster: Geofabrik extracts for the same states, a vector tile server and a routing engine
      (OSRM or Valhalla) with a map page on the start page.
- [ ] Paper: a current road atlas, the state DOT highway map, and printed routes (two each,
      not sharing a bridge or highway) to family, hospitals and a meeting point.

**Phase 3: survive a cut cable**
The cluster today depends on the internet in ways that only show when it is gone:
- [ ] Fleet pulls this repo from GitHub: running apps keep running, nothing new deploys.
      Mirror the repo to a local Forgejo and point a second GitRepo at it.
- [ ] Charts come from public Helm repos and images from Docker Hub and ghcr.io. A pod that moves
      to another node offline cannot pull its image. Add a local registry (Zot or Harbor) as a
      pull-through mirror in each node's `/etc/rancher/rke2/registries.yaml`.
- [ ] Time: the firewall already serves NTP to the cluster; a USB GPS receiver on it would keep
      the time right with no internet (etcd and TLS both break on clock drift).
- [ ] RKE2 certificates last a year and renew when RKE2 restarts within 90 days of expiry;
      note the expiry date in the paper binder.
- [ ] `local-path` keeps each volume on one disk with no copy. Keep a second copy of the library
      on an external drive (the 4 TB Seagate, once triaged).

**Phase 4: power, radio, drill**
- [ ] Measure what each box draws (plug-in watt meter); a power station + solar panel for the
      phone/laptop/radio tier. The cluster is the first thing to switch off.
- [ ] Hand-crank AM/FM/NOAA weather radio; Meshtastic LoRa radios for texting with no towers;
      GMRS license ($35, no exam, covers family).
- [ ] Drill: block the internet at the firewall for a day and write down everything that breaks.

## Non-digital basics

Water (1 gallon per person per day, two weeks if possible), food, a month of
prescriptions, cash in small bills (card networks fail first), first aid kit,
printed phone numbers and 2FA recovery codes, and an encrypted USB drive with IDs,
insurance, medical records and a password manager export (one copy off-site).
