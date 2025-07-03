# Teddy Bear Soft-Body Physics Simulation - Detailed Pseudocode

## Overview
This pseudocode defines the complete implementation logic for a teddy bear soft-body physics simulation using Matter.js, following the specifications created in Phase 1. The simulation creates a realistic rag doll with soft-body physics for interactive manipulation.

## 1. Initialization and Setup

### 1.1 Main Initialization Function
```
FUNCTION initializeTeddyBearSimulation()
    // Validate environment and dependencies
    IF NOT Matter.js available THEN
        THROW "Matter.js physics engine not available"
    END IF
    
    // Initialize core systems
    engine = setupPhysicsEngine()
    world = engine.world
    render = setupRenderer()
    mouseConstraint = setupMouseInteraction()
    
    // Create teddy bear
    teddyBear = createTeddyBear()
    
    // Start simulation loop
    startSimulationLoop()
    
    RETURN {engine, render, teddyBear, mouseConstraint}
END FUNCTION
```

### 1.2 Physics Engine Configuration
```
FUNCTION setupPhysicsEngine()
    // Create Matter.js engine with optimized settings
    engine = Matter.Engine.create({
        gravity: {x: 0, y: 0.8, scale: 0.001},
        timing: {
            timeScale: 1,
            fixedDelta: 1000/60  // 60 FPS target
        },
        velocityIterations: 4,
        positionIterations: 6,
        constraintIterations: 2,
        enableSleeping: true,
        sleepThreshold: 60
    })
    
    // Configure collision detection
    engine.world.bounds = {
        min: {x: -2000, y: -2000},
        max: {x: 2000, y: 2000}
    }
    
    RETURN engine
END FUNCTION
```

### 1.3 Renderer Setup
```
FUNCTION setupRenderer()
    render = Matter.Render.create({
        element: document.body,
        engine: engine,
        options: {
            width: 800,
            height: 600,
            wireframes: true,
            showAngleIndicator: false,
            showVelocity: false,
            showIds: false,
            showSeparations: false,
            showCollisions: false,
            showVertexNumbers: false,
            showConvexHulls: false,
            showBounds: false,
            showDebug: false,
            background: 'transparent',
            wireframeBackground: 'transparent'
        }
    })
    
    RETURN render
END FUNCTION
```

## 2. Teddy Bear Construction

### 2.1 Main Bear Creation Function
```
FUNCTION createTeddyBear()
    // Define bear proportions and positioning
    bearConfig = {
        centerX: 400,
        centerY: 200,
        scale: 1.0,
        softness: 0.7,  // 0 = rigid, 1 = very soft
        segments: {}
    }
    
    // Create all body segments
    head = createHeadSegment(bearConfig)
    torso = createTorsoSegment(bearConfig)
    leftArm = createArmSegment(bearConfig, "left")
    rightArm = createArmSegment(bearConfig, "right")
    leftLeg = createLegSegment(bearConfig, "left")
    rightLeg = createLegSegment(bearConfig, "right")
    
    // Store segments for reference
    bearConfig.segments = {head, torso, leftArm, rightArm, leftLeg, rightLeg}
    
    // Connect segments with joints
    joints = createJointConnections(bearConfig.segments)
    
    // Add all bodies to world
    allBodies = []
    FOR each segment IN bearConfig.segments
        allBodies.concat(segment.bodies)
    END FOR
    Matter.World.add(world, allBodies)
    Matter.World.add(world, joints)
    
    RETURN {segments: bearConfig.segments, joints: joints, allBodies: allBodies}
END FUNCTION
```

### 2.2 Segment Creation Algorithm
```
FUNCTION createSegment(config)
    INPUT: config = {
        centerX, centerY,     // Center position
        width, height,        // Segment dimensions
        particleSize,         // Individual particle radius
        softness,            // Constraint stiffness (0-1)
        density,             // Mass density
        frictionAir,         // Air resistance
        segmentType          // "head", "torso", "arm", "leg"
    }
    
    // Calculate particle grid dimensions
    particlesX = Math.ceil(config.width / (config.particleSize * 2))
    particlesY = Math.ceil(config.height / (config.particleSize * 2))
    
    // Ensure minimum particle density
    IF particlesX < 3 THEN particlesX = 3
    IF particlesY < 3 THEN particlesY = 3
    
    // Create particle grid
    particles = []
    FOR row = 0 TO particlesY - 1
        FOR col = 0 TO particlesX - 1
            x = config.centerX - (config.width/2) + (col * config.width / (particlesX-1))
            y = config.centerY - (config.height/2) + (row * config.height / (particlesY-1))
            
            // Create circular particle
            particle = Matter.Bodies.circle(x, y, config.particleSize, {
                density: config.density,
                frictionAir: config.frictionAir,
                collisionFilter: {
                    group: -1,  // Prevent self-collision
                    category: getCollisionCategory(config.segmentType),
                    mask: getCollisionMask(config.segmentType)
                }
            })
            
            particles.push(particle)
        END FOR
    END FOR
    
    // Create constraint network
    constraints = createConstraintNetwork(particles, particlesX, particlesY, config.softness)
    
    RETURN {
        bodies: particles,
        constraints: constraints,
        dimensions: {width: config.width, height: config.height},
        center: {x: config.centerX, y: config.centerY},
        type: config.segmentType
    }
END FUNCTION
```

### 2.3 Constraint Network Creation
```
FUNCTION createConstraintNetwork(particles, gridWidth, gridHeight, softness)
    constraints = []
    stiffness = 1.0 - softness  // Convert softness to stiffness
    
    FOR row = 0 TO gridHeight - 1
        FOR col = 0 TO gridWidth - 1
            currentIndex = row * gridWidth + col
            currentParticle = particles[currentIndex]
            
            // Horizontal connection (right neighbor)
            IF col < gridWidth - 1 THEN
                rightIndex = currentIndex + 1
                rightParticle = particles[rightIndex]
                
                constraint = Matter.Constraint.create({
                    bodyA: currentParticle,
                    bodyB: rightParticle,
                    stiffness: stiffness,
                    damping: 0.1,
                    length: calculateDistance(currentParticle.position, rightParticle.position)
                })
                constraints.push(constraint)
            END IF
            
            // Vertical connection (bottom neighbor)
            IF row < gridHeight - 1 THEN
                bottomIndex = currentIndex + gridWidth
                bottomParticle = particles[bottomIndex]
                
                constraint = Matter.Constraint.create({
                    bodyA: currentParticle,
                    bodyB: bottomParticle,
                    stiffness: stiffness,
                    damping: 0.1,
                    length: calculateDistance(currentParticle.position, bottomParticle.position)
                })
                constraints.push(constraint)
            END IF
            
            // Diagonal connections for stability
            IF row < gridHeight - 1 AND col < gridWidth - 1 THEN
                // Bottom-right diagonal
                diagonalIndex = (row + 1) * gridWidth + (col + 1)
                diagonalParticle = particles[diagonalIndex]
                
                constraint = Matter.Constraint.create({
                    bodyA: currentParticle,
                    bodyB: diagonalParticle,
                    stiffness: stiffness * 0.5,  // Weaker diagonal constraints
                    damping: 0.1,
                    length: calculateDistance(currentParticle.position, diagonalParticle.position)
                })
                constraints.push(constraint)
            END IF
            
            // Bottom-left diagonal
            IF row < gridHeight - 1 AND col > 0 THEN
                diagonalIndex = (row + 1) * gridWidth + (col - 1)
                diagonalParticle = particles[diagonalIndex]
                
                constraint = Matter.Constraint.create({
                    bodyA: currentParticle,
                    bodyB: diagonalParticle,
                    stiffness: stiffness * 0.5,
                    damping: 0.1,
                    length: calculateDistance(currentParticle.position, diagonalParticle.position)
                })
                constraints.push(constraint)
            END IF
        END FOR
    END FOR
    
    RETURN constraints
END FUNCTION
```

## 3. Specific Segment Creation

### 3.1 Head Segment
```
FUNCTION createHeadSegment(bearConfig)
    headConfig = {
        centerX: bearConfig.centerX,
        centerY: bearConfig.centerY - 80,
        width: 60,
        height: 50,
        particleSize: 4,
        softness: bearConfig.softness * 0.8,  // Slightly firmer than body
        density: 0.001,
        frictionAir: 0.01,
        segmentType: "head"
    }
    
    head = createSegment(headConfig)
    
    // Add facial features (optional rigid bodies)
    eyes = createEyes(headConfig.centerX, headConfig.centerY - 10)
    nose = createNose(headConfig.centerX, headConfig.centerY + 5)
    
    head.features = {eyes, nose}
    
    RETURN head
END FUNCTION
```

### 3.2 Torso Segment
```
FUNCTION createTorsoSegment(bearConfig)
    torsoConfig = {
        centerX: bearConfig.centerX,
        centerY: bearConfig.centerY,
        width: 80,
        height: 100,
        particleSize: 5,
        softness: bearConfig.softness,
        density: 0.001,
        frictionAir: 0.01,
        segmentType: "torso"
    }
    
    RETURN createSegment(torsoConfig)
END FUNCTION
```

### 3.3 Limb Segments
```
FUNCTION createArmSegment(bearConfig, side)
    offsetX = IF side == "left" THEN -50 ELSE 50
    
    armConfig = {
        centerX: bearConfig.centerX + offsetX,
        centerY: bearConfig.centerY - 20,
        width: 25,
        height: 70,
        particleSize: 3,
        softness: bearConfig.softness * 1.2,  // More flexible than torso
        density: 0.0008,
        frictionAir: 0.015,
        segmentType: "arm"
    }
    
    RETURN createSegment(armConfig)
END FUNCTION

FUNCTION createLegSegment(bearConfig, side)
    offsetX = IF side == "left" THEN -25 ELSE 25
    
    legConfig = {
        centerX: bearConfig.centerX + offsetX,
        centerY: bearConfig.centerY + 80,
        width: 30,
        height: 80,
        particleSize: 4,
        softness: bearConfig.softness * 1.1,
        density: 0.0012,
        frictionAir: 0.012,
        segmentType: "leg"
    }
    
    RETURN createSegment(legConfig)
END FUNCTION
```

## 4. Joint Connection System

### 4.1 Main Joint Creation
```
FUNCTION createJointConnections(segments)
    joints = []
    
    // Head to torso connection
    headToTorso = createFlexibleJoint(
        segments.head, segments.torso,
        "bottom", "top",
        {stiffness: 0.8, damping: 0.2}
    )
    joints.concat(headToTorso)
    
    // Arms to torso connections
    leftArmToTorso = createFlexibleJoint(
        segments.leftArm, segments.torso,
        "top", "left-shoulder",
        {stiffness: 0.6, damping: 0.15}
    )
    joints.concat(leftArmToTorso)
    
    rightArmToTorso = createFlexibleJoint(
        segments.rightArm, segments.torso,
        "top", "right-shoulder",
        {stiffness: 0.6, damping: 0.15}
    )
    joints.concat(rightArmToTorso)
    
    // Legs to torso connections
    leftLegToTorso = createFlexibleJoint(
        segments.leftLeg, segments.torso,
        "top", "left-hip",
        {stiffness: 0.7, damping: 0.18}
    )
    joints.concat(leftLegToTorso)
    
    rightLegToTorso = createFlexibleJoint(
        segments.rightLeg, segments.torso,
        "top", "right-hip",
        {stiffness: 0.7, damping: 0.18}
    )
    joints.concat(rightLegToTorso)
    
    RETURN joints
END FUNCTION
```

### 4.2 Flexible Joint Creation
```
FUNCTION createFlexibleJoint(segmentA, segmentB, connectionPointA, connectionPointB, properties)
    // Get connection particles from segments
    particlesA = getConnectionParticles(segmentA, connectionPointA)
    particlesB = getConnectionParticles(segmentB, connectionPointB)
    
    joints = []
    
    // Create multiple constraint connections for flexibility
    FOR i = 0 TO min(particlesA.length, particlesB.length) - 1
        constraint = Matter.Constraint.create({
            bodyA: particlesA[i],
            bodyB: particlesB[i],
            stiffness: properties.stiffness,
            damping: properties.damping,
            length: calculateDistance(particlesA[i].position, particlesB[i].position)
        })
        joints.push(constraint)
    END FOR
    
    RETURN joints
END FUNCTION
```

### 4.3 Connection Point Identification
```
FUNCTION getConnectionParticles(segment, connectionPoint)
    particles = segment.bodies
    gridWidth = calculateGridWidth(segment)
    gridHeight = calculateGridHeight(segment)
    
    SWITCH connectionPoint
        CASE "top":
            RETURN getTopRowParticles(particles, gridWidth)
        CASE "bottom":
            RETURN getBottomRowParticles(particles, gridWidth, gridHeight)
        CASE "left-shoulder":
            RETURN getLeftShoulderParticles(particles, gridWidth, gridHeight)
        CASE "right-shoulder":
            RETURN getRightShoulderParticles(particles, gridWidth, gridHeight)
        CASE "left-hip":
            RETURN getLeftHipParticles(particles, gridWidth, gridHeight)
        CASE "right-hip":
            RETURN getRightHipParticles(particles, gridWidth, gridHeight)
        DEFAULT:
            RETURN [particles[0]]  // Fallback to first particle
    END SWITCH
END FUNCTION
```

## 5. Interaction System

### 5.1 Mouse/Touch Interaction Setup
```
FUNCTION setupMouseInteraction()
    mouseConstraint = Matter.MouseConstraint.create(engine, {
        mouse: Matter.Mouse.create(render.canvas),
        constraint: {
            stiffness: 0.2,
            render: {
                visible: false
            }
        }
    })
    
    // Add event listeners for enhanced interaction
    Matter.Events.on(mouseConstraint, 'mousedown', handleMouseDown)
    Matter.Events.on(mouseConstraint, 'mouseup', handleMouseUp)
    Matter.Events.on(mouseConstraint, 'mousemove', handleMouseMove)
    
    RETURN mouseConstraint
END FUNCTION
```

### 5.2 Interaction Event Handlers
```
FUNCTION handleMouseDown(event)
    // Identify which segment is being grabbed
    targetBody = event.source.body
    IF targetBody THEN
        grabbedSegment = identifySegment(targetBody)
        IF grabbedSegment THEN
            // Increase constraint stiffness for better control
            event.source.constraint.stiffness = 0.8
            
            // Optional: Add visual feedback
            highlightSegment(grabbedSegment)
        END IF
    END IF
END FUNCTION

FUNCTION handleMouseUp(event)
    // Reset constraint stiffness
    event.source.constraint.stiffness = 0.2
    
    // Remove visual feedback
    clearHighlights()
END FUNCTION

FUNCTION handleMouseMove(event)
    // Optional: Implement drag prediction or smoothing
    IF event.source.constraint.bodyB THEN
        // Smooth the mouse movement for more natural feel
        smoothMousePosition(event.mouse.position)
    END IF
END FUNCTION
```

## 6. Rendering and Animation

### 6.1 Main Rendering Loop
```
FUNCTION startSimulationLoop()
    FUNCTION animate()
        // Update physics
        Matter.Engine.update(engine, 1000/60)
        
        // Custom rendering (if needed beyond Matter.js default)
        performCustomRendering()
        
        // Performance monitoring
        updatePerformanceMetrics()
        
        // Schedule next frame
        requestAnimationFrame(animate)
    END FUNCTION
    
    // Start the loop
    animate()
END FUNCTION
```

### 6.2 Custom Rendering Functions
```
FUNCTION performCustomRendering()
    // Clear canvas
    clearCanvas()
    
    // Draw segments with custom styling
    FOR each segment IN teddyBear.segments
        drawSegment(segment)
    END FOR
    
    // Draw joints/connections
    FOR each joint IN teddyBear.joints
        drawJoint(joint)
    END FOR
    
    // Draw UI elements
    drawUI()
END FUNCTION

FUNCTION drawSegment(segment)
    // Draw particles
    FOR each particle IN segment.bodies
        drawCircle(particle.position, particle.circleRadius, {
            strokeColor: getSegmentColor(segment.type),
            fillColor: 'transparent',
            lineWidth: 2
        })
    END FOR
    
    // Draw constraints (optional)
    IF showConstraints THEN
        FOR each constraint IN segment.constraints
            drawLine(
                constraint.bodyA.position,
                constraint.bodyB.position,
                {color: 'rgba(255,255,255,0.3)', lineWidth: 1}
            )
        END FOR
    END IF
END FUNCTION
```

## 7. Performance Optimization

### 7.1 Collision Filtering
```
FUNCTION getCollisionCategory(segmentType)
    SWITCH segmentType
        CASE "head": RETURN 0x0001
        CASE "torso": RETURN 0x0002
        CASE "arm": RETURN 0x0004
        CASE "leg": RETURN 0x0008
        DEFAULT: RETURN 0x0001
    END SWITCH
END FUNCTION

FUNCTION getCollisionMask(segmentType)
    // Allow collision with environment but not with other teddy parts
    RETURN 0x0010  // Environment collision category
END FUNCTION
```

### 7.2 Performance Monitoring
```
FUNCTION updatePerformanceMetrics()
    currentTime = performance.now()
    frameTime = currentTime - lastFrameTime
    lastFrameTime = currentTime
    
    // Track FPS
    fps = 1000 / frameTime
    
    // Track physics performance
    physicsTime = engine.timing.lastElapsed
    
    // Adaptive quality based on performance
    IF fps < 30 THEN
        reduceQuality()
    ELSE IF fps > 50 AND qualityReduced THEN
        restoreQuality()
    END IF
END FUNCTION

FUNCTION reduceQuality()
    // Reduce constraint iterations
    engine.constraintIterations = 1
    
    // Reduce particle count if necessary
    IF fps < 20 THEN
        decimateParticles()
    END IF
    
    qualityReduced = true
END FUNCTION
```

### 7.3 Memory Management
```
FUNCTION cleanupSimulation()
    // Remove all bodies from world
    Matter.World.clear(world)
    
    // Clear event listeners
    Matter.Events.off(mouseConstraint, 'mousedown')
    Matter.Events.off(mouseConstraint, 'mouseup')
    Matter.Events.off(mouseConstraint, 'mousemove')
    
    // Stop rendering
    Matter.Render.stop(render)
    
    // Clear engine
    Matter.Engine.clear(engine)
END FUNCTION
```

## 8. Error Handling and Validation

### 8.1 Input Validation
```
FUNCTION validateBearConfig(config)
    IF NOT config THEN
        THROW "Bear configuration is required"
    END IF
    
    IF config.centerX < 0 OR config.centerX > canvasWidth THEN
        THROW "Invalid centerX position"
    END IF
    
    IF config.centerY < 0 OR config.centerY > canvasHeight THEN
        THROW "Invalid centerY position"
    END IF
    
    IF config.scale <= 0 OR config.scale > 5 THEN
        THROW "Scale must be between 0 and 5"
    END IF
    
    IF config.softness < 0 OR config.softness > 1 THEN
        THROW "Softness must be between 0 and 1"
    END IF
END FUNCTION
```

### 8.2 Runtime Error Handling
```
FUNCTION safeExecute(operation, fallback)
    TRY
        RETURN operation()
    CATCH error
        console.error("Operation failed:", error)
        IF fallback THEN
            RETURN fallback()
        END IF
        RETURN null
    END TRY
END FUNCTION
```

## 9. Integration Points and Utilities

### 9.1 Utility Functions
```
FUNCTION calculateDistance(pointA, pointB)
    dx = pointA.x - pointB.x
    dy = pointA.y - pointB.y
    RETURN Math.sqrt(dx * dx + dy * dy)
END FUNCTION

FUNCTION calculateGridWidth(segment)
    RETURN Math.ceil(segment.dimensions.width / (segment.particleSize * 2))
END FUNCTION

FUNCTION calculateGridHeight(segment)
    RETURN Math.ceil(segment.dimensions.height / (segment.particleSize * 2))
END FUNCTION

FUNCTION identifySegment(body)
    FOR each segment IN teddyBear.segments
        IF segment.bodies.includes(body) THEN
            RETURN segment
        END IF
    END FOR
    RETURN null
END FUNCTION
```

### 9.2 Configuration Management
```
FUNCTION loadConfiguration()
    defaultConfig = {
        physics: {
            gravity: 0.8,
            airFriction: 0.01,
            constraintIterations: 2
        },
        bear: {
            scale: 1.0,
            softness: 0.7,
            density: 0.001
        },
        rendering: {
            wireframes: true,
            showConstraints: false,
            particleColors: {
                head: '#ff6b6b',
                torso: '#4ecdc4',
                arm: '#45b7d1',
                leg: '#96ceb4'
            }
        }
    }
    
    // Load user configuration if available
    userConfig = loadUserConfig()
    IF userConfig THEN
        config = mergeConfigurations(defaultConfig, userConfig)
    ELSE
        config = defaultConfig
    END IF
    
    RETURN config
END FUNCTION
```

## 10. Main Entry Point
```
FUNCTION main()
    TRY
        // Load configuration
        config = loadConfiguration()
        
        // Validate environment
        validateEnvironment()
        
        // Initialize simulation
        simulation = initializeTeddyBearSimulation()
        
        // Set up cleanup on page unload
        window.addEventListener('beforeunload', cleanupSimulation)
        
        console.log("Teddy bear simulation initialized successfully")
        
    CATCH error
        console.error("Failed to initialize teddy bear simulation:", error)
        displayErrorMessage("Failed to load simulation. Please refresh the page.")
    END TRY
END FUNCTION

// Auto-start when DOM is ready
document.addEventListener('DOMContentLoaded', main)
```

## Implementation Notes

### Key Design Decisions:
1. **Modular Architecture**: Each segment is created independently and then connected
2. **Flexible Constraints**: Multiple constraint types provide realistic soft-body behavior
3. **Performance Scaling**: Adaptive quality based on frame rate
4. **Error Resilience**: Comprehensive error handling and validation
5. **Extensible Design**: Easy to add new segments or modify existing ones

### Performance Considerations:
- Particle count balanced between realism and performance
- Collision filtering prevents unnecessary computations
- Constraint iterations optimized for real-time performance
- Memory cleanup prevents leaks during long sessions

### Integration Points:
- Clear separation between physics engine and rendering
- Configurable parameters for easy customization
- Event system for user interaction
- Modular design for easy testing and maintenance

This pseudocode provides a complete roadmap for implementing the teddy bear soft-body physics simulation, with enough detail for systematic development while maintaining flexibility for refinement during implementation.