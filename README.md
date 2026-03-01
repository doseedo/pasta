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

PART 2:
import React, { useState, useEffect, useRef, useCallback, useMemo } from "react"
import { motion, AnimatePresence } from "framer-motion"
import { addPropertyControls, ControlType } from "framer"

// ============================================================
// Doseedo DAW Demo Animation v2 — With Drag-to-Create
// ============================================================

const C = {
    bgDarkest: "#0a0a0a",
    bgDark: "#1a1a1a",
    bgMid: "#1a1a2e",
    bgLight: "#2a2a2a",
    bgLighter: "#333",
    border: "rgba(255,255,255,0.08)",
    borderLight: "#333",
    borderSubtle: "rgb(40,40,40)",
    text: "#fff",
    textSec: "#ccc",
    textMuted: "#888",
    textDim: "#555",
    blue: "#667eea",
    purple: "#8b5cf6",
    purpleDark: "#764ba2",
    green: "#4CAF50",
    gradPrimary: "linear-gradient(135deg, #667eea 0%, #764ba2 100%)",
    sidebarGrad: "linear-gradient(180deg, rgb(28,28,28), rgb(18,18,18))",
} as const

function seededRandom(seed: number) {
    let s = seed
    return () => {
        s = (s * 16807 + 0) % 2147483647
        return s / 2147483647
    }
}

function WaveformSVG({
    color,
    seed,
    width,
    height,
}: {
    color: string
    seed: number
    width: number
    height: number
}) {
    const pathD = useMemo(() => {
        const rand = seededRandom(seed)
        const mid = height / 2
        const step = 3
        const bars = Math.floor(width / step)
        let d = ""
        for (let i = 0; i < bars; i++) {
            const x = i * step
            const pos = i / bars
            const envelope = Math.sin(pos * Math.PI) * 0.6 + 0.4
            const burst = rand() > 0.85 ? 1.3 : 1
            const amp = rand() * mid * 0.8 * envelope * burst
            d += `M${x},${mid - amp} L${x},${mid + amp} `
        }
        return d
    }, [color, seed, width, height])

    return (
        <svg
            width={width}
            height={height}
            viewBox={`0 0 ${width} ${height}`}
            style={{ display: "block" }}
        >
            <path
                d={pathD}
                stroke={color}
                strokeWidth={2}
                strokeLinecap="round"
                fill="none"
                opacity={0.8}
                style={{ transition: "stroke 0.6s ease" }}
            />
        </svg>
    )
}

function NoiseWaveform({
    width,
    height,
    color = "rgba(139,92,246,0.6)",
    settling = false,
}: {
    width: number
    height: number
    color?: string
    settling?: boolean
}) {
    const canvasRef = useRef<HTMLCanvasElement | null>(null)
    const animRef = useRef<number | null>(null)
    const prevAmps = useRef<number[] | null>(null)
    const settleStartRef = useRef(0)

    useEffect(() => {
        const canvas = canvasRef.current
        if (!canvas || width <= 0 || height <= 0) return

        const ctx = canvas.getContext("2d")
        if (!ctx) return

        const dpr = window.devicePixelRatio || 1
        canvas.width = width * dpr
        canvas.height = height * dpr
        canvas.style.width = `${width}px`
        canvas.style.height = `${height}px`
        ctx.scale(dpr, dpr)

        const lineCount = Math.floor(width / 4)
        const centerY = height / 2

        // Parse color for glow variant
        const glowColor = color.replace(
            /[\d.]+\)$/,
            (m) => `${parseFloat(m) * 0.5})`
        )

        if (!prevAmps.current || prevAmps.current.length !== lineCount) {
            prevAmps.current = new Array(lineCount)
                .fill(0)
                .map(() => (Math.random() - 0.5) * height * 0.3)
        }

        if (settling && settleStartRef.current === 0) {
            settleStartRef.current = Date.now()
        }

        const animate = () => {
            ctx.clearRect(0, 0, width, height)
            ctx.strokeStyle = color
            ctx.lineWidth = 2

            const elapsed = settling
                ? (Date.now() - settleStartRef.current) / 1000
                : 0

            for (let i = 0; i < lineCount; i++) {
                const x = (i / lineCount) * width
                const prev = prevAmps.current![i]

                let amp: number

                if (settling) {
                    const progress = Math.min(elapsed / 0.5, 1)
                    const ease = 1 - Math.pow(1 - progress, 3)
                    const noise =
                        (1 - ease) * ((Math.random() - 0.5) * height * 0.5)
                    amp = noise
                } else {
                    const target = (Math.random() - 0.5) * height * 0.6
                    amp = prev + (target - prev) * 0.08
                }

                prevAmps.current![i] = amp

                ctx.beginPath()
                ctx.moveTo(x, centerY - amp)
                ctx.lineTo(x, centerY + amp)
                ctx.stroke()

                // Subtle glow on random bars
                if (Math.random() > 0.95) {
                    ctx.strokeStyle = glowColor
                    ctx.lineWidth = 4
                    ctx.beginPath()
                    ctx.moveTo(x, centerY - amp)
                    ctx.lineTo(x, centerY + amp)
                    ctx.stroke()
                    ctx.strokeStyle = color
                    ctx.lineWidth = 2
                }
            }

            if (!settling || elapsed < 0.5) {
                animRef.current = requestAnimationFrame(animate)
            }
        }

        animate()
        return () => {
            if (animRef.current) cancelAnimationFrame(animRef.current)
        }
    }, [width, height, color, settling])

    return (
        <canvas
            ref={canvasRef}
            style={{ width, height, borderRadius: 4, display: "block" }}
        />
    )
}

const Icons = {
    play: (
        <svg width="10" height="12" viewBox="0 0 10 12" fill="currentColor">
            <polygon points="0,0 10,6 0,12" />
        </svg>
    ),
    pause: (
        <svg width="10" height="12" viewBox="0 0 10 12" fill="currentColor">
            <rect x="0" y="0" width="3" height="12" rx="1" />
            <rect x="7" y="0" width="3" height="12" rx="1" />
        </svg>
    ),
    stop: (
        <svg width="10" height="10" viewBox="0 0 10 10" fill="currentColor">
            <rect x="0" y="0" width="10" height="10" rx="1" />
        </svg>
    ),
    mic: (
        <img
            src="https://doseedo.com/assets/icons/microphone.png"
            width="26"
            height="26"
            style={{ filter: "invert(1)", objectFit: "contain" }}
            alt=""
        />
    ),
    violin: (
        <img
            src="https://doseedo.com/assets/icons/violin.png"
            width="26"
            height="26"
            style={{ filter: "invert(1)", objectFit: "contain" }}
            alt=""
        />
    ),
    keys: (
        <img
            src="https://doseedo.com/assets/icons/keyboard.png"
            width="26"
            height="26"
            style={{ filter: "invert(1)", objectFit: "contain" }}
            alt=""
        />
    ),
    wind: (
        <img
            src="https://doseedo.com/assets/icons/sax.png"
            width="26"
            height="26"
            style={{ filter: "invert(1)", objectFit: "contain" }}
            alt=""
        />
    ),
    home: (
        <svg
            width="16"
            height="16"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            strokeWidth="2"
        >
            <path d="M3 12l9-8 9 8" />
            <path d="M5 10v10h14V10" />
        </svg>
    ),
    search: (
        <svg
            width="16"
            height="16"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            strokeWidth="2"
        >
            <circle cx="11" cy="11" r="7" />
            <line x1="16" y1="16" x2="21" y2="21" />
        </svg>
    ),
    music: (
        <svg
            width="16"
            height="16"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            strokeWidth="2"
        >
            <circle cx="6" cy="18" r="3" />
            <circle cx="18" cy="16" r="3" />
            <line x1="9" y1="18" x2="9" y2="4" />
            <line x1="21" y1="16" x2="21" y2="2" />
            <path d="M9 4l12-2" />
        </svg>
    ),
    settings: (
        <svg
            width="16"
            height="16"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            strokeWidth="2"
        >
            <circle cx="12" cy="12" r="3" />
            <path d="M12 1v2m0 18v2M4.22 4.22l1.42 1.42m12.72 12.72l1.42 1.42M1 12h2m18 0h2M4.22 19.78l1.42-1.42M18.36 5.64l1.42-1.42" />
        </svg>
    ),
    wand: (
        <svg
            width="16"
            height="16"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            strokeWidth="2"
        >
            <path d="M15 4l5 5L7 22 2 17z" />
            <path d="M18 2l4 4" />
        </svg>
    ),
    chat: (
        <svg
            width="16"
            height="16"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            strokeWidth="2"
        >
            <path d="M21 15a2 2 0 01-2 2H7l-4 4V5a2 2 0 012-2h14a2 2 0 012 2z" />
        </svg>
    ),
    chevDown: (
        <svg width="8" height="8" viewBox="0 0 8 8">
            <path
                d="M1 2.5l3 3 3-3"
                stroke="currentColor"
                strokeWidth="1.5"
                fill="none"
            />
        </svg>
    ),
    piano: (
        <img
            src="https://doseedo.com/assets/icons/piano.png"
            width="24"
            height="24"
            style={{ filter: "invert(1)", objectFit: "contain" }}
            alt=""
        />
    ),
    guitar: (
        <img
            src="https://doseedo.com/assets/icons/acguitar.png"
            width="24"
            height="24"
            style={{ filter: "invert(1)", objectFit: "contain" }}
            alt=""
        />
    ),
    bass: (
        <img
            src="https://doseedo.com/assets/icons/elecbass.png"
            width="24"
            height="24"
            style={{ filter: "invert(1)", objectFit: "contain" }}
            alt=""
        />
    ),
    trumpet: (
        <img
            src="https://doseedo.com/assets/icons/tpt.png"
            width="28"
            height="28"
            style={{ filter: "invert(1)", objectFit: "contain" }}
            alt=""
        />
    ),
    drums: (
        <img
            src="https://doseedo.com/assets/icons/drumkit.png"
            width="24"
            height="24"
            style={{ filter: "invert(1)", objectFit: "contain" }}
            alt=""
        />
    ),
    synth: (
        <img
            src="https://doseedo.com/assets/icons/keyboard.png"
            width="24"
            height="24"
            style={{ filter: "invert(1)", objectFit: "contain" }}
            alt=""
        />
    ),
}

// --- Instrument pool for drag-created tracks ---
const instrumentPool = [
    {
        name: "Vocals",
        icon: Icons.mic,
        color: "rgba(168,127,255,0.7)",
        colorBg: "rgba(168,127,255,0.06)",
    },
    {
        name: "Strings",
        icon: Icons.violin,
        color: "rgba(102,126,234,0.7)",
        colorBg: "rgba(102,126,234,0.06)",
    },
    {
        name: "Piano",
        icon: Icons.piano,
        color: "rgba(72,202,228,0.7)",
        colorBg: "rgba(72,202,228,0.06)",
    },
    {
        name: "Winds",
        icon: Icons.wind,
        color: "rgba(100,220,150,0.7)",
        colorBg: "rgba(100,220,150,0.06)",
    },
    {
        name: "Drums",
        icon: Icons.drums,
        color: "rgba(255,165,0,0.7)",
        colorBg: "rgba(255,165,0,0.06)",
    },
    {
        name: "Bass",
        icon: Icons.bass,
        color: "rgba(255,100,100,0.7)",
        colorBg: "rgba(255,100,100,0.06)",
    },
    {
        name: "Synth",
        icon: Icons.synth,
        color: "rgba(200,100,255,0.7)",
        colorBg: "rgba(200,100,255,0.06)",
    },
    {
        name: "Guitar",
        icon: Icons.guitar,
        color: "rgba(255,200,50,0.7)",
        colorBg: "rgba(255,200,50,0.06)",
    },
]

// Instruments selectable in the generation panel
const panelInstruments = [
    { name: "Piano", icon: Icons.piano },
    { name: "Guitar", icon: Icons.guitar },
    { name: "Bass", icon: Icons.bass },
    { name: "Strings", icon: Icons.violin },
    { name: "Brass", icon: Icons.trumpet },
    { name: "Winds", icon: Icons.wind },
    { name: "Drums", icon: Icons.drums },
    { name: "Synth", icon: Icons.synth },
    { name: "Vocals", icon: Icons.mic },
]

// Track with position info
interface TrackItem {
    id: string
    name: string
    icon: React.ReactNode
    color: string
    colorBg: string
    seed: number
    startFrac: number // 0-1 position along timeline
    widthFrac: number // 0-1 width along timeline
    isPlaceholder: boolean
    row: number // which row/slot
    genreClip?: boolean // true for clips generated by genre switches
    genreIndex?: number // which genre created this clip
}

const genres = [
    {
        name: "Normal",
        bg: "linear-gradient(135deg, rgba(102,126,234,0.2), rgba(118,75,162,0.15))",
        glow: "100,160,255",
    },
    {
        name: "Jazz",
        bg: "linear-gradient(135deg, rgba(140,80,220,0.2), rgba(100,50,180,0.15))",
        glow: "120,60,200",
    },
    {
        name: "Reggae",
        bg: "linear-gradient(135deg, rgba(76,175,80,0.2), rgba(255,235,59,0.15))",
        glow: "50,200,80",
    },
    {
        name: "Electronic",
        bg: "linear-gradient(135deg, rgba(0,188,212,0.2), rgba(156,39,176,0.15))",
        glow: "30,60,180",
    },
]

// Genre → which panelInstruments indices are active (6 per genre)
// panelInstruments: 0=Piano, 1=Guitar, 2=Bass, 3=Strings, 4=Brass, 5=Winds, 6=Drums, 7=Synth, 8=Vocals
const genreInstruments: Record<number, Set<number>> = {
    0: new Set([0, 3, 4, 6, 7, 8]), // Normal: Piano, Strings, Brass, Drums, Synth, Vocals
    1: new Set([0, 2, 4, 5, 6, 8]), // Jazz: Piano, Bass, Brass, Winds, Drums, Vocals
    2: new Set([1, 2, 6, 0, 8, 5]), // Reggae: Guitar, Bass, Drums, Piano, Vocals, Winds
    3: new Set([2, 6, 7, 0, 8, 1]), // Electronic: Bass, Drums, Synth, Piano, Vocals, Guitar
}

// ============================================================
// MAIN COMPONENT
// ============================================================
export default function DAW2Animation({
    trackDuration = 30,
    width = 1200,
    height = 640,
    glowIntensity = 100,
    panelWidth = 210,
    videoExtendW = 0,
    videoExtendH = 0,
    swayIntensity = 6,
    panelExtendH = 60,
    bgVideoOpacity = 50,
    bgVideoScale = 100,
    style: frameStyle,
}: {
    trackDuration?: number
    width?: number
    height?: number
    glowIntensity?: number
    panelWidth?: number
    videoExtendW?: number
    videoExtendH?: number
    swayIntensity?: number
    panelExtendH?: number
    bgVideoOpacity?: number
    bgVideoScale?: number
    style?: React.CSSProperties
}) {
    // --- State ---
    const [isPlaying, setIsPlaying] = useState(false)
    const [selectedGenre, setSelectedGenre] = useState(0)
    const [activeTrackGenre, setActiveTrackGenre] = useState(0) // tracks which genre's clips are colored (fires 2s early)
    const [selectedInstruments, setSelectedInstruments] = useState<Set<number>>(
        new Set(genreInstruments[0])
    )
    const [genProgress, setGenProgress] = useState(0)
    const [isGenerating, setIsGenerating] = useState(false)
    const [tracks, setTracks] = useState<TrackItem[]>([])
    const [mutedRows, setMutedRows] = useState<Set<number>>(new Set())
    const [soloRow, setSoloRow] = useState<number | null>(null)
    const [playheadProgress, setPlayheadProgress] = useState(0)
    const [timeDisplay, setTimeDisplay] = useState("0:00")
    const nextIdRef = useRef(1)

    // Initial fade-in: hidden for 1s then 1s fade
    const [bgReady, setBgReady] = useState(false)
    useEffect(() => {
        const t = setTimeout(() => setBgReady(true), 1000)
        return () => clearTimeout(t)
    }, [])

    // Marquee drag state
    const [videoLoaded, setVideoLoaded] = useState(false)
    const videoRef = useRef<HTMLVideoElement | null>(null)
    const bgVideoRef = useRef<HTMLVideoElement | null>(null)
    const [bgVideoLoaded, setBgVideoLoaded] = useState(false)
    const [isDragging, setIsDragging] = useState(false)
    const [marqueeStart, setMarqueeStart] = useState({ x: 0, y: 0 })
    const [marqueeEnd, setMarqueeEnd] = useState({ x: 0, y: 0 })
    const dragRef = useRef({ active: false, startX: 0, startY: 0 })

    const isPlayingRef = useRef(false)
    const playheadRef = useRef<number | null>(null)
    const startTimeRef = useRef(0)
    const pausedAtRef = useRef(0)
    const hasAutoPlayed = useRef(false)
    const genIntervalRef = useRef<ReturnType<typeof setInterval> | null>(null)
    const genTimeoutRef = useRef<ReturnType<typeof setTimeout> | null>(null)
    const staggerTimeoutsRef = useRef<ReturnType<typeof setTimeout>[]>([])
    const rootRef = useRef<HTMLDivElement | null>(null)
    const trackAreaRef = useRef<HTMLDivElement | null>(null)
    const waveAreaRef = useRef<HTMLDivElement | null>(null)
    const [rootW, setRootW] = useState(width)
    const [rootH, setRootH] = useState(height)
    const [measuredWaveW, setMeasuredWaveW] = useState(400)

    // Measure actual root size + waveform area
    useEffect(() => {
        const measure = () => {
            if (rootRef.current) {
                const r = rootRef.current.getBoundingClientRect()
                if (r.width > 0) setRootW(r.width)
                if (r.height > 0) setRootH(r.height)
            }
            if (waveAreaRef.current) {
                const w = waveAreaRef.current.offsetWidth
                if (w > 0) setMeasuredWaveW(w)
            }
        }
        measure()
        const ro = new ResizeObserver(measure)
        if (rootRef.current) ro.observe(rootRef.current)
        if (waveAreaRef.current) ro.observe(waveAreaRef.current)
        return () => ro.disconnect()
    }, [])

    // Layout
    const labelW = 140
    const genPanelW = panelWidth
    const bodyH = rootH
    const dawHeight = Math.max(200, bodyH * 0.48)
    const vizHeight = bodyH - dawHeight
    const trackHeight = 64
    const timelineH = 28
    const controlsH = 36

    // --- Playhead ---
    const playheadDuration = trackDuration

    const animatePlayhead = useCallback(() => {
        const elapsed =
            pausedAtRef.current + (Date.now() - startTimeRef.current) / 1000
        const progress = (elapsed % playheadDuration) / playheadDuration
        setPlayheadProgress(progress)
        const secs = Math.floor(elapsed % playheadDuration)
        setTimeDisplay(
            `${Math.floor(secs / 60)}:${(secs % 60).toString().padStart(2, "0")}`
        )
        playheadRef.current = requestAnimationFrame(animatePlayhead)
    }, [playheadDuration])

    // --- Auto-generate initial 4 tracks (full-width, stagger jump-in) ---
    const autoGenerateTracks = useCallback(() => {
        // Cancel any pending stagger timeouts from a previous call
        staggerTimeoutsRef.current.forEach(clearTimeout)
        staggerTimeoutsRef.current = []

        const initialTracks: TrackItem[] = instrumentPool
            .slice(0, 4)
            .map((inst, i) => ({
                id: `track-${nextIdRef.current++}`,
                name: inst.name,
                icon: inst.icon,
                color: inst.color,
                colorBg: inst.colorBg,
                seed: Math.floor(Math.random() * 10000),
                startFrac: 0,
                widthFrac: 1,
                isPlaceholder: false,
                row: i,
            }))

        // Stagger: add tracks one at a time for the jump-in animation
        initialTracks.forEach((t, i) => {
            const tid = setTimeout(() => {
                setTracks((curr) => [...curr, t])
            }, i * 200)
            staggerTimeoutsRef.current.push(tid)
        })
    }, [])

    // --- Seek to a fraction (0-1) of the 30s loop ---
    const seekTo = useCallback(
        (frac: number) => {
            const clampedFrac = Math.max(0, Math.min(1, frac))
            const seekSeconds = clampedFrac * playheadDuration
            const seekMs = clampedFrac * CYCLE_DURATION

            // Update playhead
            pausedAtRef.current = seekSeconds
            startTimeRef.current = Date.now()
            setPlayheadProgress(clampedFrac)
            const secs = Math.floor(seekSeconds)
            setTimeDisplay(
                `${Math.floor(secs / 60)}:${(secs % 60).toString().padStart(2, "0")}`
            )

            // Update cycle position
            cycleStartRef.current = Date.now() - seekMs
            cyclePausedElapsedRef.current = seekMs

            // Sync videos
            const vid = videoRef.current
            if (vid && videoLoaded) {
                vid.currentTime =
                    seekSeconds % (vid.duration || playheadDuration)
            }
            const bgVid = bgVideoRef.current
            if (bgVid) {
                bgVid.currentTime =
                    seekSeconds % (bgVid.duration || playheadDuration)
            }

            // Update genre/header to match seek position immediately
            const elapsed = seekMs % CYCLE_DURATION
            if (elapsed < PAUSE_DURATION / 2) {
                setCycleFade(elapsed / (PAUSE_DURATION / 2))
            } else if (elapsed < PAUSE_DURATION) {
                setCycleFade(1)
            } else if (elapsed >= CYCLE_DURATION - PAUSE_DURATION / 2) {
                const fadeOutElapsed =
                    elapsed - (CYCLE_DURATION - PAUSE_DURATION / 2)
                setCycleFade(1 - fadeOutElapsed / (PAUSE_DURATION / 2))
            } else {
                setCycleFade(1)
            }

            const contentElapsed = elapsed - PAUSE_DURATION
            if (contentElapsed >= 0 && contentElapsed < CONTENT_DURATION) {
                // Track generation + visual genre switch 2s early so they stay in sync
                const GENRE_ANTICIPATION = 2000
                const genreIdx = Math.min(
                    genres.length - 1,
                    Math.floor(
                        (contentElapsed + GENRE_ANTICIPATION) / GENRE_DURATION
                    )
                )
                if (genreIdx !== lastGenreRef.current) {
                    lastGenreRef.current = genreIdx
                    setSelectedGenre(genreIdx)
                    setSelectedInstruments(
                        new Set(
                            genreInstruments[genreIdx] || genreInstruments[0]
                        )
                    )
                }
                if (genreIdx !== lastTrackGenreRef.current) {
                    lastTrackGenreRef.current = genreIdx
                    handleGenreClickRef.current(genreIdx)
                }
                const headerIdx =
                    Math.floor(contentElapsed / HEADER_DURATION) %
                    headerTexts.length
                if (headerIdx !== headerIndexRef.current) {
                    headerIndexRef.current = headerIdx
                    setHeaderIndex(headerIdx)
                }
            }
        },
        [playheadDuration, videoLoaded]
    )

    // --- Playhead drag state ---
    const seekDragRef = useRef(false)
    const waveAreaRectRef = useRef<DOMRect | null>(null)

    const handlePlay = useCallback(() => {
        if (isPlaying) {
            // --- PAUSE ---
            if (playheadRef.current) cancelAnimationFrame(playheadRef.current)
            pausedAtRef.current += (Date.now() - startTimeRef.current) / 1000
            // Pause sequence cycle
            cyclePausedElapsedRef.current =
                (Date.now() - cycleStartRef.current) % CYCLE_DURATION
            if (sequenceTimerRef.current) {
                clearInterval(sequenceTimerRef.current)
                sequenceTimerRef.current = null
            }
            // Pause videos
            const vid = videoRef.current
            if (vid) vid.pause()
            const bgVid = bgVideoRef.current
            if (bgVid) bgVid.pause()
            isPlayingRef.current = false
            setIsPlaying(false)
        } else {
            // --- PLAY / RESUME ---
            if (tracks.length === 0) {
                autoGenerateTracks()
            }
            // Resume sequence cycle from where we left off
            cycleStartRef.current = Date.now() - cyclePausedElapsedRef.current
            lastCycleCountRef.current = 0
            sequenceTimerRef.current = setInterval(() => {
                const rawElapsed = Date.now() - cycleStartRef.current
                const cycleCount = Math.floor(rawElapsed / CYCLE_DURATION)
                const elapsed = rawElapsed % CYCLE_DURATION

                if (cycleCount > lastCycleCountRef.current) {
                    lastCycleCountRef.current = cycleCount
                    const vid = videoRef.current
                    if (vid) {
                        vid.currentTime = 0
                        vid.play().catch(() => {})
                    }
                    const bgVid = bgVideoRef.current
                    if (bgVid) {
                        bgVid.currentTime = 0
                        bgVid.play().catch(() => {})
                    }
                    pausedAtRef.current = 0
                    startTimeRef.current = Date.now()
                    headerIndexRef.current = -1
                    setHeaderIndex(-1)
                    lastGenreRef.current = 0
                    lastTrackGenreRef.current = 0
                    setSelectedGenre(0)
                    setActiveTrackGenre(0)
                    setSelectedInstruments(new Set(genreInstruments[0]))
                    setTracks([])
                    setIsGenerating(false)
                    setTimeout(() => autoGenerateTracksRef.current(), 50)
                }

                if (elapsed < PAUSE_DURATION / 2) {
                    setCycleFade(elapsed / (PAUSE_DURATION / 2))
                } else if (elapsed < PAUSE_DURATION) {
                    setCycleFade(1)
                } else if (elapsed >= CYCLE_DURATION - PAUSE_DURATION / 2) {
                    const fadeOutElapsed =
                        elapsed - (CYCLE_DURATION - PAUSE_DURATION / 2)
                    setCycleFade(1 - fadeOutElapsed / (PAUSE_DURATION / 2))
                } else {
                    setCycleFade(1)
                }

                const contentElapsed = elapsed - PAUSE_DURATION
                if (contentElapsed >= 0 && contentElapsed < CONTENT_DURATION) {
                    const GENRE_ANTICIPATION = 2000
                    const genreIdx = Math.min(
                        genres.length - 1,
                        Math.floor(
                            (contentElapsed + GENRE_ANTICIPATION) /
                                GENRE_DURATION
                        )
                    )
                    if (genreIdx !== lastGenreRef.current) {
                        lastGenreRef.current = genreIdx
                        setSelectedGenre(genreIdx)
                        setSelectedInstruments(
                            new Set(
                                genreInstruments[genreIdx] ||
                                    genreInstruments[0]
                            )
                        )
                    }
                    if (genreIdx !== lastTrackGenreRef.current) {
                        lastTrackGenreRef.current = genreIdx
                        handleGenreClickRef.current(genreIdx)
                    }
                    const headerIdx =
                        Math.floor(contentElapsed / HEADER_DURATION) %
                        headerTexts.length
                    if (headerIdx !== headerIndexRef.current) {
                        headerIndexRef.current = headerIdx
                        setHeaderIndex(headerIdx)
                    }
                }
            }, 50)
            // Resume videos
            const vid = videoRef.current
            if (vid && videoLoaded) vid.play().catch(() => {})
            const bgVid = bgVideoRef.current
            if (bgVid) bgVid.play().catch(() => {})
            startTimeRef.current = Date.now()
            isPlayingRef.current = true
            setIsPlaying(true)
            playheadRef.current = requestAnimationFrame(animatePlayhead)
        }
    }, [
        isPlaying,
        animatePlayhead,
        tracks.length,
        autoGenerateTracks,
        videoLoaded,
    ])

    // Auto-play is handled by startSequence → restartCycle on mount

    // --- Video: detect when ready, retry on 500 cold-start errors ---
    const VIDEO_URL =
        "https://storage.googleapis.com/audiocraft-411005.appspot.com/assets/sitedemo.mp4"
    const videoRetryRef = useRef(0)
    useEffect(() => {
        const vid = videoRef.current
        if (!vid) return
        let retryTimer: ReturnType<typeof setTimeout> | null = null
        const onReady = () => {
            videoRetryRef.current = 0
            setVideoLoaded(true)
        }
        const onError = () => {
            // Retry up to 5 times with increasing delay (2s, 3s, 4s, 5s, 6s)
            if (videoRetryRef.current < 5) {
                videoRetryRef.current++
                const delay = 1000 + videoRetryRef.current * 1000
                retryTimer = setTimeout(() => {
                    vid.src = VIDEO_URL + "?r=" + videoRetryRef.current
                    vid.load()
                }, delay)
            }
        }
        vid.addEventListener("canplay", onReady)
        vid.addEventListener("loadeddata", onReady)
        vid.addEventListener("error", onError)
        if (vid.readyState >= 2) onReady()
        return () => {
            vid.removeEventListener("canplay", onReady)
            vid.removeEventListener("loadeddata", onReady)
            vid.removeEventListener("error", onError)
            if (retryTimer) clearTimeout(retryTimer)
        }
    }, [])

    // --- Video auto-play when loaded ---
    useEffect(() => {
        const vid = videoRef.current
        if (!vid || !videoLoaded) return
        vid.play().catch(() => {})
    }, [videoLoaded])

    // --- Genre click → split tracks and generate colored clip at genre time slot ---
    const greyColor = "rgba(100,100,100,0.5)"
    const greyColorBg = "rgba(100,100,100,0.04)"

    const handleGenreClick = useCallback(
        (genreIndex: number) => {
            if (isGenerating) return

            // Genre 0: tracks already loaded by autoGenerateTracks, nothing to do
            if (genreIndex === 0) {
                setActiveTrackGenre(0)
                return
            }

            // Genres 1-3: split existing tracks and insert colored clip at the genre's time slot
            const startFrac =
                (PAUSE_DURATION + genreIndex * GENRE_DURATION) / CYCLE_DURATION
            const endFrac = Math.min(
                1,
                (PAUSE_DURATION + (genreIndex + 1) * GENRE_DURATION) /
                    CYCLE_DURATION
            )

            const survivingTracks: TrackItem[] = []
            const affectedRows = new Set<number>()

            tracks.forEach((t) => {
                const tEnd = t.startFrac + t.widthFrac

                // No overlap with the new clip region — keep as-is (renderer handles coloring)
                if (tEnd <= startFrac || t.startFrac >= endFrac) {
                    survivingTracks.push(t)
                    return
                }

                affectedRows.add(t.row)

                // Before portion — inherit parent's genreIndex (stays grey if parent was grey),
                // falls back to activeTrackGenre for original unsplit tracks
                if (t.startFrac < startFrac) {
                    survivingTracks.push({
                        ...t,
                        id: `track-${nextIdRef.current++}`,
                        widthFrac: startFrac - t.startFrac,
                        isPlaceholder: false,
                        genreClip: true,
                        genreIndex: t.genreIndex ?? activeTrackGenre,
                    })
                }

                // After portion — same logic
                if (tEnd > endFrac) {
                    survivingTracks.push({
                        ...t,
                        id: `track-${nextIdRef.current++}`,
                        startFrac: endFrac,
                        widthFrac: tEnd - endFrac,
                        seed: t.seed + 500,
                        isPlaceholder: false,
                        genreClip: true,
                        genreIndex: t.genreIndex ?? activeTrackGenre,
                    })
                }
            })

            // Create new colored clips at the genre position
            const rows = Array.from(affectedRows).sort((a, b) => a - b)
            const targetRows = rows.length > 0 ? rows : [0, 1, 2, 3]

            const newTracks: TrackItem[] = targetRows.map((row, i) => {
                const instIndex =
                    (nextIdRef.current + i) % instrumentPool.length
                const inst = instrumentPool[instIndex]
                return {
                    id: `track-${nextIdRef.current++}`,
                    name: inst.name,
                    icon: inst.icon,
                    color: inst.color,
                    colorBg: inst.colorBg,
                    seed: Math.floor(Math.random() * 10000),
                    startFrac,
                    widthFrac: endFrac - startFrac,
                    isPlaceholder: true,
                    row,
                    genreClip: true,
                    genreIndex: genreIndex,
                }
            })

            setTracks([...survivingTracks, ...newTracks])
            setIsGenerating(true)
            setGenProgress(0)

            let p = 0
            genIntervalRef.current = setInterval(() => {
                p += 0.025
                if (p >= 1) {
                    if (genIntervalRef.current)
                        clearInterval(genIntervalRef.current)
                    setGenProgress(1)
                } else {
                    setGenProgress(p)
                }
            }, 30)

            genTimeoutRef.current = setTimeout(() => {
                setIsGenerating(false)
                setActiveTrackGenre(genreIndex)
                newTracks.forEach((t, i) => {
                    setTimeout(() => {
                        setTracks((curr) =>
                            curr.map((c) =>
                                c.id === t.id
                                    ? { ...c, isPlaceholder: false }
                                    : c
                            )
                        )
                    }, i * 200)
                })
            }, 1500)
        },
        [isGenerating, tracks, activeTrackGenre]
    )

    // --- Cycling header texts (2.5s each, 2 per genre color = synced with 5s genre rotation) ---
    const headerTexts = [
        "Deconstruct",
        "Generate",
        "All From One Audio File",
        "Real Time Control",
        "Experiment",
        "Any Genre",
        "Unlimited Possibility",
        "Complete Control",
    ]
    const [headerIndex, setHeaderIndex] = useState(-1)
    const [cycleFade, setCycleFade] = useState(0) // start faded out

    // --- Master sequence: 30s total ---
    // 2s pause (fade in) at start, then 4 genres × 7s = 28s content, fade out at end
    const GENRE_DURATION = 7000
    const HEADER_DURATION = 3500 // 2 headers per genre
    const CONTENT_DURATION = 28000 // 4 genres × 7s
    const PAUSE_DURATION = 2000
    const CYCLE_DURATION = 30000

    const sequenceTimerRef = useRef<ReturnType<typeof setInterval> | null>(null)
    const cycleStartRef = useRef(Date.now())
    const cyclePausedElapsedRef = useRef(0) // how far into cycle when paused
    const handleGenreClickRef = useRef(handleGenreClick)
    handleGenreClickRef.current = handleGenreClick
    const selectedGenreRef = useRef(selectedGenre)
    selectedGenreRef.current = selectedGenre
    const headerIndexRef = useRef(0)
    const lastGenreRef = useRef(0)
    const lastTrackGenreRef = useRef(0)

    const autoGenerateTracksRef = useRef(autoGenerateTracks)
    autoGenerateTracksRef.current = autoGenerateTracks

    const restartCycle = useCallback(() => {
        cycleStartRef.current = Date.now()
        headerIndexRef.current = -1
        setHeaderIndex(-1)
        lastGenreRef.current = 0
        lastTrackGenreRef.current = 0
        setSelectedGenre(0)
        setActiveTrackGenre(0)
        setSelectedInstruments(new Set(genreInstruments[0]))
        // Clear tracks then re-do the stagger drop-in animation
        setTracks([])
        setIsGenerating(false)
        setTimeout(() => autoGenerateTracksRef.current(), 50)
        setCycleFade(0) // start faded, will fade in during first 1s
        // Restart videos from beginning
        const vid = videoRef.current
        if (vid && videoLoaded) {
            vid.currentTime = 0
            vid.play().catch(() => {})
        }
        const bgVid = bgVideoRef.current
        if (bgVid) {
            bgVid.currentTime = 0
            bgVid.play().catch(() => {})
        }
        // Reset playhead and start animating immediately
        pausedAtRef.current = 0
        startTimeRef.current = Date.now()
        setPlayheadProgress(0)
        setTimeDisplay("0:00")
        isPlayingRef.current = true
        setIsPlaying(true)
        if (playheadRef.current) cancelAnimationFrame(playheadRef.current)
        const startAnim = () => {
            const el =
                pausedAtRef.current + (Date.now() - startTimeRef.current) / 1000
            const prog = (el % playheadDuration) / playheadDuration
            setPlayheadProgress(prog)
            const s = Math.floor(el % playheadDuration)
            setTimeDisplay(
                `${Math.floor(s / 60)}:${(s % 60).toString().padStart(2, "0")}`
            )
            playheadRef.current = requestAnimationFrame(startAnim)
        }
        playheadRef.current = requestAnimationFrame(startAnim)
    }, [videoLoaded, playheadDuration])

    const lastCycleCountRef = useRef(0)

    const startSequence = useCallback(() => {
        if (sequenceTimerRef.current) clearInterval(sequenceTimerRef.current)

        restartCycle()
        lastCycleCountRef.current = 0

        sequenceTimerRef.current = setInterval(() => {
            const rawElapsed = Date.now() - cycleStartRef.current
            const cycleCount = Math.floor(rawElapsed / CYCLE_DURATION)
            const elapsed = rawElapsed % CYCLE_DURATION

            // Seamless loop: when cycle wraps, reset tracks/video/playhead
            if (cycleCount > lastCycleCountRef.current) {
                lastCycleCountRef.current = cycleCount
                // Reset videos to start
                const vid = videoRef.current
                if (vid) {
                    vid.currentTime = 0
                    vid.play().catch(() => {})
                }
                const bgVid2 = bgVideoRef.current
                if (bgVid2) {
                    bgVid2.currentTime = 0
                    bgVid2.play().catch(() => {})
                }
                // Reset playhead
                pausedAtRef.current = 0
                startTimeRef.current = Date.now()
                // Reset tracks with stagger animation
                headerIndexRef.current = -1
                setHeaderIndex(-1)
                lastGenreRef.current = 0
                lastTrackGenreRef.current = 0
                setSelectedGenre(0)
                setActiveTrackGenre(0)
                setSelectedInstruments(new Set(genreInstruments[0]))
                setTracks([])
                setIsGenerating(false)
                setTimeout(() => autoGenerateTracksRef.current(), 50)
            }

            if (elapsed < PAUSE_DURATION / 2) {
                // First 1s: fade in
                setCycleFade(elapsed / (PAUSE_DURATION / 2))
            } else if (elapsed < PAUSE_DURATION) {
                setCycleFade(1)
            } else if (elapsed >= CYCLE_DURATION - PAUSE_DURATION / 2) {
                // Last 1s: fade out
                const fadeOutElapsed =
                    elapsed - (CYCLE_DURATION - PAUSE_DURATION / 2)
                setCycleFade(1 - fadeOutElapsed / (PAUSE_DURATION / 2))
            } else {
                setCycleFade(1)
            }

            // Content phase starts after the fade-in
            const contentElapsed = elapsed - PAUSE_DURATION
            if (contentElapsed >= 0 && contentElapsed < CONTENT_DURATION) {
                const GENRE_ANTICIPATION = 2000
                const genreIdx = Math.min(
                    genres.length - 1,
                    Math.floor(
                        (contentElapsed + GENRE_ANTICIPATION) / GENRE_DURATION
                    )
                )
                if (genreIdx !== lastGenreRef.current) {
                    lastGenreRef.current = genreIdx
                    setSelectedGenre(genreIdx)
                    setSelectedInstruments(
                        new Set(
                            genreInstruments[genreIdx] || genreInstruments[0]
                        )
                    )
                }
                if (genreIdx !== lastTrackGenreRef.current) {
                    lastTrackGenreRef.current = genreIdx
                    handleGenreClickRef.current(genreIdx)
                }
                // Header: changes every 3.5s
                const headerIdx =
                    Math.floor(contentElapsed / HEADER_DURATION) %
                    headerTexts.length
                if (headerIdx !== headerIndexRef.current) {
                    headerIndexRef.current = headerIdx
                    setHeaderIndex(headerIdx)
                }
            }
        }, 50)
    }, [restartCycle])

    useEffect(() => {
        startSequence()

        // Re-sync loop when returning to the tab — browsers pause video
        // while the tab is hidden but Date.now() keeps ticking, breaking sync.
        const onVisibility = () => {
            if (
                document.visibilityState === "visible" &&
                isPlayingRef.current
            ) {
                restartCycle()
            }
        }
        document.addEventListener("visibilitychange", onVisibility)

        return () => {
            if (sequenceTimerRef.current)
                clearInterval(sequenceTimerRef.current)
            document.removeEventListener("visibilitychange", onVisibility)
        }
    }, [startSequence, restartCycle])

    const navigateGenre = useCallback(
        (dir: 1 | -1) => {
            if (isGenerating) return
            const next = (selectedGenre + dir + genres.length) % genres.length
            setSelectedGenre(next)
            setSelectedInstruments(
                new Set(genreInstruments[next] || genreInstruments[0])
            )
            handleGenreClick(next)
            // Reset cycle from current position
            cycleStartRef.current =
                Date.now() - (PAUSE_DURATION + next * GENRE_DURATION)
            lastGenreRef.current = next
            lastTrackGenreRef.current = next
            headerIndexRef.current = next * 2
            setHeaderIndex(next * 2)
        },
        [isGenerating, selectedGenre, handleGenreClick]
    )

    // --- Marquee drag handlers ---
    const handleTrackAreaMouseDown = useCallback((e: React.MouseEvent) => {
        if (e.button !== 0) return
        // Allow drag on tracks AND empty space — drag replaces/splits existing clips
        const target = e.target as HTMLElement
        if (target.closest("button")) return // don't interfere with buttons

        const container = trackAreaRef.current
        if (!container) return

        const rect = container.getBoundingClientRect()
        const x = e.clientX - rect.left
        const y = e.clientY - rect.top

        dragRef.current = { active: true, startX: x, startY: y }
        setMarqueeStart({ x, y })
        setMarqueeEnd({ x, y })
        setIsDragging(true)
        e.preventDefault()
    }, [])

    useEffect(() => {
        const handleMouseMove = (e: MouseEvent) => {
            // Playhead scrubbing
            if (seekDragRef.current && waveAreaRectRef.current) {
                const rect = waveAreaRectRef.current
                const frac = (e.clientX - rect.left) / rect.width
                seekTo(Math.max(0, Math.min(1, frac)))
                return
            }
            if (!dragRef.current.active) return
            const container = trackAreaRef.current
            if (!container) return
            const rect = container.getBoundingClientRect()
            const x = Math.max(
                0,
                Math.min(e.clientX - rect.left, container.offsetWidth)
            )
            const y = Math.max(
                0,
                Math.min(e.clientY - rect.top, container.offsetHeight)
            )
            setMarqueeEnd({ x, y })
        }

        const handleMouseUp = (e: MouseEvent) => {
            if (seekDragRef.current) {
                seekDragRef.current = false
                waveAreaRectRef.current = null
                return
            }
            if (!dragRef.current.active) return
            dragRef.current.active = false
            setIsDragging(false)

            const container = trackAreaRef.current
            if (!container) return
            const rect = container.getBoundingClientRect()
            const endX = Math.max(
                0,
                Math.min(e.clientX - rect.left, container.offsetWidth)
            )
            const endY = Math.max(
                0,
                Math.min(e.clientY - rect.top, container.offsetHeight)
            )

            const sx = dragRef.current.startX
            const sy = dragRef.current.startY

            // Selection box (in container coordinates)
            const left = Math.min(sx, endX)
            const top = Math.min(sy, endY)
            const w = Math.abs(endX - sx)
            const h = Math.abs(endY - sy)

            // Minimum drag size to avoid accidental clicks
            if (w < 10 || h < 10) return

            // Convert pixel position to timeline fractions
            // Track area: labelW is the label column, rest is waveform area
            const waveLeft = Math.max(0, left - labelW)
            const waveRight = Math.max(0, left + w - labelW)
            const startFrac = Math.max(0, Math.min(1, waveLeft / measuredWaveW))
            const endFrac = Math.max(0, Math.min(1, waveRight / measuredWaveW))
            const widthFrac = endFrac - startFrac

            if (widthFrac < 0.01) return

            // Determine which rows the drag covers based on midpoint crossing
            // A row is included if the selection box crosses its vertical midpoint
            const bottom = top + h
            const maxRow = Math.floor(bottom / trackHeight) + 1
            const affectedRows = new Set<number>()
            for (let r = 0; r <= maxRow; r++) {
                const rowMid = r * trackHeight + trackHeight / 2
                if (top < rowMid && bottom > rowMid) affectedRows.add(r)
            }
            if (affectedRows.size === 0) return
            const startRow = Math.min(...affectedRows)
            const numTracks = affectedRows.size

            const survivingTracks: TrackItem[] = []
            const splitTracks: TrackItem[] = []

            tracks.forEach((t) => {
                if (!affectedRows.has(t.row)) {
                    // Not in an affected row — keep as-is
                    survivingTracks.push(t)
                    return
                }
                const tEnd = t.startFrac + t.widthFrac
                // No overlap — keep as-is
                if (tEnd <= startFrac || t.startFrac >= endFrac) {
                    survivingTracks.push(t)
                    return
                }
                // Overlap — split into before & after portions
                // Before portion
                if (t.startFrac < startFrac) {
                    splitTracks.push({
                        ...t,
                        id: `track-${nextIdRef.current++}`,
                        widthFrac: startFrac - t.startFrac,
                        isPlaceholder: false,
                    })
                }
                // After portion
                if (tEnd > endFrac) {
                    splitTracks.push({
                        ...t,
                        id: `track-${nextIdRef.current++}`,
                        startFrac: endFrac,
                        widthFrac: tEnd - endFrac,
                        seed: t.seed + 500, // different waveform for the split
                        isPlaceholder: false,
                    })
                }
                // The overlapping portion is replaced by the new drag tracks
            })

            // Create new placeholder tracks at the drag position
            const newTracks: TrackItem[] = []
            for (let i = 0; i < numTracks; i++) {
                const instIndex =
                    (nextIdRef.current + i) % instrumentPool.length
                const inst = instrumentPool[instIndex]
                newTracks.push({
                    id: `track-${nextIdRef.current++}`,
                    name: inst.name,
                    icon: inst.icon,
                    color: inst.color,
                    colorBg: inst.colorBg,
                    seed: Math.floor(Math.random() * 10000),
                    startFrac,
                    widthFrac,
                    isPlaceholder: true,
                    row: startRow + i,
                })
            }

            setTracks([...survivingTracks, ...splitTracks, ...newTracks])
            setIsGenerating(true)
            setGenProgress(0)

            // Animate progress
            let p = 0
            genIntervalRef.current = setInterval(() => {
                p += 0.025
                if (p >= 1) {
                    if (genIntervalRef.current)
                        clearInterval(genIntervalRef.current)
                    setGenProgress(1)
                } else {
                    setGenProgress(p)
                }
            }, 30)

            // Stagger-reveal new tracks
            genTimeoutRef.current = setTimeout(() => {
                setIsGenerating(false)
                newTracks.forEach((t, i) => {
                    setTimeout(() => {
                        setTracks((curr) =>
                            curr.map((c) =>
                                c.id === t.id
                                    ? { ...c, isPlaceholder: false }
                                    : c
                            )
                        )
                    }, i * 200)
                })
            }, 1500)
        }

        window.addEventListener("mousemove", handleMouseMove)
        window.addEventListener("mouseup", handleMouseUp)
        return () => {
            window.removeEventListener("mousemove", handleMouseMove)
            window.removeEventListener("mouseup", handleMouseUp)
        }
    }, [measuredWaveW, tracks, seekTo])

    // Computed marquee box
    const marqueeBox = useMemo(() => {
        if (!isDragging) return null
        return {
            left: Math.min(marqueeStart.x, marqueeEnd.x),
            top: Math.min(marqueeStart.y, marqueeEnd.y),
            width: Math.abs(marqueeEnd.x - marqueeStart.x),
            height: Math.abs(marqueeEnd.y - marqueeStart.y),
        }
    }, [isDragging, marqueeStart, marqueeEnd])

    // Cleanup
    useEffect(
        () => () => {
            if (playheadRef.current) cancelAnimationFrame(playheadRef.current)
            if (genIntervalRef.current) clearInterval(genIntervalRef.current)
            if (genTimeoutRef.current) clearTimeout(genTimeoutRef.current)
            if (sequenceTimerRef.current)
                clearInterval(sequenceTimerRef.current)
        },
        []
    )

    // Sort tracks by row for rendering
    const sortedTracks = useMemo(
        () => [...tracks].sort((a, b) => a.row - b.row),
        [tracks]
    )

    const visibleDuration = trackDuration
    const tickInterval = trackDuration <= 15 ? 1 : trackDuration <= 30 ? 5 : 10
    const ticks = useMemo(() => {
        const arr: { time: number; pct: string }[] = []
        for (let t = 0; t <= visibleDuration; t += tickInterval) {
            arr.push({ time: t, pct: `${(t / visibleDuration) * 100}%` })
        }
        return arr
    }, [visibleDuration, tickInterval])

    const showProgress = isGenerating

    // ============================================================
    // Smooth color theme transition — lerp RGB over ~500ms
    // ============================================================
    const targetGlow = genres[selectedGenre]?.glow || "100,160,255"
    const [r, g, b] = targetGlow.split(",").map(Number)
    const glowAnimRef = useRef({ r, g, b })
    const [smoothGlow, setSmoothGlow] = useState({ r, g, b })
    const glowRafRef = useRef<number | null>(null)
    const glowTargetRef = useRef({ r, g, b })

    useEffect(() => {
        glowTargetRef.current = { r, g, b }
        let lastTime = performance.now()

        const animate = (now: number) => {
            const dt = Math.min((now - lastTime) / 1000, 0.05) // cap delta
            lastTime = now
            const speed = 4 // ~250ms to converge (higher = faster)
            const cur = glowAnimRef.current
            const tgt = glowTargetRef.current
            const nr = cur.r + (tgt.r - cur.r) * Math.min(speed * dt, 1)
            const ng = cur.g + (tgt.g - cur.g) * Math.min(speed * dt, 1)
            const nb = cur.b + (tgt.b - cur.b) * Math.min(speed * dt, 1)
            glowAnimRef.current = { r: nr, g: ng, b: nb }

            // Only update state when the change is visible (>0.5 per channel)
            if (
                Math.abs(nr - tgt.r) > 0.5 ||
                Math.abs(ng - tgt.g) > 0.5 ||
                Math.abs(nb - tgt.b) > 0.5
            ) {
                setSmoothGlow({
                    r: Math.round(nr),
                    g: Math.round(ng),
                    b: Math.round(nb),
                })
                glowRafRef.current = requestAnimationFrame(animate)
            } else {
                glowAnimRef.current = { ...tgt }
                setSmoothGlow({ r: tgt.r, g: tgt.g, b: tgt.b })
            }
        }

        glowRafRef.current = requestAnimationFrame(animate)
        return () => {
            if (glowRafRef.current) cancelAnimationFrame(glowRafRef.current)
        }
    }, [r, g, b])

    // ============================================================
    // RENDER
    // ============================================================
    const glowRGB = `${smoothGlow.r},${smoothGlow.g},${smoothGlow.b}`
    const headerGlowRGB = genres[selectedGenre]?.glow || "100,160,255" // raw, not interpolated — CSS transition handles the fade
    const gi = glowIntensity / 100 // normalize: 1 = default

    return (
        <div
            style={{
                width: "100%",
                height: "100%",
                minWidth: width,
                minHeight: height,
                ...frameStyle,
                position: "relative",
                padding: "20px 60px 60px 60px",
                boxSizing: "border-box",
                display: "flex",
                flexDirection: "column",
            }}
        >
            {/* Background video — behind everything */}
            <video
                ref={bgVideoRef}
                src="https://storage.googleapis.com/audiocraft-411005.appspot.com/assets/sitebg2.mp4"
                autoPlay
                playsInline
                muted
                loop
                preload="auto"
                onCanPlay={() => setBgVideoLoaded(true)}
                style={{
                    position: "absolute",
                    top: "50%",
                    left: "50%",
                    width: `${bgVideoScale}%`,
                    height: `${bgVideoScale}%`,
                    transform: "translate(-50%, -50%)",
                    objectFit: "cover",
                    opacity:
                        bgVideoLoaded && bgReady ? bgVideoOpacity / 100 : 0,
                    transition: "opacity 1s ease",
                    pointerEvents: "none",
                    zIndex: 0,
                }}
            />

            {/* Luminous glow behind the DAW — color shifts with genre */}
            <div
                style={{
                    position: "absolute",
                    inset: -20,
                    background: `radial-gradient(ellipse at 50% 55%, rgba(${glowRGB},${0.25 * gi}) 0%, rgba(${glowRGB},${0.12 * gi}) 25%, rgba(${glowRGB},${0.05 * gi}) 50%, transparent 75%)`,
                    opacity: bgReady ? 1 : 0,
                    transition: "opacity 1s ease, background 1.5s ease",
                    pointerEvents: "none",
                }}
            />

            {/* Cycling header */}
            <div
                style={{
                    position: "relative",
                    height: 110,
                    opacity: cycleFade,
                    transition: "opacity 0.4s ease",
                    display: "flex",
                    alignItems: "flex-end",
                    justifyContent: "center",
                    overflow: "hidden",
                    flexShrink: 0,
                    paddingBottom: 14,
                }}
            >
                <AnimatePresence mode="wait">
                    <motion.div
                        key={headerIndex}
                        initial={{ opacity: 0, y: 22, filter: "blur(10px)" }}
                        animate={{ opacity: 1, y: 0, filter: "blur(0px)" }}
                        exit={{ opacity: 0, y: -22, filter: "blur(10px)" }}
                        transition={{
                            duration: 0.5,
                            ease: [0.25, 0.8, 0.25, 1],
                        }}
                        style={{
                            fontSize: 54,
                            fontWeight: 700,
                            letterSpacing: -1.5,
                            fontFamily:
                                "-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif",
                            background: `linear-gradient(135deg, rgba(${headerGlowRGB},0.95) 0%, rgba(${headerGlowRGB},0.55) 100%)`,
                            WebkitBackgroundClip: "text",
                            WebkitTextFillColor: "transparent",
                            backgroundClip: "text",
                            textAlign: "center" as const,
                            whiteSpace: "nowrap" as const,
                            textShadow: "none",
                            transition: "background 1.5s ease",
                        }}
                    >
                        {headerTexts[headerIndex]}
                    </motion.div>
                </AnimatePresence>
            </div>

            <motion.div
                ref={rootRef}
                animate={{ y: [0, -swayIntensity, 0, swayIntensity * 0.67, 0] }}
                transition={{
                    duration: 8,
                    repeat: Infinity,
                    ease: "easeInOut",
                }}
                style={{
                    width: "100%",
                    flex: 1,
                    minHeight: 0,
                    overflow: "visible",
                    position: "relative",
                    fontFamily:
                        "-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif",
                    userSelect: "none",
                    transition: "all 1.5s ease",
                }}
            >
                {/* SVG filter for liquid glass edge refraction */}
                <svg width="0" height="0" style={{ position: "absolute" }}>
                    <defs>
                        <filter
                            id="liquid-edge"
                            x="-5%"
                            y="-5%"
                            width="110%"
                            height="110%"
                        >
                            <feTurbulence
                                type="fractalNoise"
                                baseFrequency="0.015"
                                numOctaves="3"
                                seed="2"
                                result="noise"
                            />
                            <feDisplacementMap
                                in="SourceGraphic"
                                in2="noise"
                                scale="3"
                                xChannelSelector="R"
                                yChannelSelector="G"
                            />
                        </filter>
                        <filter id="glass-noise">
                            <feTurbulence
                                type="fractalNoise"
                                baseFrequency="0.8"
                                numOctaves="4"
                                stitchTiles="stitch"
                                result="noise"
                            />
                            <feColorMatrix
                                type="saturate"
                                values="0"
                                in="noise"
                                result="mono"
                            />
                            <feBlend
                                in="SourceGraphic"
                                in2="mono"
                                mode="overlay"
                            />
                        </filter>
                    </defs>
                </svg>

                {/* Inner glow — bleeds through all glass panels */}
                <div
                    style={{
                        position: "absolute",
                        inset: 0,
                        background: `radial-gradient(ellipse at 50% 60%, rgba(${glowRGB},${0.18 * gi}) 0%, rgba(${glowRGB},${0.08 * gi}) 35%, rgba(${glowRGB},${0.03 * gi}) 60%, transparent 85%)`,
                        transition: "background 1.5s ease",
                        pointerEvents: "none",
                        zIndex: 0,
                    }}
                />

                {/* Glass noise texture overlay */}
                <div
                    style={{
                        position: "absolute",
                        inset: 0,
                        borderRadius: 14,
                        filter: "url(#glass-noise)",
                        opacity: 0.03,
                        background: "rgba(255,255,255,0.5)",
                        pointerEvents: "none",
                        zIndex: 100,
                        mixBlendMode: "overlay" as const,
                    }}
                />

                {/* Top highlight edge — refracted glass feel */}
                <div
                    style={{
                        position: "absolute",
                        top: 0,
                        left: 20,
                        right: 20,
                        height: 1,
                        background:
                            "linear-gradient(90deg, transparent, rgba(255,255,255,0.08) 20%, rgba(255,255,255,0.12) 50%, rgba(255,255,255,0.08) 80%, transparent)",
                        filter: "url(#liquid-edge)",
                        pointerEvents: "none",
                        zIndex: 100,
                    }}
                />

                <style>{`
                @keyframes marquee-pulse {
                    0% { border-color: rgba(${glowRGB}, 0.6); box-shadow: 0 0 8px rgba(${glowRGB},0.2), inset 0 0 15px rgba(${glowRGB},0.05); }
                    100% { border-color: rgba(${glowRGB}, 0.9); box-shadow: 0 0 12px rgba(${glowRGB},0.4), inset 0 0 25px rgba(${glowRGB},0.15); }
                }
            `}</style>

                {/* ===== VIDEO CONTAINER — base layer, clipped with own radius ===== */}
                <div
                    style={{
                        position: "absolute",
                        top: 0,
                        left: 0,
                        right: 0,
                        bottom: 0,
                        overflow: "hidden",
                        borderRadius: 12,
                        background: "rgba(10,10,14,0.2)",
                        backdropFilter: "blur(2px)",
                        border: `1px solid rgba(${glowRGB},0.1)`,
                        boxShadow: `0 0 80px rgba(${glowRGB},${0.12 * gi}), 0 8px 32px rgba(0,0,0,0.5)`,
                        transition: "all 1.5s ease, transform 0.3s ease",
                        cursor: "default",
                    }}
                    onMouseEnter={(e) => {
                        e.currentTarget.style.transform = "translateY(-2px)"
                    }}
                    onMouseLeave={(e) => {
                        e.currentTarget.style.transform = "translateY(0)"
                    }}
                >
                    {/* Video background — positioned at viz container by default, extends with sliders */}
                    <video
                        ref={videoRef}
                        src={VIDEO_URL}
                        playsInline
                        muted
                        loop
                        preload="auto"
                        style={{
                            position: "absolute",
                            top: 0,
                            left: genPanelW - videoExtendW,
                            width: `calc(100% - ${genPanelW - videoExtendW}px)`,
                            height: vizHeight + videoExtendH,
                            objectFit: "cover",
                            opacity: videoLoaded && bgReady ? cycleFade : 0,
                            transition: "opacity 1s ease",
                            pointerEvents: "none",
                        }}
                    />

                    {/* Glass blur overlays — sit between video and panels, blur the video in extended areas */}
                    {videoExtendW > 0 && (
                        <div
                            style={{
                                position: "absolute",
                                top: 0,
                                left: 0,
                                width: genPanelW,
                                height: vizHeight,
                                backdropFilter: "blur(28px) saturate(160%)",
                                WebkitBackdropFilter:
                                    "blur(28px) saturate(160%)",
                                pointerEvents: "none",
                            }}
                        />
                    )}
                    {videoExtendH > 0 && (
                        <div
                            style={{
                                position: "absolute",
                                left: 0,
                                right: 0,
                                bottom: 0,
                                height: dawHeight,
                                backdropFilter: "blur(24px) saturate(150%)",
                                WebkitBackdropFilter:
                                    "blur(24px) saturate(150%)",
                                pointerEvents: "none",
                            }}
                        />
                    )}

                    {/* Video area — top right, left edge aligns with gen panel right edge */}
                    <div
                        style={{
                            position: "absolute",
                            left: genPanelW,
                            top: 0,
                            right: 0,
                            height: vizHeight,
                            display: "flex",
                            flexDirection: "column",
                            overflow: "hidden",
                        }}
                    >
                        <div
                            style={{
                                display: "flex",
                                gap: 0,
                                borderBottom:
                                    "1px solid rgba(255,255,255,0.06)",
                                background: "rgba(6,6,12,0.25)",
                                backdropFilter: "blur(16px) saturate(140%)",
                                WebkitBackdropFilter:
                                    "blur(16px) saturate(140%)",
                            }}
                        >
                            {["Image", "FX", "Video", "MIDI", "Audio"].map(
                                (tab, i) => {
                                    const isActive = i === 2
                                    return (
                                        <div
                                            key={tab}
                                            style={{
                                                padding: "7px 14px",
                                                fontSize: 10,
                                                fontWeight: 500,
                                                color: isActive
                                                    ? `rgba(${glowRGB},1)`
                                                    : C.textMuted,
                                                borderBottom: isActive
                                                    ? `2px solid rgba(${glowRGB},1)`
                                                    : "2px solid transparent",
                                                background: isActive
                                                    ? `rgba(${glowRGB},0.06)`
                                                    : "transparent",
                                                transition: "all 0.2s ease",
                                            }}
                                        >
                                            {tab}
                                        </div>
                                    )
                                }
                            )}
                        </div>
                        <div
                            style={{
                                flex: 1,
                                position: "relative",
                                overflow: "hidden",
                            }}
                        >
                            {/* Fallback animation — shows while video is loading */}
                            {!videoLoaded && (
                                <div
                                    style={{
                                        position: "absolute",
                                        inset: 0,
                                        display: "flex",
                                        alignItems: "center",
                                        justifyContent: "center",
                                        background: `radial-gradient(ellipse at center, rgba(${glowRGB},0.08) 0%, transparent 70%)`,
                                    }}
                                >
                                    <div
                                        style={{
                                            display: "flex",
                                            flexDirection: "column",
                                            alignItems: "center",
                                            gap: 8,
                                        }}
                                    >
                                        <motion.div
                                            animate={{ rotate: 360 }}
                                            transition={{
                                                duration: 8,
                                                repeat: Infinity,
                                                ease: "linear",
                                            }}
                                            style={{
                                                width: 80,
                                                height: 80,
                                                borderRadius: "50%",
                                                border: `1px solid rgba(${glowRGB},0.25)`,
                                                background: `rgba(${glowRGB},0.04)`,
                                                backdropFilter: "blur(12px)",
                                                boxShadow: `inset 0 0 20px rgba(${glowRGB},0.08), 0 0 30px rgba(${glowRGB},0.08)`,
                                                display: "flex",
                                                alignItems: "center",
                                                justifyContent: "center",
                                            }}
                                        >
                                            <motion.div
                                                animate={{
                                                    scale: [1, 1.15, 1],
                                                }}
                                                transition={{
                                                    duration: 1.5,
                                                    repeat: Infinity,
                                                }}
                                                style={{
                                                    width: 40,
                                                    height: 40,
                                                    borderRadius: "50%",
                                                    background: `linear-gradient(135deg, rgba(${glowRGB},0.8), rgba(${glowRGB},0.5))`,
                                                    opacity: 0.6,
                                                }}
                                            />
                                        </motion.div>
                                        <span
                                            style={{
                                                fontSize: 10,
                                                color: C.textMuted,
                                            }}
                                        >
                                            Loading video...
                                        </span>
                                    </div>
                                </div>
                            )}
                        </div>
                    </div>
                </div>
                {/* ===== END VIDEO CONTAINER ===== */}

                {/* Generation Panel — floating layer, beneath DAW, above video */}
                <div
                    style={{
                        position: "absolute",
                        left: 0,
                        top: -8,
                        width: genPanelW,
                        height: vizHeight + 28 + panelExtendH,
                        zIndex: 1,
                        background: "rgba(10,10,16,0.35)",
                        backdropFilter: "blur(32px) saturate(170%)",
                        WebkitBackdropFilter: "blur(32px) saturate(170%)",
                        borderRadius: "12px 12px 0 0",
                        border: "1px solid rgba(255,255,255,0.1)",
                        borderBottom: "none",
                        boxShadow: `0 6px 24px rgba(0,0,0,0.5), 0 2px 6px rgba(0,0,0,0.4), inset 0 1px 0 rgba(255,255,255,0.08), 0 0 40px rgba(${glowRGB},0.06)`,
                        overflowX: "hidden" as const,
                        overflowY: "auto" as const,
                        padding: 14,
                        paddingBottom: 34,
                        boxSizing: "border-box",
                        display: "flex",
                        flexDirection: "column",
                        gap: 12,
                        transition: "transform 0.3s ease, box-shadow 0.3s ease",
                        cursor: "default",
                    }}
                    onMouseEnter={(e) => {
                        e.currentTarget.style.transform = "translateY(-3px)"
                        e.currentTarget.style.boxShadow = `0 10px 32px rgba(0,0,0,0.6), 0 4px 10px rgba(0,0,0,0.5), inset 0 1px 0 rgba(255,255,255,0.1), 0 0 50px rgba(${glowRGB},0.1)`
                    }}
                    onMouseLeave={(e) => {
                        e.currentTarget.style.transform = "translateY(0)"
                        e.currentTarget.style.boxShadow = `0 6px 24px rgba(0,0,0,0.5), 0 2px 6px rgba(0,0,0,0.4), inset 0 1px 0 rgba(255,255,255,0.08), 0 0 40px rgba(${glowRGB},0.06)`
                    }}
                >
                    <div
                        style={{
                            fontSize: 13,
                            fontWeight: 700,
                            color: C.text,
                            paddingBottom: 10,
                            borderBottom: "1px solid rgba(255,255,255,0.06)",
                            textShadow: `0 0 20px rgba(${glowRGB},0.15)`,
                        }}
                    >
                        Generate
                    </div>

                    <div>
                        <div
                            style={{
                                fontSize: 10,
                                color: C.textMuted,
                                marginBottom: 6,
                                fontWeight: 600,
                                textTransform: "uppercase" as const,
                                letterSpacing: 0.5,
                            }}
                        >
                            Genre
                        </div>
                        <div
                            style={{
                                display: "flex",
                                alignItems: "center",
                                gap: 4,
                            }}
                        >
                            <div
                                onClick={() => navigateGenre(-1)}
                                style={{
                                    width: 18,
                                    height: 18,
                                    borderRadius: 5,
                                    display: "flex",
                                    alignItems: "center",
                                    justifyContent: "center",
                                    cursor: isGenerating
                                        ? "default"
                                        : "pointer",
                                    color: C.textMuted,
                                    flexShrink: 0,
                                    background: "rgba(255,255,255,0.05)",
                                    backdropFilter: "blur(8px)",
                                    border: "1px solid rgba(255,255,255,0.08)",
                                    boxShadow:
                                        "inset 0 1px 0 rgba(255,255,255,0.04)",
                                }}
                            >
                                <svg width="8" height="8" viewBox="0 0 8 8">
                                    <path
                                        d="M5.5 1L2.5 4l3 3"
                                        stroke="currentColor"
                                        strokeWidth="1.5"
                                        fill="none"
                                    />
                                </svg>
                            </div>
                            <div
                                style={{
                                    flex: 1,
                                    position: "relative",
                                    height: 28,
                                    overflow: "hidden",
                                    borderRadius: 8,
                                    background: `linear-gradient(135deg, rgba(${glowRGB},0.2), rgba(${glowRGB},0.1))`,
                                    backdropFilter: "blur(12px) saturate(150%)",
                                    WebkitBackdropFilter:
                                        "blur(12px) saturate(150%)",
                                    border: "1px solid rgba(255,255,255,0.06)",
                                    boxShadow:
                                        "inset 0 1px 0 rgba(255,255,255,0.04), inset 0 -1px 0 rgba(0,0,0,0.2)",
                                    transition: "background 0.4s ease",
                                }}
                            >
                                {genres.map((g, i) => {
                                    const offset = i - selectedGenre
                                    const wrappedOffset =
                                        ((offset +
                                            genres.length +
                                            Math.floor(genres.length / 2)) %
                                            genres.length) -
                                        Math.floor(genres.length / 2)
                                    const isCenter = wrappedOffset === 0
                                    if (Math.abs(wrappedOffset) > 1) return null
                                    return (
                                        <div
                                            key={g.name}
                                            onClick={() => {
                                                if (
                                                    !isCenter &&
                                                    !isGenerating
                                                ) {
                                                    setSelectedGenre(i)
                                                    setSelectedInstruments(
                                                        new Set(
                                                            genreInstruments[
                                                                i
                                                            ] ||
                                                                genreInstruments[0]
                                                        )
                                                    )
                                                    handleGenreClick(i)
                                                    cycleStartRef.current =
                                                        Date.now() -
                                                        i * GENRE_DURATION
                                                    lastGenreRef.current = i
                                                    lastTrackGenreRef.current =
                                                        i
                                                    headerIndexRef.current =
                                                        i * 2
                                                    setHeaderIndex(i * 2)
                                                }
                                            }}
                                            style={{
                                                position: "absolute",
                                                top: 0,
                                                bottom: 0,
                                                left: `${50 + wrappedOffset * 100}%`,
                                                transform: "translateX(-50%)",
                                                display: "flex",
                                                alignItems: "center",
                                                justifyContent: "center",
                                                whiteSpace: "nowrap" as const,
                                                padding: "0 12px",
                                                fontSize: isCenter ? 11 : 9,
                                                fontWeight: isCenter
                                                    ? 700
                                                    : 500,
                                                color: isCenter
                                                    ? "#fff"
                                                    : "rgba(255,255,255,0.2)",
                                                cursor:
                                                    isCenter || isGenerating
                                                        ? "default"
                                                        : "pointer",
                                                transition: "all 0.4s ease",
                                                pointerEvents:
                                                    Math.abs(wrappedOffset) > 1
                                                        ? ("none" as const)
                                                        : ("auto" as const),
                                            }}
                                        >
                                            {g.name}
                                        </div>
                                    )
                                })}
                            </div>
                            <div
                                onClick={() => navigateGenre(1)}
                                style={{
                                    width: 18,
                                    height: 18,
                                    borderRadius: 5,
                                    display: "flex",
                                    alignItems: "center",
                                    justifyContent: "center",
                                    cursor: isGenerating
                                        ? "default"
                                        : "pointer",
                                    color: C.textMuted,
                                    flexShrink: 0,
                                    background: "rgba(255,255,255,0.05)",
                                    backdropFilter: "blur(8px)",
                                    border: "1px solid rgba(255,255,255,0.08)",
                                    boxShadow:
                                        "inset 0 1px 0 rgba(255,255,255,0.04)",
                                }}
                            >
                                <svg width="8" height="8" viewBox="0 0 8 8">
                                    <path
                                        d="M2.5 1l3 3-3 3"
                                        stroke="currentColor"
                                        strokeWidth="1.5"
                                        fill="none"
                                    />
                                </svg>
                            </div>
                        </div>
                    </div>

                    <div>
                        <div
                            style={{
                                fontSize: 10,
                                color: C.textMuted,
                                marginBottom: 6,
                                fontWeight: 600,
                                textTransform: "uppercase" as const,
                                letterSpacing: 0.5,
                            }}
                        >
                            Instruments
                        </div>
                        <div
                            style={{
                                display: "grid",
                                gridTemplateColumns: "repeat(3, 1fr)",
                                gap: 4,
                            }}
                        >
                            {Array.from(
                                genreInstruments[selectedGenre] ||
                                    genreInstruments[0]
                            ).map((idx) => {
                                const inst = panelInstruments[idx]
                                if (!inst) return null
                                const isSelected = selectedInstruments.has(idx)
                                return (
                                    <div
                                        key={inst.name}
                                        onClick={() => {
                                            if (isGenerating) return
                                            setSelectedInstruments((prev) => {
                                                const next = new Set(prev)
                                                if (next.has(idx))
                                                    next.delete(idx)
                                                else next.add(idx)
                                                return next
                                            })
                                        }}
                                        style={{
                                            display: "flex",
                                            flexDirection: "column" as const,
                                            alignItems: "center",
                                            gap: 3,
                                            padding: "6px 2px",
                                            borderRadius: 8,
                                            cursor: isGenerating
                                                ? "default"
                                                : "pointer",
                                            background: isSelected
                                                ? `linear-gradient(135deg, rgba(${glowRGB},0.2), rgba(${glowRGB},0.1))`
                                                : "rgba(255,255,255,0.03)",
                                            backdropFilter:
                                                "blur(10px) saturate(140%)",
                                            WebkitBackdropFilter:
                                                "blur(10px) saturate(140%)",
                                            border: `1px solid ${isSelected ? `rgba(${glowRGB},0.4)` : "rgba(255,255,255,0.06)"}`,
                                            boxShadow: isSelected
                                                ? `inset 0 1px 0 rgba(${glowRGB},0.15), 0 0 10px rgba(${glowRGB},0.1)`
                                                : "inset 0 1px 0 rgba(255,255,255,0.03)",
                                            color: isSelected
                                                ? "#fff"
                                                : C.textMuted,
                                            transition: "all 0.2s ease",
                                        }}
                                    >
                                        <div
                                            style={{
                                                opacity: isSelected ? 1 : 0.6,
                                            }}
                                        >
                                            {inst.icon}
                                        </div>
                                        <span
                                            style={{
                                                fontSize: 7,
                                                fontWeight: 600,
                                                letterSpacing: 0.3,
                                            }}
                                        >
                                            {inst.name}
                                        </span>
                                    </div>
                                )
                            })}
                        </div>
                    </div>

                    {/* Progress */}
                    <div
                        style={{
                            opacity: showProgress ? 1 : 0,
                            transition: "opacity 0.3s ease",
                            pointerEvents: showProgress
                                ? ("auto" as const)
                                : ("none" as const),
                            background: `rgba(${glowRGB},0.06)`,
                            backdropFilter: "blur(12px) saturate(150%)",
                            WebkitBackdropFilter: "blur(12px) saturate(150%)",
                            border: `1px solid rgba(${glowRGB},0.15)`,
                            boxShadow: `inset 0 1px 0 rgba(${glowRGB},0.08), 0 0 20px rgba(${glowRGB},0.05)`,
                            borderRadius: 10,
                            padding: 10,
                        }}
                    >
                        <div style={{ marginBottom: 5 }}>
                            <span
                                style={{
                                    fontSize: 10,
                                    color: `rgba(${glowRGB},1)`,
                                    fontWeight: 600,
                                }}
                            >
                                Generating...
                            </span>
                        </div>
                        <div
                            style={{
                                height: 5,
                                borderRadius: 3,
                                background: "rgba(255,255,255,0.04)",
                                boxShadow: "inset 0 1px 2px rgba(0,0,0,0.3)",
                                overflow: "hidden",
                            }}
                        >
                            <div
                                style={{
                                    width: `${genProgress * 100}%`,
                                    height: "100%",
                                    borderRadius: 3,
                                    background: `linear-gradient(135deg, rgba(${glowRGB},0.9), rgba(${glowRGB},0.6))`,
                                    transition: "width 0.3s ease",
                                    boxShadow: `0 0 8px rgba(${glowRGB},0.4)`,
                                }}
                            />
                        </div>
                    </div>

                    <div
                        style={{
                            padding: "10px 0",
                            borderRadius: 10,
                            textAlign: "center" as const,
                            background: isGenerating
                                ? `rgba(${glowRGB},0.12)`
                                : `linear-gradient(135deg, rgba(${glowRGB},0.85) 0%, rgba(${glowRGB},0.55) 100%)`,
                            backdropFilter: "blur(12px) saturate(160%)",
                            WebkitBackdropFilter: "blur(12px) saturate(160%)",
                            border: isGenerating
                                ? `1px solid rgba(${glowRGB},0.2)`
                                : "1px solid rgba(255,255,255,0.15)",
                            boxShadow: isGenerating
                                ? "none"
                                : `inset 0 1px 0 rgba(255,255,255,0.2), 0 4px 16px rgba(${glowRGB},0.25), 0 0 30px rgba(${glowRGB},0.1)`,
                            color: "#fff",
                            fontSize: 12,
                            fontWeight: 600,
                            cursor: isGenerating ? "default" : "pointer",
                            transition: "all 0.3s ease",
                        }}
                    >
                        {isGenerating ? "Generating..." : "Generate"}
                    </div>
                </div>

                {/* ===== DAW ===== top floating layer, wider than video container */}
                <div
                    style={{
                        position: "absolute",
                        left: -10,
                        right: -10,
                        bottom: -4,
                        height: dawHeight + 12,
                        zIndex: 2,
                        display: "flex",
                        flexDirection: "column",
                        background: "rgba(6,6,10,0.3)",
                        backdropFilter: "blur(28px) saturate(170%)",
                        WebkitBackdropFilter: "blur(28px) saturate(170%)",
                        borderRadius: "16px 16px 0 0",
                        border: "1px solid rgba(255,255,255,0.12)",
                        borderBottom: "none",
                        boxShadow: `0 -10px 40px rgba(0,0,0,0.6), 0 -3px 12px rgba(0,0,0,0.5), inset 0 1px 0 rgba(255,255,255,0.1), 0 0 50px rgba(${glowRGB},0.08)`,
                        transition: "transform 0.3s ease, box-shadow 0.3s ease",
                        cursor: "default",
                    }}
                    onMouseEnter={(e) => {
                        e.currentTarget.style.transform = "translateY(-3px)"
                        e.currentTarget.style.boxShadow = `0 -14px 48px rgba(0,0,0,0.7), 0 -4px 16px rgba(0,0,0,0.6), inset 0 1px 0 rgba(255,255,255,0.12), 0 0 60px rgba(${glowRGB},0.12)`
                    }}
                    onMouseLeave={(e) => {
                        e.currentTarget.style.transform = "translateY(0)"
                        e.currentTarget.style.boxShadow = `0 -10px 40px rgba(0,0,0,0.6), 0 -3px 12px rgba(0,0,0,0.5), inset 0 1px 0 rgba(255,255,255,0.1), 0 0 50px rgba(${glowRGB},0.08)`
                    }}
                >
                    {/* Transport controls */}
                    <div
                        style={{
                            height: controlsH,
                            minHeight: controlsH,
                            display: "flex",
                            alignItems: "center",
                            padding: "0 10px",
                            background: "rgba(8,8,14,0.15)",
                            backdropFilter: "blur(24px) saturate(170%)",
                            WebkitBackdropFilter: "blur(24px) saturate(170%)",
                            borderBottom: "1px solid rgba(255,255,255,0.08)",
                            borderRadius: "16px 16px 0 0",
                            boxShadow:
                                "inset 0 1px 0 rgba(255,255,255,0.05), inset 0 -1px 0 rgba(255,255,255,0.03)",
                            gap: 6,
                        }}
                    >
                        <motion.button
                            onClick={handlePlay}
                            whileHover={{ scale: 1.08 }}
                            whileTap={{ scale: 0.92 }}
                            style={{
                                width: 26,
                                height: 26,
                                borderRadius: 6,
                                border: `1px solid ${isPlaying ? `rgba(${glowRGB},0.5)` : "rgba(255,255,255,0.1)"}`,
                                background: isPlaying
                                    ? `rgba(${glowRGB},0.2)`
                                    : "rgba(255,255,255,0.06)",
                                backdropFilter: "blur(12px)",
                                boxShadow: isPlaying
                                    ? `inset 0 1px 0 rgba(${glowRGB},0.2), 0 0 12px rgba(${glowRGB},0.15)`
                                    : "inset 0 1px 0 rgba(255,255,255,0.06)",
                                color: isPlaying ? "#fff" : "#ccc",
                                cursor: "pointer",
                                display: "flex",
                                alignItems: "center",
                                justifyContent: "center",
                                outline: "none",
                                padding: 0,
                            }}
                        >
                            {isPlaying ? Icons.pause : Icons.play}
                        </motion.button>
                        <div
                            style={{
                                width: 26,
                                height: 26,
                                borderRadius: 6,
                                border: "1px solid rgba(255,255,255,0.1)",
                                background: "rgba(255,255,255,0.06)",
                                backdropFilter: "blur(12px)",
                                boxShadow:
                                    "inset 0 1px 0 rgba(255,255,255,0.06)",
                                color: "#ccc",
                                display: "flex",
                                alignItems: "center",
                                justifyContent: "center",
                            }}
                        >
                            {Icons.stop}
                        </div>
                        <div
                            style={{
                                color: "#fff",
                                fontSize: 11,
                                fontWeight: 600,
                                minWidth: 42,
                                padding: "0 6px",
                                background: "rgba(6,6,12,0.2)",
                                backdropFilter: "blur(14px)",
                                border: "1px solid rgba(255,255,255,0.08)",
                                boxShadow:
                                    "inset 0 1px 0 rgba(255,255,255,0.04)",
                                borderRadius: 6,
                                height: 26,
                                display: "flex",
                                alignItems: "center",
                                justifyContent: "center",
                                fontVariantNumeric: "tabular-nums",
                            }}
                        >
                            {timeDisplay}
                        </div>
                        {/* Volume slider */}
                        <div
                            style={{
                                display: "flex",
                                alignItems: "center",
                                gap: 5,
                                marginLeft: 2,
                            }}
                        >
                            <svg
                                width="12"
                                height="12"
                                viewBox="0 0 24 24"
                                fill="none"
                                stroke="rgba(255,255,255,0.4)"
                                strokeWidth="2"
                            >
                                <path d="M11 5L6 9H2v6h4l5 4V5z" />
                                <path d="M19.07 4.93a10 10 0 010 14.14M15.54 8.46a5 5 0 010 7.07" />
                            </svg>
                            <div
                                style={{
                                    width: 60,
                                    height: 4,
                                    borderRadius: 2,
                                    background: "rgba(255,255,255,0.08)",
                                    overflow: "hidden",
                                }}
                            >
                                <div
                                    style={{
                                        width: "80%",
                                        height: "100%",
                                        borderRadius: 2,
                                        background: "rgba(255,255,255,0.3)",
                                    }}
                                />
                            </div>
                        </div>
                        <div style={{ flex: 1 }} />
                        <div
                            style={{
                                padding: "3px 8px",
                                borderRadius: 6,
                                fontSize: 10,
                                background: "rgba(255,255,255,0.04)",
                                backdropFilter: "blur(10px)",
                                border: "1px solid rgba(255,255,255,0.08)",
                                boxShadow:
                                    "inset 0 1px 0 rgba(255,255,255,0.04)",
                                color: C.textMuted,
                                fontVariantNumeric: "tabular-nums",
                            }}
                        >
                            120 BPM
                        </div>
                        <div style={{ display: "flex", gap: 3 }}>
                            {["-", "+"].map((z) => (
                                <div
                                    key={z}
                                    style={{
                                        width: 22,
                                        height: 22,
                                        borderRadius: 3,
                                        color: C.textMuted,
                                        display: "flex",
                                        alignItems: "center",
                                        justifyContent: "center",
                                        fontSize: 12,
                                    }}
                                >
                                    {z}
                                </div>
                            ))}
                        </div>
                    </div>

                    {/* Timeline */}
                    <div
                        style={{
                            height: timelineH,
                            minHeight: timelineH,
                            display: "flex",
                            borderBottom: "1px solid rgba(255,255,255,0.06)",
                            background: "rgba(6,6,12,0.15)",
                            backdropFilter: "blur(20px) saturate(150%)",
                            WebkitBackdropFilter: "blur(20px) saturate(150%)",
                        }}
                    >
                        <div
                            style={{
                                width: labelW,
                                minWidth: labelW,
                                background: "rgba(6,6,12,0.15)",
                                borderRight: "1px solid rgba(255,255,255,0.06)",
                                display: "flex",
                                alignItems: "center",
                                paddingLeft: 10,
                            }}
                        >
                            <span style={{ fontSize: 9, color: C.textDim }}>
                                + Add Track
                            </span>
                        </div>
                        <div
                            style={{
                                flex: 1,
                                position: "relative",
                                background: "rgba(6,6,10,0.1)",
                                cursor: "pointer",
                            }}
                            onClick={(e) => {
                                const rect =
                                    e.currentTarget.getBoundingClientRect()
                                const frac =
                                    (e.clientX - rect.left) / rect.width
                                seekTo(frac)
                            }}
                        >
                            {ticks.map((tick) => (
                                <div
                                    key={tick.time}
                                    style={{
                                        position: "absolute",
                                        left: tick.pct,
                                        top: 0,
                                        height: "100%",
                                    }}
                                >
                                    <span
                                        style={{
                                            position: "absolute",
                                            top: 5,
                                            left: 3,
                                            color: C.textDim,
                                            fontSize: 8,
                                            whiteSpace: "nowrap",
                                        }}
                                    >
                                        {tick.time}s
                                    </span>
                                    <div
                                        style={{
                                            position: "absolute",
                                            bottom: 0,
                                            left: 0,
                                            width: 1,
                                            height: 8,
                                            background: "#333",
                                        }}
                                    />
                                </div>
                            ))}
                        </div>
                    </div>

                    {/* Track area — drag-to-create target */}
                    <div
                        ref={trackAreaRef}
                        onMouseDown={handleTrackAreaMouseDown}
                        style={{
                            flex: 1,
                            position: "relative",
                            overflow: "hidden",
                            cursor: isDragging ? "crosshair" : "crosshair",
                        }}
                    >
                        {/* Row backgrounds + labels (one per unique row) */}
                        {Array.from(new Set(sortedTracks.map((t) => t.row)))
                            .sort((a, b) => a - b)
                            .map((row) => {
                                const rowTracks = sortedTracks.filter(
                                    (t) => t.row === row
                                )
                                const primary =
                                    rowTracks.find((t) => !t.isPlaceholder) ||
                                    rowTracks[0]
                                if (!primary) return null
                                return (
                                    <div
                                        key={`row-${row}`}
                                        style={{
                                            position: "absolute",
                                            top: row * trackHeight,
                                            left: 0,
                                            right: 0,
                                            height: trackHeight,
                                            borderBottom:
                                                "1px solid rgba(255,255,255,0.04)",
                                            zIndex: 1,
                                        }}
                                    >
                                        <div
                                            style={{
                                                position: "absolute",
                                                left: 0,
                                                top: 0,
                                                bottom: 0,
                                                width: labelW,
                                                display: "flex",
                                                alignItems: "center",
                                                padding: "0 6px",
                                                gap: 6,
                                                background: "rgba(8,8,14,0.2)",
                                                backdropFilter:
                                                    "blur(20px) saturate(150%)",
                                                WebkitBackdropFilter:
                                                    "blur(20px) saturate(150%)",
                                                borderRight:
                                                    "1px solid rgba(255,255,255,0.06)",
                                                boxShadow:
                                                    "inset -1px 0 0 rgba(255,255,255,0.03)",
                                            }}
                                        >
                                            <div
                                                style={{
                                                    width: 4,
                                                    height: 38,
                                                    borderRadius: 2,
                                                    background: `linear-gradient(180deg, rgba(${glowRGB},0.8), rgba(${glowRGB},0.4))`,
                                                }}
                                            />
                                            {primary.isPlaceholder ? (
                                                <motion.span
                                                    animate={{
                                                        opacity: [1, 0.4, 1],
                                                    }}
                                                    transition={{
                                                        duration: 1.2,
                                                        repeat: Infinity,
                                                        ease: "easeInOut",
                                                    }}
                                                    style={{
                                                        color: `rgba(${glowRGB},1)`,
                                                        fontSize: 11,
                                                        fontWeight: 600,
                                                        whiteSpace: "nowrap",
                                                    }}
                                                >
                                                    {primary.name}
                                                    {[0, 1, 2].map((i) => (
                                                        <motion.span
                                                            key={i}
                                                            animate={{
                                                                opacity: [
                                                                    0.2, 1, 0.2,
                                                                ],
                                                            }}
                                                            transition={{
                                                                duration: 1.2,
                                                                repeat: Infinity,
                                                                delay: i * 0.2,
                                                            }}
                                                        >
                                                            .
                                                        </motion.span>
                                                    ))}
                                                </motion.span>
                                            ) : (
                                                <>
                                                    <div
                                                        style={{
                                                            width: 30,
                                                            height: 30,
                                                            borderRadius: 6,
                                                            display: "flex",
                                                            alignItems:
                                                                "center",
                                                            justifyContent:
                                                                "center",
                                                            color: C.text,
                                                            opacity: 0.9,
                                                        }}
                                                    >
                                                        {primary.icon}
                                                    </div>
                                                    <div
                                                        style={{
                                                            flex: 1,
                                                            minWidth: 0,
                                                        }}
                                                    >
                                                        <div
                                                            style={{
                                                                fontSize: 11,
                                                                fontWeight: 600,
                                                                color: C.text,
                                                                whiteSpace:
                                                                    "nowrap",
                                                                overflow:
                                                                    "hidden",
                                                                textOverflow:
                                                                    "ellipsis",
                                                            }}
                                                        >
                                                            {primary.name}
                                                        </div>
                                                        <div
                                                            style={{
                                                                display: "flex",
                                                                gap: 3,
                                                                marginTop: 3,
                                                            }}
                                                        >
                                                            <div
                                                                onClick={() =>
                                                                    setMutedRows(
                                                                        (
                                                                            prev
                                                                        ) => {
                                                                            const next =
                                                                                new Set(
                                                                                    prev
                                                                                )
                                                                            if (
                                                                                next.has(
                                                                                    row
                                                                                )
                                                                            )
                                                                                next.delete(
                                                                                    row
                                                                                )
                                                                            else
                                                                                next.add(
                                                                                    row
                                                                                )
                                                                            return next
                                                                        }
                                                                    )
                                                                }
                                                                style={{
                                                                    width: 16,
                                                                    height: 14,
                                                                    borderRadius: 3,
                                                                    border: `1px solid ${mutedRows.has(row) ? "rgba(220,180,50,0.4)" : "rgba(255,255,255,0.08)"}`,
                                                                    background:
                                                                        mutedRows.has(
                                                                            row
                                                                        )
                                                                            ? "rgba(220,180,50,0.25)"
                                                                            : "transparent",
                                                                    color: mutedRows.has(
                                                                        row
                                                                    )
                                                                        ? "rgba(220,180,50,1)"
                                                                        : "rgba(255,255,255,0.3)",
                                                                    fontSize: 7,
                                                                    fontWeight: 700,
                                                                    cursor: "pointer",
                                                                    display:
                                                                        "flex",
                                                                    alignItems:
                                                                        "center",
                                                                    justifyContent:
                                                                        "center",
                                                                }}
                                                            >
                                                                M
                                                            </div>
                                                            <div
                                                                onClick={() =>
                                                                    setSoloRow(
                                                                        (
                                                                            prev
                                                                        ) =>
                                                                            prev ===
                                                                            row
                                                                                ? null
                                                                                : row
                                                                    )
                                                                }
                                                                style={{
                                                                    width: 16,
                                                                    height: 14,
                                                                    borderRadius: 3,
                                                                    border: `1px solid ${soloRow === row ? "rgba(80,180,255,0.4)" : "rgba(255,255,255,0.08)"}`,
                                                                    background:
                                                                        soloRow ===
                                                                        row
                                                                            ? "rgba(80,180,255,0.25)"
                                                                            : "transparent",
                                                                    color:
                                                                        soloRow ===
                                                                        row
                                                                            ? "rgba(80,180,255,1)"
                                                                            : "rgba(255,255,255,0.3)",
                                                                    fontSize: 7,
                                                                    fontWeight: 700,
                                                                    cursor: "pointer",
                                                                    display:
                                                                        "flex",
                                                                    alignItems:
                                                                        "center",
                                                                    justifyContent:
                                                                        "center",
                                                                }}
                                                            >
                                                                S
                                                            </div>
                                                            <div
                                                                style={{
                                                                    flex: 1,
                                                                    height: 3,
                                                                    borderRadius: 2,
                                                                    background:
                                                                        "rgba(255,255,255,0.06)",
                                                                    marginTop: 5,
                                                                    marginLeft: 4,
                                                                    overflow:
                                                                        "hidden",
                                                                }}
                                                            >
                                                                <div
                                                                    style={{
                                                                        width: "75%",
                                                                        height: "100%",
                                                                        borderRadius: 2,
                                                                        background:
                                                                            "rgba(255,255,255,0.2)",
                                                                    }}
                                                                />
                                                            </div>
                                                        </div>
                                                    </div>
                                                </>
                                            )}
                                        </div>
                                        <div
                                            style={{
                                                position: "absolute",
                                                left: labelW,
                                                top: 0,
                                                right: 0,
                                                bottom: 0,
                                                background: "rgba(6,6,10,0.15)",
                                                backdropFilter:
                                                    "blur(20px) saturate(150%)",
                                                WebkitBackdropFilter:
                                                    "blur(20px) saturate(150%)",
                                            }}
                                        />
                                    </div>
                                )
                            })}

                        {/* Waveform area — percentage-based so timeline always fills */}
                        <div
                            ref={waveAreaRef}
                            style={{
                                position: "absolute",
                                left: labelW,
                                top: 0,
                                right: 0,
                                bottom: 0,
                                zIndex: 2,
                            }}
                        >
                            {/* Grid lines */}
                            {ticks.map((tick) => (
                                <div
                                    key={`g${tick.time}`}
                                    style={{
                                        position: "absolute",
                                        top: 0,
                                        left: tick.pct,
                                        width: 1,
                                        height: "100%",
                                        background: "rgba(255,255,255,0.025)",
                                        pointerEvents: "none",
                                    }}
                                />
                            ))}

                            {/* Clips — percentage positioned, fade with cycle */}
                            <div style={{ opacity: cycleFade }}>
                                <AnimatePresence>
                                    {sortedTracks.map((track) => {
                                        // Color logic — renderer determines all colors from activeTrackGenre:
                                        // - Placeholders: dynamic glow (current visual color during loading)
                                        // Mute/solo override: forced grey regardless of genre
                                        const isMutedOrSuppressed =
                                            mutedRows.has(track.row) ||
                                            (soloRow !== null &&
                                                track.row !== soloRow)
                                        // Genre-based active check
                                        const isActive =
                                            !isMutedOrSuppressed &&
                                            (track.isPlaceholder ||
                                                (track.genreClip &&
                                                    track.genreIndex ===
                                                        activeTrackGenre) ||
                                                (!track.genreClip &&
                                                    activeTrackGenre === 0))
                                        let tColor: string
                                        let tColorBg: string

                                        const shadeShifts = [
                                            [0, 0, 0], // base
                                            [30, -20, 20], // warmer
                                            [-20, 20, 30], // cooler
                                            [20, 10, -30], // shifted
                                            [-10, 30, -10], // green-tinted
                                            [30, -10, -20], // red-tinted
                                        ]

                                        if (isActive) {
                                            // Placeholders use live glow; completed tracks use static genre glow
                                            // so CSS transition handles color fade in sync with grey fade
                                            const glow = track.isPlaceholder
                                                ? glowRGB
                                                : genres[activeTrackGenre]
                                                      ?.glow || "100,160,255"
                                            const [gr, gg, gb] = glow
                                                .split(",")
                                                .map(Number)
                                            const shift =
                                                shadeShifts[
                                                    track.row %
                                                        shadeShifts.length
                                                ]
                                            const cr = Math.max(
                                                0,
                                                Math.min(255, gr + shift[0])
                                            )
                                            const cg = Math.max(
                                                0,
                                                Math.min(255, gg + shift[1])
                                            )
                                            const cb = Math.max(
                                                0,
                                                Math.min(255, gb + shift[2])
                                            )
                                            tColor = `rgba(${cr},${cg},${cb},0.7)`
                                            tColorBg = `rgba(${cr},${cg},${cb},0.06)`
                                        } else {
                                            // Inactive: grey
                                            tColor = greyColor
                                            tColorBg = greyColorBg
                                        }
                                        return (
                                            <motion.div
                                                key={track.id}
                                                initial={{ scale: 0.92, y: 6 }}
                                                animate={{ scale: 1, y: 0 }}
                                                transition={{
                                                    duration: 0.4,
                                                    ease: [0.2, 0.8, 0.2, 1],
                                                }}
                                                whileHover={
                                                    track.isPlaceholder
                                                        ? {}
                                                        : {
                                                              scale: 1.012,
                                                              y: -1.5,
                                                              transition: {
                                                                  duration: 0.2,
                                                              },
                                                          }
                                                }
                                                style={{
                                                    position: "absolute",
                                                    top:
                                                        track.row *
                                                            trackHeight +
                                                        3,
                                                    left: `calc(${track.startFrac * 100}% + 3px)`,
                                                    width: `calc(${track.widthFrac * 100}% - 6px)`,
                                                    height: trackHeight - 6,
                                                    zIndex: 2,
                                                    overflow: "hidden",
                                                    borderRadius: 8,
                                                    transformStyle:
                                                        "preserve-3d" as const,
                                                    perspective: 800,
                                                }}
                                            >
                                                {/* Container — transitions from plain dark (placeholder) to themed glass (completed) */}
                                                <div
                                                    style={{
                                                        position: "absolute",
                                                        inset: 0,
                                                        borderRadius: 8,
                                                        ...(isActive ? {
                                                            background: track.isPlaceholder
                                                                ? `rgba(${glowRGB},0.06)`
                                                                : `linear-gradient(135deg, ${tColor.replace("0.7", "0.38")} 0%, ${tColor.replace("0.7", "0.20")} 40%, ${tColor.replace("0.7", "0.14")} 60%, ${tColor.replace("0.7", "0.30")} 100%)`,
                                                            backdropFilter: track.isPlaceholder
                                                                ? "none"
                                                                : "blur(6px) saturate(200%) brightness(1.15)",
                                                            border: track.isPlaceholder
                                                                ? `1px solid rgba(${glowRGB},0.12)`
                                                                : `1px solid ${tColor.replace("0.7", "0.3")}`,
                                                            borderLeft: track.isPlaceholder
                                                                ? `1px solid rgba(${glowRGB},0.12)`
                                                                : `2px solid ${tColor.replace("0.7", "0.55")}`,
                                                            borderTop: track.isPlaceholder
                                                                ? `1px solid rgba(${glowRGB},0.12)`
                                                                : `1px solid ${tColor.replace("0.7", "0.35")}`,
                                                            boxShadow: track.isPlaceholder
                                                                ? `0 4px 16px rgba(0,0,0,0.3), 0 1px 3px rgba(0,0,0,0.2), inset 0 1px 0 rgba(255,255,255,0.06), 0 0 20px ${tColor.replace("0.7", "0.06")}`
                                                                : `0 4px 16px rgba(0,0,0,0.35), 0 1px 3px rgba(0,0,0,0.25), inset 0 1px 0 rgba(255,255,255,0.12), inset 0 -1px 0 rgba(0,0,0,0.1), inset 0 0 30px ${tColor.replace("0.7", "0.08")}, 0 0 24px ${tColor.replace("0.7", "0.1")}`,
                                                        } : {
                                                            background: `rgba(100,100,100,0.06)`,
                                                            border: `1px solid rgba(100,100,100,0.12)`,
                                                            boxShadow: `0 4px 16px rgba(0,0,0,0.3), 0 1px 3px rgba(0,0,0,0.2), inset 0 1px 0 rgba(255,255,255,0.06)`,
                                                        }),
                                                        transition: "background 0.6s ease, border 0.6s ease, border-left 0.6s ease, border-top 0.6s ease, box-shadow 0.6s ease, backdrop-filter 0.6s ease",
                                                        display: "flex",
                                                        alignItems: "center",
                                                        padding: "0 4px",
                                                    }}
                                                >
                                                    {track.isPlaceholder ? (
                                                        <NoiseWaveform
                                                            width={Math.max(
                                                                50,
                                                                track.widthFrac *
                                                                    measuredWaveW -
                                                                    14
                                                            )}
                                                            height={
                                                                trackHeight - 22
                                                            }
                                                            color={isActive ? tColor : greyColor}
                                                        />
                                                    ) : (
                                                        <WaveformSVG
                                                            color={isActive ? "rgba(255,255,255,0.85)" : greyColor}
                                                            seed={
                                                                track.seed
                                                            }
                                                            width={Math.max(
                                                                50,
                                                                track.widthFrac *
                                                                    measuredWaveW -
                                                                    14
                                                            )}
                                                            height={
                                                                trackHeight -
                                                                22
                                                            }
                                                        />
                                                    )}
                                                </div>
                                            </motion.div>
                                        )
                                    })}
                                </AnimatePresence>
                            </div>

                            {/* Playhead */}
                            {(isPlaying || playheadProgress > 0) && (
                                <>
                                    <div
                                        style={{
                                            position: "absolute",
                                            top: 0,
                                            left: `${playheadProgress * 100}%`,
                                            width: 1.5,
                                            height: "100%",
                                            background: "#fff",
                                            boxShadow:
                                                "0 0 8px rgba(255,255,255,0.25)",
                                            zIndex: 50,
                                            pointerEvents: "none",
                                        }}
                                    />
                                    {/* Draggable playhead handle — wide hit area */}
                                    <div
                                        style={{
                                            position: "absolute",
                                            top: -4,
                                            left: `calc(${playheadProgress * 100}% - 10px)`,
                                            width: 20,
                                            height: 16,
                                            zIndex: 52,
                                            cursor: "ew-resize",
                                            display: "flex",
                                            alignItems: "flex-start",
                                            justifyContent: "center",
                                        }}
                                        onMouseDown={(e) => {
                                            e.preventDefault()
                                            e.stopPropagation()
                                            seekDragRef.current = true
                                            const waveArea = waveAreaRef.current
                                            if (waveArea)
                                                waveAreaRectRef.current =
                                                    waveArea.getBoundingClientRect()
                                        }}
                                    >
                                        <div
                                            style={{
                                                width: 0,
                                                height: 0,
                                                borderLeft:
                                                    "5px solid transparent",
                                                borderRight:
                                                    "5px solid transparent",
                                                borderTop: "8px solid #fff",
                                                marginTop: 3,
                                            }}
                                        />
                                    </div>
                                </>
                            )}
                        </div>

                        {/* Empty state hint */}
                        {tracks.length === 0 && !isDragging && (
                            <div
                                style={{
                                    display: "flex",
                                    alignItems: "center",
                                    justifyContent: "center",
                                    height: "100%",
                                    color: "#2a2a2a",
                                    fontSize: 12,
                                    flexDirection: "column",
                                    gap: 6,
                                    pointerEvents: "none",
                                }}
                            >
                                <div style={{ opacity: 0.5 }}>{Icons.wand}</div>
                                <span>Drag to create tracks</span>
                            </div>
                        )}

                        {/* Marquee selection box */}
                        {marqueeBox && (
                            <div
                                style={{
                                    position: "absolute",
                                    left: marqueeBox.left,
                                    top: marqueeBox.top,
                                    width: marqueeBox.width,
                                    height: marqueeBox.height,
                                    background: `rgba(${glowRGB}, 0.1)`,
                                    backdropFilter: "blur(4px) saturate(130%)",
                                    border: `1.5px solid rgba(${glowRGB}, 0.6)`,
                                    borderRadius: 4,
                                    boxShadow: `inset 0 0 20px rgba(${glowRGB},0.08), 0 0 16px rgba(${glowRGB},0.15)`,
                                    pointerEvents: "none",
                                    zIndex: 1000,
                                    animation:
                                        "marquee-pulse 0.8s ease-in-out infinite alternate",
                                }}
                            />
                        )}
                    </div>
                </div>
                {/* ===== END DAW ===== */}
            </motion.div>
        </div>
    )
}

addPropertyControls(DAW2Animation, {
    trackDuration: {
        type: ControlType.Number,
        title: "Duration (s)",
        defaultValue: 30,
        min: 5,
        max: 120,
        step: 1,
    },
    glowIntensity: {
        type: ControlType.Number,
        title: "Glow Intensity",
        defaultValue: 100,
        min: 0,
        max: 200,
        step: 5,
    },
    panelWidth: {
        type: ControlType.Number,
        title: "Panel Width",
        defaultValue: 210,
        min: 120,
        max: 350,
        step: 5,
    },
    videoExtendW: {
        type: ControlType.Number,
        title: "Vid Extend W",
        defaultValue: 0,
        min: 0,
        max: 400,
        step: 10,
    },
    videoExtendH: {
        type: ControlType.Number,
        title: "Vid Extend H",
        defaultValue: 0,
        min: 0,
        max: 400,
        step: 10,
    },
    swayIntensity: {
        type: ControlType.Number,
        title: "Sway",
        defaultValue: 6,
        min: 0,
        max: 30,
        step: 1,
    },
    panelExtendH: {
        type: ControlType.Number,
        title: "Panel Extend H",
        defaultValue: 60,
        min: -100,
        max: 300,
        step: 5,
    },
    bgVideoOpacity: {
        type: ControlType.Number,
        title: "BG Vid Opacity",
        defaultValue: 50,
        min: 0,
        max: 100,
        step: 5,
    },
    bgVideoScale: {
        type: ControlType.Number,
        title: "BG Vid Scale",
        defaultValue: 100,
        min: 50,
        max: 300,
        step: 5,
    },
    width: {
        type: ControlType.Number,
        title: "Width",
        defaultValue: 1200,
        min: 600,
        max: 1800,
    },
    height: {
        type: ControlType.Number,
        title: "Height",
        defaultValue: 640,
        min: 400,
        max: 1000,
    },
})
---

## 11. AUDIO SYSTEM

### 11.1 — Audio Architecture

```cpp
// Source/PastaWarfare/Audio/PastaAudioManager.h
#pragma once

#include "CoreMinimal.h"
#include "Subsystems/WorldSubsystem.h"
#include "PastaAudioManager.generated.h"

UENUM(BlueprintType)
enum class EMusicState : uint8
{
    MainMenu,
    Lobby,
    BusFlight,
    EarlyGame,      // Calm ambient
    MidGame,        // Building tension
    LateGame,       // Intense combat music
    FinalCircle,    // Maximum intensity
    Victory,
    Elimination     // Death sting
};

UCLASS()
class PASTAWARFARE_API UPastaAudioManager : public UWorldSubsystem
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintCallable, Category = "Audio")
    void SetMusicState(EMusicState NewState);

    UFUNCTION(BlueprintCallable, Category = "Audio")
    void PlayStinger(const FString& StingerName);

    UFUNCTION(BlueprintCallable, Category = "Audio")
    void SetAmbientBiome(const FString& BiomeType);

protected:
    UPROPERTY(EditDefaultsOnly, Category = "Audio|Music")
    TMap<EMusicState, USoundBase*> MusicTracks;

    UPROPERTY(EditDefaultsOnly, Category = "Audio|Stingers")
    TMap<FString, USoundBase*> Stingers;

    UPROPERTY()
    UAudioComponent* MusicComponent;

    UPROPERTY()
    UAudioComponent* AmbientComponent;

    float CurrentMusicIntensity;
    EMusicState CurrentState;
};
```

### 11.2 — Sound Design Specifications

```
AUDIO DESIGN DOCUMENT:

=== WEAPON SOUNDS ===
All weapon sounds must incorporate pasta-related foley:

Penne Puncher (Assault Rifle):
  Fire: Crisp snap + hollow thunk (penne tube popping) + mechanical cycling
  Reload: Clinking of penne tubes sliding into magazine + snap click
  Impact: Ceramic/pasta cracking on surface

Farfalle Flinger (Shotgun):
  Fire: Deep boom + flutter of pasta wings + scattered impact
  Reload: Sliding shell sound + pasta shuffle
  Pump: Wet pasta squish + mechanical rack

Linguine Longshot (Sniper):
  Fire: Sharp whip crack (linguine cutting air) + reverb tail
  Bolt action: Sliding flat noodle sound
  Bullet flyby: High-pitched whistle + noodle flutter

Lasagna Launcher (RPG):
  Fire: Heavy wet THWUMP + sizzle of hot cheese
  Projectile flight: Whistling + bubbling sauce sound
  Explosion: Splat + boom + cheese sizzle decay

Spaghetti Whip (Melee):
  Swing: Wet noodle whip crack
  Hit: Slap + squelch
  Miss: Whoosh with noodle flutter

=== CHARACTER SOUNDS ===
Per-pasta type footsteps and movement:

Spaghetti: Wet squish + light noodle rustle
Penne: Hollow clicking/clacking (like bamboo)
Farfalle: Light fluttery rustle
Rigatoni: Heavy hollow thuds (deep resonance)
Fusilli: Springy spiral bounce sounds
Lasagna: Wet layered squelch + heavy step
Ravioli: Soft puffy pillow sounds
Macaroni: Curved tube clinking
Linguine: Flat ribbon slap sounds
Tortellini: Round rolling sounds
Orzo: Cascading tiny grain sounds (like sand/rice)

=== AMBIENT BIOME SOUNDS ===
Desert (Iraq/Syria/Yemen/Sudan):
  - Wind through ruins, distant muezzin call, sand granules
  - Occasional helicopter/vehicle in distance
  - Crackling fires, settling rubble

Temperate (Ukraine):
  - Wind through trees, distant birds, river flow
  - Occasional distant artillery (non-directional ambient)
  - Creaking structures, wind howl

Tropical (Myanmar/Somalia):
  - Insects, tropical birds, waves (coastal)
  - Dense vegetation rustle, rain drips
  - Distant city sounds

Highland (Ethiopia):
  - Mountain wind, sparse bird calls
  - Distant village sounds, livestock
  - Echo effects from valleys

=== MUSIC SYSTEM ===
Adaptive music using horizontal layering (Wwise-style in UE5):

Base Layer: Always playing ambient pad per biome
Rhythm Layer: Fades in during mid-game, percussion based on region
  - Middle East zones: Frame drum / darbuka patterns
  - Eastern European: Martial snare patterns
  - African: Djembe and talking drum
  - Asian: Taiko-influenced patterns
Melody Layer: Fades in during late game, Italian-inspired motifs
  - Accordion/mandolin melodic fragments
  - Operatic vocal stabs for elimination streaks
Intensity Layer: Swells during final circle
  - Full orchestral surge
  - Choir stabs on eliminations

Victory Music: Triumphant Italian-themed orchestral piece
  - Mandolins, accordion, strings, brass
  - "That's-a spicy meatball!" voice line at match win

=== UI SOUNDS ===
Menu navigation: Soft pasta snap clicks
Button hover: Light noodle squish
Button press: Satisfying pasta crunch
Match found: Italian kitchen bell + sizzle
Elimination popup: Quick orchestral hit + pasta snap
Supply drop incoming: Descending whistle + pot lid clang
Storm warning: Deep rumbling + boiling water sounds
Chest opening: Creaking + pasta tumbling out
```

### 11.3 — AI Audio Generation

```python
# Scripts/AssetGen/audio_generator.py
"""
Generate placeholder audio using AI audio generation APIs.
Final audio should be produced by a sound designer, but this provides
functional placeholders for development.

Options:
  - Stability Audio API (stability.ai)
  - ElevenLabs SFX generation
  - Freesound API for Creative Commons samples
"""

import requests
import os

ELEVENLABS_API_KEY = os.environ.get("ELEVENLABS_API_KEY")

def generate_sfx_elevenlabs(prompt: str, output_path: str, 
                             duration_seconds: float = 2.0) -> str:
    """Generate sound effect using ElevenLabs Sound Effects API."""
    headers = {
        "xi-api-key": ELEVENLABS_API_KEY,
        "Content-Type": "application/json"
    }
    
    payload = {
        "text": prompt,
        "duration_seconds": duration_seconds,
        "prompt_influence": 0.5
    }
    
    response = requests.post(
        "https://api.elevenlabs.io/v1/sound-generation",
        headers=headers,
        json=payload
    )
    
    os.makedirs(os.path.dirname(output_path), exist_ok=True)
    with open(output_path, 'wb') as f:
        f.write(response.content)
    
    return output_path

# Generate all game sound effects
SOUND_EFFECTS = {
    # Weapon fire sounds
    "SFX_Penne_Puncher_Fire": ("Rapid hollow tube popping sound, crisp mechanical, assault rifle pasta weapon fire, game sound effect", 0.3),
    "SFX_Farfalle_Flinger_Fire": ("Deep boom with fluttering, shotgun blast with pasta scatter, game sound effect", 0.4),
    "SFX_Linguine_Longshot_Fire": ("Sharp whip crack with reverb, high velocity snap, sniper rifle shot, game sound", 0.5),
    "SFX_Lasagna_Launcher_Fire": ("Heavy wet thump with sizzle, rocket launcher fire, game sound effect", 0.6),
    "SFX_Orzo_Obliterator_Fire": ("Rapid tiny pellet spray, cascading grain fire, SMG burst, game sound", 0.2),
    "SFX_Fusilli_Fury_Fire": ("Double barrel boom with spiral whoosh, heavy shotgun, game sound", 0.5),
    "SFX_Spaghetti_Whip_Swing": ("Wet noodle whip crack, melee swing, game sound effect", 0.4),
    
    # Impact sounds
    "SFX_Pasta_Impact_Light": ("Ceramic cracking, light pasta breaking on surface, game impact sound", 0.3),
    "SFX_Pasta_Impact_Heavy": ("Heavy pasta shattering, explosive splat, game impact sound", 0.5),
    "SFX_Sauce_Splat": ("Wet sauce splatter on surface, tomato sauce impact, game sound", 0.3),
    "SFX_Cheese_Sizzle": ("Hot melted cheese sizzling and bubbling, game ambient sound", 1.5),
    
    # Explosion sounds
    "SFX_Meatball_Explosion": ("Meaty explosion with bread crumb scatter, grenade explosion, game sound", 0.8),
    "SFX_Marinara_Molotov": ("Glass breaking with wet sauce splash and fire ignition, game sound", 0.6),
    "SFX_Lasagna_Explosion": ("Heavy layered explosion with cheese splatter, rocket impact, game sound", 1.0),
    
    # Character sounds
    "SFX_Footstep_Spaghetti": ("Wet squishing footstep, noodle step on ground, game footstep sound", 0.2),
    "SFX_Footstep_Penne": ("Hollow tube clicking footstep, hard pasta on ground, game footstep", 0.2),
    "SFX_Footstep_Rigatoni": ("Heavy hollow thud footstep, large tube step, game footstep sound", 0.2),
    
    # Ability sounds
    "SFX_Noodle_Lasso_Throw": ("Wet rope throwing sound with whoosh, lasso toss, game ability sound", 0.5),
    "SFX_Noodle_Lasso_Hit": ("Wet grab and tighten, rope catching, game ability impact", 0.3),
    "SFX_Spiral_Dash": ("Spinning whoosh with springy coil sound, movement ability, game sound", 0.4),
    "SFX_Layer_Up_Shield": ("Layered stacking sound, building up, shield ability, game sound", 0.6),
    
    # UI sounds
    "SFX_UI_Click": ("Crisp pasta snap, clean UI click, game interface sound", 0.1),
    "SFX_UI_Hover": ("Light soft pasta squish, subtle UI hover, game interface sound", 0.1),
    "SFX_UI_Confirm": ("Satisfying pasta crunch, confirmation, game interface sound", 0.2),
    "SFX_Chest_Open": ("Creaky wooden box opening with pasta tumbling, loot chest, game sound", 1.0),
    "SFX_Pickup_Item": ("Quick swoosh with subtle chime, item pickup, game sound", 0.3),
    "SFX_Elimination_Sting": ("Quick dramatic orchestral hit, elimination notification, game sound", 0.5),
    "SFX_Victory_Fanfare": ("Triumphant Italian fanfare with mandolin and accordion, victory music, game sound", 3.0),
    
    # Ambient sounds
    "SFX_Boiling_Water": ("Pot of water boiling and bubbling, kitchen ambient, game ambient sound", 5.0),
    "SFX_Storm_Rumble": ("Deep rumbling with boiling water undertone, storm ambient, game sound", 5.0),
    "SFX_Bus_Engine": ("Fantastical flying engine hum with bubbling pot sounds, vehicle, game sound", 5.0),
}

def generate_all_sounds():
    """Generate all game sound effects."""
    for name, (prompt, duration) in SOUND_EFFECTS.items():
        output_path = f"Content/Audio/SFX/{name}.wav"
        try:
            generate_sfx_elevenlabs(prompt, output_path, duration)
            print(f"  ✓ Generated: {name}")
        except Exception as e:
            print(f"  ⚠ Failed: {name} — {e}")

if __name__ == "__main__":
    generate_all_sounds()
```

---

## 12. VFX & SHADER SYSTEMS

### 12.1 — Niagara VFX Systems Required

```
NIAGARA VFX LIST:

=== WEAPON VFX ===
NS_MuzzleFlash_Pasta:
  - Burst of tiny pasta fragments + steam puff
  - Parametric: color tint per weapon type
  - Life: 0.1s, 20-30 particles

NS_Tracer_PenneTube:
  - Spinning penne tube flying through air
  - Ribbon trail behind it
  - GPU particles for performance

NS_Tracer_SpaghettiNeedle:
  - Thin fast line with slight wave
  - Hair-thin ribbon trail

NS_Tracer_FarfalleSpread:
  - Multiple farfalle shapes spreading outward
  - Flutter animation on each particle (mesh particles)

NS_Impact_PastaBreak:
  - Pasta fragments scattering on impact
  - Dust cloud + small sparks
  - Decal spawn (pasta splat)
  - Parametric: surface type response

NS_Explosion_Meatball:
  - Spherical shockwave ring
  - Meat and breadcrumb particle scatter
  - Smoke column (brown-red)
  - Ground scorch decal
  - Camera shake trigger

NS_Explosion_Lasagna:
  - Layered rectangular explosion
  - Cheese string particles stretching and falling
  - Sauce splatter spray
  - Larger smoke plume

NS_Fire_Marinara:
  - Red-orange fire effect
  - Sauce bubbling on ground
  - Steam rising
  - Area-of-effect visualization ring

NS_Smoke_Parmesan:
  - Dense yellow-white smoke cloud
  - Cheese particle motes floating
  - Volumetric density over time

NS_Slip_Alfredo:
  - Creamy white liquid spreading on ground
  - Glossy reflective surface decal
  - Small bubble particles

=== CHARACTER VFX ===
NS_Spaghetti_IdleStrands:
  - Gentle sway of loose noodle strands
  - Spline-based hair simulation particles
  - Responds to movement direction

NS_Sauce_Drip:
  - Occasional sauce drip from characters
  - Parametric: sauce color (red/white/green)
  - Ground splat on landing

NS_Orzo_BodyShimmer:
  - Constant tiny grain movement on Orzo character
  - Grain particles separating and rejoining
  - Parametric: intensity based on movement speed

NS_Elimination_PastaExplosion:
  - Per-character-type elimination burst
  - Spaghetti: noodles flying everywhere
  - Penne: tubes scattering
  - Farfalle: butterflies dispersing
  - Etc. — each pasta type has unique death particles

NS_HealEffect_Steam:
  - Rising steam particles when consuming healables
  - Green tint for health, blue for shield
  - Gentle upward spiral

NS_BuildPlace_Flour:
  - Flour dust poof when placing building
  - Brief white particle burst
  - Settling dust

=== ENVIRONMENT VFX ===
NS_Storm_Wall:
  - Massive vertical wall of boiling pasta water
  - Rolling cloud with lightning
  - Steam at the base
  - Translucent from distance, opaque up close
  - Damage indicator (red pulse when inside)
  - GPU heavy — use LOD system:
    - Close: Full particle simulation
    - Medium: Simplified billboard particles
    - Far: Shader-only (no particles)

NS_SupplyDrop_Trail:
  - Steam trail from descending pasta pot supply drop
  - Spiral descent animation
  - Glow effect on pot

NS_Chest_Glow:
  - Subtle glow emanating from chests
  - Rarity-colored particles:
    - Common: gray motes
    - Uncommon: green sparkles
    - Rare: blue sparkles
    - Epic: purple swirl
    - Legendary: gold beam + particle shower

NS_HeatmapGlow:
  - Emissive glow on terrain matching heatmap intensity
  - Pulsing ember particles in high-conflict areas
  - Smoke wisps rising from intense zones
```

### 12.2 — Custom Shaders & Materials

```
SHADER SYSTEMS:

=== M_Storm_Wall (Material) ===
Custom translucent material for storm wall:
- Scrolling noise texture for turbulence
- Fresnel for edge glow
- Depth fade for ground intersection
- Refraction for distortion
- Color: murky brownish water with red lightning flashes
- Opacity: distance-based (more opaque closer)
- Emissive: pulsing to indicate damage zone

=== M_Heatmap_Overlay (Material) ===
Applied to terrain as decal or landscape layer:
- Samples dynamic heatmap texture from ConflictHeatmapSubsystem
- Color ramp: transparent → blue → green → yellow → orange → red
- Emissive intensity scales with conflict intensity
- Animated noise for organic look
- Additive blend over terrain

=== M_Ghost_Build (Material) ===
For building placement preview:
- Translucent with grid pattern
- Color parameter: green (valid) / red (invalid)
- Edge highlight (fresnel)
- Pulse animation

=== M_Shield_Hit (Material) ===
Shield impact visualization:
- Hexagonal grid pattern
- Ripple from impact point
- Blue-white glow fading over 0.3s
- Fresnel for sphere shape

=== M_Rarity_Glow (Material Function) ===
Reusable glow for items by rarity:
- Parameter: rarity enum → color mapping
- Pulsing emissive
- Particle spawn trigger

=== PP_DamageVignette (Post Process) ===
Screen-space damage indication:
- Red vignette at low health
- Directional damage indicator overlay
- Screen shake integration
- Desaturation at critical health

=== PP_StormDamage (Post Process) ===
Screen effect when inside storm:
- Edge distortion
- Color shift to brownish
- Particle overlay (water droplets)
- Intensity scales with storm damage
```

---

## 13. OPTIMIZATION & STREAMING

### 13.1 — Performance Targets

```
TARGET PERFORMANCE:

Platform         | Resolution | FPS Target | Quality
─────────────────┼────────────┼────────────┼─────────
PC High-end      | 4K         | 60 FPS     | Epic
PC Mid-range     | 1440p      | 60 FPS     | High
PC Low-end       | 1080p      | 60 FPS     | Medium
PS5              | 4K (upscaled) | 60 FPS  | High
Xbox Series X    | 4K (upscaled) | 60 FPS  | High
Xbox Series S    | 1080p      | 60 FPS     | Medium
```

### 13.2 — World Partition Strategy

```cpp
// Source/PastaWarfare/World/PastaWorldPartitionSubsystem.h

/*
WORLD PARTITION CONFIGURATION:

The game map uses UE5's World Partition system with custom streaming logic:

GRID SIZE: 256m × 256m cells (each cell is a streaming unit)

STREAMING DISTANCES:
  - Character detail:  500m (full LOD0 within this range)
  - Building detail:   800m (structures fully loaded)
  - Terrain detail:    1500m (full landscape resolution)
  - Vegetation:        600m (foliage instances)
  - Far terrain:       5000m (low-LOD terrain, Nanite simplified)
  - Globe backdrop:    Infinite (always loaded during drop)

PRIORITY LOADING:
  1. Cell containing player (immediate)
  2. Cells in movement direction (predictive)
  3. Adjacent cells (ring buffer)
  4. Cells near other players in view (network-informed)
  5. Background cells (low priority)

NANITE USAGE:
  - ALL static meshes use Nanite (buildings, rocks, props, terrain)
  - Characters do NOT use Nanite (need skeletal mesh)
  - Weapons do NOT use Nanite (attached to characters)
  - Building pieces: Nanite enabled for placed pieces, disabled for ghost preview

HLOD (Hierarchical Level of Detail):
  - Generated automatically for each World Partition cell
  - HLOD0: Full detail (within streaming distance)
  - HLOD1: Merged simplified meshes (500m-1500m)
  - HLOD2: Billboard impostors (1500m-5000m)
  - HLOD3: Color-averaged blocks (5000m+, visible during drop)

VIRTUAL TEXTURING:
  - Landscape uses Runtime Virtual Texturing (RVT)
  - Virtual texture pool: 4GB
  - Tile size: 256×256 (streaming unit)
  - Feedback buffer: half resolution

DATA LAYERS:
  - Layer_Terrain: Heightmaps, splatmaps, landscape actors
  - Layer_Buildings: Procedurally placed structures
  - Layer_Vegetation: Foliage instances, trees
  - Layer_Loot: Spawn points, chests, floor loot
  - Layer_Gameplay: Triggers, zones, spawn points
  - Layer_VFX: Ambient effects, heatmap decals
*/
```

### 13.3 — Memory & Draw Call Budget

```
MEMORY BUDGET (8GB GPU target):

Category              | Budget   | Notes
──────────────────────┼──────────┼──────────────────────
Terrain textures      | 1.5 GB   | Virtual texturing pool
Building meshes       | 1.0 GB   | Nanite mesh data
Character meshes      | 0.5 GB   | 11 types × ~45MB each
Weapon meshes         | 0.3 GB   | ~20 weapons × ~15MB
VFX textures          | 0.3 GB   | Particle sprites, flipbooks
UI textures           | 0.2 GB   | Atlas sheets
Animation data        | 0.4 GB   | Compressed sequences
Audio                 | 0.3 GB   | Compressed, streaming
Render targets        | 1.5 GB   | GBuffer, shadows, etc.
Overhead/system       | 2.0 GB   | Engine, shaders, buffers
──────────────────────┼──────────┼──────────────────────
TOTAL                 | 8.0 GB   |

DRAW CALL BUDGET:
- Target: <3000 draw calls per frame
- Characters visible: ~20 at any time (LOD system)
- Buildings in view: ~200 (Nanite handles this)
- Vegetation: Instanced (1 draw call per foliage type)
- Particles: GPU Niagara (minimal CPU draw calls)
- UI: Batched Slate rendering

TRIANGLE BUDGET:
- Nanite handles geometric complexity automatically
- Proxy triangle budget: ~5M triangles visible
- Nanite will render from billions of source triangles
```

### 13.4 — Network Optimization

```
NETWORK OPTIMIZATION:

Replication Frequency Tiers:
  Critical (every net tick, 60Hz):
    - Own pawn position/rotation
    - Fire/hit events
  
  High (30Hz):
    - Nearby enemy positions (within 100m)
    - Active ability effects nearby
  
  Medium (10Hz):
    - Distant player positions (100m-500m)
    - Health/shield changes
    - Inventory updates
  
  Low (2Hz):
    - Players beyond 500m (position only)
    - Storm circle updates
    - Score updates
  
  Event-Only:
    - Eliminations
    - Building placement (once, then static)
    - Loot pickups

Compression:
  - Positions: FVector_NetQuantize10 (1mm precision, saves 50% bandwidth)
  - Rotations: FRotator compressed to 16-bit per component
  - Velocities: Quantized to 1cm/s precision
  - Health/Shield: uint8 (0-200 range is sufficient)

Relevancy:
  - Players beyond 500m: position-only updates
  - Players beyond 1000m: not replicated (use minimap dots only)
  - Building pieces: relevant only within 300m
  - Loot items: relevant only within 200m
  - VFX actors: not replicated (spawned client-side from RPC)

Bandwidth Target:
  - Server upstream: 5 MB/s total (100 players)
  - Per-player downstream: 50 KB/s average, 150 KB/s peak
  - Per-player upstream: 10 KB/s average
```

---

## 14. BACKEND SERVICES

### 14.1 — Backend Architecture

```
BACKEND SERVICES ARCHITECTURE:

┌─────────────────────────────────────────────────────────┐
│                    GAME CLIENT                           │
└──────────────┬──────────────────────────┬───────────────┘
               │                          │
    ┌──────────▼──────────┐    ┌──────────▼──────────┐
    │   MATCHMAKING       │    │   ACCOUNT SERVICE   │
    │   (EOS / Custom)    │    │   (REST API)        │
    │   - Region routing  │    │   - Auth (OAuth2)   │
    │   - Skill-based     │    │   - Profile         │
    │   - Squad formation │    │   - Inventory       │
    └──────────┬──────────┘    │   - Purchases       │
               │               │   - Stats           │
    ┌──────────▼──────────┐    └──────────┬──────────┘
    │   GAME SERVER       │               │
    │   (Dedicated UE5)   │    ┌──────────▼──────────┐
    │   - 100 player      │    │   DATABASE          │
    │   - Server authority│    │   (PostgreSQL)      │
    │   - Match state     │    │   - Player accounts │
    │   - Anti-cheat      │    │   - Match history   │
    └──────────┬──────────┘    │   - Leaderboards    │
               │               │   - Inventory       │
    ┌──────────▼──────────┐    └─────────────────────┘
    │   GAME SERVER       │
    │   FLEET MANAGER     │    ┌─────────────────────┐
    │   (Agones / Gamelift│    │  CONFLICT DATA      │
    │    / Custom K8s)    │    │  SERVICE            │
    │   - Auto-scaling    │    │  - ACLED API proxy  │
    │   - Region deploy   │    │  - UCDP API proxy   │
    │   - Health checks   │    │  - Cache layer      │
    └─────────────────────┘    │  - Heatmap compute  │
                               └─────────────────────┘

TECHNOLOGY CHOICES:
  - Matchmaking: Epic Online Services (EOS) — free, cross-platform
  - Account/Profile: Custom REST API (Node.js + Express or Go)
  - Database: PostgreSQL (accounts, stats) + Redis (leaderboards, cache)
  - Game Servers: UE5 Dedicated Server on AWS/GCP
  - Fleet Management: Agones (Kubernetes-based) or AWS GameLift
  - Conflict Data: Custom microservice caching ACLED/UCDP data
  - CDN: CloudFront/Fastly for patches and assets
  - Voice Chat: EOS Voice or Vivox integration
```

### 14.2 — Matchmaking Flow

```
MATCHMAKING SEQUENCE:

1. CLIENT → MATCHMAKER: RequestMatch(region, mode, squad_members)
2. MATCHMAKER: Check player pool for region
   - If pool >= MinPlayers (50): Create match
   - If pool < MinPlayers: Wait up to 30s, then fill with bots
3. MATCHMAKER → FLEET MANAGER: RequestServer(region, map_config)
4. FLEET MANAGER: Spin up or allocate existing server
5. FLEET MANAGER → MATCHMAKER: ServerReady(ip, port, encryption_key)
6. MATCHMAKER → CLIENT: MatchFound(server_ip, port, token)
7. CLIENT: Connect to dedicated server with auth token
8. SERVER: Validate token, spawn player in lobby

REGIONS:
  - NA-East (Virginia)
  - NA-West (Oregon)
  - EU-West (London/Frankfurt)
  - EU-East (Warsaw)
  - ME (Bahrain)  — Important for Iraq/Yemen/Syria zones
  - Asia (Singapore/Tokyo)
  - OCE (Sydney)
  - SA (São Paulo)
  - Africa (Cape Town)

Bot backfill fills remaining slots with AI players.
Bots use Behavior Trees with difficulty scaling based on remaining human player skill.
```

### 14.3 — Account & Progression System

```
PLAYER PROGRESSION:

Account Level: 1-1000 (XP from matches)
  XP Sources:
    - Eliminations: 50 XP each
    - Assists: 25 XP
    - Top 25: 100 XP
    - Top 10: 200 XP  
    - Top 5: 350 XP
    - Victory Royale: 500 XP
    - Damage dealt: 1 XP per 10 damage
    - Materials harvested: 1 XP per 50 mats
    - Survival time: 1 XP per 30 seconds
    - First game of the day: 200 XP bonus

Pasta Pass (Battle Pass equivalent):
  - 100 tiers per season (10 weeks)
  - Free track: Basic cosmetics every 5 tiers
  - Premium track ($9.99): Cosmetic every tier
  - Rewards include: skins, gliders, harvesting tools, emotes, 
    loading screens, sprays, Penne Points

Challenges (Daily/Weekly):
  - Daily (3 per day, reset at midnight UTC):
    - "Eliminate 3 players with shotguns"
    - "Deal 500 damage with abilities"
    - "Harvest 1000 materials"
  - Weekly (7 per week):
    - "Win a match in Iraq zone"
    - "Eliminate a player with every weapon type"
    - "Place 100 building pieces in a single match"
  - Season-long:
    - "Win 50 matches"
    - "Play as each pasta type at least once"
    - "Visit all conflict zones"

Leaderboards:
  - Global rankings by: Wins, K/D, Win Rate
  - Per-zone rankings (top players in each conflict zone)
  - Per-pasta-type rankings
  - Seasonal and all-time
```

---

## 15. BUILD, PACKAGE & RELEASE

### 15.1 — Build Pipeline

```bash
#!/bin/bash
# Scripts/Build/build_all.sh
# Complete build pipeline for all platforms

set -e

PROJECT_DIR="/path/to/PastaWarfare"
UE_DIR="/path/to/UnrealEngine"
BUILD_DIR="/builds"

# === STEP 1: Cook Content ===
echo "=== Cooking Content ==="
"${UE_DIR}/Engine/Build/BatchFiles/RunUAT.sh" BuildCookRun \
    -project="${PROJECT_DIR}/PastaWarfare.uproject" \
    -noP4 \
    -platform=Win64 \
    -clientconfig=Shipping \
    -serverconfig=Shipping \
    -cook \
    -build \
    -stage \
    -pak \
    -archive \
    -archivedirectory="${BUILD_DIR}/Win64" \
    -compressed \
    -prereqs \
    -distribution \
    -createreleaseversion=1.0.0 \
    -nodebuginfo

# === STEP 2: Build Dedicated Server ===
echo "=== Building Dedicated Server (Linux) ==="
"${UE_DIR}/Engine/Build/BatchFiles/RunUAT.sh" BuildCookRun \
    -project="${PROJECT_DIR}/PastaWarfare.uproject" \
    -noP4 \
    -platform=Linux \
    -server \
    -serverconfig=Shipping \
    -cook \
    -build \
    -stage \
    -pak \
    -archive \
    -archivedirectory="${BUILD_DIR}/LinuxServer" \
    -compressed \
    -nodebuginfo \
    -noclient

# === STEP 3: Package for Distribution ===
echo "=== Packaging ==="

# Create installer (NSIS or Inno Setup for Windows)
# Or prepare for Steam/Epic distribution

# === STEP 4: Docker Container for Server ===
echo "=== Building Server Docker Image ==="
cat > "${BUILD_DIR}/Dockerfile.server" << 'DOCKERFILE'
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    libfontconfig1 \
    libfreetype6 \
    libc6 \
    libstdc++6 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /server

COPY LinuxServer/ .

RUN chmod +x PastaWarfareServer

EXPOSE 7777/udp
EXPOSE 27015/udp

ENTRYPOINT ["./PastaWarfareServer", "-log", "-port=7777"]
DOCKERFILE

cd "${BUILD_DIR}"
docker build -f Dockerfile.server -t pastawarfare-server:latest .

# === STEP 5: Push Server Image ===
echo "=== Pushing Server Image ==="
# docker push to ECR/GCR for fleet management

echo "=== BUILD COMPLETE ==="
echo "Client build:
