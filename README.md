# Teddy Bear Soft-Body Physics Simulation

A interactive physics simulation featuring a teddy bear with soft-body dynamics built using Matter.js.

## Overview

This simulation demonstrates advanced soft-body physics by creating a teddy bear composed of interconnected particles and constraints. The bear responds realistically to gravity, user interaction, and environmental forces while maintaining its characteristic shape.

## Files

- `bear.html` - Main simulation file (self-contained)
- `bear.csv` - Bear shape pattern data (34x24 grid)
- `support/` - Additional documentation and specifications

## Features

- **Soft-body Physics**: Realistic deformation and movement
- **Interactive Controls**: Drag and manipulate the bear
- **Real-time Parameters**: Adjust softness and gravity
- **Performance Monitoring**: FPS and physics timing display
- **Debug Mode**: Detailed system information

## Usage

1. Open `bear.html` in a web browser
2. Use mouse to drag and interact with the bear
3. Adjust controls:
   - **Reset Bear**: Return to initial position
   - **Pause/Resume**: Control simulation
   - **Softness**: Adjust bear flexibility (0-100%)
   - **Gravity**: Control gravitational force (0-200%)
   - **Debug Mode**: Show detailed metrics

## Keyboard Shortcuts

- `R` - Reset bear
- `Space` - Pause/Resume
- `D` - Toggle debug mode

## Technical Details

- Built with Matter.js physics engine
- 816 particles in precise teddy bear shape
- Multi-layer constraint network for stability
- Optimized for 60 FPS performance
- Responsive canvas-based rendering

## Dependencies

- Matter.js (loaded from CDN)
- Modern web browser with Canvas support

## Architecture

The simulation follows SPARC methodology with modular design:
- Core simulation engine
- Performance monitoring system
- Interactive controls
- Debug information display

See `support/` folder for detailed technical specifications.