
# [MC-Skins: Minecraft Skin Editor](/mc-skins)

A Next.js/React application for pixel-level Minecraft skin editing with real-time 3D preview and optional live collaboration.

## Overview

Built for the Minecraft community (my kids and their friends), this editor allows users to create and modify Minecraft character skins directly in the browser. It combines a canvas-based paint interface with a live 3D viewer, and includes an experimental collaboration mode over WebSockets.

![MC-Skins Screenshot](https://www.nickhedberg.com/images/FRkl7V19xfc9IvyjKoO0G7RNP5Q=/fit-in/1200x1200/s3-us-west-2.amazonaws.com/nick-hedberg/img%2F1894%3A2856%2F33230e0ef02b834d7f7ae9c5c14f1e018417bedc.png)

## Key Features

### Pixel Art Editor
- **Canvas-Based Editing**: Precise pixel-level control using HTML5 Canvas
- **Area Isolation**: Focus on specific body regions (head/arms/torso/legs and sides) while editing
- **Color Controls**: Palette + picker workflow for fast color changes
- **Quick Fill**: Fill grouped regions in one action
- **Undo/Redo History**: Local history stack for stepping backward/forward through edits

### 3D Preview System
- **Real-Time Rendering**: Live 3D character model updates as you edit
- **Multiple Views**: Front, back, sides, and rotating views of the character
- **Animation Support**: Character poses and walking animations
- **Lighting Controls**: Adjustable lighting to preview skin appearance in-game

### Skin Management
- **Save & Load**: Persistent skin storage with user accounts
- **Import/Export**: PNG upload and export support (64x64 validation on upload)
- **Per-User Library**: APIs scoped to the authenticated user for listing/creating/updating skins

## Technical Implementation

### Canvas Architecture
```
components/PixelPainter/
└── index.js                 # Main canvas editor and paint interactions

components/Preview/
└── index.js                 # 3D viewer integration (react-skinview3d)

hooks/useSkinData.js         # Shared skin state, history, persistence, ws sync
components/EditControls/     # Upload/export/history/area controls
```

### Pixel Manipulation System
The editor uses a layered canvas approach:
- **Base Layer**: Primary skin pixels (Steve/Alex model support)
- **Overlay Layer**: Additional details like jackets, hats, sleeves
- **Preview Integration**: Real-time texture mapping to 3D model
- **Format Conversion**: Automatic conversion between skin format versions

### 3D Rendering Pipeline
```javascript
// 3D model system
const skinRenderer = {
  loadModel: (skinData) => { /* Parse Minecraft skin format */ },
  applyTexture: (canvas) => { /* Map 2D pixels to 3D surfaces */ },
  animate: (pose) => { /* Character animation system */ },
  render: (scene) => { /* WebGL rendering pipeline */ }
}
```

### Authentication & Storage
- **User Accounts**: Auth0 integration for personal skin libraries
- **Database Storage**: MySQL backend for skin metadata and pixel data
- **Secure APIs**: CRUD routes validate user ownership before updates/deletes

## Development Challenges

### Pixel-Perfect Editing
Minecraft skins require precise pixel placement:
- **Zoom Controls**: Multiple zoom levels for detailed work
- **Grid Overlay**: Optional pixel grid for alignment
- **Touch Support**: Mobile-friendly editing with touch gestures
- **Performance**: Smooth editing even with complex brush operations

### 3D Model Accuracy
Ensuring the preview matches in-game appearance:
- **Minecraft Geometry**: Accurate replication of character model dimensions
- **Texture Mapping**: Proper UV mapping for all model faces
- **Lighting Model**: Realistic lighting that matches game environment
- **Animation System**: Character poses and movement animations

### Collaboration and Sync
- **WebSocket Patches**: The editor can broadcast pixel patch updates for shared editing sessions.
- **Client Filtering**: Client IDs prevent echoing your own patches back into your view.

## User Interface Design

### Editor Layout
- **Tool Palette**: Left sidebar with drawing tools and options
- **Canvas Area**: Center stage with zoom and pan controls
- **3D Preview**: Right panel with rotating character model
- **Color Controls**: Bottom panel with color picker and palette

### Responsive Design
The editor adapts to different screen sizes:
- **Desktop**: Full multi-panel layout with all tools visible
- **Tablet**: Collapsible panels with touch-optimized controls
- **Mobile**: Stacked layout with swipe navigation between editor and preview

## Performance Optimization

### Canvas Rendering
- **Efficient Redraws**: Only redraw changed regions for smooth editing
- **Layer Compositing**: Hardware-accelerated layer blending
- **Memory Management**: Proper cleanup of canvas contexts and textures

### 3D Rendering
- **WebGL Optimization**: Efficient vertex buffers and texture management
- **Frame Rate Control**: Maintain 60fps during real-time preview updates
- **Resource Loading**: Progressive loading of 3D assets and textures

The MC-Skins editor combines precise pixel art tools with real-time 3D preview to create an intuitive skin creation experience that rivals desktop applications while running entirely in the browser.
