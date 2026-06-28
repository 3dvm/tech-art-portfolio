# Case Study: "The Time Capsule" Environment Manager & Launcher

[← Back to Home](./index.md)

<div align="center">
  <video width="800" controls autoplay loop muted playsinline preload="metadata" style="border-radius: 8px;">
    <source src="https://estudiomacuare.com/uploads/tu_video_launcher.mp4" type="video/mp4">
    Tu navegador no soporta la reproducción de video nativo.
  </video>
</div>
<br>

---

### 1. The Problem: "Dependency Hell" and Broken Pipelines
Updating software mid-production is a massive risk for any studio. Global installations and cross-dependencies often mean that upgrading a tool for a new project completely breaks legacy projects. Artists waste hours dealing with Python `AttributeError` crashes, missing add-ons, and manual path configurations just to open an older file.

The challenge was to eliminate this friction entirely, ensuring that every project retains its original "DNA" and can be opened years later without breaking the current studio ecosystem.

### 2. The Solution: A Lightweight "rez-like" Studio Hub
To solve this, I developed the **Macuare Hub**, a custom environment manager written in Python. Functioning as a lightweight alternative to frameworks like *rez*, this launcher creates an invisible sandbox for artists. 

It scans our hybrid network (Nextcloud + SVN) and parses a unique configuration file (`project_config.json`) for each project. Based on this data, the Hub dynamically injects the exact network topology, loads the specific Blender executable (e.g., v5.0.1 vs v5.1.2), and isolates the required custom add-ons without relying on global system installations. 

Artists can even run multiple projects simultaneously with entirely different, conflicting environments, and neither will crash.

---

### 3. Tools & Infrastructure

| Category | Technology Used |
| :--- | :--- |
| **Language** | Python (`os`, `json`, `subprocess`) |
| **GUI Framework** | CustomTkinter (Modern, artist-friendly interface) |
| **Network / VCS** | SVN (Apache Subversion), Nextcloud, Git LFS |
| **Core Architecture** | Environment Isolation, JSON Data Parsing, Dynamic Path Injection |

### 4. Impact & Pipeline Metrics
* **Deployment Time:** Reduced artist onboarding and environment setup from hours of manual configuration to **1 click**.
* **Zero Global Installations:** Add-ons and extensions (like Kitsu and custom Asset Pipelines) are loaded dynamically per session, keeping the artist's local machine completely clean.
* **100% Backward Compatibility:** Legacy projects are frozen in time. An asset built in a 2024 pipeline opens flawlessly in 2026 without breaking modern tools.

---

### 5. Code Architecture Snapshot

*(A snippet of the Python logic used to parse the project DNA and inject the sandboxed environment before launching the DCC).*

```python
# Snapshot: Parsing Project DNA and Injecting Isolated Paths
import os
import json
import subprocess

def launch_sandboxed_environment(project_path):
    config_file = os.path.join(project_path, "project_config.json")
    
    with open(config_file, 'r') as file:
        dna = json.load(file)
        
    # Inject isolated paths for scripts and add-ons (Zero Global Installation)
    env = os.environ.copy()
    env["BLENDER_USER_SCRIPTS"] = dna["isolated_scripts_path"]
    env["MACUARE_ASSET_LIBRARY"] = dna["svn_repo_path"]
    
    executable = dna["blender_executable_path"]
    
    print(f"[SUCCESS] Injecting environment for {dna['project_name']} - Blender {dna['version']}")
    subprocess.Popen([executable], env=env)

```

---

[← Back to Main Portfolio](https://www.google.com/search?q=./index.html)

**Ernesto Del Valle Macuare** | Pipeline TD & Technical Artist

📧 edelvallemacuare@gmail.com
