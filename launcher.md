# Case Study: "The Time Capsule" Environment Manager & Launcher

[← Back to Home](./index.md) | [🔗 View Full Repository on GitHub](https://github.com/3dvm/macuare-hub)

<div align="center">
  <video width="800" controls autoplay loop muted playsinline preload="metadata" style="border-radius: 8px;">
    <source src="https://estudiomacuare.com/uploads/tu_video_launcher.mp4" type="video/mp4">
    Your browser does not support native video playback.
  </video>
  <p><em>Macuare Hub in action: Single Sign-On (SSO), dynamic environment building, and isolated DCC launching.</em></p>
</div>
<br>

---

### 1. The Problem: "Dependency Hell" and Security Risks
Updating software mid-production is a massive risk for any studio. Global installations and cross-dependencies often mean that upgrading a tool for a new project completely breaks legacy projects. Artists waste hours dealing with Python `AttributeError` crashes, missing add-ons, and manual path configurations just to open an older file.

Furthermore, traditional pipelines often rely on saving plain-text SVN or network credentials on local disks, creating significant security vulnerabilities in shared studio environments.

The challenge was twofold: **eliminate environment friction entirely** (ensuring every project retains its original "DNA" to be opened years later) and **secure network credentials** without compromising the artist's user experience.

### 2. The Solution: A Lightweight "rez-like" Studio Hub
To solve this, I developed the **Macuare Hub**, a custom environment manager and standalone executable written in Python. Functioning as a lightweight, OS-agnostic alternative to frameworks like *rez*, this launcher creates an absolute, invisible sandbox for artists. 

It acts as a bridge between our production tracker (Kitsu), our version control (SVN), and our hybrid storage (Nextcloud). Based on the project's JSON manifest, the Hub dynamically extracts specific DCC binaries, isolates required add-ons locally, and injects runtime variables—all without touching the OS global registry.

<div align="center">
  <img src="assets/img/macuare_hub_artist_dashboard.png" alt="Macuare Hub - Artist Dashboard" width="700" style="border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.2);">
  <br><em>Artist Dashboard built with CustomTkinter. Projects are populated dynamically based on Kitsu assignments.</em>
</div>

<div class="mermaid">
flowchart TD
    subgraph Cloud [Studio Cloud Infrastructure]
        K[🦊 Kitsu API<br>SSO & Assignments]
        N[☁️ Nextcloud<br>Software Vault & Manifests]
        S[🐘 SVN Server<br>Production Assets & Shots]
    end

    subgraph Workstation [Artist Local Machine]
        direction TB
        MH{⚙️ Macuare Hub<br>Standalone Executable}
        RAM[(🧠 In-Memory Vault<br>Volatile Credentials)]
        SB[📦 Ephemeral Sandbox<br>./06_conf_LOCAL/]
        DCC[🎨 Blender Subprocess]
    end

    %% Data Flow
    K <-->|Role Auth / JSON| MH
    N -->|Downloads Add-ons .zip| MH
    MH -->|Writes Extensions & Configs| SB
    MH -->|Stores Passwords temporarily| RAM
    
    %% Injection
    RAM -.->|Injects ENV variables| DCC
    SB -.->|BLENDER_USER_RESOURCES override| DCC
    
    %% DCC connection
    DCC <-->|Commits/Updates Data| S

    %% Styling
    classDef cloud fill:#2c3e50,stroke:#fff,stroke-width:2px,color:#fff;
    classDef local fill:#34495e,stroke:#fff,stroke-width:2px,color:#fff;
    classDef hub fill:#e67e22,stroke:#fff,stroke-width:3px,color:#fff;
    classDef security fill:#e74c3c,stroke:#fff,stroke-width:2px,color:#fff;

    class K,N,S cloud;
    class SB,DCC local;
    class MH hub;
    class RAM security;
</div>

---

### 3. Key Pipeline Features

* **Absolute Runtime Isolation (Sandbox):** By dynamically overriding variables like `BLENDER_USER_RESOURCES`, the Hub forces the DCC to read and write preferences, extensions, and scripts strictly within a local project folder (`06_conf_LOCAL`). Artists can run Blender 3.6 and Blender 4.2 simultaneously without cross-contamination.
* **In-Memory Credential Vault (JIT Security):** The Hub implements a Just-In-Time interception pattern. SVN passwords are asked once via a modal and kept strictly in volatile RAM. They are injected into the DCC as environment variables and wiped entirely upon logout.
* **Role-Based Access Control (Kitsu SSO):** Integrated with the Gazu API, the Hub detects the user's role. Technical Directors (TDs) get a specialized dashboard to deploy new pipelines and sync manifests, while Artists get a streamlined, read-only launcher.
* **Custom App Templates & Splash Screens:** Automatically deploys studio-specific UI templates and project-specific Splash Screens to ensure artists instantly know they are in the correct context.

<div align="center">
  <img src="assets/img/macuare_hub_td_wizard.png" alt="Macuare Hub - TD Project Builder Wizard" width="500" style="border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.2);">
  <br><em>Technical Director view: Initializing a new project with specific DCC versions, mandatory extensions, and a custom Splash Screen.</em>
</div>

---

### 4. Tools & Infrastructure Architecture

| Category | Technology Used |
| :--- | :--- |
| **Language / UI** | Python 3.12, CustomTkinter (Artist-proof UI) |
| **Production Tracking** | Kitsu (Gazu API for SSO and Role Validation) |
| **Network / VCS** | SVN (Apache Subversion), Nextcloud, Git LFS |
| **Core Architecture** | MVC Pattern, In-Memory Vault, Ephemeral Sandboxing |
| **Distribution** | PyInstaller (Standalone frozen executable, no Python required for artists) |

---

### 5. Code Architecture Snapshots

#### Snapshot 1: Enforcing Absolute Isolation (Sandbox)
Instead of extracting add-ons globally, the Hub builds an ephemeral environment. It hijacks the DCC's resource paths before launching the subprocess, guaranteeing zero global footprint.

```python
import os
import subprocess
from pathlib import Path

def lanzar_blender(project_root: Path, config_path: Path, svn_user: str, svn_pwd: str, kitsu_user: str, kitsu_pwd: str):
    # ... (Manifest parsing and binary validation omitted) ...

    env = os.environ.copy()
    env["MACUARE_PROJECT_CONFIG"] = str(config_path)
    
    # === ABSOLUTE ISOLATION (SANDBOX) ===
    # Force the DCC to use a local, project-specific folder for all configs and extensions
    sandbox_dir = project_root / "06_conf_LOCAL" / "blender_data"
    sandbox_dir.mkdir(parents=True, exist_ok=True)
    env["BLENDER_USER_RESOURCES"] = str(sandbox_dir)
    
    # Inject RAM-secured credentials
    env["MACUARE_SVN_USER"] = svn_user
    env["MACUARE_SVN_PASSWORD"] = svn_pwd
    env["KITSU_USER"] = kitsu_user
    env["KITSU_PWD"] = kitsu_pwd

    # Launch subprocess with the custom App Template
    cmd = [str(blender_bin), "--app-template", template_name]
    subprocess.Popen(cmd, env=env)

```

#### Snapshot 2: Just-In-Time (JIT) Credential Interception

To prevent saving passwords on disk, the UI components intercept the launch action. If the RAM Vault is empty, execution pauses, prompts the user, stores the keys in volatile memory, and resumes automatically via callbacks.

```python
    def iniciar_proyecto_hilo(self, project_root: Path, config_path: Path):
        # === JIT INTERCEPTOR FOR SECURE LAUNCH ===
        if not self.vault.has_svn_credentials():
            SVNLoginWindow(
                parent=self.winfo_toplevel(),
                vault_manager=self.vault,
                on_success_callback=lambda: self.iniciar_proyecto_hilo(project_root, config_path)
            )
            return

        # Extract full credentials strictly from RAM
        svn_user, svn_pwd = self.vault.get_svn_credentials()
        kitsu_user, kitsu_pwd = self.vault.get_kitsu_credentials()

        threading.Thread(
            target=lanzar_blender, 
            args=(project_root, config_path, svn_user, svn_pwd, kitsu_user, kitsu_pwd), 
            daemon=True
        ).start()

```

---

### 6. Impact & Pipeline Metrics

* **Deployment Time:** Reduced artist onboarding and environment setup from hours of manual configuration to **1 click**.
* **Bulletproof Security:** Zero plain-text credentials on local disks. Passwords live strictly in RAM during the active session.
* **100% Backward Compatibility:** Legacy projects are effectively "time capsules". An asset built in a 2024 pipeline opens flawlessly in 2026 with its own frozen tools, without breaking the modern studio ecosystem.

---

[← Back to Main Portfolio](./index.md)

**Ernesto Del Valle Macuare** | Pipeline TD & Tools Developer

[🔗 LinkedIn](https://www.google.com/search?q=https://www.linkedin.com/in/tu-perfil) | [🔗 GitHub](https://www.google.com/search?q=https://github.com/3dvm) | 📧 edelvallemacuare@gmail.com
