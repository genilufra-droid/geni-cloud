# Plani i Deploy-it të Përhershëm — Sistemi Genit Cloud (Django)

Ky dokument përshkruan hapat konkretë për ta bërë URL-në **të përhershme** (jo quick tunnel që vdes), duke deployuar në një host publik me PostgreSQL.

---

## Pse quick tunnel-i vdes?

Quick tunnel-i i Cloudflare (`*.trycloudflare.com`) është **falas dhe pa llogari**, por:
- Nuk ka garanci uptime (Cloudflare mund ta mbyllë në çdo moment).
- Vdes kur procesi `cloudflared` ndalon ose kur sandbox-i rindizet.
- URL-ja ndryshon çdo herë që rindizet.

Për një URL **të përhershme** duhet një host i dedikuar (Render / Railway / PythonAnywhere) me PostgreSQL.

---

## Opsioni A — Render (rekomanduar për fillestarë)

### Çfarë duhet nga përdoruesi
1. **Llogari falas Render** → https://render.com (regjistrohu me email/GitHub).
2. **Repo GitHub** me kodin e projektit (ose Render mund të deployojë direkt nga GitHub).
3. **PostgreSQL** — Render ofron PostgreSQL falas (plan "Free") direkt nga dashboard-i.

### Hapat
1. Pusho kodin në GitHub (repo private ose publike).
2. Në Render: **New → Web Service** → lidh repon.
3. **Build Command:**
   ```bash
   pip install -r requirements.txt && python manage.py migrate && python manage.py collectstatic --noinput
   ```
4. **Start Command:**
   ```bash
   gunicorn genit_cloud.wsgi:application --bind 0.0.0.0:$PORT --workers 2
   ```
5. **Environment Variables:**
   - `DJANGO_SECRET_KEY` = një varg i gjatë i rastësishëm
   - `DJANGO_DEBUG` = `False`
   - `DJANGO_ALLOWED_HOSTS` = `<emri-i-shërbimit>.onrender.com`
   - `DATABASE_URL` = URL e PostgreSQL (nga Render DB)
6. **Krijo adminin** (pasi DB të jetë gati):
   ```bash
   python manage.py createsuperuser
   ```
   (ose përdor një management command / shell në Render)
7. Render të jep URL të përhershme: `https://<emri>.onrender.com`

---

## Opsioni B — Railway

1. **Llogari Railway** → https://railway.app
2. **New Project → Deploy from GitHub** → lidh repon.
3. Shto **PostgreSQL** plugin (Railway e jep automatikisht `DATABASE_URL`).
4. **Start Command:** `gunicorn config.wsgi:application --bind 0.0.0.0:$PORT`
5. Shto env: `DJANGO_SECRET_KEY`, `DJANGO_DEBUG=False`, `DJANGO_ALLOWED_HOSTS=*.up.railway.app`.
6. URL e përhershme: `https://<emri>.up.railway.app`

---

## Opsioni C — PythonAnywhere

1. **Llogari PythonAnywhere** → https://www.pythonanywhere.com (plan falas ose paid).
2. Ngarko kodin (git clone ose upload).
3. Krijo virtualenv dhe `pip install -r requirements.txt`.
4. Konfiguro **Web tab** → WSGI file që tregon `genit_cloud.wsgi`.
5. PostgreSQL: PythonAnywhere ofron DB falas (MySQL) ose PostgreSQL (paid).
6. URL e përhershme: `https://<username>.pythonanywhere.com`

---

## Çfarë duhet të ndryshohet në kod për prodhim real

Kodi aktual (v2) tashmë e mbështet prodhimin:
- `settings.py` lexon `DATABASE_URL`, `DJANGO_SECRET_KEY`, `DJANGO_DEBUG`, `DJANGO_ALLOWED_HOSTS` nga environment.
- WhiteNoise shërben static files (CompressedManifestStaticFilesStorage).
- Gunicorn si WSGI server.

**Vetëm sigurohu që:**
- `DATABASE_URL` të tregojë te PostgreSQL (jo SQLite) në prodhim.
- `DJANGO_ALLOWED_HOSTS` të përfshijë domain-in e host-it.
- Migrimet të ekzekutohen në deploy (`migrate`).
- Admini të krijohet pas migrimeve.

---

## Kredencialet aktuale (demo)

- **Username:** `admin`
- **Password:** `GenitCloud#2026!Admin`

> ⚠️ Në prodhim real, ndryshoje këtë password menjëherë pas deploy-it.

---

## Përmbledhje e shpejtë

| Host | URL e përhershme | PostgreSQL | Kosto |
|------|------------------|------------|-------|
| Render | `<emri>.onrender.com` | Po (free) | Free tier |
| Railway | `<emri>.up.railway.app` | Po | Trial/paid |
| PythonAnywhere | `<username>.pythonanywhere.com` | Po (paid) | Free/paid |

**Rekomandim:** Render me PostgreSQL — më i thjeshtë për fillestarët dhe ka free tier.