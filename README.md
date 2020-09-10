# 3D Mouse Picking for Bevy

This is a 3D mouse picking plugin for [Bevy](https://github.com/bevyengine/bevy). The plugin will cast a ray into the scene and check for intersection against all meshes tagged with the `PickableMesh` component. The built-in highlighting and selection state management features are opt-in.

**Expect Breaking Changes - Issues/PRs Welcome**

![Picking demo](docs/demo.webp)

## Getting Started

To run the `3d_scene` example - a modified version of the `bevy` example of the same name - clone this repository and run:
```console
cargo run --example 3d_scene
```

## Usage

### Setup

Add the repo to your dependencies in Cargo.toml

```toml
bevy_mod_picking = { git = "https://github.com/aevyrie/bevy_mod_picking", branch = "master" }
```

Import the plugin:

```rust
use bevy_mod_picking::*;
```

Add it to your App::build() in the plugins section of your Bevy app:

```rust
.add_plugin(PickingPlugin)
```

### Marking Entities for Picking

Make sure you have your camera's entity on hand, you could do the following in your setup system:

```rust
let camera_entity = Entity::new();

// ...

.spawn_as_entity(camera_entity, Camera3dComponents {

// ...
```

Now all you have to do is mark any mesh entities with the `PickableMesh` component:

```rust
.with(PickableMesh::new(camera_entity))
```

If you want it to highlight when you hover, add the `HighlightablePickMesh` component:

```rust
.with(HighlightablePickMesh::new())
```

If you also want to select meshes and keep them highlighted with the left mouse button, add the `SelectablePickMesh` component:

```rust
.with(SelectablePickMesh::new())
```

### Getting Pick Data

#### Pick Intersections Under the Cursor

Mesh picking intersection are reported in [NDC](http://www.songho.ca/opengl/gl_projectionmatrix.html) and World Coordinates. You can use the `PickState` resource to either get the topmost entity, or a list of all entities sorted by distance (near -> far) under the cursor:

```rust
fn get_picks(
    pick_state: Res<PickState>,
) {
    println!("All entities:\n{:?}", pick_state.list());
    println!("Top entity:\n{:?}", pick_state.top());
}
```

Alternatively, you can create a query to iterate over all `PickableMesh`s and get the entity's pick coordinates with `get_pick_coord_ndc()`.

#### World coordinates

You can get the pick world coordinates with the `get_pick_coord_world()` function in the `PickIntersection` returned from `pick_state.top()`. You will have to pass in your camera's `projection_matrix` and `view_matrix`:

```rust
fn get_world_coords(
    pick_state: ResMut<PickState>,
    mut query: Query<(&DebugCursor, &mut Translation)>,
    mut camera_query: Query<(&Transform, &Camera)>,
) {
    // Get the camera
    let mut view_matrix = Mat4::zero();
    let mut projection_matrix = Mat4::zero();
    for (transform, camera) in &mut camera_query.iter() {
        view_matrix = transform.value.inverse();
        projection_matrix = camera.projection_matrix;
    }

    // Get the top pick's world position
    if let Some(top_pick) = pick_state.top() {
        let world_position: Vec3 = top_pick.get_pick_coord_world(projection_matrix, view_matrix);

        // Do something with world_pos...
    }
}
```

#### Selection State

If you're using the `SelectablePickMesh` component for selection, you can access the selection state by querying all selectable entities and accessing the `.selected()` function.

### Plugin Parameters

If you're using the built in `HighlightablePickMash` component for highlighting, you can change the colors by accessing the `PickHighlightParams` and setting the colors:

```rust
fn set_highlight_params(
    mut highlight_params: ResMut<PickHighlightParams>,
) {
    highlight_params.set_hover_color(Color::rgb(1.0, 0.0, 0.0));
    highlight_params.set_selection_color(Color::rgb(1.0, 0.0, 1.0));
}
```
