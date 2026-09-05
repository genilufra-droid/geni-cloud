# 🚀 Deploy i Përhershëm në Render — "Sistemi Genit Cloud"

Ky udhëzues shpjegon hap-pas-hapi si ta deployosh aplikacionin Django **"Sistemi Genit Cloud"** në **Render** me **PostgreSQL**, në mënyrë që URL-ja të jetë **e përhershme** (jo si quick tunnel-i që vdes).

---

## ✅ Çfarë është gati në këtë projekt

| Skedar | Përshkrim |
|--------|-----------|
| `render.yaml` | Blueprint i Render — krijon automatikisht **PostgreSQL (free)** + **Web Service** |
| `Procfile` | Komanda e startimit me Gunicorn |
| `runtime.txt` | Versioni i Python (3.11.9) |
| `requirements.txt` | Django, gunicorn, psycopg2-binary, dj-database-url, whitenoise, python-dotenv |
| `genit_cloud/settings.py` | Lexon `DATABASE_URL`, `DJANGO_SECRET_KEY`, `DJANGO_DEBUG`, `DJANGO_ALLOWED_HOSTS` nga env; WhiteNoise i konfiguruar |
| `.gitignore` | Përjashton `db.sqlite3`, `.env`, `staticfiles/`, etj. |

---

## 🧾 Çfarë llogarish/kredencialesh duhen nga TI

| Çfarë | Ku ta marrësh | Pse duhet |
|-------|---------------|-----------|
| **Llogari GitHub** | https://github.com | Për të pushuar kodin dhe për ta lidhur me Render |
| **Llogari Render** | https://render.com (Sign up → GitHub) | Për të krijuar Web Service + PostgreSQL |

> Render ka **plan falas** — nuk kërkon kartë krediti për të filluar.

---

## 📋 Hapat e Deploy-it

### Hapi 1 — Pusho kodin në GitHub
```bash
cd documents/genit-cloud-django_v3
git init
git add .
git commit -m "Sistemi Genit Cloud - Django"
# Krijo një repo të ri në GitHub (pa README) dhe:
git remote add origin https://github.com/<USERNAME>/genit-cloud.git
git push -u origin main
```

### Hapi 2 — Krijo llogari Render dhe lidh GitHub
1. Shko te [render.com](https://render.com) → **Sign up** → **Continue with GitHub**.
2. Autorizo Render të aksesojë repon tënde GitHub.

### Hapi 3 — Deploy me Blueprint
1. Në dashboard-in e Render: **New +** → **Blueprint**.
2. Zgjidh repon `genit-cloud` (ose si e quajte).
3. Render do të lexojë `render.yaml` dhe do të krijojë automatikisht:
   - **PostgreSQL** (`genit-cloud-db`, plan free)
   - **Web Service** (`genit-cloud`, plan free)
4. Shtyp **Apply** → Render ndërton dhe deployon vetë.

### Hapi 4 — Krijo adminin (superuser)
Pas deploy-it të parë, hap **Render Dashboard → genit-cloud → Shell** dhe ekzekuto:
```bash
python manage.py createsuperuser
```
Vendos username + password (p.sh. `admin` / një password i fortë).

> ⚠️ Render-i ekzekuton `migrate` dhe `collectstatic` automatikisht në build (shih `render.yaml`), kështu që DB-ja PostgreSQL krijohet dhe migrohet vetë.

### Hapi 5 — Merre URL-në publike
- Render të jep URL: **`https://genit-cloud.onrender.com`**
- Hap `/login/` → hyr me superuser-in që krijoje.

---

## 🔧 Env Vars (të vendosura automatikisht nga render.yaml)

| Var | Vlera |
|-----|-------|
| `DJANGO_SECRET_KEY` | Gjenerohet automatikisht nga Render |
| `DJANGO_DEBUG` | `False` |
| `DJANGO_ALLOWED_HOSTS` | `*` (ose vendos domain-in e Render) |
| `DATABASE_URL` | Lidhet automatikisht me PostgreSQL |
| `PYTHON_VERSION` | `3.11.9` |

---

## 🧪 Testimi pas deploy-it

```bash
curl -I https://<username>.onrender.com/login/
# Prit: HTTP/1.1 200 OK
```

---

## ⚠️ Shënime të rëndësishme

- **Plan free i Render** "sleeps" pas ~15 min pa trafik — faqja hapet pak më ngadalë herën e parë pas pushimi (normal).
- **Mos e pusho `db.sqlite3`** në GitHub — `.gitignore` e përjashton. Në prodhim përdoret PostgreSQL.
- Për të përditësuar: `git push` në `main` → Render ri-deployon automatikisht.

---

## 🎯 Përmbledhje

Me këtë setup, aplikacioni **"Sistemi Genit Cloud"** do të jetë i aksesueshëm publikisht në një URL **të përhershme** (`https://<username>.onrender.com`) me **PostgreSQL** si database prodhimi, pa pasur nevojë për quick tunnel që vdes.