
# Experiment 8: Generating nParticle swams and Bubble masses with expressions.
## Introduction: 

Particle systems are powerful tools in Unity that allow you to create complex visual effects by simulating large numbers of small objects (particles). In this tutorial, we'll create three distinct particle effects:

- Particle Swarms (Boids Simulation): Simulates flocking behavior like birds, fish, or insects
- Bubble Masses: Creates floating bubbles with physics-based movement
- Expression-Driven Effects: Uses mathematical expressions to create spiral patterns

Each effect demonstrates different approaches to particle control:

- Swarm: Emergent behavior from simple rules
- Bubbles: Physics simulation with randomness
- Expressions: Direct mathematical control over particle movement


## Part 1: Creating Particle Swarms (Boids Simulation)
Understanding the Boids Algorithm
The boids algorithm simulates flocking behavior using three simple rules:

- Cohesion: Particles move toward the center of nearby particles
- Separation: Particles avoid crowding neighbors
- Alignment: Particles align with the average direction of neighbors

Step 1: Create the Swarm Particle System
- Create a Particle System GameObject 
   - Right-click in the Hierarchy window 
   - Select "Effects" → "Particle System"
   - Rename it to "SwarmParticles"

Step 2: Configure the Particle System

In the Inspector, adjust these settings:
```
Duration: 5
Start Lifetime: 10
Start Speed: 0
Start Size: 0.1
Start Color: A bright color (e.g., yellow)
Emission:
Rate over Time: 100
Shape:
Shape: Sphere
Radius: 1
Renderer:
Material: Create a new material (right-click in Project window → Create → Material)
Set Shader to "Unlit/Color"
Choose a bright color
```
Step 3: Create the Swarm Controller Script

- Create the Script
 - Right-click in the Scripts folder → Create → C# Script
-  Name it "SwarmController"

```
using UnityEngine;

public class SwarmController : MonoBehaviour
{
    [Header("Swarm Settings")]
    public float cohesionRadius = 2f;      // How close particles need to be to affect each other
    public float separationRadius = 0.5f;   // Minimum distance between particles
    public float maxSpeed = 2f;            // Maximum speed particles can move
    public float cohesionStrength = 1f;    // How strongly particles move toward group center
    public float separationStrength = 1.5f; // How strongly particles avoid crowding
    public float alignmentStrength = 1f;   // How strongly particles align with neighbors

    private ParticleSystem.Particle[] particles;
    private ParticleSystem ps;

    void Start()
    {
        // Get the Particle System component
        ps = GetComponent<ParticleSystem>();
        
        // Initialize the particles array
        particles = new ParticleSystem.Particle[ps.main.maxParticles];
    }

    void Update()
    {
        // Get all active particles
        int count = ps.GetParticles(particles);
        
        // Process each particle
        for (int i = 0; i < count; i++)
        {
            Vector3 position = particles[i].position;
            Vector3 velocity = particles[i].velocity;
            
            // Initialize forces
            Vector3 cohesionForce = Vector3.zero;
            Vector3 separationForce = Vector3.zero;
            Vector3 alignmentForce = Vector3.zero;
            int neighborCount = 0;
            
            // Check all other particles
            for (int j = 0; j < count; j++)
            {
                if (i == j) continue; // Skip self
                
                Vector3 neighborPosition = particles[j].position;
                float distance = Vector3.Distance(position, neighborPosition);
                
                // Only consider particles within cohesion radius
                if (distance < cohesionRadius)
                {
                    // Cohesion: move toward center of neighbors
                    cohesionForce += neighborPosition;
                    
                    // Alignment: align with neighbor's velocity
                    alignmentForce += particles[j].velocity;
                    
                    neighborCount++;
                    
                    // Separation: avoid crowding
                    if (distance < separationRadius)
                    {
                        separationForce += (position - neighborPosition) / distance;
                    }
                }
            }
            
            // Calculate average forces
            if (neighborCount > 0)
            {
                cohesionForce = (cohesionForce / neighborCount - position).normalized * cohesionStrength;
                alignmentForce = (alignmentForce / neighborCount - velocity).normalized * alignmentStrength;
            }
            
            separationForce = separationForce.normalized * separationStrength;
            
            // Apply forces to velocity
            velocity += cohesionForce + separationForce + alignmentForce;
            
            // Limit speed
            velocity = Vector3.ClampMagnitude(velocity, maxSpeed);
            
            // Update particle
            particles[i].velocity = velocity;
            particles[i].position += velocity * Time.deltaTime;
        }
        
        // Apply changes to the particle system
        ps.SetParticles(particles, count);
    }
}
```
Attach the Script to the particle.

### Understanding the Code

Start(): Initializes the particle system and array

Update(): Main loop that processes all particles

Nested Loops: The outer loop processes each particle, the inner loop checks neighbors

Force Calculation: Each rule (cohesion, separation, alignment) calculates a force vector

Velocity Update: Combines all forces and applies them to the particle's velocity

Position Update: Moves the particle based on its velocity

## Part 2: Creating Bubble Masses
Bubbles have several key characteristics:

* Buoyancy: They float upward
* Wobble: They drift randomly side to side
* Popping: They disappear randomly after a while

Step 1: Create the Bubble Particle System

  - Create a Particle System GameObject
  - Right-click in the `Hierarchy` → `Effects` → `Particle System`
  - Rename it to "BubbleParticles"

Step 2: Configure the Particle System
In the Inspector:
```
Duration: 5
Start Lifetime: Random between 3 and 8
Start Speed: 0
Start Size: Random between 0.2 and 0.5
Start Color: A light blue or white
Emission:
Rate over Time: 30
Shape:
Shape: Hemisphere
Radius: 1
Renderer:
Material: Create a new bubble material
Shader: "Standard"
Rendering Mode: Transparent
Albedo: Light blue with low alpha
Enable "Fresnel Effects" for a realistic bubble look
```
Step 3: Create the Bubble Controller Script
```
using UnityEngine;

public class BubbleController : MonoBehaviour
{
    [Header("Bubble Settings")]
    public float buoyancy = 1f;          // How fast bubbles rise
    public float wobbleStrength = 0.5f;   // How much bubbles drift side to side
    public float popChance = 0.01f;      // Chance per second that a bubble will pop

    private ParticleSystem.Particle[] particles;
    private ParticleSystem ps;

    void Start()
    {
        ps = GetComponent<ParticleSystem>();
        particles = new ParticleSystem.Particle[ps.main.maxParticles];
    }

    void Update()
    {
        int count = ps.GetParticles(particles);
        
        for (int i = 0; i < count; i++)
        {
            // Get current particle properties
            Vector3 velocity = particles[i].velocity;
            
            // Apply buoyancy (upward force)
            velocity.y += buoyancy * Time.deltaTime;
            
            // Apply wobble (random horizontal movement)
            float noise = Mathf.PerlinNoise(Time.time * 2f, particles[i].randomSeed);
            velocity.x += (noise - 0.5f) * wobbleStrength;
            velocity.z += (noise - 0.5f) * wobbleStrength;
            
            // Random popping
            if (Random.value < popChance * Time.deltaTime)
            {
                particles[i].remainingLifetime = 0;
            }
            else
            {
                // Update particle velocity
                particles[i].velocity = velocity;
            }
        }
        
        ps.SetParticles(particles, count);
    }
}
```
Attach the Script

### Understanding the Code
Buoyancy: Adds upward force to velocity every frame

Wobble: Uses Perlin noise to create smooth, random horizontal movement

Popping: Randomly sets particles' remaining lifetime to 0
RandomSeed: Each particle has a unique random seed for consistent noise patterns

## Part 3: Expression-Driven Effects (Spiral Motion)
Understanding Mathematical Expressions
- Expression-driven effects use mathematical functions to control particle movement directly. This gives us precise control over patterns like spirals, waves, and orbits.

Step 1: Create the Expression-Driven Particle System
  - Create a Particle System GameObject
  - Right-click in the Hierarchy → "Effects" → "Particle System"
  - Rename it to "SpiralParticles"
Configure the Particle System
In the Inspector:
```
Duration: 10
Start Lifetime: 5
Start Speed: 0
Start Size: 0.1
Start Color: A vibrant color (e.g., magenta)
Emission:
Rate over Time: 50
Shape:
Shape: Sphere
Radius: 0.5
Renderer:
Material: Create a new material with an unlit shader
```
Step 2: Create the Expression-Driven Script
  - Create the Script
  - Right-click in the Scripts folder → Create → C# Script
  - Name it "ExpressionDrivenParticles"
```
using UnityEngine;

public class ExpressionDrivenParticles : MonoBehaviour
{
    [Header("Spiral Settings")]
    public float spiralSpeed = 2f;      // How fast the spiral rotates
    public float riseSpeed = 0.5f;      // How fast particles rise
    public float radiusGrowth = 0.2f;   // How fast the spiral expands

    private ParticleSystem.Particle[] particles;
    private ParticleSystem ps;

    void Start()
    {
        ps = GetComponent<ParticleSystem>();
        particles = new ParticleSystem.Particle[ps.main.maxParticles];
    }

    void Update()
    {
        int count = ps.GetParticles(particles);
        float time = Time.time;

        for (int i = 0; i < count; i++)
        {
            // Get unique seed for each particle
            float seed = particles[i].randomSeed;
            
            // Calculate spiral position
            float angle = time * spiralSpeed + seed * Mathf.PI * 2;
            float radius = 0.5f + time * radiusGrowth;
            
            // Apply spiral motion
            particles[i].position = new Vector3(
                Mathf.Cos(angle) * radius,  // X position (circular motion)
                particles[i].position.y + riseSpeed * Time.deltaTime,  // Y position (rising)
                Mathf.Sin(angle) * radius   // Z position (circular motion)
            );
        }
        
        ps.SetParticles(particles, count);
    }
}
```
Attach the Script

Press Play to see the spiral effect
### Customizing Expressions
You can modify the Update() method to create different effects:

```
particles[i].position = new Vector3(
    particles[i].position.x, // Keep original X
    particles[i].position.y + Mathf.Sin(time * 3f + particles[i].randomSeed * 10) * 0.1f, // Sine wave Y
    particles[i].position.z  // Keep original Z
);
```
Orbiting Around a Point
```
Vector3 center = Vector3.zero; // Orbit around origin
float orbitRadius = 2f;
particles[i].position = center + new Vector3(
    Mathf.Cos(time + particles[i].randomSeed * Mathf.PI * 2) * orbitRadius,
    0,
    Mathf.Sin(time + particles[i].randomSeed * Mathf.PI * 2) * orbitRadius
);
```
Random Drift with Noise
```
float noiseScale = 0.5f;
particles[i].position += new Vector3(
    (Mathf.PerlinNoise(time, particles[i].randomSeed) - 0.5f) * noiseScale,
    0,
    (Mathf.PerlinNoise(time + 1000, particles[i].randomSeed) - 0.5f) * noiseScale
);
```

## Output
![Expression-driver, Swarm, Bubble n-particles generated](https://github.com/shubham23bcg10021/Graphics/blob/main/Practical%208/vecg7ss/Screenshot%202025-09-25%20023200.png)
