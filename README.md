# PacmanUnity

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

From here it's a pretty standard Vector calculation to determine direction. The only caveat to that is, that I slightly shrunk the Pacman sprite to 95% of its original size to fit better, so there was the potential the sprite could be misaligned and mess up future moves. Before any potential movements, I round Pacman's position to the nearest 0.25 (based on the grid size) depending on the axis he's moving on. Even with the initial rounding, I found that occasionally the alignment would be incorrect, so a secondary rounding is placed and truncates the position to 2 decimal places. 

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
