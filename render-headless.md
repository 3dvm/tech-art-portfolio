# Case Study: Art-Driven Render Pipeline (Historias Nativas)

[← Back to Home](./index.md) | [🔗 View Full Repository on GitHub](https://github.com/3dvm/historias-nativas-pipeline)

<div align="center">
  <video width="600" controls autoplay loop muted playsinline preload="metadata" style="border-radius: 8px;">
    <source src="https://estudiomacuare.com/wp-content/uploads/atancha_360_export.mp4" type="video/mp4">
    Your browser does not support native video playback.
  </video>
  <p><em>The "Painting with Polygons" effect: A complex, art-driven result requiring strict Render Layer separation and dynamic scene mutation at render time.</em></p>
</div>
<br>

---

### 1. The Problem: Art Direction vs. Farm Downtime
In stylized animation pipelines, splitting scenes into rendering passes (e.g., Backgrounds vs. Characters) requires drastically different DCC settings to achieve specific visual goals. Characters required motion blur to drive vector displacement, while backgrounds required raytracing without blur. 

Additionally, the project suffered from asset duplication, where character and prop libraries were copied locally into every episode's folder. Relying on artists to manually resolve broken library links, toggle engine settings, and route compositing nodes before submitting to the render farm guaranteed human error, broken dependencies, and massive downtime.

### 2. The Solution: Autonomous QA & Headless Mutation
To eliminate GUI overhead and human intervention, I engineered a headless rendering framework driven by Bash and executed via Blender's Python API (`bpy`). 

1. **Asset Centralization:** A headless Python script automatically iterated through `bpy.data.libraries` to batch-migrate hundreds of shots to a centralized `libs/` directory structure without opening the GUI.
2. **Automated Sanity Check (QA):** An OS-level script scans the output directories against the original scene's length, mathematically detecting missing frames and writing them to a state-tracking manifest (`omit.list`).
3. **Headless Orchestration:** The core Bash script parses the manifest and launches Blender in strict background mode (`-b`), actively intercepting the initialization sequence to inject custom Python configuration payloads.

<div class="mermaid">
flowchart TD
    %% QA & Sanity Check Phase
    A[📁 Master Scene .blend] --> Z{🔍 render_qa_checker.py}
    Z -->|Scans PNGs vs Scene length| C[(📄 omit.list log)]
    
    %% Orchestration Phase
    A --> B{⚙️ render_missing_passes.sh}
    
    subgraph Orchestration [Bash Render Orchestration]
        B <-->|Reads Status| C
        B -->|If Env pass missing| D[Blender + setup_environment_pass.py]
        B -->|If Char pass missing| E[Blender + setup_character_pass.py]
    end
    
    subgraph Mutation [Python API Mutation - Headless]
        D -.->|Overrides| F[Anti-Aliasing: ON<br>Motion Blur: OFF<br>Z-Mask: OFF]
        E -.->|Overrides| G[Anti-Aliasing: OFF<br>Motion Blur: ON<br>Z-Mask: ON]
    end
    
    F --> H[🖼️ Environment Sequence]
    G --> I[🖼️ Character Sequence]

    H --> J((🎬 Final Compositing))
    I --> J

    %% Styling
    classDef bash fill:#4EAA25,stroke:#fff,stroke-width:2px,color:#fff;
    classDef python fill:#3776AB,stroke:#fff,stroke-width:2px,color:#fff;
    classDef file fill:#E26F25,stroke:#fff,stroke-width:2px,color:#fff;
    
    class B bash;
    class Z,D,E python;
    class A,C file;
</div>

---

### 3. Impact & Pipeline Metrics
* **Idempotent Queue:** The Bash orchestrator safely re-launches crashed jobs. If an overnight render fails at frame 150, the parser (`cut -d" "`) surgically injects `-s 150 -e 300` into the command line, preventing re-rendering of completed sequences.
* **Resource Optimization:** Bypassing the Blender GUI entirely saved significant RAM overhead per node, allowing heavier scenes to render on standard studio workstations.
* **Zero Human Error:** Hardcoding layer settings and compositing paths through Python injections guaranteed that Characters and Backgrounds were always rendered with their exact technical requirements.

---

### 4. Code Architecture Snapshot

*(A snippet of the Python injection payload used to configure the Character Pass. It utilizes historical API syntax compliant with Blender 2.75 (Python 3.4) to isolate Render Layers with Z-masking and audit the Compositing tree).*

```python
import bpy

# 1. Pipeline-Specific Render Settings (Character Pass)
bpy.context.scene.render.use_antialiasing = False
bpy.context.scene.render.use_motion_blur = True
bpy.context.scene.render.use_raytrace = False

# 2. Dynamic Output Path Generation based on File Data
filename = bpy.path.display_name_from_filepath(bpy.data.filepath)
base_path = "//../../render/" + filename[0:2] + "/" + filename[0:5] + "_personaje/"
bpy.context.scene.render.filepath = base_path

# 3. Z-Masking & Render Layer Logic
for layer in bpy.context.scene.render.layers[1:]:
    layer.use = True
    for m in range(20):
        layer.layers_zmask[m] = False # Reset masks
    
    # Activate Z-mask specifically for the background holdout layer
    layer.layers_zmask[6] = True 

# 4. Compositing Node Tree Audit
if bpy.context.scene.use_nodes:
    tree = bpy.context.scene.node_tree
    for node in tree.nodes:
        if 'File' in node.name:
            if 'fondo' in node.base_path:
                node.mute = True # Mute background outputs
            elif 'personaje' in node.base_path:
                node.base_path = base_path

```

---

[← Back to Main Portfolio](./index.md)

**Ernesto Del Valle Macuare** | Pipeline TD & Technical Artist

[🔗 LinkedIn](https://www.linkedin.com/in/ernesto-del-valle-macuare/) | [🔗 GitHub](https://github.com/3dvm) | 📧 edelvallemacuare@gmail.com






