# THREE.JS 3D CHESS — TECH MEMORY

> Mục đích: tài liệu bàn giao kỹ thuật cho mọi agent/hội thoại sau khi tiếp tục phát triển prototype Cờ Vua 3D trong repo này.

## 1. Scope

Project này độc lập. Không được dùng repo/game khác làm nơi phát triển hoặc thử nghiệm.

- Repo: `bktimmy247/lumina-preview-html`
- Entry hiện tại: `co-vua-3d/index.html`
- Live: `https://bktimmy247.github.io/lumina-preview-html/co-vua-3d/`

## 2. Core stack

Renderer / runtime:

- Three.js `0.180.0`
- WebGL via `THREE.WebGLRenderer`
- ES modules + import map
- Static HTML, không cần backend
- GitHub Pages để host preview

Gameplay chess:

- `chess.js 1.4.0`
- Luật cờ, legal moves, check/checkmate, draw, castling, en passant, promotion
- Three.js chỉ chịu trách nhiệm render + interaction

## 3. 3D approach

Không dùng Unity/Unreal/Cocos.

Không bắt buộc dùng model GLB/FBX. Bản hiện tại dựng toàn bộ bằng procedural geometry trong Three.js:

- `BoxGeometry`
- `CylinderGeometry`
- `SphereGeometry`
- `ConeGeometry`
- `TorusGeometry`
- `LatheGeometry`
- `OctahedronGeometry`
- `THREE.Group`

Các quân cờ được cấu tạo từ nhiều primitive mesh ghép thành silhouette fantasy.

Current fantasy direction:

- White faction: Ivory Kingdom
- Black faction: Obsidian Legion
- Pawn: royal/shadow guard
- Rook: fortress sentinel / golem
- Knight: armored warhorse
- Bishop: hooded battle priest
- Queen: arcane queen
- King: high king / dread king

## 4. Rendering

Camera:

- `THREE.PerspectiveCamera`
- `OrbitControls`
- drag/touch rotate
- pinch/wheel zoom
- camera flip White/Black
- intro camera animation

Materials:

- `MeshPhysicalMaterial`
- `MeshStandardMaterial`
- `MeshBasicMaterial`
- ACES filmic tone mapping
- sRGB output

Lighting:

- HemisphereLight
- DirectionalLight as primary shadow caster
- a small number of global Point/Spot lights only

IMPORTANT: Do NOT attach one real PointLight to every chess piece. On iPhone, 32 dynamic lights caused a major FPS drop.

Faction glow should be done mainly by emissive materials / small emissive meshes, not per-piece lights.

## 5. Mobile / iPhone performance rules

These rules are mandatory unless profiling proves otherwise.

### Pixel ratio

Use a mobile cap:

```js
renderer.setPixelRatio(Math.min(devicePixelRatio, 1.15))
```

Desktop can use a higher cap, currently around 1.6.

### Shadows

On mobile:

- use lower shadow map resolution
- current target around `768 x 768`
- avoid every sub-mesh casting/receiving shadows
- keep one primary global shadow light

Do not return to 2048 shadow maps on iPhone without profiling.

### Geometry LOD

Procedural mesh segments must be reduced on mobile.

Current approach:

- fewer radial segments for cylinders/cones
- fewer sphere width/height segments
- fewer torus segments
- fewer LatheGeometry segments

Desktop can render higher-detail procedural meshes.

### Interaction

On touch devices:

- do not run hover raycasting every frame
- raycast on tap/pointer-up for selection
- desktop hover effects can remain enabled

### Background

Mobile should use fewer:

- decorative pillars
- arches
- particles
- non-gameplay meshes

### FX

Particle effects on captures should have lower particle count on mobile.

Do not introduce full-screen post-processing/bloom by default on iPhone. Prefer emissive materials and cheap geometry-based glow.

## 6. Board and visual style

Current board direction:

- obsidian/dark-metal base
- brass/gold frame
- ivory / dark slate squares
- cinematic dark environment
- blue/cold faction accents for White
- red/dark faction accents for Black

UI direction:

- premium dark glass
- compact mobile HUD
- safe-area aware
- do not cover board interaction area unnecessarily

## 7. Move interaction flow

1. Tap/click own piece.
2. Query `chess.js` for legal moves.
3. Render move/capture markers.
4. Tap destination.
5. Apply move to `chess.js`.
6. Animate rendered piece.
7. Rebuild piece scene from authoritative chess state.
8. Update status/check/checkmate UI.

Important architectural rule:

`chess.js state is authoritative; Three.js scene is a projection of that state.`

Do not store independent game-rule state inside Three.js meshes.

## 8. Animation direction

Current movement:

- lift
- travel
- land

Capture:

- stronger vertical arc
- small particle burst
- dedicated sound

Future preferred direction:

- Knight: leap
- Rook: heavy straight dash
- Bishop: diagonal glide
- Queen: arcane trail
- King: weighty royal movement

Keep all animation cosmetic. Final board state must still come from `chess.js`.

## 9. Audio

Current audio uses WebAudio oscillators for lightweight procedural SFX.

Keep audio user-gesture compatible on iOS.

If adding real SFX assets later:

- preload small compressed files
- avoid large audio packages
- stop/reuse AudioBufferSource patterns where possible
- do not create many simultaneous audio nodes

## 10. Known performance lesson

A previous version looked good but lagged badly on iPhone because it combined:

- 32 per-piece PointLights
- 2048 shadow maps
- high DPR
- high geometry segment counts
- decorative scene geometry
- hover raycasting every frame

The optimization pass removed/reduced these costs.

Commit containing the main iPhone renderer optimization:

- `ed954347640987aa8b9ffb1eb1b86a342891fba6`

Fantasy silhouette upgrade before that:

- `ab2ae8cef7e9b87057c6dc306e5aced2734ff839`

## 11. Development rule for future agents

Before adding visual effects or geometry:

1. Preserve chess rules.
2. Measure the likely mobile cost.
3. Prefer one global light over many local lights.
4. Prefer emissive material over dynamic light.
5. Prefer procedural LOD over uniformly high-detail geometry.
6. Keep iPhone as the baseline performance target.
7. Make one visual/performance change at a time where possible.
8. Test the GitHub Pages build after each meaningful renderer change.

## 12. Future upgrade path

Recommended order:

1. Piece-specific movement/capture animations.
2. Better faction silhouettes.
3. Shared geometry/material caches.
4. Instancing where repeated meshes become numerous.
5. Optional GLB characters only if procedural models are no longer sufficient.
6. If GLB is introduced, use glTF/GLB + compressed textures + low-poly meshes; keep WebGL mobile budget first.

Do not switch engine unless explicitly requested. The point of this prototype is to continue using the Three.js/WebGL approach.
