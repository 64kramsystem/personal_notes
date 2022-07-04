# hecs

- [hecs](#hecs)
  - [Basic functionality](#basic-functionality)

## Basic functionality

```rs
// hecs = { version = "0.7.7", features = ["macros"] }
// hecs-macros = "0.8.1"

use hecs::*;
use hecs_macros::Bundle;

// Components
struct Position(f32);
struct Velocity(f32);

#[derive(Bundle)]
struct Player {
    pos: Position,
    vel: Velocity,
}

fn main() {
    let mut world = World::new();

    // Spawn entity; also supports tuples as param.
    //
    let eid = world.spawn(Player {
        pos: Position(1.),
        vel: Velocity(2.),
    });

    // Add components to an entity; a single component can't be added via insert()
    // Adding components of an existing type overwrites the existing ones.
    //
    world.insert_one(eid, 1.).unwrap();
    world.insert(eid, (2., "bar")).unwrap();

    // Query (iteration). Other APIs: query_one(), query_one_mut()
    //
    for (_id, (posv, strv)) in world.query_mut::<(&mut Position, &&str)>() {
        println!("P:{:.1} S:{}", posv.0, strv);

        // Changes are applied immediately.
        //
        posv.0 = 3.;
    }

    // Get the component of an entity. Other APIs: get_mut()
    //
    println!("{:.1}", world.get::<Position>(eid).unwrap().0);

    // Despawn entity
    //
    world.despawn(eid).unwrap();
}
```
