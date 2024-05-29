<!--yml
category: 未分类
date: 2024-05-27 14:59:43
-->

# Creating a Snake Game Clone with Rust/Bevy | Brad Barrows' Blog

> 来源：[https://bbarrows.com/posts/bevy-snake](https://bbarrows.com/posts/bevy-snake)

# Rust Practice with Bevy

Bevy has an amazing dependency injection system that makes its "ECS" (Entity Component System) architecture very easy and ituitive to use. Really the DI system is one of the most amazing Rust feats that I have seen so far.

Someone has taken the time to document how to create your own [Bevy based DI system here which is a great read](https://promethia-27.github.io/dependency_injection_like_bevy_from_scratch/chapter1/system.html)

There is also an unofficial Bevy book that is really helpful [here](https://bevy-cheatbook.github.io/programming/intro-data.html)

## Bevy and ECS

ECS is documented [here](https://bevyengine.org/learn/quick-start/getting-started/ecs/) It describes some of the basic concepts of how the Bevy game engine works. Combined with a powerful dependency injection (DI) system, Bevy is powerful, easy to work with, and fun to use.

For my snake game I only needed the concept of three entities really: the snake, the food, and the snake's body.

This is done by the C part of ECS. I create a Component for each. This allows me to, later on, using the DI/query system executed on each game loop, to find all the entities of a certain type and do something with them.

On setup, I create the snake and the food using a spawn command. It takes a tuple where I can provide multiple Components that make up whatever entity I am creating.

Each of the described objects above have a visual aspect for example, so each have a 2D mesh (which includes a transform/translation aka location).

Here is where I create the SnakeHead for example:

```
 let head_mesh = Mesh2dHandle(meshes.add(Rectangle::new(OBJECT_SIZE, OBJECT_SIZE)));
    let box_color = Color::rgb(0.8, 0.2, 0.1);
    commands.spawn((
        MaterialMesh2dBundle {
            mesh: head_mesh,
            material: materials.add(box_color),
            transform: Transform::from_translation(Vec3::new(0.0, 0.0, 0.0)),
            ..default()
        },
        SnakeHead {
            direction: Direction::Up,
        },
    )); 
```

The most amazing part of Bevy though that I have found so far is its "query"/DI system. With types and traits, I can query for all the entities of a certain type and do something with them each step of the game loop.

This is the function signature I have for checking for collisions for ex:

```
fn check_collisions(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<ColorMaterial>>,
    mut apple_query: Query<(&mut Transform), (With<Apple>, Without<SnakeHead>, Without<SnakeBody>)>,
    mut snake_head_query: Query<(&mut Transform, &mut SnakeHead)>,
    mut snake_body_query: Query<(&mut Transform, Entity), (With<SnakeBody>, Without<SnakeHead>)>, 
```

This function signature can be however long.

The DI system will provide me with whatever objects I need. Commands, meshes, materials, etc.. As well as decipher a simple to work with type system to provide me with all the entities that match a specific query.

For example, with the snake body. I want to be able to move each one and also remove them if the snake crashes. To move them I just need their Transform (part of what was provided from the MaterialMesh2dBundle during setup) and the Entity itself so I can send a command to despawn them or remove body parts from the game.

The DI system that powers all this is amazing to me. I had no idea this could be accomplished in Rust until someone from the Bevy Discord chat kindly pointed me to this documentation which breaks down [how it all works](https://promethia-27.github.io/dependency_injection_like_bevy_from_scratch/chapter1/system.html).

## Reading and Learning the Complicated Macros from the DI System Mentioned Above

If you take the example from the end of page 3 in the DI like Bevy from Scratch book, you can use cargo expand to see what the macro is doing.

I created a new project and added the macro to the main.rs file. Then I ran cargo expand and it showed me the expanded macro created unsugared Rust code which made understanding the macros and the documentation much easier to understand.

To install cargo expand run: `cargo install cargo-expand`

The code I ran `cargo expand > expanded.rs` on is:

```
struct FunctionSystem<Input, F> {
    f: F,
    marker: PhantomData<fn() -> Input>,
}

trait System {
    fn run(&mut self, resources: &mut HashMap<TypeId, Box<dyn Any>>);
}

macro_rules! impl_system {
    (
        $(
            $($params:ident),+
        )?
    ) => {
        #[allow(non_snake_case, unused)]
        impl<
            F: FnMut(
                $( $($params),+ )?
            )
            $(, $($params: 'static),+ )?
        > System for FunctionSystem<($( $($params,)+ )?), F> {
            fn run(&mut self, resources: &mut HashMap<TypeId, Box<dyn Any>>) {
                $($(
                    let $params = *resources.remove(&TypeId::of::<$params>()).unwrap().downcast::<$params>().unwrap();
                )+)?

                (self.f)(
                    $($($params),+)?
                );
            }
        }
    }
}

impl_system!();
impl_system!(T1);
impl_system!(T1, T2);
impl_system!(T1, T2, T3);
impl_system!(T1, T2, T3, T4);

trait IntoSystem<Input> {
    type System: System;

    fn into_system(self) -> Self::System;
}

macro_rules! impl_into_system {
    (
        $($(
                $params:ident
        ),+)?
    ) => {
        impl<F: FnMut($($($params),+)?) $(, $($params: 'static),+ )?> IntoSystem<( $($($params,)+)? )> for F {
            type System = FunctionSystem<( $($($params,)+)? ), Self>;

            fn into_system(self) -> Self::System {
                FunctionSystem {
                    f: self,
                    marker: Default::default(),
                }
            }
        }
    }
}

impl_into_system!();
impl_into_system!(T1);
impl_into_system!(T1, T2);
impl_into_system!(T1, T2, T3);
impl_into_system!(T1, T2, T3, T4);

type StoredSystem = Box<dyn System>;

struct Scheduler {
    systems: Vec<StoredSystem>,
    resources: HashMap<TypeId, Box<dyn Any>>,
}

impl Scheduler {
    pub fn run(&mut self) {
        for system in self.systems.iter_mut() {
            system.run(&mut self.resources);
        }
    }

    pub fn add_system<I, S: System + 'static>(&mut self, system: impl IntoSystem<I, System = S>) {
        self.systems.push(Box::new(system.into_system()));
    }

    pub fn add_resource<R: 'static>(&mut self, res: R) {
        self.resources.insert(TypeId::of::<R>(), Box::new(res));
    }
}

fn main() {
    let mut scheduler = Scheduler {
        systems: vec![],
        resources: HashMap::default(),
    };

    scheduler.add_system(foo);
    scheduler.add_resource(12i32);

    scheduler.run();
}

fn foo(int: i32) {
    println!("int! {int}");
} 
```

# My Snake Clone

Can be found [here](https://github.com/bebrws/bevy-snake)

# And Compiled to WASM and Played Here! (Only on WASM supported browsers - so no mobile)

<canvas id="bevy-portal" tabindex="0" data-raw-handle="1" alt="app" cursor="auto"></canvas>