# 2d Player Movement

```gdscript
extends KinematicBody2D

var velocity = Vector2.ZERO
var speed = 250

func _physics_process(delta):
	var input = Vector2.ZERO
	
	input.x = Input.get_action_strength("walk_right") - Input.get_action_strength("walk_left")
	input.y = Input.get_action_strength("walk_down") - Input.get_action_strength("walk_up")
	input = input.normalized()
	
	if input != Vector2.ZERO:
		velocity = input * speed * delta
	else:
		velocity = Vector2.ZERO

	move_and_collide(velocity)
```


# Godot Notes Site
http://kidscancode.org/godot_recipes/shaders/intro/


# Setting up player animations
Start with a player object that is a Kinematic2d type object
Under that, create a sprite node and a collsion shape 2d node
Sprite node should have a sprite sheet attached that we can split up with the animation area
Then create an animation player node
Select animation player, click animation button and create a new animation
Set the animation length to frames * 0.1
Create animations for each direction for both idle and walk by setting a property track and have that linked to the sprite frame
Should be able to use key button on player sprite to set key frames
Get a reference to the animation player in the player script
Add animation tree to the player object
Assign animation player to the animation tree by clicking assign in the inspector
Set the tree root to a new AnimationNodeStateMachine
Click the active button on animation tree
Add a blend space 2d to the canvas area for the animation tree for idle
Edit blend space 2d
Setup triangles by adding points, top point needs to be the down position, bottom point needs to be the up position (up and down are swapped)
Set blend mode to discrete
Add another blend space 2d for running
Set the blend position of both trees to 0, 0 by going to the inspector for the animation tree
Setup transition between 2 trees that go both directions
Set the idle animation tree to run on load

Below is an example of working code to get this going

```
extends KinematicBody2D

var velocity = Vector2.ZERO
var speed = 200
onready var animation_player = $AnimationPlayer
onready var animation_tree = $AnimationTree
onready var animation_state = animation_tree.get("parameters/playback")


func _physics_process(delta):
	var input = Vector2.ZERO
	
	input.x = Input.get_action_strength("walk_right") - Input.get_action_strength("walk_left")
	input.y = Input.get_action_strength("walk_down") - Input.get_action_strength("walk_up")
	input = input.normalized()
	
	if input != Vector2.ZERO:
		animation_tree.set("parameters/Idle/blend_position", input);
		animation_tree.set("parameters/Walk/blend_position", input);
		animation_state.travel("Walk")
		velocity = input * speed * delta
	else:
		animation_state.travel("Idle")
		velocity = Vector2.ZERO

	move_and_collide(velocity)


```

# Camera
One way to have a camera follow the player is to have a Camera2D as a child of the player.  A problem can happen with this setup if the player ends up being removed from the game, maybe when they die, then the camera gets removed and it gets reset to the default camera.

To fix this, you can add a RemoteTransform2D node to the player, and a Camera2D to the game.  You can point the RemoteTransform2D to point to the camera, which will make the camera follow the player, and it doesn't get destroyed if the player is removed from the scene, as the camera is a child of the game.
