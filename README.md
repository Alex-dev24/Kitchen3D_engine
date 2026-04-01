<font color="green">===========</font> <font color="lime">Welcome to the Kitchen 3D developer cave!</font> <font color="green">===========</font>

Here is a small summary of the file's contents:
1. Engine information
2. How can the engine be useful?
3. Getting started with the engine
4. Operating documentation
5. Feedback

<font color="lime">1. Engine information</font>

Kitchen 3D is a Python game engine built using the Math and Pygame libraries. The engine was built for use by both beginners and professional prokarmists to create their own projects. One of its advantages is the presence of architecture in the code, simple documentation and uncomplicated operating principles of the engine itself.

Lead developers: Alex Scott and Digital Nightfall

Publication date: --/--/----

This version of Kitchen 3D is designed for Windows 10. If you're using other versions of Windows or Linux, or other operating systems, you may experience crashes, bugs, or problems installing required libraries. The documentation is also relevant for Windows 10.

<font color="red">!!! We kindly request that you do not use images from the "textures" and "sprites" folders, or sounds from the "sound" folder, as they are for auxiliary purposes only and are not copyrighted by the company or consumers.</font>

<font color="lime"> 2. How can the engine be useful? </font>

Many people think that creating games these days is incredibly difficult. Let's be honest, to some extent this is true, but first and foremost, creating your own game is a creative process that shouldn't be influenced by external development factors. That's why we're working on a product for everyone—one that's easy to use and understand. This engine can be used as a standalone resource for creating your own games, as well as a learning tool for understanding the principles of 3D graphics and how to create them.

<font color="lime"> 3. Getting started with the engine  </font>

Getting started with the engine is quite easy! All you need to do is download Pygame. To do this, open a terminal and enter the following: 

    pip install pygame

If you encounter a problem with pip not being updated, open cmd using the Win+R keyboard shortcut and then enter the following:

    python -m pip install --upgrade pip

Or if you have multiple versions of Python:

    py -m pip install --upgrade pip 

There may also be a problem with your Python version. To resolve this, visit the Microsoft Store and download the latest version.

At this point the installation is complete and we can move on to examining the functionality and operating methods.

<font color="lime">4. Operating documentation </font>

Now we move on to the biggest and most complex section - "Operating documentation". Here we will learn how to work with the engine and how to activate its functions.

<font color="yellow">===========</font> <font color="lightyellow">Basics of 3D creation</font> <font color="yellow">===========</font>

Let's start by introducing simple concepts visually. This will allow us to understand how 3D is created from 2D. And no, this is not a mistake; there is no such thing as 3D here; everything we see is just a mathematical illusion. 

So the first thing you need to know is how to find the distance between a player and something on the map. You can see it in this picture:

![Distance](Documentation_resources\distance.png)

This is the basic knowledge that will be used everywhere from raycasting to sprite rendering. The operations on these objects themselves are described in the code. Here are some examples of using formulas

    xhor = cur_x + depth_horizontal * cos_a
    yvert = cur_y + depth_vertical * sin_a

You can find it in the "raycasting" file.

 Or:

    dx, dy = sprite_x - player.x, sprite_y - player.y

You can find it in the "sprites" file.

Next, let's look at the formulas we use to calculate a player's movement.

![Movement](Documentation_resources\movement.png)

And of course, how it is used:

    keys = pg.key.get_pressed()
        
    #calculate movement direction based on viewing angle
    sin_a = math.sin(self.angle)
    cos_a = math.cos(self.angle)
    
    new_x, new_y = self.x, self.y
    
    #calculate movement speed (frame rate independent)
    speed = player_speed * delta_time
    player_cos = speed * cos_a
    player_sin = speed * sin_a
    
    #WASD movement
    if keys[pg.K_w]:
        new_x += player_cos
        new_y += player_sin
    if keys[pg.K_s]:
        new_x -= player_cos
        new_y -= player_sin
    if keys[pg.K_a]:
        new_x += player_sin
        new_y -= player_cos
    if keys[pg.K_d]:
        new_x -= player_sin
        new_y += player_cos

You can find it in the "player" file.

Now let's look at how the player will see:

![FOV](Documentation_resources\FOV.png)

This is necessary because to find the collision of rays (the player's field of view) with walls we will use raycasting technology or simply emitting rays in the player's view for further rendering.

We can see all the specified variables in the "settings" file:

    #================== RAY CASTING ===================
    FOV = math.pi/2
    HALF_FOV = FOV/2
    NUM_RAYS = WIDTH//4
    MAX_DEPTH = 1000
    DELTA_ANGLE=FOV/NUM_RAYS
    DIST = (WIDTH/2) / math.tan(HALF_FOV)  
    PROJECTION_COEFFICIENT = DIST * TILE
    SCALE = WIDTH // NUM_RAYS
    RENDER = 15
    #==================================================

We need to understand where these variables come from, as we'll be working with projection later. A projection of the actual wall is drawn on our screen, creating a 3D effect.

Here's a top view: 

![Proj_view_1](Documentation_resources\projection_view_1.png)

========================

<font color="lightblue">d - DIST

DEPTH - depth

H - TILE

h - projection_height

d * H - PROJECTION_COEFFICIENT</font>

========================

Here's a side view:

![Proj_view_2](Documentation_resources\projection_view_2.png)

For instance:

    projection_height = min(int(PROJECTION_COEFFICIENT / depth), 2* HEIGHT)

You can find it in the "raycasting" file.

And now the technology for finding a wall to draw. Of course, we could simply extend the ray until it hits a wall, but that would require a lot of computing power. For optimization, we'll calculate the intersections of horizontal and vertical lines on an "imaginary grid".

Verticals:

![Verticals](Documentation_resources\verticals.png)

Horizontals:

![Horizontals](Documentation_resources\horizontals.png)

How to use it in code:

Verticals:

    #============ CHECKING VERTICAL LINES ============
    depth_vertical = float('inf')
    texture_vertical = "1"
    x = xm + TILE if cos_a >= 0 else xm
    dx = 1 if cos_a >= 0 else -1

    #check cycle
    for i in range(RENDER):
        depth_vertical = (x - cur_x) / cos_a
        yvert = cur_y + depth_vertical * sin_a
        tile_vertical = cell_locator(x + dx, yvert)
        if tile_vertical in world_map:
            texture_vertical = world_map[tile_vertical]
            break
        x += dx * TILE
    #==================================================

Horizontals:

    #=========== CHECKING HORIZONTAL LINES ============
    depth_horizontal = float('inf')
    texture_horizontal = "1"
    y = ym + TILE if sin_a >= 0 else ym
    dy = 1 if sin_a >= 0 else -1
    
    #check cycle
    for i in range(RENDER):
        depth_horizontal = (y - cur_y) / sin_a
        xhor = cur_x + depth_horizontal * cos_a
        tile_horizontal = cell_locator(xhor, y + dy)
        if tile_horizontal in world_map:
            texture_horizontal = world_map[tile_horizontal]
            break
        y += dy * TILE
    #==================================================

Selecting the nearest wall:

    #=========== SELECTING THE NEAREST WALL ===========
    if depth_vertical < depth_horizontal:
        depth, texture_offset, texture = depth_vertical, yvert, texture_vertical

        #points of impact of the ray on the wall
        hit_x = cur_x + depth * cos_a 
        hit_y = cur_y + depth * sin_a
    else:
        depth, texture_offset, texture = depth_horizontal, xhor, texture_horizontal
        hit_x = cur_x + depth * cos_a  
        hit_y = cur_y + depth * sin_a
    #==================================================

You can find it in the "raycasting" file.

This is enough to understand the concept of the engine. Now let's move on to its functionality.

<font color="yellow">===============</font> <font color="lightyellow">Functional</font> <font color="yellow">==============</font>

And the time has come to show what the engine is capable of and what can and should be changed to make it more convenient. We have commented some parts of the engine specifically so that they either do not conflict with each other or you can use them if necessary.

It's worth starting with the main file. This is the most important file, where the engine's core functions are called and processed in its main cycle. To run the code, you need to run it, having first saved all changes in the remaining files.

In it, you can see the commented functions for rendering the title, fps, and position in z; at the moment, they are disabled because you are using full-screen launch mode.


Here they all are: 

    # icon = pg.image.load("Kitchen 3D\icon\kitchen.png") 
    # pg.display.set_icon(icon)
    # pg.display.set_caption(f'Kitchen 3D V{1} | FPS: {clock.get_fps() :.1f} | Z: {player.z:.1f}')

And of course the most noticeable block:

    #============ COLLISION WITH A SPRITE =============
    # if event.type == pg.MOUSEBUTTONDOWN and event.button == 1:
    #     mx = pg.mouse.get_pos()[0]
    #     #find the nearest sprite 
    #     collide_sprites = [sprite for sprite in drawing.sprites if sprite.collide(player)]
    #     if collide_sprites:
    #      nearest = min(collide_sprites, key=lambda s: s.get_distance(player))
    #      drawing.sprites.remove(nearest)
    #==================================================

This function is used to check for collisions of the ray when the mouse is pressed with sprites, in which case they disappear, but you can change the last line and use it for your own purposes. You can find its beginning in the file "sprites" in line 69

Let's move on to the "map" file. Here you can see the map added to the list. As you can see, there are horizontal and vertical columns x and y respectively. This is necessary for setting coordinates for sprites and the player.

In "settings":

    #================ PLAYER SETTINGS =================
    cell_x = 4 #hc  <==
    cell_y = 2 #vc  <==
    player_position = (cell_x * TILE + TILE//2, cell_y * TILE + TILE//2) <==
    player_angle = 0 
    player_speed = 0.5
    #==================================================

In "drawing":

    self.sprites = [
        Sprite('Kitchen 3D\\sprites\\KITCHEN_SPRITE_BONE.png', (15, 12)),
        Sprite('Kitchen 3D\\sprites\\KITCHEN_SPRITE_BONE.png', (5, 5)),
        Sprite('Kitchen 3D\\sprites\\KITCHEN_SPRITE_BONE.png', (10, 9)),
        Sprite('Kitchen 3D\\sprites\\KITCHEN_SPRITE_BONE.png', (11, 7)),
    ]

You can also see that the map consists of _ and numbers - this is the void and walls with a texture number or a special sign, for example, D is a door and L is a light source.

Speaking of textures, you can change them in the "drawing" file. First, put them in the "sprites" or "textures" folder, specify the approximate path, and don't forget to replace \ with \\ to ensure the images are read correctly.

And feel free to add new blocks and sprites! For the former, add a new type designation to the self.textures set in the "drawing" file:

    self.textures = 
        { "n": pg.image.load("Kitchen 3D\\textures\\KITCHEN_WALL_BRICK.png").convert_alpha()
        }


Then, in the "map" file, inside the check for "if char != '_':", insert the following:

    if char =="1":
        world_map[(i*TILE,j * TILE)] = "n"

Where n is the texture number or name within the set.

As for sprites, just add line like this to the sprite list:
    
    Sprite('Kitchen 3D\\sprites\\KITCHEN_SPRITE_BONE.png', (15, 12))



Text




    If distance to wall > LIGHT_DISTANCE:
    brightness = AMBIENT

    else, if there are light sources:
    d = min(distance to sources)

    if d < LIGHT_RADIUS:
        t = d / LIGHT_RADIUS
        brightness = AMBIENT + LIGHT_POWER * (1 - t^FALLOFF)
    else:
        brightness = AMBIENT

    else:
    brightness = AMBIENT

    Result = max(50, min(255, brightness))
