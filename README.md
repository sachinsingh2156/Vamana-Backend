
# 🧪 Vamana Backend

The **Vamana Backend** is a robust, Dockerized REST API service developed using **Express.js**, created as part of a collaborative initiative between **IIT Jodhpur** and **AIIA Delhi (All India Institute of Ayurveda)**. The project aims to modernize traditional Ayurvedic research and practice through secure, scalable, and interoperable software solutions.

---

## 🔍 Project Overview

**Vamana** is an Ayurvedic therapeutic procedure. This backend system powers the data management, workflow orchestration, and analytics for digitalizing clinical and research data related to Vamana therapy. It is designed to serve a frontend application, research dashboards, and mobile interfaces.

---

## 🏗️ Tech Stack

- **Node.js / Express.js** – Core backend framework for REST API
- **MongoDB / PostgreSQL** *(based on configuration)* – For secure and scalable data storage
- **Docker** – Containerization for consistent deployment across environments
- **JWT** – Optional user authentication mechanism
- **Swagger / Postman** – For API documentation and testing

---

## 🐳 Deploy on any machine (Windows / Mac / Linux)

One `docker compose up` starts **MongoDB + API + Cloudflare Tunnel**, so the app is reachable on your domain (e.g. `https://vamanaaiia.space`) without opening firewall ports.

Stack:

| Service | Role |
| -------- | ---- |
| `mongodb` | Database (inside Docker) |
| `backend` | Express API on port `3000` |
| `cloudflared` | Cloudflare Tunnel → public domain |

---

### A. One-time Cloudflare setup (do once per domain)

Skip this if the tunnel and hostname are already configured and working.

1. Add the domain to [Cloudflare](https://dash.cloudflare.com) and set the registrar nameservers to Cloudflare’s.
2. Open **Zero Trust → Networks → Tunnels** → create a tunnel (e.g. `vamana-backend`), type **Cloudflared**.
3. Copy the **tunnel token** shown on the install screen (keep it secret).
4. Add a **Public Hostname**:
   - Subdomain: leave empty for the apex domain (or use `www`)
   - Domain: your domain (e.g. `vamanaaiia.space`)
   - Path: leave empty
   - Type: `HTTP`
   - URL: `localhost:3000`
5. Under the domain’s **DNS → Records**, confirm a proxied **CNAME** exists for `@` (or `www`) pointing to  
   `<tunnel-id>.cfargotunnel.com`.
6. **SSL/TLS** → mode **Full**, enable **Always Use HTTPS**.

Only **one machine** should run this tunnel token at a time. Moving to another PC means stopping Docker on the old machine first.

---

### B. Install on a new system (step by step)

#### Windows

1. Install **[Docker Desktop for Windows](https://docs.docker.com/desktop/setup/install/windows-install/)**.
2. During install, keep **WSL 2** backend enabled (recommended).
3. Restart if prompted, then open **Docker Desktop** and wait until it says **Docker is running**.
4. Get the project onto the machine (clone or copy the folder):

   ```powershell
   git clone <your-repo-url> Vamana-Backend
   cd Vamana-Backend
   ```

5. Create the env file from the example:

   ```powershell
   copy .env.example .env
   ```

6. Edit `.env` in Notepad (or any editor) and set your tunnel token:

   ```env
   CLOUDFLARE_TUNNEL_TOKEN=paste_your_tunnel_token_here
   ```

   Get the token from: Cloudflare Zero Trust → Tunnels → your tunnel → configure / install connector.

7. In PowerShell or Command Prompt, from the project folder:

   ```powershell
   docker compose up --build -d
   ```

8. Check containers:

   ```powershell
   docker compose ps
   ```

   You should see `vamana_mongodb` (healthy), `vamana_backend`, and `vamana_cloudflared` as **Up**.

9. Test locally:

   ```powershell
   curl http://localhost:3000/api/test
   ```

   Or open `http://localhost:3000/api/test` in a browser. Expect: `Connection Successful`.

10. Test the public domain: `https://vamanaaiia.space/api/test`  
    In Cloudflare → Tunnels, status should be **Healthy**.

11. Stop the stack (when leaving this machine):

    ```powershell
    docker compose down
    ```

    Data in MongoDB is kept in a Docker volume. To wipe data as well:

    ```powershell
    docker compose down -v
    ```

#### Mac / Linux

Same flow as Windows, with these differences:

1. Install [Docker Desktop for Mac](https://docs.docker.com/desktop/setup/install/mac-install/) or Docker Engine on Linux.
2. Create `.env`:

   ```bash
   cp .env.example .env
   # edit CLOUDFLARE_TUNNEL_TOKEN
   ```

3. Start:

   ```bash
   docker compose up --build -d
   ```

---

### C. Useful commands

```bash
# Start (detached)
docker compose up --build -d

# View logs (all / tunnel only)
docker compose logs -f
docker compose logs -f cloudflared

# Restart after changing .env
docker compose up -d --force-recreate cloudflared

# Stop
docker compose down
```

---

### D. Troubleshooting

| Problem | What to do |
| -------- | ---------- |
| Port `3000` already in use | Stop the other app using 3000, or change the host port in `docker-compose.yml` (`"3001:3000"`). |
| Tunnel status **Inactive** | Confirm `.env` has a valid `CLOUDFLARE_TUNNEL_TOKEN`, then `docker compose up -d --force-recreate cloudflared`. |
| Tunnel connects but domain does not resolve | Domain nameservers / DNS CNAME to `*.cfargotunnel.com` missing in Cloudflare DNS. |
| QUIC / UDP errors in logs | Already mitigated: compose uses `--protocol http2`. Recreate the `cloudflared` service if needed. |
| Docker command not found on Windows | Start **Docker Desktop** first; use PowerShell from the project directory. |
| Old machine still online | Stop Docker there (`docker compose down`) so only one connector uses the token. |

---

## 📁 Project Structure

```
vamana-backend/
├── Controller/           # Express routers
├── Service/              # Business logic
├── Models/               # Mongoose schemas
├── Repository/           # MongoDB connection
├── static/               # Static pages (e.g. privacy)
├── server.js             # Entry point
├── Dockerfile
├── docker-compose.yml    # mongodb + backend + cloudflared
├── .env.example          # Template for CLOUDFLARE_TUNNEL_TOKEN
├── .env                  # Local secrets (do not commit)
└── README.md
```

---

## 🌐 API Endpoints

A few example endpoints:

| Method | Endpoint           | Description                 |
| ------ | ------------------ | --------------------------- |
| GET    | `/api/patients`    | Fetch all patient records   |
| POST   | `/api/patients`    | Create a new patient entry  |
| GET    | `/api/vamana/logs` | Fetch Vamana therapy logs   |
| POST   | `/api/vamana/logs` | Submit a new therapy record |

➡️ API documentation can be accessed via Swagger (if integrated) or through a provided Postman collection.

---

## 🏛️ Institutional Collaboration

This project is a result of academic and clinical collaboration between:

* **Indian Institute of Technology (IIT) Jodhpur** – Tech development & AI integration
* **All India Institute of Ayurveda (AIIA), New Delhi** – Clinical expertise, data provisioning & validation

Together, the institutions aim to digitize Ayurvedic protocols for scalable and scientific exploration.

---

## 🔒 Security & Privacy

* Implements secure practices for handling patient and clinical data
* JWT authentication for authorized access *(optional)*
* HTTPS recommended for production deployments
* Follows data compliance standards in healthcare IT

---

## 📌 Future Enhancements

* Integration with national health data systems (ABDM)
* Advanced analytics and report generation
* Real-time dashboards for doctors and researchers
* AI-driven insights on Vamana outcomes
* Mobile app connectivity

---

## 🧑‍💻 Developed By

**Mayank Jonwal**
B.Tech AI & Data Science, IIT Jodhpur
Collaborating with AIIA, New Delhi

[LinkedIn](https://www.linkedin.com/in/mayankjonwal) • [GitHub](https://github.com/<your-username>)

**Sachin Singh**
M.Tech CSE, IIT Jodhpur
Collaborating with AIIA, New Delhi

[LinkedIn](https://www.linkedin.com/in/sachinsingh2156) • [GitHub](https://github.com/sachinsingh2156)

---

## 📄 License

This project is currently under institutional collaboration and not yet open-source. For access or partnership inquiries, please contact the project leads at **IIT Jodhpur** or **AIIA Delhi**.


