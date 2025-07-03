# Teddy Bear Soft-Body Simulation - System Architecture

## Executive Summary

This document defines the complete system architecture for a teddy bear soft-body physics simulation built on Matter.js. The architecture follows SOLID principles, provides clear separation of concerns, and establishes a robust foundation for Test-Driven Development (TDD) implementation.

## 1. System Overview

### 1.1 Architectural Principles

- **Modular Design**: Clear separation between physics, rendering, and interaction systems
- **Dependency Inversion**: Core simulation logic independent of rendering and input systems
- **Single Responsibility**: Each component has a well-defined, focused purpose
- **Extensibility**: Architecture supports future enhancements and modifications
- **Testability**: Components designed for comprehensive unit and integration testing

### 1.2 High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Application Layer                                │
├─────────────────────────────────────────────────────────────────────────────┤
│ TeddyBearSimulation │ ConfigurationManager │ PerformanceMonitor │ ErrorHandler │
├─────────────────────────────────────────────────────────────────────────────┤
│                          Interaction Layer                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│    UserInteractionController    │    EventManager    │    InputHandler      │
├─────────────────────────────────────────────────────────────────────────────┤
│                          Rendering Layer                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│   RenderingEngine   │   CanvasRenderer   │   DebugRenderer   │   UIRenderer  │
├─────────────────────────────────────────────────────────────────────────────┤
│                          Physics Layer                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│ PhysicsEngine │ SoftBodySystem │ ConstraintNetwork │ CollisionManager │ World │
├─────────────────────────────────────────────────────────────────────────────┤
│                          Entity Layer                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│   TeddyBear   │   BodySegment   │   ParticleSystem   │   JointSystem        │
├─────────────────────────────────────────────────────────────────────────────┤
│                          Matter.js Integration                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  Engine  │  Bodies  │  Constraints  │  Composite  │  Events  │  Render      │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 2. Component Architecture

### 2.1 Application Layer

#### 2.1.1 TeddyBearSimulation (Main Controller)
```javascript
class TeddyBearSimulation {
  constructor(config) {
    this.config = config;
    this.physicsEngine = null;
    this.renderingEngine = null;
    this.interactionController = null;
    this.teddyBear = null;
    this.state = 'UNINITIALIZED';
  }

  // Public API
  initialize() -> Promise<void>
  start() -> void
  pause() -> void
  resume() -> void
  destroy() -> void
  
  // State Management
  getState() -> SimulationState
  setState(state) -> void
  
  // Configuration
  updateConfig(config) -> void
  getConfig() -> SimulationConfig
}
```

**Responsibilities:**
- Orchestrate initialization of all subsystems
- Manage simulation lifecycle and state
- Coordinate between physics, rendering, and interaction layers
- Handle configuration updates and validation

#### 2.1.2 ConfigurationManager
```javascript
class ConfigurationManager {
  constructor() {
    this.config = null;
    this.validators = new Map();
    this.observers = new Set();
  }

  // Configuration Management
  loadConfiguration(source) -> Promise<SimulationConfig>
  validateConfiguration(config) -> ValidationResult
  mergeConfigurations(base, override) -> SimulationConfig
  
  // Runtime Updates
  updateProperty(path, value) -> void
  getProperty(path) -> any
  
  // Validation
  registerValidator(path, validator) -> void
  addObserver(observer) -> void
}
```

**Responsibilities:**
- Load and validate configuration from various sources
- Provide type-safe configuration access
- Support runtime configuration updates
- Notify observers of configuration changes

#### 2.1.3 PerformanceMonitor
```javascript
class PerformanceMonitor {
  constructor() {
    this.metrics = new Map();
    this.thresholds = new Map();
    this.callbacks = new Map();
  }

  // Metrics Collection
  startTiming(name) -> void
  endTiming(name) -> number
  recordMetric(name, value) -> void
  
  // Performance Analysis
  getMetrics() -> PerformanceMetrics
  getAverageMetric(name, window) -> number
  
  // Adaptive Quality
  setThreshold(metric, threshold, callback) -> void
  checkThresholds() -> void
}
```

**Responsibilities:**
- Monitor frame rate, physics performance, and memory usage
- Trigger adaptive quality adjustments
- Provide performance analytics and debugging information
- Alert when performance thresholds are exceeded

### 2.2 Physics Layer

#### 2.2.1 PhysicsEngine (Matter.js Wrapper)
```javascript
class PhysicsEngine {
  constructor(config) {
    this.matterEngine = null;
    this.world = null;
    this.config = config;
    this.bodies = new Map();
    this.constraints = new Map();
  }

  // Engine Management
  initialize() -> void
  update(deltaTime) -> void
  reset() -> void
  
  // Body Management
  addBody(body) -> string
  removeBody(id) -> void
  getBody(id) -> Body
  
  // Constraint Management
  addConstraint(constraint) -> string
  removeConstraint(id) -> void
  
  // World Properties
  setGravity(gravity) -> void
  setBounds(bounds) -> void
}
```

**Responsibilities:**
- Wrap Matter.js engine with application-specific API
- Manage physics world state and properties
- Provide abstraction layer for physics operations
- Handle physics engine lifecycle and updates

#### 2.2.2 SoftBodySystem
```javascript
class SoftBodySystem {
  constructor(physicsEngine) {
    this.physicsEngine = physicsEngine;
    this.softBodies = new Map();
    this.particleSystems = new Map();
    this.constraintNetworks = new Map();
  }

  // Soft Body Creation
  createSoftBody(config) -> SoftBody
  createParticleGrid(config) -> ParticleSystem
  createConstraintNetwork(particles, config) -> ConstraintNetwork
  
  // Runtime Management
  updateSoftBody(id, properties) -> void
  getSoftBodyDeformation(id) -> DeformationData
  
  // Performance Optimization
  optimizeConstraints() -> void
  decimateParticles(threshold) -> void
}
```

**Responsibilities:**
- Create and manage soft-body physics entities
- Generate particle systems and constraint networks
- Provide soft-body specific physics operations
- Optimize performance through adaptive quality

#### 2.2.3 ConstraintNetwork
```javascript
class ConstraintNetwork {
  constructor(particles, config) {
    this.particles = particles;
    this.constraints = [];
    this.config = config;
    this.topology = null;
  }

  // Network Construction
  buildNetwork() -> void
  addStructuralConstraints() -> void
  addShearConstraints() -> void
  addBendingConstraints() -> void
  
  // Network Management
  updateStiffness(stiffness) -> void
  validateNetwork() -> ValidationResult
  optimizeNetwork() -> void
  
  // Analysis
  getNetworkStats() -> NetworkStats
  findWeakConstraints() -> Constraint[]
}
```

**Responsibilities:**
- Generate constraint networks for soft-body simulation
- Manage different constraint types (structural, shear, bending)
- Provide network analysis and optimization capabilities
- Support runtime constraint property modifications

### 2.3 Entity Layer

#### 2.3.1 TeddyBear (Composite Entity)
```javascript
class TeddyBear {
  constructor(config) {
    this.id = generateId();
    this.config = config;
    this.segments = new Map();
    this.joints = new Map();
    this.state = 'INACTIVE';
  }

  // Lifecycle Management
  initialize() -> void
  activate() -> void
  deactivate() -> void
  destroy() -> void
  
  // Segment Management
  addSegment(type, segment) -> void
  getSegment(type) -> BodySegment
  removeSegment(type) -> void
  
  // Joint Management
  connectSegments(segmentA, segmentB, config) -> Joint
  disconnectSegments(segmentA, segmentB) -> void
  
  // State Management
  getPosition() -> Vector2
  setPosition(position) -> void
  getVelocity() -> Vector2
  applyForce(force, point) -> void
  
  // Interaction
  grab(point) -> GrabHandle
  release(handle) -> void
  
  // Serialization
  serialize() -> TeddyBearData
  deserialize(data) -> void
}
```

**Responsibilities:**
- Manage complete teddy bear entity composed of multiple segments
- Coordinate segment interactions and joint connections
- Provide high-level manipulation interface
- Handle entity lifecycle and state management

#### 2.3.2 BodySegment
```javascript
class BodySegment {
  constructor(type, config) {
    this.type = type;
    this.config = config;
    this.particleSystem = null;
    this.constraintNetwork = null;
    this.connectionPoints = new Map();
    this.state = 'INACTIVE';
  }

  // Lifecycle
  initialize(softBodySystem) -> void
  activate() -> void
  deactivate() -> void
  destroy() -> void
  
  // Particle Management
  createParticles() -> ParticleSystem
  getParticles() -> Particle[]
  getParticle(index) -> Particle
  
  // Connection Points
  defineConnectionPoint(name, selector) -> void
  getConnectionPoint(name) -> ConnectionPoint
  getConnectionParticles(name) -> Particle[]
  
  // Properties
  setSoftness(softness) -> void
  setDensity(density) -> void
  setFriction(friction) -> void
  
  // Interaction
  getClosestParticle(point) -> Particle
  applyForceAtPoint(force, point) -> void
  
  // Analysis
  getDeformationData() -> DeformationData
  getBounds() -> Bounds
  getCenter() -> Vector2
}
```

**Responsibilities:**
- Represent individual body segments (head, torso, limbs)
- Manage particle systems and constraint networks for each segment
- Define connection points for inter-segment joints
- Provide segment-specific physics operations

#### 2.3.3 ParticleSystem
```javascript
class ParticleSystem {
  constructor(config) {
    this.particles = [];
    this.config = config;
    this.gridDimensions = null;
    this.bounds = null;
  }

  // Particle Creation
  createParticleGrid(width, height, particleSize) -> void
  createParticle(position, properties) -> Particle
  
  // Grid Management
  getParticleAt(row, col) -> Particle
  getParticleIndex(row, col) -> number
  getGridPosition(index) -> {row, col}
  
  // Spatial Queries
  getParticlesInRadius(center, radius) -> Particle[]
  getParticlesInBounds(bounds) -> Particle[]
  getNearestParticle(point) -> Particle
  
  // Properties
  updateParticleProperties(properties) -> void
  getParticleCount() -> number
  
  // Optimization
  decimateParticles(factor) -> void
  mergeParticles(threshold) -> void
}
```

**Responsibilities:**
- Create and manage particle collections for soft-body simulation
- Provide spatial organization and query capabilities
- Support runtime particle manipulation and optimization
- Maintain particle grid topology for constraint creation

### 2.4 Rendering Layer

#### 2.4.1 RenderingEngine
```javascript
class RenderingEngine {
  constructor(config) {
    this.config = config;
    this.renderers = new Map();
    this.activeRenderer = null;
    this.canvas = null;
    this.context = null;
  }

  // Renderer Management
  initialize(canvas) -> void
  registerRenderer(name, renderer) -> void
  setActiveRenderer(name) -> void
  
  // Rendering Pipeline
  render(scene) -> void
  preRender() -> void
  postRender() -> void
  
  // Viewport Management
  setViewport(viewport) -> void
  getViewport() -> Viewport
  
  // Performance
  setTargetFPS(fps) -> void
  getActualFPS() -> number
}
```

**Responsibilities:**
- Manage multiple rendering strategies (wireframe, solid, debug)
- Coordinate rendering pipeline and viewport management
- Provide performance monitoring and optimization
- Support custom rendering modes and effects

#### 2.4.2 CanvasRenderer
```javascript
class CanvasRenderer {
  constructor(canvas) {
    this.canvas = canvas;
    this.context = canvas.getContext('2d');
    this.styles = new Map();
    this.effects = [];
  }

  // Rendering Operations
  clear() -> void
  renderParticle(particle, style) -> void
  renderConstraint(constraint, style) -> void
  renderSegment(segment, style) -> void
  
  // Style Management
  setStyle(name, style) -> void
  getStyle(name) -> RenderStyle
  
  // Effects
  addEffect(effect) -> void
  removeEffect(effect) -> void
  applyEffects() -> void
  
  // Utilities
  worldToScreen(worldPoint) -> ScreenPoint
  screenToWorld(screenPoint) -> WorldPoint
}
```

**Responsibilities:**
- Provide Canvas 2D rendering implementation
- Handle coordinate transformations and viewport mapping
- Support customizable rendering styles and effects
- Optimize rendering performance through batching and culling

#### 2.4.3 DebugRenderer
```javascript
class DebugRenderer {
  constructor(baseRenderer) {
    this.baseRenderer = baseRenderer;
    this.debugOptions = new Set();
    this.overlays = new Map();
  }

  // Debug Visualization
  renderDebugInfo(scene, debugData) -> void
  renderConstraintNetwork(network) -> void
  renderParticleGrid(particles) -> void
  renderConnectionPoints(segment) -> void
  
  // Debug Options
  enableDebug(option) -> void
  disableDebug(option) -> void
  toggleDebug(option) -> void
  
  // Overlays
  addOverlay(name, overlay) -> void
  removeOverlay(name) -> void
  renderOverlays() -> void
  
  // Performance Visualization
  renderPerformanceGraph(metrics) -> void
  renderFrameTime() -> void
}
```

**Responsibilities:**
- Provide comprehensive debug visualization capabilities
- Render constraint networks, particle grids, and connection points
- Display performance metrics and debugging information
- Support customizable debug options and overlays

### 2.5 Interaction Layer

#### 2.5.1 UserInteractionController
```javascript
class UserInteractionController {
  constructor(canvas, physicsEngine) {
    this.canvas = canvas;
    this.physicsEngine = physicsEngine;
    this.inputHandler = null;
    this.interactionState = 'IDLE';
    this.activeInteractions = new Map();
  }

  // Interaction Management
  initialize() -> void
  startInteraction(type, data) -> InteractionHandle
  updateInteraction(handle, data) -> void
  endInteraction(handle) -> void
  
  // Mouse/Touch Handling
  handleMouseDown(event) -> void
  handleMouseMove(event) -> void
  handleMouseUp(event) -> void
  handleTouchStart(event) -> void
  handleTouchMove(event) -> void
  handleTouchEnd(event) -> void
  
  // Interaction Types
  startDrag(point, target) -> DragHandle
  startPinch(points, target) -> PinchHandle
  startRotate(center, angle, target) -> RotateHandle
  
  // Utilities
  getTargetAt(point) -> Entity
  worldToScreen(point) -> Point
  screenToWorld(point) -> Point
}
```

**Responsibilities:**
- Handle user input events and convert to physics interactions
- Manage multiple simultaneous interactions (multi-touch)
- Provide interaction abstractions (drag, pinch, rotate)
- Coordinate between input events and physics engine

#### 2.5.2 InputHandler
```javascript
class InputHandler {
  constructor(canvas) {
    this.canvas = canvas;
    this.listeners = new Map();
    this.state = {
      mouse: { position: null, buttons: 0 },
      touches: new Map(),
      keyboard: new Set()
    };
  }

  // Event Registration
  on(event, callback) -> void
  off(event, callback) -> void
  emit(event, data) -> void
  
  // Input State
  getMousePosition() -> Point
  getMouseButtons() -> number
  getTouches() -> Touch[]
  getKeys() -> Set<string>
  
  // Coordinate Conversion
  getCanvasPosition(event) -> Point
  normalizeEvent(event) -> NormalizedEvent
  
  // Event Handling
  handleMouseEvent(event) -> void
  handleTouchEvent(event) -> void
  handleKeyboardEvent(event) -> void
}
```

**Responsibilities:**
- Capture and normalize input events across devices
- Maintain input state and provide query interface
- Handle coordinate conversion from screen to canvas space
- Provide event abstraction for interaction controller

## 3. Data Flow Architecture

### 3.1 Physics Update Cycle

```
User Input → InputHandler → UserInteractionController → PhysicsEngine
                                                             ↓
Performance ← PerformanceMonitor ← RenderingEngine ← Physics Update
Monitor                                ↑                      ↓
   ↓                              Render Data          TeddyBear State
Quality                                                       ↓
Adjustment → ConfigurationManager → SoftBodySystem → BodySegment Update
```

### 3.2 Rendering Pipeline

```
Scene Data → RenderingEngine → Renderer Selection → Canvas Operations
     ↓              ↓                    ↓                  ↓
Debug Data → DebugRenderer → Overlay Rendering → Screen Output
     ↓              ↓                    ↓                  ↓
UI Data → UIRenderer → Interface Elements → User Feedback
```

### 3.3 Event Flow

```
DOM Events → InputHandler → Event Normalization → UserInteractionController
                                                          ↓
Physics Events ← PhysicsEngine ← Interaction Translation → Interaction State
     ↓                ↓                                        ↓
Event Manager ← Application Events ← State Changes ← TeddyBear Updates
     ↓
Component Notifications → UI Updates + Performance Monitoring
```

## 4. API Design

### 4.1 Public API Interface

```javascript
interface TeddyBearSimulationAPI {
  // Lifecycle Management
  initialize(config: SimulationConfig): Promise<void>;
  start(): void;
  pause(): void;
  resume(): void;
  destroy(): void;
  
  // Configuration
  updateConfig(config: Partial<SimulationConfig>): void;
  getConfig(): SimulationConfig;
  
  // Teddy Bear Manipulation
  getTeddyBear(): TeddyBear;
  setTeddyBearPosition(position: Vector2): void;
  getTeddyBearPosition(): Vector2;
  
  // Interaction Control
  enableInteraction(): void;
  disableInteraction(): void;
  
  // Rendering Control
  setRenderMode(mode: RenderMode): void;
  enableDebugMode(options: DebugOptions): void;
  disableDebugMode(): void;
  
  // Performance Monitoring
  getPerformanceMetrics(): PerformanceMetrics;
  setPerformanceThreshold(metric: string, threshold: number): void;
  
  // Event Handling
  on(event: string, callback: Function): void;
  off(event: string, callback: Function): void;
  
  // Serialization
  serialize(): SimulationState;
  deserialize(state: SimulationState): void;
}
```

### 4.2 Configuration Schema

```typescript
interface SimulationConfig {
  canvas: {
    width: number;
    height: number;
    pixelRatio?: number;
  };
  
  physics: {
    gravity: Vector2;
    airFriction: number;
    constraintIterations: number;
    positionIterations: number;
    velocityIterations: number;
    enableSleeping: boolean;
    sleepThreshold: number;
  };
  
  bear: {
    position: Vector2;
    scale: number;
    softness: number;
    density: number;
    segments: {
      head: SegmentConfig;
      torso: SegmentConfig;
      leftArm: SegmentConfig;
      rightArm: SegmentConfig;
      leftLeg: SegmentConfig;
      rightLeg: SegmentConfig;
    };
  };
  
  rendering: {
    mode: 'wireframe' | 'solid' | 'debug';
    backgroundColor: string;
    particleColors: Record<string, string>;
    constraintColors: Record<string, string>;
    showConstraints: boolean;
    showParticles: boolean;
    showConnectionPoints: boolean;
  };
  
  interaction: {
    enabled: boolean;
    dragStiffness: number;
    multiTouch: boolean;
    touchRadius: number;
  };
  
  performance: {
    targetFPS: number;
    adaptiveQuality: boolean;
    performanceThresholds: Record<string, number>;
  };
}

interface SegmentConfig {
  dimensions: {
    width: number;
    height: number;
  };
  particleSize: number;
  particleCount: {
    x: number;
    y: number;
  };
  softness: number;
  density: number;
  frictionAir: number;
  connectionPoints: Record<string, ConnectionPointConfig>;
}
```

## 5. Integration Patterns

### 5.1 Matter.js Integration Strategy

#### 5.1.1 Engine Wrapper Pattern
```javascript
class MatterEngineWrapper {
  constructor(config) {
    this.engine = Matter.Engine.create(config.matterConfig);
    this.world = this.engine.world;
    this.bodies = new Map();
    this.constraints = new Map();
  }
  
  // Abstraction Methods
  createBody(definition) {
    const body = Matter.Bodies[definition.type](...definition.args);
    const id = this.generateId();
    this.bodies.set(id, body);
    Matter.World.add(this.world, body);
    return id;
  }
  
  createConstraint(definition) {
    const constraint = Matter.Constraint.create(definition);
    const id = this.generateId();
    this.constraints.set(id, constraint);
    Matter.World.add(this.world, constraint);
    return id;
  }
}
```

#### 5.1.2 Event Bridge Pattern
```javascript
class MatterEventBridge {
  constructor(matterEngine, eventManager) {
    this.matterEngine = matterEngine;
    this.eventManager = eventManager;
    this.setupEventMapping();
  }
  
  setupEventMapping() {
    Matter.Events.on(this.matterEngine, 'collisionStart', (event) => {
      this.eventManager.emit('physics:collision:start', {
        pairs: event.pairs.map(this.transformPair.bind(this))
      });
    });
    
    Matter.Events.on(this.matterEngine, 'beforeUpdate', (event) => {
      this.eventManager.emit('physics:before-update', {
        timestamp: event.timestamp,
        deltaTime: event.delta
      });
    });
  }
}
```

### 5.2 Soft-Body Implementation Pattern

#### 5.2.1 Particle-Constraint Network
```javascript
class SoftBodyImplementation {
  createSoftBody(config) {
    // Create particle grid
    const particles = this.createParticleGrid(config);
    
    // Create constraint network
    const constraints = this.createConstraintNetwork(particles, config);
    
    // Register with physics engine
    const bodyIds = particles.map(p => this.physicsEngine.addBody(p));
    const constraintIds = constraints.map(c => this.physicsEngine.addConstraint(c));
    
    return new SoftBody({
      particles: bodyIds,
      constraints: constraintIds,
      config: config
    });
  }
  
  createConstraintNetwork(particles, config) {
    const constraints = [];
    const gridWidth = config.particleCount.x;
    const gridHeight = config.particleCount.y;
    
    // Structural constraints (horizontal and vertical)
    for (let row = 0; row < gridHeight; row++) {
      for (let col = 0; col < gridWidth; col++) {
        const current = particles[row * gridWidth + col];
        
        // Horizontal constraint
        if (col < gridWidth - 1) {
          const right = particles[row * gridWidth + (col + 1)];
          constraints.push(this.createConstraint(current, right, config));
        }
        
        // Vertical constraint
        if (row < gridHeight - 1) {
          const bottom = particles[(row + 1) * gridWidth + col];
          constraints.push(this.createConstraint(current, bottom, config));
        }
      }
    }
    
    return constraints;
  }
}
```

## 6. Performance Architecture

### 6.1 Memory Management Strategy

#### 6.1.1 Object Pooling
```javascript
class ObjectPool {
  constructor(createFn, resetFn, initialSize = 10) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    this.pool = [];
    this.active = new Set();
    
    // Pre-populate pool
    for (let i = 0; i < initialSize; i++) {
      this.pool.push(this.createFn());
    }
  }
  
  acquire() {
    let object;
    if (this.pool.length > 0) {
      object = this.pool.pop();
    } else {
      object = this.createFn();
    }
    
    this.active.add(object);
    return object;
  }
  
  release(object) {
    if (this.active.has(object)) {
      this.active.delete(object);
      this.resetFn(object);
      this.pool.push(object);
    }
  }
}
```

#### 6.1.2 Adaptive Quality System
```javascript
class AdaptiveQualitySystem {
  constructor(performanceMonitor, config) {
    this.performanceMonitor = performanceMonitor;
    this.config = config;
    this.qualityLevel = 1.0;
    this.thresholds = {
      reduce: { fps: 30, physicsTime: 16 },
      restore: { fps: 50, physicsTime: 10 }
    };
  }
  
  update() {
    const metrics = this.performanceMonitor.getMetrics();
    
    if (this.shouldReduceQuality(metrics)) {
      this.reduceQuality();
    } else if (this.shouldRestoreQuality(metrics)) {
      this.restoreQuality();
    }
  }
  
  reduceQuality() {
    this.qualityLevel = Math.max(0.1, this.qualityLevel - 0.1);
    this.applyQualitySettings();
  }
  
  applyQualitySettings() {
    // Reduce constraint iterations
    this.config.physics.constraintIterations = Math.max(1, 
      Math.floor(this.config.physics.constraintIterations * this.qualityLevel));
    
    // Reduce particle density
    this.config.bear.segments.forEach(segment => {
      segment.particleCount.x = Math.max(3, 
        Math.floor(segment.particleCount.x * this.qualityLevel));
      segment.particleCount.y = Math.max(3, 
        Math.floor(segment.particleCount.y * this.qualityLevel));
    });
  }
}
```

### 6.2 Optimization Strategies

#### 6.2.1 Spatial Partitioning
```javascript
class SpatialHashGrid {
  constructor(cellSize) {
    this.cellSize = cellSize;
    this.grid = new Map();
  }
  
  insert(object, bounds) {
    const cells = this.getCells(bounds);
    
    cells.forEach(cell => {
      if (!this.grid.has(cell)) {
        this.grid.set(cell, new Set());
      }
      this.grid.get(cell).add(object);
    });
  }
  
  query(bounds) {
    const cells = this.getCells(bounds);
    const results = new Set();
    
    cells.forEach(cell => {
      if (this.grid.has(cell)) {
        this.grid.get(cell).forEach(object => results.add(object));
      }
    });
    
    return Array.from(results);
  }
  
  getCells(bounds) {
    const cells = [];
    const minX = Math.floor(bounds.min.x / this.cellSize);
    const maxX = Math.floor(bounds.max.x / this.cellSize);
    const minY = Math.floor(bounds.min.y / this.cellSize);
    const maxY = Math.floor(bounds.max.y / this.cellSize);
    
    for (let x = minX; x <= maxX; x++) {
      for (let y = minY; y <= maxY; y++) {
        cells.push(`${x},${y}`);
      }
    }
    
    return cells;
  }
}
```

## 7. Error Handling Architecture

### 7.1 Error Hierarchy

```javascript
class SimulationError extends Error {
  constructor(message, code, context) {
    super(message);
    this.name = 'SimulationError';
    this.code = code;
    this.context = context;
    this.timestamp = Date.now();
  }
}

class PhysicsError extends SimulationError {
  constructor(message, context) {
    super(message, 'PHYSICS_ERROR', context);
    this.name = 'PhysicsError';
  }
}

class RenderingError extends SimulationError {
  constructor(message, context) {
    super(message, 'RENDERING_ERROR', context);
    this.name = 'RenderingError';
  }
}

class ConfigurationError extends SimulationError {
  constructor(message, context) {
    super(message, 'CONFIGURATION_ERROR', context);
    this.name = 'ConfigurationError';
  }
}
```

### 7.2 Error Handling Strategy

```javascript
class ErrorHandler {
  constructor(config) {
    this.config = config;
    this.errorLog = [];
    this.recoveryStrategies = new Map();
    this.setupRecoveryStrategies();
  }
  
  setupRecoveryStrategies() {
    this.recoveryStrategies.set('PHYSICS_ERROR', this.recoverFromPhysicsError.bind(this));
    this.recoveryStrategies.set('RENDERING_ERROR', this.recoverFromRenderingError.bind(this));
    this.recoveryStrategies.set('CONFIGURATION_ERROR', this.recoverFromConfigurationError.bind(this));
  }
  
  handleError(error) {
    // Log error
    this.logError(error);
    
    // Attempt recovery
    const strategy = this.recoveryStrategies.get(error.code);
    if (strategy) {
      try {
        strategy(error);
      } catch (recoveryError) {
        this.handleFatalError(recoveryError);
      }
    } else {
      this.handleUnknownError(error);
    }
  }
  
  recoverFromPhysicsError(error) {
    // Reset physics engine
    this.physicsEngine.reset();
    
    // Reload configuration
    this.configurationManager.reloadConfiguration();
    
    // Reinitialize teddy bear
    this.teddyBear.initialize();
  }
  
  recoverFromRenderingError(error) {
    // Switch to fallback renderer
    this.renderingEngine.setActiveRenderer('fallback');
    
    // Reduce rendering quality
    this.renderingEngine.setQuality(0.5);
  }
}
```

## 8. Testing Architecture

### 8.1 Test Structure

```
tests/
├── unit/
│   ├── entities/
│   │   ├── teddy-bear.test.js
│   │   ├── body-segment.test.js
│   │   └── particle-system.test.js
│   ├── physics/
│   │   ├── physics-engine.test.js
│   │   ├── soft-body-system.test.js
│   │   └── constraint-network.test.js
│   ├── rendering/
│   │   ├── rendering-engine.test.js
│   │   ├── canvas-renderer.test.js
│   │   └── debug-renderer.test.js
│   └── interaction/
│       ├── user-interaction-controller.test.js
│       └── input-handler.test.js
├── integration/
│   ├── physics-rendering.test.js
│   ├── interaction-physics.test.js
│   └── complete-simulation.test.js
├── e2e/
│   ├── simulation-lifecycle.test.js
│   ├── user-interaction.test.js
│   └── performance.test.js
├── fixtures/
│   ├── test-configurations.js
│   ├── mock-physics-engine.js
│   └── test-scenarios.js
└── helpers/
    ├── test-utils.js
    ├── performance-helpers.js
    └── assertion-helpers.js
```

### 8.2 Test Interface Design

```javascript
// Test Interface for Physics Engine
class PhysicsEngineTestInterface {
  constructor(physicsEngine) {
    this.physicsEngine = physicsEngine;
  }
  
  // Test Helpers
  getBodyCount() {
    return this.physicsEngine.bodies.size;
  }
  
  getConstraintCount() {
    return this.physicsEngine.constraints.size;
  }
  
  simulateSteps(steps) {
    for (let i = 0; i < steps; i++) {
      this.physicsEngine.update(1000 / 60);
    }
  }
  
  // Validation Helpers
  validatePhysicsState() {
    // Check for NaN positions
    // Validate constraint integrity
    // Check energy conservation
  }
  
  // Mock Helpers
  createMockBody(properties) {
    return {
      id: 'mock-body',
      position: { x: 0, y: 0 },
      velocity: { x: 0, y: 0 },
      ...properties
    };
  }
}
```

### 8.3 Performance Testing Framework

```javascript
class PerformanceTestSuite {
  constructor() {
    this.benchmarks = new Map();
    this.results = [];
  }
  
  addBenchmark(name, testFn, config) {
    this.benchmarks.set(name, { testFn, config });
  }
  
  async runBenchmark(name) {
    const benchmark = this.benchmarks.get(name);
    if (!benchmark) throw new Error(`Benchmark ${name} not found`);
    
    const results = [];
    const iterations = benchmark.config.iterations || 100;
    
    for (let i = 0; i < iterations; i++) {
      const start = performance.now();
      await benchmark.testFn();
      const end = performance.now();
      results.push(end - start);
    }
    
    return {
      name,
      iterations,
      min: Math.min(...results),
      max: Math.max(...results),
      avg: results.reduce((a, b) => a + b) / results.length,
      median: this.calculateMedian(results)
    };
  }
  
  calculateMedian(values) {
    const sorted = values.sort((a, b) => a - b);
    const mid = Math.floor(sorted.length / 2);
    return sorted.length % 2 === 0 
      ? (sorted[mid - 1] + sorted[mid]) / 2 
      : sorted[mid];
  }
}
```

## 9. Extensibility Points

### 9.1 Plugin Architecture

```javascript
class PluginManager {
  constructor() {
    this.plugins = new Map();
    this.hooks = new Map();
  }
  
  registerPlugin(name, plugin) {
    this.plugins.set(name, plugin);
    
    // Register hooks
    if (plugin.hooks) {
      plugin.hooks.forEach((hook, hookName) => {
        if (!this.hooks.has(hookName)) {
          this.hooks.set(hookName, []);
        }
        this.hooks.get(hookName).push(hook);
      });
    }
  }
  
  executeHook(hookName, data) {
    const hooks = this.hooks.get(hookName) || [];
    return hooks.reduce((result, hook) => hook(result), data);
  }
  
  getPlugin(name) {
    return this.plugins.get(name);
  }
}
```

### 9.2 Custom Renderer Interface

```javascript
interface CustomRenderer {
  initialize(canvas: HTMLCanvasElement): void;
  render(scene: SceneData): void;
  setViewport(viewport: Viewport): void;
  cleanup(): void;
}

class CustomRendererRegistry {
  constructor() {
    this.renderers = new Map();
  }
  
  register(name, rendererClass) {
    this.renderers.set(name, rendererClass);
  }
  
  create(name, config) {
    const RendererClass = this.renderers.get(name);
    if (!RendererClass) {
      throw new Error(`Renderer ${name} not found`);
    }
    return new RendererClass(config);
  }
}
```

## 10. Deployment Architecture

### 10.1 Build Configuration

```javascript
// webpack.config.js
module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'teddy-bear-simulation.js',
    library: 'TeddyBearSimulation',
    libraryTarget: 'umd'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: 'babel-loader'
      }
    ]
  },
  externals: {
    'matter-js': 'Matter'
  },
  optimization: {
    minimize: true,
    usedExports: true,
    sideEffects: false
  }
};
```

### 10.2 Performance Monitoring

```javascript
class ProductionMonitor {
  constructor() {
    this.metrics = {
      frameRate: new RollingAverage(60),
      physicsTime: new RollingAverage(60),
      renderTime: new RollingAverage(60),
      memoryUsage: new RollingAverage(10)
    };
  }
  
  recordFrame(frameData) {
    this.metrics.frameRate.add(1000 / frameData.deltaTime);
    this.metrics.physicsTime.add(frameData.physicsTime);
    this.metrics.renderTime.add(frameData.renderTime);
    
    // Check for performance issues
    if (this.metrics.frameRate.getAverage() < 30) {
      this.reportPerformanceIssue('low-fps', {
        fps: this.metrics.frameRate.getAverage(),
        physicsTime: this.metrics.physicsTime.getAverage(),
        renderTime: this.metrics.renderTime.getAverage()
      });
    }
  }
  
  recordMemoryUsage() {
    if (performance.memory) {
      this.metrics.memoryUsage.add(performance.memory.usedJSHeapSize);
    }
  }
  
  reportPerformanceIssue(type, data) {
    // Send to analytics service
    // Log to console in development
    // Trigger adaptive quality adjustments
  }
}
```

## 11. Summary

This system architecture provides a comprehensive blueprint for implementing the teddy bear soft-body simulation with the following key characteristics:

### 11.1 Architectural Strengths

1. **Modularity**: Clear separation of concerns with well-defined interfaces
2. **Testability**: Each component designed for comprehensive testing
3. **Extensibility**: Plugin architecture and extensibility points for future enhancements
4. **Performance**: Adaptive quality system and optimization strategies
5. **Maintainability**: Clean architecture with dependency inversion
6. **Robustness**: Comprehensive error handling and recovery mechanisms

### 11.2 Implementation Guidelines

1. **Start with Core Components**: Begin with Entity and Physics layers
2. **Implement Test Infrastructure**: Set up testing framework early
3. **Add Rendering Layer**: Implement basic rendering before advanced features
4. **Integrate Interaction**: Add user interaction capabilities
5. **Optimize Performance**: Implement adaptive quality and monitoring
6. **Add Extensions**: Implement plugin system and custom renderers

### 11.3 Quality Assurance

1. **Unit Test Coverage**: Minimum 95% coverage for all core components
2. **Integration Testing**: Comprehensive testing of component interactions
3. **Performance Testing**: Automated performance benchmarking
4. **Cross-browser Testing**: Validation across target browsers and devices
5. **Memory Leak Testing**: Automated detection of memory leaks
6. **User Acceptance Testing**: Validation of user interaction scenarios

This architecture provides a solid foundation for systematic Test-Driven Development, ensuring that each component can be implemented, tested, and validated independently while maintaining system cohesion and performance.