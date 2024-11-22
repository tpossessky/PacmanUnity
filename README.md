# PacmanUnity Devlog

## Overview 

Yet another fairly large step up in complexity. Difficult, pixel-perfect movement behavior and AI were the two defining challenges here.

## Collision/Movement
### Collision
Games 1-4 all featured fairly open layouts where simple box colliders did the trick. With Pacman, I dealt with a tilemap in 8x8pixel grids and character sprites in 16x16 grids. This meant the gaps in the maze Pacman could move in were nearly pixel-perfect. My first approach was to try a `BoxCollider2D` and `TileMapCollider` to handle everything. I ran into issues immediately with the movements not being pixel-perfect and getting stuck in the maze.

My solution was to use a series of `RayCast2D` emitted by Pacman on all sides instead. 

![image](https://github.com/user-attachments/assets/4118b834-4bec-4118-a6ef-b31e05c85f24)


With 5 rays per side, I can accurately tell when Pacman can move in a given direction. Initially, I tried 4 rays representing the corners of the Sprite, but as you can see in the image I ran into edge cases where only a thin strip of wall blocks movement. Each frame, these rays are updated and a `Movements` object (with booleans representing available movement directions) is given to the `PlayerMovement` script. 

I thought of optimizing this to only run every other frame, but with the window for a valid move being so small, I ran into the issue of a majority of potential moves being skipped.

### Movement

With the collision detection being so precise, expecting the player to press the WASD keys at exactly the right frame was not feasible. Instead, I implemented a movement buffer in the form of a Queue and timer. When a player inputs a move, it gets added to the queue along with the current time.   

```
    public void HandlePlayerMovement(InputAction.CallbackContext context)
    {
        if (context.performed)
        {
            inputQueue.Enqueue(context.ReadValue<Vector2>());
            lastInputTime = Time.time; 
        }
    }
```

From here it's a pretty standard Vector calculation to determine direction. The only caveat is that I slightly shrunk the Pacman sprite to 95% of its original size to fit better which added the potential the sprite could be misaligned and mess up future moves. Before any potential movements, I round Pacman's position to the nearest 0.25 (based on the grid size) depending on the axis he's moving on. Even with the initial rounding, I found that occasionally the alignment would be incorrect, so a secondary rounding is placed and truncates the position to 2 decimal places. 

```
    private void AlignPositionOnAxis(bool alignX)
    {
        var position = playerRigidBody.position;
        if (alignX)
        {
            position.x = Mathf.Round(position.x * 4) / 4f; // Adjust to grid (e.g., 1/4 unit)
            position.x = (float) Math.Round(position.x,2);

        }
        else
        {
            position.y = Mathf.Round(position.y * 4) / 4f; // Adjust to grid (e.g., 1/4 unit)
            position.y = (float) Math.Round(position.y,2);
        }
        playerRigidBody.position = position;
    }
```
## AI Behavior

When deciding on how to handle the AI, I had a choice between a fully custom solution and using Unity's built-in Navmesh systems. I decided to go the fully custom route to understand AI at a baseline level better.

In the original Pacman game, the AI's behavior was very advanced for the time and I wanted to implement it similarly to how it was originally done. I read a great article (https://gameinternals.com/understanding-pac-man-ghost-behavior) on how the AIs behave and started by implementing a simple movement script. 

The movement logic I implemented only makes the AIs make a decision when they reach a predefined point on the map. Since the game operates on a linear grid, the AIs then use a simplified version of the `RayCast2D` check Pacman does to determine their available movements once they reach a decision point.

Green - Decision Point

Purple - Corner 

![image](https://github.com/user-attachments/assets/8bbf4636-f8d0-416d-a470-1115ccc29b8c)

The AIs inherit from the base `BaseAIMovement` script. This script is responsible for actually moving the AIs around and the individual ghosts can request their preference of movement in a `Queue<Vector2>`.

### Red Ghost

The Red Ghost's job is to go directly towards the player. This is fairly simple behavior where all that's needed is to calculate which direction to move in based on relative x and y distances, and then add them to the queue. 

### Pink Ghost

The Pink Ghost chooses a location 4 units in front of the player based on their current velocity

### Orange Ghost

The Orange Ghost switches its behavior based on its linear distance to the player. Above 8 units, it acts the same as red but when it gets too close it defaults back to its Scatter location in a corner. 

### Blue Ghost

The Blue Ghost is the most complex of the 4. It calculates a position 2 units in front of the player then draws a vector from Red's position to that target and doubles it. This gives the final target.

## AI Sample Video 

The below video is from an early debug build where I was testing the AI targeting. 

Green lines - "Chase mode" 

Red lines - "Scatter mode" 

![Pacman-SampleScene-WindowsMacLinux-Unity66000 0 27f1_DX11_2024-11-2121-39-37-ezgif com-optimize](https://github.com/user-attachments/assets/1c1cc2f2-6881-4708-a762-2d1100e4f547)
