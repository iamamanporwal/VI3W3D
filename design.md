# VI3W - AI 3D Editor Design Document

## 1. System Overview

VI3W (pronounced "view") is an AI-powered 3D modeling platform that enables users to create and edit 3D models through natural language prompts and image inputs. The system runs entirely in the browser, eliminating the need for high-end hardware or specialized 3D software.

**Core Value Proposition**: "3D in 3 Clicks" - democratizing 3D modeling by making it accessible, fast, and collaborative through AI assistance.

**Target Users**:
- Engineers for rapid prototyping
- Architects for concept visualization
- Game developers for asset creation
- Educators and students for learning 3D concepts

**Key Differentiators**:
- Browser-based execution (no installation required)
- Multi-modal input (text + images)
- Real-time AI-assisted modeling
- Collaborative environment support
- Focus on low-resource environments (Bharat focus)

## 2. High-Level Architecture

VI3W follows a client-centric architecture with AI orchestration:

```
┌─────────────────────────────────────────────────────────────┐
│                    Browser Client (React)                    │
├─────────────┬──────────────┬──────────────┬─────────────────┤
│   Chat UI   │  3D Viewer   │ Parts Panel  │  History Panel  │
│   (Gemini)  │ (Three.js)   │              │                 │
└─────────────┴──────────────┴──────────────┴─────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                    AI Orchestration Layer                    │
├─────────────────┬─────────────────┬─────────────────────────┤
│   FAST Mode     │    CAD Mode     │   Image-to-3D Mode      │
│  (Primitives)   │ (Python Scripts)│   (Visual Analysis)     │
└─────────────────┴─────────────────┴─────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                    Execution Environment                     │
├─────────────────┬─────────────────┬─────────────────────────┤
│  Pyodide Runtime│  Asset Manager  │   CSG Operations        │
│  (Browser CAD)  │  (Mesh Caching) │   (Hole Cutting)        │
└─────────────────┴─────────────────┴─────────────────────────┘
```

**Architecture Principles**:
1. **Client-First**: All computation happens in the browser for maximum accessibility
2. **AI-Centric**: Gemini AI orchestrates modeling decisions and code generation
3. **Multi-Modal**: Support for text prompts, image inputs, and parametric editing
4. **Progressive Enhancement**: Fallback from CAD to primitive representations
5. **State Isolation**: Multi-environment support with independent workspaces

## 3. Component Breakdown

### 3.1 Frontend Components

**Core UI Components**:
- `App.tsx`: Main application orchestrator with environment management
- `ChatPanel.tsx`: Natural language interface with AI mode selection
- `ViewerPanel.tsx`: Three.js-based 3D visualization with WebGL rendering
- `LeftPanel.tsx`: Parts hierarchy and history management
- `Header.tsx`: System status and environment controls

**State Management**:
- **Environment System**: Multi-workspace support with localStorage persistence
- **Part Registry**: Unified scene representation across different generation methods
- **Visualization State**: Primitive-based fallback for compatibility
- **History Stack**: Version control with snapshot restoration

### 3.2 Service Layer

**AI Services**:
- `geminiService.ts`: Multi-mode AI orchestration (FAST/CAD/Image-to-3D)
- `schemaValidator.ts`: Response validation and sanitization

**Execution Services**:
- `pythonEngine.ts`: Browser-based Python execution via Pyodide
- `csgHoles.ts`: Client-side constructive solid geometry operations
- `assetManager.ts`: Mesh caching and blob URL management
- `exportService.ts`: 3D model export capabilities

**Utility Services**:
- `sceneUtils.ts`: Geometry validation and sanitization
- `partRegistry.ts`: Part lifecycle and reconciliation logic

### 3.3 Data Models

**Core Types** (`types.ts`):
- `Part`: Unified representation for all generation methods (primitive/voxel/CAD/external_mesh)
- `VizPrimitive`: Legacy primitive representation for backward compatibility
- `Environment`: Isolated workspace with complete state
- `ChatMessage`: Conversation history with multimodal attachments
- `HoleSpec`: Parametric feature definition for CAD operations

**AI Response Schema**:
- Structured JSON responses with intent detection (create/edit/explain)
- Multi-part scene descriptions with method-specific metadata
- Visualization fallback for legacy compatibility

## 4. Data Flow

### 4.1 Primary User Journey

```
User Input
    │
    ▼
[Chat Interface] → Text/Image Prompt + AI Mode Selection
    │
    ▼
[Gemini AI Service] → Mode-specific prompt engineering
    │                    │
    │                    ▼
    │           [Schema Validation] → Response sanitization
    │
    ▼
[Scene Assembly] → Part registry creation + visualization fallback
    │
    ▼
[CAD Execution] → Python script execution (if CAD mode)
    │                    │
    │                    ▼
    │           [Pyodide Runtime] → Mesh generation in browser
    │
    ▼
[3D Viewer Update] → Real-time rendering with Three.js
    │
    ▼
[State Persistence] → Environment save + history snapshot
```

### 4.2 Multi-Mode Execution Paths

**FAST Mode Path**:
```
Text Prompt → Gemini (FAST prompt) → Primitive/Voxel Parts → Direct Rendering
```

**CAD Mode Path**:
```
Text Prompt → Gemini (CAD prompt) → Python Script Generation → 
Pyodide Execution → Mesh Generation → CSG Operations → Final Rendering
```

**Image-to-3D Path**:
```
Image Upload → Visual Analysis → 3D Structure Inference → 
Multi-mode Generation → Hybrid Rendering
```

### 4.3 State Synchronization

```
User Action → Environment Update → LocalStorage Persistence → 
UI Re-render → Viewer Update → History Snapshot
```

## 5. AI / Model Design

### 5.1 Multi-Mode AI Orchestration

**FAST Mode**:
- Purpose: Rapid prototyping with primitive shapes
- AI Behavior: Decomposes objects into boxes, cylinders, spheres
- Output: Direct primitive visualization
- Use Case: Concept exploration, low-fidelity models

**CAD Mode**:
- Purpose: Precision modeling with parametric control
- AI Behavior: Generates Python scripts for mesh generation
- Output: Executable Python code + mesh data
- Use Case: Engineering parts, mechanical components

**Image-to-3D Mode**:
- Purpose: Convert 2D images to 3D models
- AI Behavior: Visual analysis + 3D structure inference
- Output: Hybrid representation (primitives + generated meshes)
- Use Case: Reference-based modeling, asset conversion

### 5.2 Prompt Engineering Strategy

**System Prompts**:
- `VI3WCAD_SYSTEM_PROMPT`: CAD-focused with strict wheel/tire rules
- `FAST_SYSTEM_PROMPT`: Primitive-focused for speed
- `SYSTEM_PROMPT`: Legacy multi-mode orchestrator

**Response Schema Enforcement**:
- Structured JSON output with validation
- Backward compatibility with visualization array
- Error handling with fallback responses
- Edit intent detection with partial matching

### 5.3 Model Constraints & Optimizations

**Performance Considerations**:
- Token limits: 8192 for CAD, 4096 for FAST
- Timeout handling: 90s for CAD, 60s for FAST
- Retry logic: Exponential backoff with 3 attempts for CAD
- Response caching: Script hash-based mesh caching

**Quality Controls**:
- Wheel/tire validation: Perfect cylinders only
- Dimension sanitization: Bounds checking + zero-size prevention
- Python script validation: Disallowed import filtering
- Mesh validation: Triangle count + vertex limits

## 6. Deployment Strategy

### 6.1 Development Environment

**Local Development**:
- Vite dev server on port 3000
- Hot module replacement for rapid iteration
- Environment variable management via .env
- TypeScript strict mode for type safety

**Build Process**:
- Vite-based bundling with tree shaking
- Code splitting for optimal load times
- Asset optimization for 3D models
- Source maps for debugging

### 6.2 Production Deployment

**Static Hosting**:
- Target: Any static hosting service (Vercel, Netlify, GitHub Pages)
- Requirements: HTTPS, modern browser support
- Assets: Pre-compiled JavaScript + HTML
- No server-side dependencies

**CDN Strategy**:
- Pyodide runtime from jsDelivr CDN
- Three.js libraries from unpkg
- Gemini API calls directly from client
- Mesh caching in IndexedDB

### 6.3 Environment Configuration

**Required Environment Variables**:
- `GEMINI_API_KEY`: Google Gemini API access
- (Optional) Custom CDN endpoints for Pyodide

**Browser Requirements**:
- WebGL 2.0 support for 3D rendering
- ES2020+ JavaScript support
- Web Workers for Pyodide execution
- LocalStorage for state persistence

## 7. Scalability Considerations

### 7.1 Client-Side Scalability

**Performance Optimizations**:
- Lazy loading of Pyodide runtime
- Mesh caching with hash-based invalidation
- Virtualized part lists for large scenes
- Progressive rendering for complex models

**Memory Management**:
- Object URL cleanup to prevent leaks
- Pyodide runtime isolation per environment
- Mesh data compression for storage
- Garbage collection triggers

### 7.2 AI Service Scalability

**API Optimization**:
- Request batching for multi-part scenes
- Response caching for common prompts
- Token usage monitoring and optimization
- Fallback strategies for API limits

**Cost Management**:
- Mode-based token budgeting
- User session rate limiting
- Response size optimization
- Cached result reuse

### 7.3 Data Scalability

**Scene Complexity**:
- Part count limits per environment
- Mesh complexity controls (triangle count)
- Hierarchical scene organization
- Progressive detail loading

**Storage Management**:
- LocalStorage quota management
- IndexedDB for large mesh data
- Environment pruning for inactive workspaces
- Export/import for scene portability

## 8. Security & Privacy

### 8.1 Data Security

**Client-Side Security**:
- No sensitive data transmission beyond API keys
- Local execution of Python scripts (sandboxed)
- Input sanitization for AI prompts
- XSS prevention through React sanitization

**API Security**:
- Environment variable protection for API keys
- HTTPS enforcement for all external requests
- Request signing for Gemini API calls
- Rate limiting and abuse prevention

### 8.2 Privacy Considerations

**Data Collection**:
- No user data collection beyond local storage
- Optional telemetry for error reporting
- Anonymous usage statistics (opt-in)
- Clear data retention policies

**User Content**:
- Local processing of uploaded images
- No external storage of user models
- Export-only data sharing
- Privacy-by-design architecture

### 8.3 Sandbox Security

**Python Execution**:
- Import blacklisting (os, sys, socket, etc.)
- Resource limits (timeout, memory)
- Network access prevention
- Filesystem isolation

**3D Rendering**:
- WebGL context isolation
- Shader validation and sanitization
- Texture size limits
- Render loop throttling

## 9. Limitations & Risks

### 9.1 Technical Limitations

**Browser Constraints**:
- Pyodide loading time (10-30 seconds)
- Memory limits for complex scenes
- WebGL compatibility across devices
- LocalStorage size restrictions (5-10MB)

**AI Limitations**:
- Prompt understanding accuracy
- Complex geometry generation quality
- Image-to-3D conversion fidelity
- Parametric constraint adherence

**Performance Limitations**:
- Real-time rendering of complex meshes
- Python execution speed in browser
- Large scene manipulation responsiveness
- Mobile device compatibility

### 9.2 Business Risks

**Dependency Risks**:
- Gemini API availability and pricing
- Pyodide project maintenance
- Three.js ecosystem stability
- Browser API standardization

**Market Risks**:
- Competition from established 3D tools
- User adoption of AI-assisted modeling
- Feature parity with professional tools
- Monetization strategy viability

**Technical Debt**:
- Legacy visualization system maintenance
- Multi-mode orchestration complexity
- State synchronization overhead
- Cross-browser compatibility testing

### 9.3 Mitigation Strategies

**Progressive Enhancement**:
- Fallback to primitive rendering
- Graceful degradation for unsupported features
- Polyfill strategy for browser APIs
- Feature detection before execution

**Monitoring & Alerting**:
- Client-side error tracking
- Performance metric collection
- User behavior analytics
- System health dashboards

## 10. Future Enhancements

### 10.1 Short-term Roadmap (3-6 months)

**Enhanced AI Capabilities**:
- Multi-step reasoning for complex models
- Physics-aware generation (balance, stability)
- Material property simulation
- Assembly relationship understanding

**Collaboration Features**:
- Real-time multi-user editing
- Version control integration
- Comment and annotation system
- Shareable environment links

**Export Improvements**:
- Additional format support (STEP, IGES)
- 3D printing optimization
- Animation export (GLB with keyframes)
- CAD software compatibility

### 10.2 Medium-term Vision (6-18 months)

**Advanced Modeling**:
- Boolean operations in browser
- NURBS surface modeling
- Parametric constraint solving
- Generative design algorithms

**Integration Ecosystem**:
- Plugin system for custom generators
- API for third-party tool integration
- Cloud rendering service
- Marketplace for pre-built components

**Accessibility Features**:
- Voice command interface
- Haptic feedback integration
- Screen reader compatibility
- Keyboard navigation enhancements

### 10.3 Long-term Vision (18+ months)

**AI Breakthroughs**:
- Real-time neural rendering
- Photorealistic material generation
- Physics simulation integration
- Autonomous design optimization

**Platform Expansion**:
- Mobile native applications
- AR/VR visualization
- Desktop application with local AI
- Enterprise deployment options

**Community Building**:
- Open-source model training
- Community prompt library
- Skill-based matchmaking
- Educational curriculum integration

---

## Design Principles Summary

1. **Accessibility First**: Browser-based, low-resource operation for Bharat and global markets
2. **AI Empowerment**: Human-in-the-loop design with AI assistance, not replacement
3. **Progressive Complexity**: Simple primitives to advanced CAD with clear learning paths
4. **Collaboration Ready**: Multi-environment, shareable workspaces from day one
5. **Open Foundation**: Built on open-source technologies with extensible architecture

This design positions VI3W as a transformative tool in the 3D modeling space, bridging the gap between professional CAD software and accessible creative tools through intelligent AI assistance.
