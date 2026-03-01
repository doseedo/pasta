# PASTA WARFARE — Complete Game Development Prompt for Claude Code Agent (Unreal Engine 5.4+)

---

## MISSION STATEMENT

You are building **"Pasta Warfare"** — a AAA-quality Fortnite-style battle royale game in Unreal Engine 5.4+ where every playable character is an anthropomorphic pasta type, every weapon and ability is pasta-themed, and the map is a 1:1 stylized recreation of planet Earth that displays a **live heatmap of real-world global conflicts** pulled from open data APIs. Players drop into real conflict zones (Iraq, Ukraine, Sudan, etc.) and fight in recognizable real-world terrain rendered in Fortnite's signature stylized-realism art direction — except everyone is pasta and the arsenal is carb-based.

This prompt covers **every system, asset, pipeline, and line of architecture** needed to ship a release-candidate build. Execute every section sequentially. Do not skip sections. Do not stub out systems — implement them fully. Where AI-generated assets are specified, use the exact pipelines described.

---

## TABLE OF CONTENTS

1. [Project Setup & Directory Structure](#1-project-setup--directory-structure)
2. [AI Asset Generation Pipeline](#2-ai-asset-generation-pipeline)
3. [Character System — Pasta Fighters](#3-character-system--pasta-fighters)
4. [Weapon System — Pasta Arsenal](#4-weapon-system--pasta-arsenal)
5. [Ability System — Pasta Powers](#5-ability-system--pasta-powers)
6. [World Map — Earth Recreation](#6-world-map--earth-recreation)
7. [Conflict Heatmap System](#7-conflict-heatmap-system)
8. [Battle Royale Core Loop](#8-battle-royale-core-loop)
9. [Networking & Multiplayer](#9-networking--multiplayer)
10. [UI/UX System](#10-uiux-system)
11. [Audio System](#11-audio-system)
12. [VFX & Shader Systems](#12-vfx--shader-systems)
13. [Optimization & Streaming](#13-optimization--streaming)
14. [Backend Services](#14-backend-services)
15. [Build, Package & Release](#15-build-package--release)

---

## 1. PROJECT SETUP & DIRECTORY STRUCTURE

### 1.1 — Create UE5 Project

```bash
# Use UnrealBuildTool to generate the project
# Engine version: 5.4.x or latest stable
# Template: Blank C++ project (NOT Blueprint-only — we need C++ for performance-critical systems)
# Target platforms: Win64, Linux Server (dedicated server), PS5, Xbox Series X (stubs for console)
# Rendering: Lumen (Global Illumination + Reflections), Nanite enabled, Virtual Shadow Maps
# Networking: Push Model replication enabled from the start

mkdir -p PastaWarfare
cd PastaWarfare
# Generate via UE5 project generator or copy from template
```

### 1.2 — Directory Layout

```
PastaWarfare/
├── Source/
│   ├── PastaWarfare/           # Primary game module
│   │   ├── Core/               # Game instance, game mode, game state
│   │   ├── Characters/         # Pasta character classes, components
│   │   ├── Weapons/            # Weapon base, projectiles, hitscans
│   │   ├── Abilities/          # GAS (Gameplay Ability System) setup
│   │   ├── World/              # Map streaming, heatmap, zone system
│   │   ├── Vehicles/           # Meatball tank, spaghetti zipline, etc.
│   │   ├── UI/                 # Slate + UMG widgets
│   │   ├── Networking/         # Replication, prediction, server auth
│   │   ├── AI/                 # Bot behavior trees for backfill
│   │   ├── Backend/            # REST client, matchmaking interface
│   │   └── Audio/              # Audio manager, dynamic music
│   └── PastaWarfareServer/     # Dedicated server module
├── Content/
│   ├── Characters/
│   │   ├── Spaghetti/          # Mesh, skeleton, anims, materials
│   │   ├── Penne/
│   │   ├── Farfalle/
│   │   ├── Rigatoni/
│   │   ├── Fusilli/
│   │   ├── Lasagna/
│   │   ├── Ravioli/
│   │   ├── Macaroni/
│   │   ├── Linguine/
│   │   ├── Tortellini/
│   │   ├── Orzo/
│   │   └── Shared/             # Shared anim montages, sockets
│   ├── Weapons/
│   │   ├── Meshes/
│   │   ├── Animations/
│   │   ├── VFX/
│   │   └── Audio/
│   ├── World/
│   │   ├── Terrain/            # Heightmap tiles, biome materials
│   │   ├── Buildings/          # Modular building kit
│   │   ├── Vegetation/         # Trees, foliage per biome
│   │   ├── Landmarks/          # Iconic structures
│   │   └── Heatmap/            # Heatmap materials, data textures
│   ├── UI/
│   ├── VFX/
│   ├── Audio/
│   └── Cinematics/
├── Plugins/
│   ├── PastaNetworking/        # Custom networking plugin
│   ├── ConflictHeatmap/        # Heatmap data + rendering plugin
│   └── PastaAssetGen/          # AI asset generation tools (editor-only)
├── Scripts/
│   ├── AssetGen/               # Python scripts for AI asset pipeline
│   ├── MapGen/                 # Terrain generation from real-world data
│   └── Build/                  # CI/CD scripts
├── Config/
│   ├── DefaultEngine.ini
│   ├── DefaultGame.ini
│   ├── DefaultInput.ini
│   └── DefaultScalability.ini
└── Tools/
    ├── blender_scripts/        # Blender Python for mesh post-processing
    └── substance_scripts/      # Substance automation for texturing
```

### 1.3 — Required Plugins (Enable in .uproject)

```json
{
  "Plugins": [
    { "Name": "GameplayAbilities", "Enabled": true },
    { "Name": "EnhancedInput", "Enabled": true },
    { "Name": "CommonUI", "Enabled": true },
    { "Name": "Niagara", "Enabled": true },
    { "Name": "MetaHumans", "Enabled": false },
    { "Name": "Water", "Enabled": true },
    { "Name": "GeometryScripting", "Enabled": true },
    { "Name": "WorldPartition", "Enabled": true },
    { "Name": "MassEntity", "Enabled": true },
    { "Name": "OnlineSubsystem", "Enabled": true },
    { "Name": "OnlineSubsystemEOS", "Enabled": true },
    { "Name": "PixelStreaming", "Enabled": false },
    { "Name": "ChaosVehicles", "Enabled": true },
    { "Name": "ProceduralMeshComponent", "Enabled": true }
  ]
}
```

### 1.4 — Build Configuration

```ini
; DefaultEngine.ini additions
[/Script/Engine.RendererSettings]
r.GenerateMeshDistanceFields=True
r.DynamicGlobalIlluminationMethod=1  ; Lumen
r.ReflectionMethod=1                  ; Lumen
r.Shadow.Virtual.Enable=1
r.Nanite.MaxPixelsPerEdge=1
r.Streaming.PoolSize=4096

[/Script/Engine.StreamingSettings]
s.LevelStreamingActorsUpdateTimeLimit=10.0
s.PriorityAsyncLoadingExtraTime=20.0

[/Script/Engine.PhysicsSettings]
DefaultDegreesOfFreedom=Full3D
bSupportUVFromHitResults=True

[SystemSettings]
net.AllowPushModelReplication=1
```

---

## 2. AI ASSET GENERATION PIPELINE

This is the most critical section. Every visual asset in the game (characters, weapons, environments, props) must be generated using AI tools, then refined through an automated pipeline to be game-ready.

### 2.1 — Pipeline Architecture Overview

```
[Text Prompt] → [AI Image Gen (concept art)] → [AI 3D Gen (mesh)] → [Blender Post-Process] → [Auto-UV/Retopo] → [AI Texture Gen] → [Substance Refinement] → [UE5 Import] → [Auto-Rig (characters)] → [Anim Retarget] → [Material Setup] → [LOD Gen] → [Nanite Processing]
```

### 2.2 — AI 3D Model Generation

Use **multiple AI 3D generation services** via their APIs. For each asset, generate from multiple services and pick the best result:

#### Primary: Meshy API (meshy.ai)
```python
# Scripts/AssetGen/meshy_generator.py
import requests
import json
import time
import os

MESHY_API_KEY = os.environ.get("MESHY_API_KEY")
BASE_URL = "https://api.meshy.ai/v2"

def generate_3d_model(prompt: str, art_style: str = "realistic", 
                       topology: str = "quad", target_polycount: int = 50000,
                       negative_prompt: str = "") -> str:
    """
    Generate a 3D model from text prompt using Meshy API.
    Returns the task ID for polling.
    """
    headers = {
        "Authorization": f"Bearer {MESHY_API_KEY}",
        "Content-Type": "application/json"
    }
    
    payload = {
        "mode": "preview",  # First pass: preview for speed
        "prompt": prompt,
        "art_style": art_style,
        "negative_prompt": negative_prompt or "low quality, blurry, deformed, ugly",
        "topology": topology,
        "target_polycount": target_polycount
    }
    
    response = requests.post(f"{BASE_URL}/text-to-3d", headers=headers, json=payload)
    task_id = response.json()["result"]
    
    # Poll until complete
    while True:
        status = requests.get(f"{BASE_URL}/text-to-3d/{task_id}", headers=headers).json()
        if status["status"] == "SUCCEEDED":
            return status  # Contains model_urls with glb, fbx, obj, usdz
        elif status["status"] == "FAILED":
            raise Exception(f"Generation failed: {status.get('error')}")
        time.sleep(10)

def refine_model(task_id: str, texture_richness: str = "high") -> str:
    """
    Refine a preview model to production quality.
    """
    headers = {
        "Authorization": f"Bearer {MESHY_API_KEY}",
        "Content-Type": "application/json"
    }
    
    payload = {
        "mode": "refine",
        "preview_task_id": task_id,
        "texture_richness": texture_richness
    }
    
    response = requests.post(f"{BASE_URL}/text-to-3d", headers=headers, json=payload)
    refine_task_id = response.json()["result"]
    
    while True:
        status = requests.get(f"{BASE_URL}/text-to-3d/{refine_task_id}", headers=headers).json()
        if status["status"] == "SUCCEEDED":
            return status
        elif status["status"] == "FAILED":
            raise Exception(f"Refinement failed: {status.get('error')}")
        time.sleep(15)

def download_model(model_urls: dict, output_dir: str, name: str):
    """Download all format variants of the generated model."""
    os.makedirs(output_dir, exist_ok=True)
    for fmt, url in model_urls.items():
        if url:
            response = requests.get(url)
            filepath = os.path.join(output_dir, f"{name}.{fmt}")
            with open(filepath, 'wb') as f:
                f.write(response.content)
            print(f"Downloaded {filepath}")
```

#### Secondary: Tripo3D API (tripo3d.ai)
```python
# Scripts/AssetGen/tripo_generator.py
import requests
import os
import time

TRIPO_API_KEY = os.environ.get("TRIPO_API_KEY")
BASE_URL = "https://api.tripo3d.ai/v2/openapi"

def generate_3d_tripo(prompt: str, model_version: str = "v2.0-20240919",
                       face_limit: int = 50000) -> dict:
    """Generate 3D model using Tripo API."""
    headers = {
        "Authorization": f"Bearer {TRIPO_API_KEY}",
        "Content-Type": "application/json"
    }
    
    # Create task
    payload = {
        "type": "text_to_model",
        "prompt": prompt,
        "model_version": model_version,
        "face_limit": face_limit,
        "texture": True,
        "pbr": True  # PBR textures for UE5 compatibility
    }
    
    response = requests.post(f"{BASE_URL}/task", headers=headers, json=payload)
    task_id = response.json()["data"]["task_id"]
    
    # Poll
    while True:
        status = requests.get(f"{BASE_URL}/task/{task_id}", headers=headers).json()
        if status["data"]["status"] == "success":
            return status["data"]["output"]
        elif status["data"]["status"] == "failed":
            raise Exception(f"Tripo generation failed")
        time.sleep(8)

def download_tripo_model(output: dict, output_dir: str, name: str):
    """Download Tripo model output (GLB with PBR textures)."""
    os.makedirs(output_dir, exist_ok=True)
    model_url = output.get("model")
    if model_url:
        response = requests.get(model_url)
        filepath = os.path.join(output_dir, f"{name}.glb")
        with open(filepath, 'wb') as f:
            f.write(response.content)
```

#### Tertiary: Stability AI (for concept art → 3D via image-to-3D)
```python
# Scripts/AssetGen/stability_generator.py
import requests
import os

STABILITY_API_KEY = os.environ.get("STABILITY_API_KEY")

def generate_concept_art(prompt: str, output_path: str, 
                          style_preset: str = "digital-art",
                          width: int = 1024, height: int = 1024) -> str:
    """Generate concept art using Stability AI for reference images."""
    headers = {
        "Authorization": f"Bearer {STABILITY_API_KEY}",
        "Content-Type": "application/json",
        "Accept": "application/json"
    }
    
    payload = {
        "text_prompts": [
            {
                "text": prompt,
                "weight": 1.0
            },
            {
                "text": "blurry, low quality, watermark, text, signature, deformed",
                "weight": -1.0
            }
        ],
        "cfg_scale": 7,
        "width": width,
        "height": height,
        "samples": 4,  # Generate 4 variants to pick best
        "steps": 50,
        "style_preset": style_preset
    }
    
    response = requests.post(
        "https://api.stability.ai/v1/generation/stable-diffusion-xl-1024-v1-0/text-to-image",
        headers=headers,
        json=payload
    )
    
    data = response.json()
    os.makedirs(os.path.dirname(output_path), exist_ok=True)
    
    paths = []
    for i, image in enumerate(data["artifacts"]):
        import base64
        img_path = output_path.replace(".png", f"_v{i}.png")
        with open(img_path, 'wb') as f:
            f.write(base64.b64decode(image["base64"]))
        paths.append(img_path)
    
    return paths

def image_to_3d_stability(image_path: str, output_path: str) -> str:
    """Convert concept art image to 3D model using Stability's Stable Fast 3D."""
    headers = {
        "Authorization": f"Bearer {STABILITY_API_KEY}",
    }
    
    with open(image_path, 'rb') as f:
        files = {"image": f}
        data = {
            "texture_resolution": 2048,
            "foreground_ratio": 0.85
        }
        response = requests.post(
            "https://api.stability.ai/v2beta/3d/stable-fast-3d",
            headers=headers,
            files=files,
            data=data
        )
    
    with open(output_path, 'wb') as f:
        f.write(response.content)
    
    return output_path
```

### 2.3 — Blender Post-Processing Pipeline

Every AI-generated mesh must go through Blender for cleanup:

```python
# Scripts/AssetGen/blender_postprocess.py
# Run via: blender --background --python blender_postprocess.py --   

import bpy
import sys
import json
import os
import math

argv = sys.argv
argv = argv[argv.index("--") + 1:]
input_path = argv[0]
output_path = argv[1]
config_path = argv[2]

with open(config_path, 'r') as f:
    config = json.load(f)

# Clear scene
bpy.ops.wm.read_factory_settings(use_empty=True)

# Import
ext = os.path.splitext(input_path)[1].lower()
if ext == '.glb' or ext == '.gltf':
    bpy.ops.import_scene.gltf(filepath=input_path)
elif ext == '.fbx':
    bpy.ops.import_scene.fbx(filepath=input_path)
elif ext == '.obj':
    bpy.ops.import_scene.obj(filepath=input_path)

# Select all mesh objects
mesh_objects = [obj for obj in bpy.context.scene.objects if obj.type == 'MESH']

for obj in mesh_objects:
    bpy.context.view_layer.objects.active = obj
    obj.select_set(True)
    
    # 1. SCALE NORMALIZATION — standardize to UE5 scale (1 unit = 1cm)
    target_height = config.get("target_height_cm", 180)  # Default human height
    bbox = obj.dimensions
    max_dim = max(bbox)
    if max_dim > 0:
        scale_factor = target_height / (max_dim * 100)
        obj.scale *= scale_factor
        bpy.ops.object.transform_apply(scale=True)
    
    # 2. CENTER ORIGIN
    bpy.ops.object.origin_set(type='ORIGIN_CENTER_OF_VOLUME', center='MEDIAN')
    obj.location = (0, 0, 0)
    bpy.ops.object.transform_apply(location=True)
    
    # 3. RETOPOLOGY — reduce to target polycount using Quadriflow
    bpy.ops.object.mode_set(mode='EDIT')
    bpy.ops.mesh.select_all(action='SELECT')
    
    target_faces = config.get("target_polycount", 15000)
    current_faces = len(obj.data.polygons)
    
    if current_faces > target_faces * 1.5:
        bpy.ops.object.mode_set(mode='OBJECT')
        
        # Decimate modifier for initial reduction
        decimate = obj.modifiers.new(name="Decimate", type='DECIMATE')
        decimate.ratio = target_faces / current_faces
        decimate.use_collapse_triangulate = False
        bpy.ops.object.modifier_apply(modifier="Decimate")
        
        # Remesh for cleaner topology
        if config.get("use_remesh", True):
            remesh = obj.modifiers.new(name="Remesh", type='REMESH')
            remesh.mode = 'VOXEL'
            remesh.voxel_size = config.get("voxel_size", 0.01)
            remesh.use_smooth_shade = True
            bpy.ops.object.modifier_apply(modifier="Remesh")
    
    bpy.ops.object.mode_set(mode='OBJECT')
    
    # 4. UV UNWRAP — Smart UV Project for AI-generated meshes
    bpy.ops.object.mode_set(mode='EDIT')
    bpy.ops.mesh.select_all(action='SELECT')
    bpy.ops.uv.smart_project(
        angle_limit=math.radians(66),
        island_margin=0.01,
        area_weight=0.0,
        correct_aspect=True,
        scale_to_bounds=True
    )
    bpy.ops.object.mode_set(mode='OBJECT')
    
    # 5. NORMALS — Recalculate and smooth
    bpy.ops.object.mode_set(mode='EDIT')
    bpy.ops.mesh.normals_make_consistent(inside=False)
    bpy.ops.object.mode_set(mode='OBJECT')
    
    if config.get("smooth_shading", True):
        bpy.ops.object.shade_smooth()
        obj.data.use_auto_smooth = True
        obj.data.auto_smooth_angle = math.radians(config.get("smooth_angle", 60))
    
    # 6. CLEAN GEOMETRY
    bpy.ops.object.mode_set(mode='EDIT')
    bpy.ops.mesh.select_all(action='SELECT')
    bpy.ops.mesh.remove_doubles(threshold=0.0001)
    bpy.ops.mesh.delete_loose()
    bpy.ops.mesh.fill_holes(sides=4)
    bpy.ops.object.mode_set(mode='OBJECT')
    
    # 7. GENERATE LODs
    if config.get("generate_lods", True):
        lod_ratios = config.get("lod_ratios", [1.0, 0.5, 0.25, 0.1])
        for i, ratio in enumerate(lod_ratios):
            if ratio < 1.0:
                lod_obj = obj.copy()
                lod_obj.data = obj.data.copy()
                lod_obj.name = f"{obj.name}_LOD{i}"
                bpy.context.collection.objects.link(lod_obj)
                
                decimate = lod_obj.modifiers.new(name=f"LOD{i}", type='DECIMATE')
                decimate.ratio = ratio
                bpy.context.view_layer.objects.active = lod_obj
                bpy.ops.object.modifier_apply(modifier=f"LOD{i}")

    obj.select_set(False)

# 8. EXPORT as FBX for UE5
bpy.ops.export_scene.fbx(
    filepath=output_path,
    use_selection=False,
    global_scale=1.0,
    apply_unit_scale=True,
    apply_scale_options='FBX_SCALE_ALL',
    axis_forward='-Y',  # UE5 coordinate system
    axis_up='Z',
    object_types={'MESH', 'ARMATURE'},
    use_mesh_modifiers=True,
    mesh_smooth_type='FACE',
    use_custom_props=False,
    path_mode='COPY',
    embed_textures=True,
    batch_mode='OFF'
)

print(f"Exported: {output_path}")
```

### 2.4 — AI Texture Generation Pipeline

```python
# Scripts/AssetGen/texture_generator.py
"""
Generate PBR texture sets for 3D models using AI.
Pipeline: Meshy Texturing API → Enhance with img2img → Split PBR channels
"""
import requests
import os
from PIL import Image
import numpy as np

MESHY_API_KEY = os.environ.get("MESHY_API_KEY")
STABILITY_API_KEY = os.environ.get("STABILITY_API_KEY")

def generate_textures_meshy(model_path: str, prompt: str, 
                             resolution: int = 2048,
                             art_style: str = "realistic") -> dict:
    """
    Use Meshy's text-to-texture API to generate PBR textures on an existing mesh.
    """
    headers = {"Authorization": f"Bearer {MESHY_API_KEY}"}
    
    # Upload model first
    with open(model_path, 'rb') as f:
        files = {"model_file": f}
        data = {
            "prompt": prompt,
            "art_style": art_style,
            "resolution": resolution,
            "negative_prompt": "low quality, blurry, tiling artifacts, seams"
        }
        response = requests.post(
            "https://api.meshy.ai/v2/text-to-texture",
            headers=headers,
            files=files,
            data=data
        )
    
    task_id = response.json()["result"]
    
    import time
    while True:
        status = requests.get(
            f"https://api.meshy.ai/v2/text-to-texture/{task_id}",
            headers=headers
        ).json()
        if status["status"] == "SUCCEEDED":
            return status["texture_urls"]  # albedo, normal, roughness, metallic, ao
        elif status["status"] == "FAILED":
            raise Exception("Texture generation failed")
        time.sleep(10)

def generate_pbr_from_albedo(albedo_path: str, output_dir: str, name: str):
    """
    Generate full PBR texture set from albedo using neural estimation.
    Creates: Normal, Roughness, Metallic, AO, Height maps
    Uses Stability AI's image-to-image for enhancement.
    """
    os.makedirs(output_dir, exist_ok=True)
    
    albedo = Image.open(albedo_path)
    albedo_np = np.array(albedo)
    
    # Generate normal map estimation (Sobel-based for quick results)
    from scipy import ndimage
    gray = np.mean(albedo_np[:,:,:3], axis=2)
    
    # Sobel for normal map
    dx = ndimage.sobel(gray, axis=1)
    dy = ndimage.sobel(gray, axis=0)
    dz = np.ones_like(gray) * 255
    
    normal = np.stack([
        ((dx / (np.max(np.abs(dx)) + 1e-8)) * 127 + 128).astype(np.uint8),
        ((dy / (np.max(np.abs(dy)) + 1e-8)) * 127 + 128).astype(np.uint8),
        dz.astype(np.uint8)
    ], axis=2)
    
    Image.fromarray(normal).save(os.path.join(output_dir, f"{name}_Normal.png"))
    
    # Roughness estimation (inverse of local contrast)
    local_std = ndimage.generic_filter(gray, np.std, size=5)
    roughness = 255 - (local_std / (np.max(local_std) + 1e-8) * 200).astype(np.uint8)
    Image.fromarray(roughness).save(os.path.join(output_dir, f"{name}_Roughness.png"))
    
    # Metallic (very low for pasta — organic material)
    metallic = np.full_like(gray, 10, dtype=np.uint8)  # Pasta is non-metallic
    Image.fromarray(metallic).save(os.path.join(output_dir, f"{name}_Metallic.png"))
    
    # AO estimation (darken crevices)
    ao = ndimage.gaussian_filter(gray, sigma=3)
    ao = ((ao / (np.max(ao) + 1e-8)) * 255).astype(np.uint8)
    Image.fromarray(ao).save(os.path.join(output_dir, f"{name}_AO.png"))
    
    # Height map
    height = gray.astype(np.uint8)
    Image.fromarray(height).save(os.path.join(output_dir, f"{name}_Height.png"))
    
    # Save enhanced albedo
    albedo.save(os.path.join(output_dir, f"{name}_Albedo.png"))
    
    return {
        "albedo": os.path.join(output_dir, f"{name}_Albedo.png"),
        "normal": os.path.join(output_dir, f"{name}_Normal.png"),
        "roughness": os.path.join(output_dir, f"{name}_Roughness.png"),
        "metallic": os.path.join(output_dir, f"{name}_Metallic.png"),
        "ao": os.path.join(output_dir, f"{name}_AO.png"),
        "height": os.path.join(output_dir, f"{name}_Height.png"),
    }

def enhance_texture_stability(input_path: str, prompt: str, output_path: str,
                                strength: float = 0.35):
    """
    Use Stability AI img2img to enhance/refine an AI-generated texture.
    Low strength preserves structure while improving quality.
    """
    headers = {
        "Authorization": f"Bearer {STABILITY_API_KEY}",
        "Accept": "application/json"
    }
    
    with open(input_path, 'rb') as f:
        files = {"init_image": f}
        data = {
            "text_prompts[0][text]": prompt,
            "text_prompts[0][weight]": 1.0,
            "text_prompts[1][text]": "seams, tiling artifacts, stretching, blur",
            "text_prompts[1][weight]": -1.0,
            "cfg_scale": 7,
            "image_strength": 1.0 - strength,  # API uses inverse
            "samples": 1,
            "steps": 40
        }
        
        response = requests.post(
            "https://api.stability.ai/v1/generation/stable-diffusion-xl-1024-v1-0/image-to-image",
            headers=headers,
            files=files,
            data=data
        )
    
    import base64
    data = response.json()
    with open(output_path, 'wb') as f:
        f.write(base64.b64decode(data["artifacts"][0]["base64"]))
    
    return output_path
```

### 2.5 — Auto-Rigging Pipeline for Characters

```python
# Scripts/AssetGen/auto_rig.py
"""
Auto-rigging pipeline for pasta characters using Mixamo API + Blender.
Since pasta characters are humanoid (anthropomorphic), we use standard humanoid rigs
then customize bone proportions per pasta type.
"""

import requests
import os
import time

def rig_with_mixamo(fbx_path: str, output_path: str) -> str:
    """
    Upload mesh to Mixamo for auto-rigging.
    Requires Mixamo credentials (Adobe account).
    Falls back to Blender Rigify if Mixamo is unavailable.
    """
    # NOTE: Mixamo API is unofficial — implement as web automation or use Rigify fallback
    
    # FALLBACK: Blender Rigify auto-rig
    return rig_with_rigify(fbx_path, output_path)

def rig_with_rigify(fbx_path: str, output_path: str) -> str:
    """
    Use Blender's Rigify addon to auto-rig the character.
    Executed as subprocess calling Blender.
    """
    import subprocess
    
    blender_script = f'''
import bpy
import os

# Clear and import
bpy.ops.wm.read_factory_settings(use_empty=True)
bpy.ops.import_scene.fbx(filepath="{fbx_path}")

# Find mesh
mesh_obj = None
for obj in bpy.context.scene.objects:
    if obj.type == 'MESH':
        mesh_obj = obj
        break

if mesh_obj is None:
    raise Exception("No mesh found in FBX")

# Add Rigify meta-rig
bpy.ops.object.armature_human_metarig_add()
metarig = bpy.context.active_object

# Scale metarig to match character
mesh_height = mesh_obj.dimensions.z
metarig_height = metarig.dimensions.z
if metarig_height > 0:
    scale = mesh_height / metarig_height
    metarig.scale = (scale, scale, scale)
    bpy.ops.object.transform_apply(scale=True)

# Position metarig at mesh origin
metarig.location = mesh_obj.location

# Generate Rigify rig
bpy.ops.pose.rigify_generate()
rig = bpy.context.active_object

# Parent mesh to rig with automatic weights
mesh_obj.select_set(True)
rig.select_set(True)
bpy.context.view_layer.objects.active = rig
bpy.ops.object.parent_set(type='ARMATURE_AUTO')

# Export
bpy.ops.export_scene.fbx(
    filepath="{output_path}",
    use_selection=False,
    add_leaf_bones=False,
    axis_forward='-Y',
    axis_up='Z',
    global_scale=1.0,
    apply_scale_options='FBX_SCALE_ALL',
    object_types={{'MESH', 'ARMATURE'}},
    use_armature_deform_only=True,
    bake_anim=False
)
'''
    
    script_path = "/tmp/rigify_script.py"
    with open(script_path, 'w') as f:
        f.write(blender_script)
    
    subprocess.run([
        "blender", "--background", "--python", script_path
    ], check=True)
    
    return output_path
```

### 2.6 — Animation Retargeting Pipeline

```python
# Scripts/AssetGen/anim_retarget.py
"""
Download Mixamo animations and retarget to pasta character skeletons.
We use a standard humanoid animation set that covers all battle royale actions.
"""

REQUIRED_ANIMATIONS = {
    # Locomotion
    "Idle": "idle standing breathing",
    "Walk_Forward": "walking forward casual",
    "Walk_Backward": "walking backward",  
    "Walk_Left": "strafe walk left",
    "Walk_Right": "strafe walk right",
    "Run_Forward": "running forward",
    "Sprint": "sprinting fast",
    "Crouch_Idle": "crouching idle",
    "Crouch_Walk": "crouch walking",
    "Prone_Idle": "prone lying down idle",
    "Prone_Crawl": "prone crawling forward",
    
    # Jumping
    "Jump_Start": "jump start anticipation",
    "Jump_Loop": "falling loop",
    "Jump_Land": "jump landing",
    "Double_Jump": "double jump flip",
    
    # Combat - Generic
    "Aim_Idle": "aiming rifle idle",
    "Aim_Walk": "aiming rifle walking",
    "Fire_Rifle": "shooting rifle",
    "Fire_Shotgun": "shooting shotgun recoil",
    "Fire_Pistol": "shooting pistol one hand",
    "Reload": "reloading rifle",
    "Melee_Swing": "sword swing overhead",
    "Melee_Combo_1": "punch combo 1",
    "Melee_Combo_2": "kick combo",
    "Throw": "throwing overhand",
    
    # Actions
    "Build_Place": "placing object down",
    "Harvest_Pickaxe": "pickaxe mining swing",
    "Emote_Dance_1": "hip hop dancing",
    "Emote_Dance_2": "chicken dance",
    "Emote_Dance_3": "robot dance",
    "Emote_Wave": "waving hello",
    "Emote_Laugh": "laughing pointing",
    "Revive": "kneeling helping up",
    "DBNO_Enter": "getting knocked down",
    "DBNO_Crawl": "crawling injured",
    "Death": "death falling backward",
    "Death_Headshot": "head shot death",
    
    # Skydiving / Drop
    "Skydive": "skydiving spread eagle",
    "Parachute": "parachute gliding",
    "Glider_Ride": "hang gliding",
    
    # Vehicle
    "Drive_Sit": "sitting driving",
    "Passenger_Sit": "sitting passenger idle",
    
    # Traversal
    "Climb_Up": "climbing wall",
    "Slide": "sliding on knees",
    "Mantle": "vaulting over obstacle",
    "Swim_Surface": "swimming surface",
    "Swim_Underwater": "swimming underwater",
    "Zipline": "zipline hanging",
}

def create_animation_blueprint_config():
    """
    Generate the UE5 Animation Blueprint configuration.
    This creates an ABP that blends all animations based on movement state.
    """
    return {
        "state_machine": {
            "states": {
                "Idle": {
                    "animation": "Idle",
                    "blend_time": 0.2
                },
                "Walking": {
                    "blend_space": "BS_Walk",
                    "axes": {
                        "horizontal": "Direction",
                        "vertical": "Speed"
                    },
                    "animations": ["Walk_Forward", "Walk_Backward", "Walk_Left", "Walk_Right"],
                    "blend_time": 0.15
                },
                "Running": {
                    "animation": "Run_Forward",
                    "blend_time": 0.2
                },
                "Sprinting": {
                    "animation": "Sprint",
                    "blend_time": 0.25
                },
                "Crouching": {
                    "blend_space": "BS_Crouch",
                    "animations": ["Crouch_Idle", "Crouch_Walk"],
                    "blend_time": 0.2
                },
                "Prone": {
                    "blend_space": "BS_Prone",
                    "animations": ["Prone_Idle", "Prone_Crawl"],
                    "blend_time": 0.3
                },
                "Jumping": {
                    "sequence": ["Jump_Start", "Jump_Loop", "Jump_Land"],
                    "blend_time": 0.1
                },
                "Falling": {
                    "animation": "Jump_Loop",
                    "blend_time": 0.15
                },
                "Skydiving": {
                    "animation": "Skydive",
                    "blend_time": 0.3
                },
                "Gliding": {
                    "animation": "Glider_Ride",
                    "blend_time": 0.3
                },
                "Swimming": {
                    "blend_space": "BS_Swim",
                    "animations": ["Swim_Surface", "Swim_Underwater"],
                    "blend_time": 0.3
                },
                "DBNO": {
                    "sequence": ["DBNO_Enter", "DBNO_Crawl"],
                    "blend_time": 0.2
                },
                "Dead": {
                    "animations": ["Death", "Death_Headshot"],
                    "blend_time": 0.0
                }
            },
            "layers": {
                "UpperBody": {
                    "slot": "UpperBody",
                    "blend_mode": "layered",
                    "animations": [
                        "Aim_Idle", "Aim_Walk", "Fire_Rifle", "Fire_Shotgun",
                        "Fire_Pistol", "Reload", "Throw", "Build_Place",
                        "Harvest_Pickaxe"
                    ]
                },
                "FullBody_Override": {
                    "slot": "FullBody",
                    "blend_mode": "override",
                    "animations": [
                        "Melee_Swing", "Melee_Combo_1", "Melee_Combo_2",
                        "Emote_Dance_1", "Emote_Dance_2", "Emote_Dance_3",
                        "Emote_Wave", "Emote_Laugh", "Revive",
                        "Mantle", "Climb_Up", "Slide"
                    ]
                }
            }
        }
    }
```

### 2.7 — Master Asset Generation Orchestrator

```python
# Scripts/AssetGen/generate_all_assets.py
"""
MASTER ORCHESTRATOR — Generates ALL game assets.
Run this script to produce every character, weapon, prop, and environment asset.
"""

import os
import json
import subprocess
from meshy_generator import generate_3d_model, refine_model, download_model
from tripo_generator import generate_3d_tripo, download_tripo_model
from stability_generator import generate_concept_art, image_to_3d_stability
from texture_generator import generate_textures_meshy, generate_pbr_from_albedo
from auto_rig import rig_with_rigify
from blender_postprocess import *  # Used via subprocess

OUTPUT_BASE = "Content/"

# ============================================================
# CHARACTER DEFINITIONS — Every pasta type with exact prompts
# ============================================================
CHARACTERS = {
    "Spaghetti": {
        "prompt": "Highly detailed anthropomorphic spaghetti pasta character, humanoid body made of long thin spaghetti noodles bundled together forming arms legs and torso, head is a round ball of tangled spaghetti with two eyes and expressive face, wearing a tactical vest and combat boots, Fortnite art style, stylized realism, game-ready character, T-pose, full body, white background, 4K",
        "negative": "realistic human skin, smooth body, no pasta texture, blurry, low quality",
        "height_cm": 185,
        "polycount": 25000,
        "special_features": ["noodle_physics_strands", "sauce_drip_vfx"],
        "hitbox_type": "standard_humanoid",
        "passive_ability": "Slippery — 10% movement speed bonus on wet surfaces"
    },
    "Penne": {
        "prompt": "Highly detailed anthropomorphic penne pasta character, humanoid body composed of large penne rigate tubes, torso is a massive penne tube with ridged texture, arms and legs are smaller penne tubes connected, head is a penne tube with a face on the flat end, military tactical gear, Fortnite art style, stylized realism, game-ready, T-pose, full body, white background",
        "negative": "smooth texture, no ridges, realistic human, blurry",
        "height_cm": 195,
        "polycount": 20000,
        "special_features": ["hollow_body_ricochet", "ridged_armor_texture"],
        "hitbox_type": "standard_humanoid_wide",
        "passive_ability": "Hollow Core — 5% chance to have projectiles pass through"
    },
    "Farfalle": {
        "prompt": "Highly detailed anthropomorphic farfalle bowtie pasta character, body shaped like a large farfalle with pinched center forming waist, butterfly wing-like pasta flaps as arms, small farfalle as hands, legs are ruffled pasta shapes, head is a small farfalle with cute face, colorful tactical outfit, Fortnite art style, T-pose, full body, white background, game-ready 3D character",
        "negative": "flat, no depth, realistic human body, blurry",
        "height_cm": 170,
        "polycount": 22000,
        "special_features": ["wing_glide_extended", "flutter_jump"],
        "hitbox_type": "standard_humanoid_slim",
        "passive_ability": "Butterfly Glide — Glider deploys 20% faster, 15% slower descent"
    },
    "Rigatoni": {
        "prompt": "Highly detailed anthropomorphic rigatoni pasta character, massive tank-like body made of one giant rigatoni tube as torso, thick ridged arms and legs from rigatoni sections, head is a rigatoni tube cap with stern face, heavy military armor and gear, Fortnite art style but imposing and strong, T-pose, full body, white background, game-ready 3D model",
        "negative": "thin, smooth, no ridges, delicate, blurry",
        "height_cm": 210,
        "polycount": 22000,
        "special_features": ["heavy_armor_plating", "ground_pound"],
        "hitbox_type": "large_humanoid",
        "passive_ability": "Ridged Defense — Takes 10% less explosive damage"
    },
    "Fusilli": {
        "prompt": "Highly detailed anthropomorphic fusilli spiral pasta character, entire body is twisted spiral pasta form, corkscrew shape torso, spiral arms and legs that can extend and retract, head is a tight fusilli spiral with playful face, agile scout outfit with pouches, Fortnite art style, T-pose, full body, white background, game-ready character model",
        "negative": "straight, no spirals, smooth, realistic human, blurry",
        "height_cm": 175,
        "polycount": 28000,
        "special_features": ["spiral_spin_attack", "corkscrew_movement"],
        "hitbox_type": "standard_humanoid_slim",
        "passive_ability": "Spiral Agility — Can spin-dodge once every 8 seconds, briefly becoming harder to hit"
    },
    "Lasagna": {
        "prompt": "Highly detailed anthropomorphic lasagna character, body is stacked layers of lasagna sheets with visible meat sauce and cheese between layers, thick and wide body type, arms and legs are rolled lasagna tubes, head is a lasagna square with melted cheese hair, heavy assault gear, Fortnite art style, T-pose, full body, white background, game-ready 3D character",
        "negative": "thin, single layer, no cheese, no sauce, blurry",
        "height_cm": 190,
        "polycount": 18000,
        "special_features": ["layer_shield", "cheese_trail"],
        "hitbox_type": "large_humanoid_wide",
        "passive_ability": "Layered Up — Has 25 bonus shield that regenerates slowly out of combat"
    },
    "Ravioli": {
        "prompt": "Highly detailed anthropomorphic ravioli character, body is a large puffy ravioli pillow shape with crimped edges, arms and legs extend from the ravioli body, head is a smaller ravioli with cute face and crimped edge crown, contains visible filling when damaged, stealth operative outfit, Fortnite art style, T-pose, full body, white background, game-ready",
        "negative": "flat, deflated, no crimped edges, realistic, blurry",
        "height_cm": 165,
        "polycount": 18000,
        "special_features": ["stuffed_explosion_on_death", "pillow_bounce"],
        "hitbox_type": "standard_humanoid_round",
        "passive_ability": "Stuffed — On elimination, drops a healing filling pool (50 HP over 5s)"
    },
    "Macaroni": {
        "prompt": "Highly detailed anthropomorphic elbow macaroni character, body made of connected curved macaroni tubes, distinctive C-curved torso, arms are articulated macaroni elbows, head is a large macaroni tube with face on the opening, cheese-themed tactical gear with orange accents, Fortnite art style, T-pose, full body, white background, game-ready 3D model",
        "negative": "straight tubes, no curve, realistic human, blurry, low quality",
        "height_cm": 175,
        "polycount": 20000,
        "special_features": ["cheese_gunk_trail", "elbow_strike"],
        "hitbox_type": "standard_humanoid",
        "passive_ability": "Mac Attack — Melee strikes deal 15% bonus damage"
    },
    "Linguine": {
        "prompt": "Highly detailed anthropomorphic linguine pasta character, body made of flat wide linguine noodles layered and flowing, elegant and tall, limbs are bundles of linguine ribbons, head is wrapped linguine with sophisticated face, sniper/marksman tactical outfit, Fortnite art style, T-pose, full body, white background, game-ready character",
        "negative": "round noodles, spaghetti, thick, blurry, realistic",
        "height_cm": 195,
        "polycount": 24000,
        "special_features": ["ribbon_whip", "flat_profile_peek"],
        "hitbox_type": "tall_slim_humanoid",
        "passive_ability": "Flat Profile — 8% smaller hitbox when facing enemies directly"
    },
    "Tortellini": {
        "prompt": "Highly detailed anthropomorphic tortellini character, body is a large round tortellini ring with folded pasta forming limbs, distinctive belly-button fold shape, arms and legs tucked from the ring body, head emerges from the top fold with mischievous face, demolition expert outfit with explosives, Fortnite art style, T-pose, full body, white background, game-ready",
        "negative": "flat, unfolded, no ring shape, realistic, blurry",
        "height_cm": 160,
        "polycount": 20000,
        "special_features": ["roll_attack", "ring_toss"],
        "hitbox_type": "small_round_humanoid",
        "passive_ability": "Pocket Pasta — Carries 1 extra grenade/throwable slot"
    },
    "Orzo": {
        "prompt": "Highly detailed anthropomorphic orzo pasta character, tiny but numerous orzo grains forming a humanoid collective body like a golem, body shimmers and shifts as individual orzo pieces move, head is a concentrated cluster of orzo with glowing eyes, scout/recon outfit, Fortnite art style, T-pose, full body, white background, game-ready character model",
        "negative": "solid body, single piece, realistic human, blurry, no grain texture",
        "height_cm": 155,
        "polycount": 20000,
        "special_features": ["scatter_dodge", "grain_storm_vfx"],
        "hitbox_type": "small_humanoid",
        "passive_ability": "Grain Scatter — When hit, briefly disperses (0.3s invulnerability, 20s cooldown)"
    }
}

# ============================================================
# WEAPON DEFINITIONS
# ============================================================
WEAPONS = {
    # === ASSAULT RIFLES ===
    "Penne_Puncher": {
        "prompt": "A tactical assault rifle made entirely of penne pasta tubes, the barrel is a long penne tube, magazine is stacked penne, stock is curved penne, fires small penne projectiles, detailed mechanical pasta weapon, game-ready 3D model, white background, Fortnite art style",
        "type": "assault_rifle",
        "damage": 27,
        "fire_rate": 5.5,
        "magazine_size": 30,
        "reload_time": 2.2,
        "rarity_range": ["Common", "Uncommon", "Rare"],
        "projectile": "penne_tube_small"
    },
    "Rigatoni_Repeater": {
        "prompt": "A heavy assault rifle constructed from rigatoni pasta tubes, large ribbed barrel made of rigatoni, rotating drum magazine of rigatoni tubes, heavy stock from giant rigatoni piece, fires explosive rigatoni, detailed weapon, game-ready, white background, Fortnite style",
        "type": "assault_rifle",
        "damage": 35,
        "fire_rate": 3.8,
        "magazine_size": 25,
        "reload_time": 2.8,
        "rarity_range": ["Rare", "Epic", "Legendary"],
        "projectile": "rigatoni_explosive"
    },
    
    # === SHOTGUNS ===
    "Farfalle_Flinger": {
        "prompt": "A combat shotgun made of farfalle bowtie pasta pieces, wide barrel opening for spread, stock is large farfalle piece, pump action mechanism made of pasta, fires burst of sharp farfalle, detailed pasta weapon, game-ready 3D model, white background, Fortnite art style",
        "type": "shotgun",
        "damage": 85,
        "fire_rate": 1.0,
        "magazine_size": 5,
        "reload_time": 4.5,
        "rarity_range": ["Common", "Uncommon", "Rare"],
        "projectile": "farfalle_spread",
        "pellet_count": 10
    },
    "Fusilli_Fury": {
        "prompt": "A double-barrel shotgun made from twisted fusilli pasta, two spiral fusilli barrels side by side, ornate pasta engravings, break-action mechanism, fires devastating spiral fusilli slugs, detailed weapon, game-ready, white background, Fortnite style",
        "type": "shotgun",
        "damage": 120,
        "fire_rate": 0.7,
        "magazine_size": 2,
        "reload_time": 3.0,
        "rarity_range": ["Rare", "Epic", "Legendary"],
        "projectile": "fusilli_spiral_slug"
    },
    
    # === SMGs ===
    "Orzo_Obliterator": {
        "prompt": "A compact submachine gun made from tiny orzo pasta grains compressed together, short barrel, high-capacity drum magazine filled with orzo, rapid-fire mechanism, sprays orzo pellets at high speed, compact tactical pasta weapon, game-ready 3D, white background, Fortnite style",
        "type": "smg",
        "damage": 18,
        "fire_rate": 10.0,
        "magazine_size": 35,
        "reload_time": 1.8,
        "rarity_range": ["Common", "Uncommon", "Rare"],
        "projectile": "orzo_pellet_burst"
    },
    
    # === SNIPER RIFLES ===
    "Linguine_Longshot": {
        "prompt": "A long-range sniper rifle made from flat linguine noodles, extremely long barrel made of bundled linguine, scope made from a pasta ring, precision stock, fires a single penetrating linguine bolt at extreme velocity, elegant detailed pasta weapon, game-ready 3D model, white background, Fortnite style",
        "type": "sniper_rifle",
        "damage": 110,
        "fire_rate": 0.33,
        "magazine_size": 1,
        "reload_time": 3.5,
        "rarity_range": ["Rare", "Epic", "Legendary"],
        "projectile": "linguine_bolt",
        "headshot_multiplier": 2.5
    },
    "Spaghetti_Sniper": {
        "prompt": "A semi-automatic marksman rifle made from spaghetti noodles, long thin spaghetti barrel, woven spaghetti body, scope attachment, extended spaghetti stock, fires hardened spaghetti needles, tactical pasta weapon, game-ready 3D, white background, Fortnite art style",
        "type": "sniper_rifle",
        "damage": 75,
        "fire_rate": 0.75,
        "magazine_size": 6,
        "reload_time": 2.5,
        "rarity_range": ["Uncommon", "Rare", "Epic"],
        "projectile": "spaghetti_needle"
    },
    
    # === PISTOLS ===
    "Tortellini_Tosser": {
        "prompt": "A semi-automatic pistol made from tortellini pasta, compact ring-shaped body, small tortellini barrel, single-hand grip made of folded pasta, fires miniature tortellini rings, sidearm pasta weapon, game-ready 3D model, white background, Fortnite art style",
        "type": "pistol",
        "damage": 24,
        "fire_rate": 6.0,
        "magazine_size": 16,
        "reload_time": 1.3,
        "rarity_range": ["Common", "Uncommon"],
        "projectile": "mini_tortellini"
    },
    "Ravioli_Revolver": {
        "prompt": "A heavy revolver made from ravioli pasta, large ravioli cylinder with 6 chambers, heavy barrel from stacked ravioli, ornate pasta grip with crimped edges, fires explosive ravioli rounds, powerful sidearm, game-ready 3D, white background, Fortnite style",
        "type": "pistol",
        "damage": 55,
        "fire_rate": 1.5,
        "magazine_size": 6,
        "reload_time": 2.0,
        "rarity_range": ["Rare", "Epic", "Legendary"],
        "projectile": "ravioli_heavy"
    },
    
    # === LAUNCHERS ===
    "Lasagna_Launcher": {
        "prompt": "A rocket launcher made from lasagna layers, wide rectangular barrel for launching lasagna sheets, layered body showing cheese and sauce between pasta layers, heavy two-hand weapon, fires explosive lasagna slabs, devastating area damage weapon, game-ready 3D model, white background, Fortnite art style",
        "type": "launcher",
        "damage": 100,
        "fire_rate": 0.5,
        "magazine_size": 1,
        "reload_time": 4.0,
        "rarity_range": ["Rare", "Epic", "Legendary"],
        "projectile": "lasagna_slab",
        "splash_radius": 350,
        "splash_damage": 60
    },
    "Macaroni_Mortar": {
        "prompt": "A grenade launcher made from curved elbow macaroni, curved barrel that arcs shots, rotating cylinder of macaroni grenades, cheese-encrusted body, fires arcing macaroni bombs, tactical area denial weapon, game-ready 3D, white background, Fortnite style",
        "type": "launcher",
        "damage": 70,
        "fire_rate": 1.0,
        "magazine_size": 6,
        "reload_time": 3.5,
        "rarity_range": ["Uncommon", "Rare", "Epic"],
        "projectile": "mac_bomb",
        "splash_radius": 250,
        "splash_damage": 40,
        "arc_trajectory": True
    },
    
    # === MELEE ===
    "Spaghetti_Whip": {
        "prompt": "A melee whip weapon made from a bundle of al dente spaghetti noodles, braided spaghetti handle, flowing spaghetti strands that swing with physics, cracks like a whip, detailed pasta melee weapon, game-ready 3D model, white background, Fortnite art style",
        "type": "melee",
        "damage": 45,
        "fire_rate": 2.0,
        "range": 400,
        "rarity_range": ["Common", "Uncommon", "Rare"],
        "special": "extended_range_melee"
    },
    "Penne_Blade": {
        "prompt": "A short sword made from a sharpened giant penne tube, the tube is sliced diagonally creating a blade edge, ridged surface provides grip texture, pasta hilt guard, fast slashing melee weapon, game-ready 3D model, white background, Fortnite style",
        "type": "melee",
        "damage": 55,
        "fire_rate": 1.5,
        "range": 200,
        "rarity_range": ["Rare", "Epic", "Legendary"],
        "special": "armor_piercing_melee"
    },
    
    # === HARVESTING TOOLS (Pickaxe Equivalent) ===
    "Fork_Twirl": {
        "prompt": "A giant dinner fork as a harvesting tool, long handle with four prongs, spaghetti wrapped around the prongs, gleaming metallic fork with pasta accents, iconic harvesting tool, game-ready 3D model, white background, Fortnite style",
        "type": "harvesting_tool",
        "damage": 20,
        "harvest_rate": 1.0
    },
    "Cheese_Grater": {
        "prompt": "A large cheese grater as a harvesting tool, box grater shape with handle on top, sharp grating surfaces, bits of parmesan stuck to it, menacing yet comedic, game-ready 3D model, white background, Fortnite style",
        "type": "harvesting_tool",
        "damage": 20,
        "harvest_rate": 1.0
    },
    "Rolling_Pin_Rusher": {
        "prompt": "A massive rolling pin as a harvesting tool, wooden pin with flour dust, heavy and powerful looking, pasta dough stuck to it, classic Italian kitchen weapon, game-ready 3D model, white background, Fortnite style",
        "type": "harvesting_tool",
        "damage": 20,
        "harvest_rate": 1.0
    }
}

# ============================================================
# THROWABLES / CONSUMABLES
# ============================================================
THROWABLES = {
    "Marinara_Molotov": {
        "prompt": "A bottle of marinara sauce as a molotov cocktail, glass bottle filled with red sauce, rag stuffed in top that's on fire, dripping sauce, throwable weapon, game-ready 3D, white background, Fortnite style",
        "type": "throwable",
        "damage": 15,
        "tick_damage": 8,
        "duration": 5.0,
        "radius": 200,
        "effect": "fire_dot_area"
    },
    "Meatball_Grenade": {
        "prompt": "A giant meatball as a grenade, perfectly round meatball with a visible pin and safety lever attached, bread crumb texture, sizzling, throwable explosive, game-ready 3D model, white background, Fortnite style",
        "type": "throwable",
        "damage": 75,
        "radius": 300,
        "fuse_time": 2.5,
        "effect": "explosion"
    },
    "Parmesan_Smoke": {
        "prompt": "A block of parmesan cheese that creates a smoke screen when thrown, dense yellow-white cheese block with smoke wisps, tactical utility item, game-ready 3D, white background, Fortnite style",
        "type": "throwable",
        "damage": 0,
        "radius": 500,
        "duration": 8.0,
        "effect": "smoke_screen"
    },
    "Alfredo_Slick": {
        "prompt": "A jar of alfredo sauce that creates a slippery surface when thrown, creamy white sauce in a jar, sloshing, creates movement hazard on ground, game-ready 3D, white background, Fortnite style",
        "type": "throwable",
        "damage": 0,
        "radius": 400,
        "duration": 10.0,
        "effect": "slip_zone"
    },
    "Garlic_Bread_Heal": {
        "prompt": "Golden crispy garlic bread as a healing consumable item, steaming hot, buttery, with visible garlic and herbs, glowing with healing energy, game-ready 3D, white background, Fortnite style",
        "type": "consumable",
        "heal_amount": 50,
        "use_time": 4.0
    },
    "Pasta_Water_Shield": {
        "prompt": "A steaming pot of pasta water as a shield potion, large pot with boiling starchy water, bubbling, grants shield on consumption, game-ready 3D, white background, Fortnite style",
        "type": "consumable",
        "shield_amount": 50,
        "use_time": 5.0
    },
    "Olive_Oil_Elixir": {
        "prompt": "An ornate bottle of extra virgin olive oil as a legendary consumable, golden green oil in decorative bottle, glowing, full heal and shield, game-ready 3D, white background, Fortnite style",
        "type": "consumable",
        "heal_amount": 100,
        "shield_amount": 100,
        "use_time": 10.0,
        "rarity": "Legendary"
    }
}

# ============================================================
# ENVIRONMENT PROPS
# ============================================================
ENVIRONMENT_PROPS = {
    # Trees per biome (pasta-ified)
    "Tree_Broccoli_Standard": {
        "prompt": "A large broccoli tree for a game environment, thick brown trunk, large bushy broccoli head canopy, stylized realistic Fortnite art style, game-ready environment prop, white background",
        "polycount": 5000
    },
    "Tree_Broccoli_Tall": {
        "prompt": "A very tall thin broccoli tree, elongated trunk, smaller broccoli canopy at top, stylized Fortnite environment prop, game-ready, white background",
        "polycount": 4000
    },
    "Bush_Basil": {
        "prompt": "A large basil bush for hiding in, dense basil leaves forming a round bush, aromatic green plant, game environment prop, Fortnite style, game-ready, white background",
        "polycount": 3000
    },
    "Rock_Parmesan": {
        "prompt": "A large rock formation made of parmesan cheese, crumbly yellow-white cheese texture, crystalline grain structure, multiple sizes grouped, game environment prop, Fortnite style, white background",
        "polycount": 3000
    },
    "Crate_Tomato": {
        "prompt": "A wooden crate filled with tomatoes, breakable supply crate, tomatoes spilling out, wooden slats, loot container, game prop, Fortnite style, white background",
        "polycount": 2000
    },
    "Barrel_Wine": {
        "prompt": "An Italian wine barrel, wooden barrel with metal bands, aged wood texture, can be used as cover, destructible, game environment prop, Fortnite style, white background",
        "polycount": 2000
    },
    "Chest_Pasta_Box": {
        "prompt": "A treasure chest made from a giant pasta box, cardboard box shaped like a chest with pasta branding, glowing slightly, loot container, game prop, Fortnite style, white background",
        "polycount": 3000
    },
    "Vehicle_Meatball_Tank": {
        "prompt": "A small tank vehicle where the turret is a giant meatball, tank body is a bread bowl, treads are made of lasagna noodles, comedic but functional military vehicle, game-ready vehicle model, Fortnite style, white background",
        "polycount": 15000
    },
    "Vehicle_Spaghetti_ATV": {
        "prompt": "An ATV/quad bike made from bundled spaghetti frame, wheels are round pasta shapes, engine is a pot of boiling water, fast nimble vehicle, game-ready vehicle model, Fortnite style, white background",
        "polycount": 12000
    },
    "Glider_Ravioli_Chute": {
        "prompt": "A parachute/glider made from a giant ravioli, the puffed ravioli acts as the chute canopy with crimped edges catching air, rope handles hanging below, game-ready glider, Fortnite style, white background",
        "polycount": 5000
    },
    "Glider_Farfalle_Wings": {
        "prompt": "A hang glider made from a giant farfalle bowtie pasta, the wing-like shape provides lift, player hangs beneath the center pinch, beautiful flowing pasta glider, game-ready, Fortnite style, white background",
        "polycount": 5000
    }
}

# ============================================================
# BUILDING PIECES (Fortnite-style building system)
# ============================================================
BUILDING_PIECES = {
    "Wall_Lasagna": {
        "prompt": "A buildable wall segment made of stacked lasagna layers, layered pasta sheets with cheese mortar between, fortification wall, game-ready building piece, Fortnite style, white background",
        "polycount": 1500,
        "health": 150,
        "build_time": 1.5
    },
    "Floor_Lasagna": {
        "prompt": "A buildable floor/ceiling platform made of a large flat lasagna sheet, rigid pasta platform, can be walked on, game-ready building piece, Fortnite style, white background",
        "polycount": 1000,
        "health": 150,
        "build_time": 1.5
    },
    "Ramp_Lasagna": {
        "prompt": "A buildable ramp/staircase made of angled lasagna sheets, ridged surface for traction, buildable incline, game-ready building piece, Fortnite style, white background",
        "polycount": 1200,
        "health": 140,
        "build_time": 1.5
    },
    "Roof_Lasagna": {
        "prompt": "A buildable roof piece shaped like lasagna sheets angled to a peak, provides overhead cover, buildable roof segment, game-ready, Fortnite style, white background",
        "polycount": 1200,
        "health": 130,
        "build_time": 1.5
    }
}

def run_full_asset_generation():
    """
    Execute the complete asset generation pipeline for all game assets.
    """
    print("=" * 60)
    print("PASTA WARFARE — FULL ASSET GENERATION PIPELINE")
    print("=" * 60)
    
    # Phase 1: Generate all concept art
    print("\n[PHASE 1] Generating concept art...")
    for name, config in {**CHARACTERS, **WEAPONS, **THROWABLES, 
                          **ENVIRONMENT_PROPS, **BUILDING_PIECES}.items():
        concept_dir = f"Content/_ConceptArt/{name}/"
        generate_concept_art(
            prompt=config["prompt"],
            output_path=f"{concept_dir}{name}.png",
            style_preset="digital-art"
        )
        print(f"  ✓ Concept art: {name}")
    
    # Phase 2: Generate 3D models
    print("\n[PHASE 2] Generating 3D models...")
    for name, config in {**CHARACTERS, **WEAPONS, **THROWABLES,
                          **ENVIRONMENT_PROPS, **BUILDING_PIECES}.items():
        raw_dir = f"Content/_Raw3D/{name}/"
        
        # Try Meshy first
        try:
            result = generate_3d_model(
                prompt=config["prompt"],
                art_style="realistic",
                target_polycount=config.get("polycount", 15000)
            )
            refined = refine_model(result["result"])
            download_model(refined["model_urls"], raw_dir, name)
            print(f"  ✓ 3D Model (Meshy): {name}")
        except Exception as e:
            print(f"  ⚠ Meshy failed for {name}: {e}")
            # Fallback to Tripo
            try:
                result = generate_3d_tripo(
                    prompt=config["prompt"],
                    face_limit=config.get("polycount", 15000)
                )
                download_tripo_model(result, raw_dir, name)
                print(f"  ✓ 3D Model (Tripo fallback): {name}")
            except Exception as e2:
                print(f"  ⚠ Tripo also failed for {name}: {e2}")
                # Final fallback: image-to-3D via concept art
                concept_path = f"Content/_ConceptArt/{name}/{name}_v0.png"
                if os.path.exists(concept_path):
                    image_to_3d_stability(concept_path, f"{raw_dir}{name}.glb")
                    print(f"  ✓ 3D Model (Stability img2mesh fallback): {name}")
    
    # Phase 3: Post-process all meshes in Blender
    print("\n[PHASE 3] Post-processing meshes...")
    for name, config in {**CHARACTERS, **WEAPONS, **THROWABLES,
                          **ENVIRONMENT_PROPS, **BUILDING_PIECES}.items():
        raw_path = f"Content/_Raw3D/{name}/{name}.glb"
        processed_path = f"Content/_Processed/{name}/{name}.fbx"
        
        if os.path.exists(raw_path):
            blender_config = {
                "target_height_cm": config.get("height_cm", 100),
                "target_polycount": config.get("polycount", 15000),
                "generate_lods": True,
                "lod_ratios": [1.0, 0.5, 0.25, 0.1],
                "smooth_shading": True,
                "smooth_angle": 60,
                "use_remesh": True,
                "voxel_size": 0.008
            }
            config_path = f"/tmp/{name}_config.json"
            with open(config_path, 'w') as f:
                json.dump(blender_config, f)
            
            subprocess.run([
                "blender", "--background", "--python",
                "Scripts/AssetGen/blender_postprocess.py",
                "--", raw_path, processed_path, config_path
            ], check=True)
            print(f"  ✓ Post-processed: {name}")
    
    # Phase 4: Generate textures
    print("\n[PHASE 4] Generating PBR textures...")
    for name, config in {**CHARACTERS, **WEAPONS, **THROWABLES,
                          **ENVIRONMENT_PROPS, **BUILDING_PIECES}.items():
        mesh_path = f"Content/_Processed/{name}/{name}.fbx"
        tex_dir = f"Content/_Textures/{name}/"
        
        if os.path.exists(mesh_path):
            try:
                tex_urls = generate_textures_meshy(
                    mesh_path, config["prompt"], resolution=2048
                )
                # Download and organize textures
                for tex_type, url in tex_urls.items():
                    if url:
                        resp = requests.get(url)
                        os.makedirs(tex_dir, exist_ok=True)
                        with open(f"{tex_dir}{name}_{tex_type}.png", 'wb') as f:
                            f.write(resp.content)
                print(f"  ✓ Textures (Meshy): {name}")
            except:
                # Fallback: generate from albedo
                albedo_path = f"Content/_ConceptArt/{name}/{name}_v0.png"
                if os.path.exists(albedo_path):
                    generate_pbr_from_albedo(albedo_path, tex_dir, name)
                    print(f"  ✓ Textures (Generated from albedo): {name}")
    
    # Phase 5: Rig characters
    print("\n[PHASE 5] Auto-rigging characters...")
    for name, config in CHARACTERS.items():
        mesh_path = f"Content/_Processed/{name}/{name}.fbx"
        rigged_path = f"Content/Characters/{name}/{name}_Rigged.fbx"
        
        if os.path.exists(mesh_path):
            os.makedirs(os.path.dirname(rigged_path), exist_ok=True)
            rig_with_rigify(mesh_path, rigged_path)
            print(f"  ✓ Rigged: {name}")
    
    print("\n" + "=" * 60)
    print("ASSET GENERATION COMPLETE")
    print("=" * 60)
    print("\nNext: Import all assets from Content/ into Unreal Engine project")
    print("Then: Set up materials, Animation Blueprints, and integrate into gameplay systems")

if __name__ == "__main__":
    run_full_asset_generation()
```

### 2.8 — UE5 Auto-Import & Material Setup Script

```python
# Scripts/AssetGen/ue5_import_setup.py
"""
Unreal Engine Python script to auto-import all processed assets
and set up materials, LODs, and physics assets.
Run inside UE5 Editor via Python console or Editor Utility Widget.
"""

import unreal
import os
import json

CONTENT_DIR = "/Game/"
PROCESSED_DIR = "C:/PastaWarfare/Content/_Processed/"
TEXTURE_DIR = "C:/PastaWarfare/Content/_Textures/"

def import_all_assets():
    """Import all processed FBX files and textures into UE5."""
    
    asset_tools = unreal.AssetToolsHelpers.get_asset_tools()
    
    # Import settings for skeletal meshes (characters)
    skeletal_import_options = unreal.FbxImportUI()
    skeletal_import_options.set_editor_property('import_mesh', True)
    skeletal_import_options.set_editor_property('import_textures', True)
    skeletal_import_options.set_editor_property('import_materials', True)
    skeletal_import_options.set_editor_property('import_as_skeletal', True)
    skeletal_import_options.set_editor_property('automated_import_should_detect_type', False)
    skeletal_import_options.skeletal_mesh_import_data.set_editor_property('import_morph_targets', True)
    skeletal_import_options.skeletal_mesh_import_data.set_editor_property('update_skeleton_reference_pose', True)
    
    # Import settings for static meshes (weapons, props)
    static_import_options = unreal.FbxImportUI()
    static_import_options.set_editor_property('import_mesh', True)
    static_import_options.set_editor_property('import_textures', True)
    static_import_options.set_editor_property('import_materials', True)
    static_import_options.set_editor_property('import_as_skeletal', False)
    static_import_options.static_mesh_import_data.set_editor_property('combine_meshes', True)
    static_import_options.static_mesh_import_data.set_editor_property('generate_lightmap_u_vs', True)
    static_import_options.static_mesh_import_data.set_editor_property('auto_generate_collision', True)
    
    # Walk processed directory and import
    for root, dirs, files in os.walk(PROCESSED_DIR):
        for file in files:
            if file.endswith('.fbx'):
                filepath = os.path.join(root, file)
                asset_name = os.path.splitext(file)[0]
                
                # Determine if character or static mesh
                is_character = "_Rigged" in file or asset_name in [
                    "Spaghetti", "Penne", "Farfalle", "Rigatoni", "Fusilli",
                    "Lasagna", "Ravioli", "Macaroni", "Linguine", "Tortellini", "Orzo"
                ]
                
                if is_character:
                    dest = f"/Game/Characters/{asset_name}/"
                    options = skeletal_import_options
                else:
                    dest = f"/Game/Weapons/{asset_name}/" if asset_name in WEAPONS else f"/Game/Props/{asset_name}/"
                    options = static_import_options
                
                # Create import task
                task = unreal.AssetImportTask()
                task.set_editor_property('filename', filepath)
                task.set_editor_property('destination_path', dest)
                task.set_editor_property('automated', True)
                task.set_editor_property('replace_existing', True)
                task.set_editor_property('save', True)
                task.set_editor_property('options', options)
                
                asset_tools.import_asset_tasks([task])
                unreal.log(f"Imported: {asset_name} → {dest}")

def create_pasta_master_material():
    """
    Create the master material for all pasta characters and weapons.
    Supports: PBR textures, subsurface scattering (pasta translucency),
    dynamic wetness, damage states, sauce overlay.
    """
    
    asset_tools = unreal.AssetToolsHelpers.get_asset_tools()
    mat_factory = unreal.MaterialFactoryNew()
    
    # Create master material
    master_mat = asset_tools.create_asset(
        "M_Pasta_Master", "/Game/Materials/",
        unreal.Material, mat_factory
    )
    
    # The material will be configured in the Material Editor with these nodes:
    # This is documented as the node graph to recreate:
    material_config = {
        "name": "M_Pasta_Master",
        "shading_model": "Subsurface",  # Pasta has slight translucency
        "blend_mode": "Opaque",
        "two_sided": False,
        "parameters": {
            # Texture Parameters
            "T_Albedo": {"type": "Texture2D", "group": "Textures"},
            "T_Normal": {"type": "Texture2D", "group": "Textures"},
            "T_Roughness": {"type": "Texture2D", "group": "Textures"},
            "T_Metallic": {"type": "Texture2D", "group": "Textures"},
            "T_AO": {"type": "Texture2D", "group": "Textures"},
            "T_Height": {"type": "Texture2D", "group": "Textures"},
            "T_SubsurfaceColor": {"type": "Texture2D", "group": "Textures"},
            
            # Scalar Parameters
            "PastaBaseColor_Tint": {"type": "LinearColor", "default": [0.95, 0.85, 0.6, 1.0], "group": "Color"},
            "Roughness_Min": {"type": "Scalar", "default": 0.3, "group": "Surface"},
            "Roughness_Max": {"type": "Scalar", "default": 0.8, "group": "Surface"},
            "Normal_Intensity": {"type": "Scalar", "default": 1.0, "group": "Surface"},
            "Subsurface_Intensity": {"type": "Scalar", "default": 0.15, "group": "Subsurface"},
            "Subsurface_Color": {"type": "LinearColor", "default": [1.0, 0.9, 0.7, 1.0], "group": "Subsurface"},
            
            # Dynamic State Parameters
            "Wetness": {"type": "Scalar", "default": 0.0, "min": 0.0, "max": 1.0, "group": "DynamicState"},
            "DamageAmount": {"type": "Scalar", "default": 0.0, "min": 0.0, "max": 1.0, "group": "DynamicState"},
            "SauceOverlay": {"type": "Scalar", "default": 0.0, "min": 0.0, "max": 1.0, "group": "DynamicState"},
            "SauceColor": {"type": "LinearColor", "default": [0.7, 0.1, 0.05, 1.0], "group": "DynamicState"},
            "CookLevel": {"type": "Scalar", "default": 0.5, "min": 0.0, "max": 1.0, "group": "DynamicState"},
        },
        "node_graph_description": """
        BASE COLOR:
        T_Albedo → Multiply(PastaBaseColor_Tint) → Lerp(SauceColor, SauceOverlay) → 
        Lerp(DarkerTint, DamageAmount) → Lerp(WetDarken, Wetness) → Base Color
        
        CookLevel controls color shift: 0=raw(pale) → 0.5=al_dente(golden) → 1.0=overcooked(brown)
        
        NORMAL:
        T_Normal → FlattenNormal(Normal_Intensity) → Normal
        
        ROUGHNESS:
        T_Roughness → Lerp(Roughness_Min, Roughness_Max) → Lerp(0.1, Wetness) → Roughness
        (Wet pasta is shinier)
        
        METALLIC:
        T_Metallic → Metallic (near zero for pasta)
        
        SUBSURFACE:
        Subsurface_Color * Subsurface_Intensity → Subsurface Color
        (Pasta is slightly translucent when backlit)
        
        AMBIENT OCCLUSION:
        T_AO → AO
        
        WORLD POSITION OFFSET:
        T_Height * Wind_Vector * Vertex_Color.R → WPO
        (For noodle strand animation on spaghetti/linguine characters)
        """
    }
    
    unreal.log(f"Master Material created: {master_mat.get_path_name()}")
    return master_mat

def setup_all_materials():
    """Create material instances for every character and weapon from the master material."""
    
    asset_tools = unreal.AssetToolsHelpers.get_asset_tools()
    mi_factory = unreal.MaterialInstanceConstantFactoryNew()
    
    master_mat_path = "/Game/Materials/M_Pasta_Master"
    master_mat = unreal.EditorAssetLibrary.load_asset(master_mat_path)
    
    # Character-specific tints and properties
    character_material_configs = {
        "Spaghetti": {"tint": [0.95, 0.88, 0.6], "roughness_min": 0.25, "roughness_max": 0.65, "subsurface": 0.2, "cook_level": 0.5},
        "Penne": {"tint": [0.92, 0.84, 0.58], "roughness_min": 0.4, "roughness_max": 0.85, "subsurface": 0.1, "cook_level": 0.5},
        "Farfalle": {"tint": [0.98, 0.92, 0.7], "roughness_min": 0.3, "roughness_max": 0.7, "subsurface": 0.15, "cook_level": 0.45},
        "Rigatoni": {"tint": [0.88, 0.80, 0.55], "roughness_min": 0.45, "roughness_max": 0.9, "subsurface": 0.08, "cook_level": 0.55},
        "Fusilli": {"tint": [0.93, 0.86, 0.62], "roughness_min": 0.35, "roughness_max": 0.75, "subsurface": 0.12, "cook_level": 0.5},
        "Lasagna": {"tint": [0.85, 0.75, 0.5], "roughness_min": 0.5, "roughness_max": 0.95, "subsurface": 0.05, "cook_level": 0.6},
        "Ravioli": {"tint": [0.96, 0.90, 0.68], "roughness_min": 0.3, "roughness_max": 0.7, "subsurface": 0.18, "cook_level": 0.48},
        "Macaroni": {"tint": [0.94, 0.87, 0.63], "roughness_min": 0.35, "roughness_max": 0.8, "subsurface": 0.12, "cook_level": 0.5},
        "Linguine": {"tint": [0.97, 0.91, 0.65], "roughness_min": 0.25, "roughness_max": 0.6, "subsurface": 0.22, "cook_level": 0.5},
        "Tortellini": {"tint": [0.90, 0.82, 0.58], "roughness_min": 0.35, "roughness_max": 0.75, "subsurface": 0.14, "cook_level": 0.52},
        "Orzo": {"tint": [0.93, 0.85, 0.6], "roughness_min": 0.4, "roughness_max": 0.85, "subsurface": 0.08, "cook_level": 0.5},
    }
    
    for char_name, props in character_material_configs.items():
        mi_path = f"/Game/Characters/{char_name}/MI_{char_name}"
        mi = asset_tools.create_asset(
            f"MI_{char_name}", f"/Game/Characters/{char_name}/",
            unreal.MaterialInstanceConstant, mi_factory
        )
        mi.set_editor_property('parent', master_mat)
        
        # Set texture parameters
        tex_dir = f"/Game/Characters/{char_name}/Textures/"
        # Parameters would be set here using mi.set_texture_parameter_value_editor_only() etc.
        
        unreal.log(f"Material Instance created: MI_{char_name}")

# Execute
if __name__ == "__main__":
    import_all_assets()
    create_pasta_master_material()
    setup_all_materials()
```

---

## 3. CHARACTER SYSTEM — PASTA FIGHTERS

### 3.1 — Base Character Class (C++)

```cpp
// Source/PastaWarfare/Characters/PastaCharacterBase.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Character.h"
#include "AbilitySystemInterface.h"
#include "GameplayAbilitySpec.h"
#include "PastaWarfare/Abilities/PastaAbilitySystemComponent.h"
#include "PastaWarfare/Abilities/PastaAttributeSet.h"
#include "PastaCharacterBase.generated.h"

UENUM(BlueprintType)
enum class EPastaType : uint8
{
    Spaghetti,
    Penne,
    Farfalle,
    Rigatoni,
    Fusilli,
    Lasagna,
    Ravioli,
    Macaroni,
    Linguine,
    Tortellini,
    Orzo
};

UENUM(BlueprintType)
enum class ECharacterState : uint8
{
    Alive,
    DBNO,        // Down But Not Out
    Eliminated,
    Spectating,
    PreGame,
    Skydiving,
    Gliding
};

USTRUCT(BlueprintType)
struct FPastaCharacterData : public FTableRowBase
{
    GENERATED_BODY()

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    EPastaType PastaType;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    FText DisplayName;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    FText Description;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    TSoftObjectPtr CharacterMesh;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    TSoftClassPtr AnimBlueprintClass;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float BaseHealth = 100.0f;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float BaseShield = 0.0f;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float BaseSpeed = 600.0f;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float SprintMultiplier = 1.5f;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float CrouchSpeedMultiplier = 0.5f;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    TArray<TSubclassOf> DefaultAbilities;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    TSubclassOf PassiveAbilityEffect;

    // Per-pasta hit reactions
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    USoundBase* HitSound;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    UNiagaraSystem* HitVFX;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    USoundBase* EliminationSound;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    UNiagaraSystem* EliminationVFX;
};

UCLASS()
class PASTAWARFARE_API APastaCharacterBase : public ACharacter, public IAbilitySystemInterface
{
    GENERATED_BODY()

public:
    APastaCharacterBase();

    // IAbilitySystemInterface
    virtual UAbilitySystemComponent* GetAbilitySystemComponent() const override;

    // Core getters
    UFUNCTION(BlueprintPure, Category = "Pasta|Character")
    EPastaType GetPastaType() const { return PastaType; }

    UFUNCTION(BlueprintPure, Category = "Pasta|Character")
    ECharacterState GetCharacterState() const { return CharacterState; }

    UFUNCTION(BlueprintPure, Category = "Pasta|Character")
    float GetCurrentHealth() const;

    UFUNCTION(BlueprintPure, Category = "Pasta|Character")
    float GetCurrentShield() const;

    UFUNCTION(BlueprintPure, Category = "Pasta|Character")
    bool IsAlive() const { return CharacterState == ECharacterState::Alive; }

    // State changes
    UFUNCTION(Server, Reliable)
    void ServerEnterDBNO();

    UFUNCTION(Server, Reliable)
    void ServerRevive(APastaCharacterBase* Reviver);

    UFUNCTION(Server, Reliable)
    void ServerEliminate(APastaCharacterBase* Eliminator);

    // Inventory
    UFUNCTION(BlueprintCallable, Category = "Pasta|Inventory")
    void EquipWeapon(int32 SlotIndex);

    UFUNCTION(BlueprintCallable, Category = "Pasta|Inventory")
    bool PickupItem(class APastaPickup* Pickup);

    UFUNCTION(BlueprintCallable, Category = "Pasta|Inventory")
    void DropItem(int32 SlotIndex);

    // Building System
    UFUNCTION(BlueprintCallable, Category = "Pasta|Building")
    void StartBuilding(EBuildPieceType PieceType);

    UFUNCTION(BlueprintCallable, Category = "Pasta|Building")
    void PlaceBuildPiece();

    UFUNCTION(BlueprintCallable, Category = "Pasta|Building")
    void RotateBuildPiece(float Direction);

    // Emotes
    UFUNCTION(Server, Reliable, BlueprintCallable, Category = "Pasta|Emote")
    void ServerPlayEmote(int32 EmoteIndex);

protected:
    virtual void BeginPlay() override;
    virtual void Tick(float DeltaTime) override;
    virtual void SetupPlayerInputComponent(class UInputComponent* InputComp) override;
    virtual void GetLifetimeReplicatedProps(TArray& OutLifetimeProps) const override;
    virtual void PossessedBy(AController* NewController) override;
    virtual void OnRep_PlayerState() override;
    virtual float TakeDamage(float Damage, FDamageEvent const& DamageEvent, 
                              AController* EventInstigator, AActor* DamageCauser) override;

    // Initialize abilities when possessed
    void InitializeAbilities();
    void ApplyPassiveAbility();

    // Components
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    UPastaAbilitySystemComponent* AbilitySystemComponent;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    class UPastaAttributeSet* AttributeSet;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    class UPastaInventoryComponent* InventoryComponent;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    class UPastaBuildingComponent* BuildingComponent;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    class UPastaInteractionComponent* InteractionComponent;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    class USpringArmComponent* CameraBoom;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    class UCameraComponent* FollowCamera;

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    class UNiagaraComponent* PastaVFXComponent;  // For ongoing pasta-specific effects

    // Character Data
    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Pasta|Config")
    FPastaCharacterData CharacterData;

    UPROPERTY(Replicated, BlueprintReadOnly, Category = "Pasta|State")
    EPastaType PastaType;

    UPROPERTY(ReplicatedUsing = OnRep_CharacterState, BlueprintReadOnly, Category = "Pasta|State")
    ECharacterState CharacterState;

    UFUNCTION()
    void OnRep_CharacterState();

    // DBNO
    UPROPERTY(EditDefaultsOnly, Category = "Pasta|DBNO")
    float DBNODuration = 30.0f;

    UPROPERTY(EditDefaultsOnly, Category = "Pasta|DBNO")
    float ReviveTime = 5.0f;

    FTimerHandle DBNOTimerHandle;

    // Input (Enhanced Input)
    UPROPERTY(EditDefaultsOnly, Category = "Input")
    class UInputMappingContext* DefaultMappingContext;

    UPROPERTY(EditDefaultsOnly, Category = "Input")
    class UInputAction* MoveAction;

    UPROPERTY(EditDefaultsOnly, Category = "Input")
    class UInputAction* LookAction;

    UPROPERTY(EditDefaultsOnly, Category = "Input")
    class UInputAction* JumpAction;

    UPROPERTY(EditDefaultsOnly, Category = "Input")
    class UInputAction* SprintAction;

    UPROPERTY(EditDefaultsOnly, Category = "Input")
    class UInputAction* CrouchAction;

    UPROPERTY(EditDefaultsOnly, Category = "Input")
    class UInputAction* FireAction;

    UPROPERTY(EditDefaultsOnly, Category = "Input")
    class UInputAction* AimAction;

    UPROPERTY(EditDefaultsOnly, Category = "Input")
    class UInputAction* ReloadAction;

    UPROPERTY(EditDefaultsOnly, Category = "Input")
    class UInputAction* InteractAction;

    UPROPERTY(EditDefaultsOnly, Category = "Input")
    class UInputAction* BuildAction;

    UPROPERTY(EditDefaultsOnly, Category = "Input")
    class UInputAction* InventorySlotActions[6]; // 5 weapon slots + harvesting tool

    // Input handlers
    void Move(const struct FInputActionValue& Value);
    void Look(const struct FInputActionValue& Value);
    void StartSprint();
    void StopSprint();
    void ToggleCrouch();
    void StartFire();
    void StopFire();
    void StartAim();
    void StopAim();
    void Reload();
    void Interact();
    void ToggleBuild();
    void SwitchWeapon(int32 SlotIndex);

private:
    bool bIsSprinting = false;
    bool bIsAiming = false;
    bool bIsBuildMode = false;
};
```

### 3.2 — Attribute Set (GAS)

```cpp
// Source/PastaWarfare/Abilities/PastaAttributeSet.h
#pragma once

#include "CoreMinimal.h"
#include "AttributeSet.h"
#include "AbilitySystemComponent.h"
#include "PastaAttributeSet.generated.h"

#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
    GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
    GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
    GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
    GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)

UCLASS()
class PASTAWARFARE_API UPastaAttributeSet : public UAttributeSet
{
    GENERATED_BODY()

public:
    UPastaAttributeSet();

    virtual void GetLifetimeReplicatedProps(TArray& OutLifetimeProps) const override;
    virtual void PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue) override;
    virtual void PostGameplayEffectExecute(const FGameplayEffectModCallbackData& Data) override;

    // Health
    UPROPERTY(BlueprintReadOnly, ReplicatedUsing = OnRep_Health, Category = "Pasta|Attributes")
    FGameplayAttributeData Health;
    ATTRIBUTE_ACCESSORS(UPastaAttributeSet, Health)

    UPROPERTY(BlueprintReadOnly, ReplicatedUsing = OnRep_MaxHealth, Category = "Pasta|Attributes")
    FGameplayAttributeData MaxHealth;
    ATTRIBUTE_ACCESSORS(UPastaAttributeSet, MaxHealth)

    // Shield
    UPROPERTY(BlueprintReadOnly, ReplicatedUsing = OnRep_Shield, Category = "Pasta|Attributes")
    FGameplayAttributeData Shield;
    ATTRIBUTE_ACCESSORS(UPastaAttributeSet, Shield)

    UPROPERTY(BlueprintReadOnly, ReplicatedUsing = OnRep_MaxShield, Category = "Pasta|Attributes")
    FGameplayAttributeData MaxShield;
    ATTRIBUTE_ACCESSORS(UPastaAttributeSet, MaxShield)

    // Resources
    UPROPERTY(BlueprintReadOnly, ReplicatedUsing = OnRep_BuildMaterials, Category = "Pasta|Attributes")
    FGameplayAttributeData BuildMaterials;  // Flour — single building resource
    ATTRIBUTE_ACCESSORS(UPastaAttributeSet, BuildMaterials)

    UPROPERTY(BlueprintReadOnly, Category = "Pasta|Attributes")
    FGameplayAttributeData MaxBuildMaterials;
    ATTRIBUTE_ACCESSORS(UPastaAttributeSet, MaxBuildMaterials)

    // Movement modifiers
    UPROPERTY(BlueprintReadOnly, ReplicatedUsing = OnRep_MoveSpeedMultiplier, Category = "Pasta|Attributes")
    FGameplayAttributeData MoveSpeedMultiplier;
    ATTRIBUTE_ACCESSORS(UPastaAttributeSet, MoveSpeedMultiplier)

    // Meta attribute for damage calculation (not replicated)
    UPROPERTY(BlueprintReadOnly, Category = "Pasta|Attributes")
    FGameplayAttributeData IncomingDamage;
    ATTRIBUTE_ACCESSORS(UPastaAttributeSet, IncomingDamage)

    UPROPERTY(BlueprintReadOnly, Category = "Pasta|Attributes")
    FGameplayAttributeData IncomingHealing;
    ATTRIBUTE_ACCESSORS(UPastaAttributeSet, IncomingHealing)

protected:
    UFUNCTION()
    void OnRep_Health(const FGameplayAttributeData& OldHealth);
    UFUNCTION()
    void OnRep_MaxHealth(const FGameplayAttributeData& OldMaxHealth);
    UFUNCTION()
    void OnRep_Shield(const FGameplayAttributeData& OldShield);
    UFUNCTION()
    void OnRep_MaxShield(const FGameplayAttributeData& OldMaxShield);
    UFUNCTION()
    void OnRep_BuildMaterials(const FGameplayAttributeData& OldBuildMaterials);
    UFUNCTION()
    void OnRep_MoveSpeedMultiplier(const FGameplayAttributeData& OldValue);
};
```

### 3.3 — Inventory Component

```cpp
// Source/PastaWarfare/Characters/PastaInventoryComponent.h
#pragma once

#include "CoreMinimal.h"
#include "Components/ActorComponent.h"
#include "PastaWarfare/Weapons/PastaWeaponBase.h"
#include "PastaInventoryComponent.generated.h"

DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(FOnWeaponChanged, int32, SlotIndex, APastaWeaponBase*, NewWeapon);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(FOnItemPickedUp, int32, SlotIndex, APastaWeaponBase*, Weapon);

UCLASS(ClassGroup=(Custom), meta=(BlueprintSpawnableComponent))
class PASTAWARFARE_API UPastaInventoryComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    UPastaInventoryComponent();

    static constexpr int32 MAX_WEAPON_SLOTS = 5;
    static constexpr int32 HARVESTING_TOOL_SLOT = 0;

    UFUNCTION(BlueprintCallable, Category = "Inventory")
    bool AddWeapon(APastaWeaponBase* Weapon);

    UFUNCTION(BlueprintCallable, Category = "Inventory")
    void RemoveWeapon(int32 SlotIndex);

    UFUNCTION(BlueprintCallable, Category = "Inventory")
    void EquipSlot(int32 SlotIndex);

    UFUNCTION(BlueprintPure, Category = "Inventory")
    APastaWeaponBase* GetEquippedWeapon() const { return EquippedWeapon; }

    UFUNCTION(BlueprintPure, Category = "Inventory")
    APastaWeaponBase* GetWeaponInSlot(int32 SlotIndex) const;

    UFUNCTION(BlueprintPure, Category = "Inventory")
    int32 GetCurrentSlot() const { return CurrentSlotIndex; }

    UFUNCTION(BlueprintPure, Category = "Inventory")
    TArray GetAllWeapons() const;

    UPROPERTY(BlueprintAssignable, Category = "Inventory")
    FOnWeaponChanged OnWeaponChanged;

    UPROPERTY(BlueprintAssignable, Category = "Inventory")
    FOnItemPickedUp OnItemPickedUp;

protected:
    virtual void GetLifetimeReplicatedProps(TArray& OutLifetimeProps) const override;

    UPROPERTY(Replicated)
    TArray WeaponSlots;

    UPROPERTY(ReplicatedUsing = OnRep_EquippedWeapon)
    APastaWeaponBase* EquippedWeapon;

    UPROPERTY(Replicated)
    int32 CurrentSlotIndex;

    UFUNCTION()
    void OnRep_EquippedWeapon();

    UFUNCTION(Server, Reliable)
    void ServerEquipSlot(int32 SlotIndex);
};
```

---

## 4. WEAPON SYSTEM — PASTA ARSENAL

### 4.1 — Base Weapon Class

```cpp
// Source/PastaWarfare/Weapons/PastaWeaponBase.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "PastaWeaponBase.generated.h"

UENUM(BlueprintType)
enum class EWeaponType : uint8
{
    AssaultRifle,
    Shotgun,
    SMG,
    SniperRifle,
    Pistol,
    Launcher,
    Melee,
    HarvestingTool,
    Throwable,
    Consumable
};

UENUM(BlueprintType)
enum class EWeaponRarity : uint8
{
    Common,      // Gray
    Uncommon,    // Green
    Rare,        // Blue
    Epic,        // Purple
    Legendary,   // Gold
    Mythic       // Red/Prismatic
};

UENUM(BlueprintType)
enum class EWeaponState : uint8
{
    Idle,
    Firing,
    Reloading,
    Equipping,
    Unequipping
};

USTRUCT(BlueprintType)
struct FWeaponStats
{
    GENERATED_BODY()

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float BaseDamage = 20.0f;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float HeadshotMultiplier = 2.0f;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float FireRate = 5.0f;  // Rounds per second

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    int32 MagazineSize = 30;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float ReloadTime = 2.0f;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float EquipTime = 0.5f;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float Range = 10000.0f;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float SpreadBase = 1.0f;  // Degrees

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float SpreadAim = 0.3f;   // ADS spread reduction

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float SpreadBloom = 0.15f; // Per-shot bloom increase

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float SpreadRecovery = 3.0f; // Bloom recovery rate

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float DamageFalloffStart = 3000.0f;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float DamageFalloffEnd = 7500.0f;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly)
    float DamageFalloffMultiplier = 0.5f;

    // Per-rarity damage multipliers
    float GetDamageForRarity(EWeaponRarity Rarity) const
    {
        static const float RarityMultipliers[] = { 1.0f, 1.05f, 1.10f, 1.15f, 1.20f, 1.30f };
        return BaseDamage * RarityMultipliers[static_cast(Rarity)];
    }
};

UCLASS(Abstract)
class PASTAWARFARE_API APastaWeaponBase : public AActor
{
    GENERATED_BODY()

public:
    APastaWeaponBase();

    // Core weapon interface
    UFUNCTION(BlueprintCallable, Category = "Weapon")
    virtual void StartFire();

    UFUNCTION(BlueprintCallable, Category = "Weapon")
    virtual void StopFire();

    UFUNCTION(BlueprintCallable, Category = "Weapon")
    virtual void StartReload();

    UFUNCTION(BlueprintCallable, Category = "Weapon")
    virtual void Equip(class APastaCharacterBase* NewOwner);

    UFUNCTION(BlueprintCallable, Category = "Weapon")
    virtual void Unequip();

    UFUNCTION(BlueprintCallable, Category = "Weapon")
    virtual void StartAim();

    UFUNCTION(BlueprintCallable, Category = "Weapon")
    virtual void StopAim();

    // Getters
    UFUNCTION(BlueprintPure, Category = "Weapon")
    EWeaponType GetWeaponType() const { return WeaponType; }

    UFUNCTION(BlueprintPure, Category = "Weapon")
    EWeaponRarity GetRarity() const { return Rarity; }

    UFUNCTION(BlueprintPure, Category = "Weapon")
    int32 GetCurrentAmmo() const { return CurrentAmmo; }

    UFUNCTION(BlueprintPure, Category = "Weapon")
    int32 GetReserveAmmo() const { return ReserveAmmo; }

    UFUNCTION(BlueprintPure, Category = "Weapon")
    const FWeaponStats& GetWeaponStats() const { return WeaponStats; }

    UFUNCTION(BlueprintPure, Category = "Weapon")
    bool CanFire() const;

    UFUNCTION(BlueprintPure, Category = "Weapon")
    float GetCurrentSpread() const;

protected:
    virtual void BeginPlay() override;
    virtual void Tick(float DeltaTime) override;
    virtual void GetLifetimeReplicatedProps(TArray& OutLifetimeProps) const override;

    // Fire implementation — override in subclasses for hitscan vs projectile
    UFUNCTION(Server, Reliable)
    virtual void ServerFire(FVector Origin, FVector Direction);

    virtual void LocalFire();  // Client-side prediction
    virtual void PlayFireEffects();
    virtual void PlayReloadEffects();

    // Hitscan trace
    FHitResult PerformLineTrace(FVector Start, FVector End);
    void ApplyDamage(FHitResult& Hit, float Damage);

    // Projectile spawn
    void SpawnProjectile(FVector Origin, FVector Direction);

    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    USkeletalMeshComponent* WeaponMesh;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Weapon|Config")
    EWeaponType WeaponType;

    UPROPERTY(Replicated, EditDefaultsOnly, BlueprintReadOnly, Category = "Weapon|Config")
    EWeaponRarity Rarity;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Weapon|Config")
    FWeaponStats WeaponStats;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Weapon|Config")
    FText WeaponName;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Weapon|Config")
    UTexture2D* WeaponIcon;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Weapon|Config")
    bool bIsHitscan = true;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Weapon|Config")
    TSubclassOf ProjectileClass;

    UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category = "Weapon|Config")
    int32 PelletCount = 1;  // >1 for shotguns

    // State
    UPROPERTY(Replicated)
    int32 CurrentAmmo;

    UPROPERTY(Replicated)
    int32 ReserveAmmo;

    UPROPERTY(Replicated)
    EWeaponState WeaponState;

    UPROPERTY()
    APastaCharacterBase* OwnerCharacter;

    float CurrentSpreadAngle;
    float LastFireTime;
    bool bWantsToFire;
    bool bIsAiming;

    FTimerHandle ReloadTimerHandle;
    FTimerHandle EquipTimerHandle;

    // VFX/Audio
    UPROPERTY(EditDefaultsOnly, Category = "Weapon|FX")
    UNiagaraSystem* MuzzleFlashVFX;

    UPROPERTY(EditDefaultsOnly, Category = "Weapon|FX")
    UNiagaraSystem* ImpactVFX;

    UPROPERTY(EditDefaultsOnly, Category = "Weapon|FX")
    UNiagaraSystem* TracerVFX;

    UPROPERTY(EditDefaultsOnly, Category = "Weapon|Audio")
    USoundBase* FireSound;

    UPROPERTY(EditDefaultsOnly, Category = "Weapon|Audio")
    USoundBase* ReloadSound;

    UPROPERTY(EditDefaultsOnly, Category = "Weapon|Audio")
    USoundBase* EquipSound;

    UPROPERTY(EditDefaultsOnly, Category = "Weapon|Audio")
    USoundBase* EmptyClickSound;

    // Animation
    UPROPERTY(EditDefaultsOnly, Category = "Weapon|Animation")
    UAnimMontage* FireMontage;

    UPROPERTY(EditDefaultsOnly, Category = "Weapon|Animation")
    UAnimMontage* ReloadMontage;

    UPROPERTY(EditDefaultsOnly, Category = "Weapon|Animation")
    UAnimMontage* EquipMontage;
};
```

### 4.2 — Projectile Base

```cpp
// Source/PastaWarfare/Weapons/PastaProjectile.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "PastaProjectile.generated.h"

UCLASS(Abstract)
class PASTAWARFARE_API APastaProjectile : public AActor
{
    GENERATED_BODY()

public:
    APastaProjectile();

    void InitProjectile(FVector Direction, float Damage, APastaCharacterBase* Instigator);

protected:
    virtual void BeginPlay() override;
    virtual void Tick(float DeltaTime) override;

    UFUNCTION()
    void OnHit(UPrimitiveComponent* HitComp, AActor* OtherActor,
               UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit);

    UFUNCTION()
    void OnOverlap(UPrimitiveComponent* OverlappedComp, AActor* OtherActor,
                    UPrimitiveComponent* OtherComp, int32 OtherBodyIndex,
                    bool bFromSweep, const FHitResult& SweepResult);

    virtual void Explode(FVector Location);
    virtual void ApplyAreaDamage(FVector Location);

    UPROPERTY(VisibleAnywhere)
    UStaticMeshComponent* ProjectileMesh;

    UPROPERTY(VisibleAnywhere)
    class UProjectileMovementComponent* ProjectileMovement;

    UPROPERTY(VisibleAnywhere)
    class USphereComponent* CollisionSphere;

    UPROPERTY(VisibleAnywhere)
    UNiagaraComponent* TrailVFX;

    UPROPERTY(EditDefaultsOnly, Category = "Projectile")
    float Speed = 5000.0f;

    UPROPERTY(EditDefaultsOnly, Category = "Projectile")
    float GravityScale = 0.3f;

    UPROPERTY(EditDefaultsOnly, Category = "Projectile")
    float LifeSpan = 5.0f;

    UPROPERTY(EditDefaultsOnly, Category = "Projectile")
    float SplashRadius = 0.0f;

    UPROPERTY(EditDefaultsOnly, Category = "Projectile")
    float SplashDamage = 0.0f;

    UPROPERTY(EditDefaultsOnly, Category = "Projectile|FX")
    UNiagaraSystem* ExplosionVFX;

    UPROPERTY(EditDefaultsOnly, Category = "Projectile|FX")
    USoundBase* ExplosionSound;

    UPROPERTY(EditDefaultsOnly, Category = "Projectile|FX")
    USoundBase* FlightSound;

    float ProjectileDamage;
    APastaCharacterBase* InstigatorCharacter;
};
```

---

## 5. ABILITY SYSTEM — PASTA POWERS

### 5.1 — Ability Definitions

Each pasta type has a unique tactical ability and an ultimate ability. Implement using UE5's Gameplay Ability System (GAS).

```cpp
// Source/PastaWarfare/Abilities/PastaAbilitySystemComponent.h
#pragma once

#include "AbilitySystemComponent.h"
#include "PastaAbilitySystemComponent.generated.h"

UCLASS()
class PASTAWARFARE_API UPastaAbilitySystemComponent : public UAbilitySystemComponent
{
    GENERATED_BODY()

public:
    void GrantDefaultAbilities(const TArray<TSubclassOf<class UPastaGameplayAbility>>& Abilities);
    void ClearAllAbilities();

    UFUNCTION(BlueprintCallable, Category = "Abilities")
    bool TryActivateAbilityByTag(FGameplayTag AbilityTag);
};

// ============================================================
// ABILITY DEFINITIONS PER PASTA TYPE
// ============================================================
/*
SPAGHETTI:
  Tactical: "Noodle Lasso" — Throw a bundle of spaghetti that grabs an enemy 
            and pulls them toward you (or you toward them). 15s cooldown.
  Ultimate: "Al Dente Fury" — Body hardens, gaining 50% damage reduction and 
            30% melee damage boost for 8 seconds. 90s cooldown.

PENNE:
  Tactical: "Tube Tunnel" — Place two penne tube portals up to 30m apart. 
            Team can travel between them for 10s. 20s cooldown.
  Ultimate: "Rigatoni Rain" — Call in an artillery strike of giant rigatoni 
            tubes in a targeted area. 120s cooldown.

FARFALLE:
  Tactical: "Butterfly Drift" — Gain wings for 5s, able to glide and gain 
            height. Silent movement while active. 18
