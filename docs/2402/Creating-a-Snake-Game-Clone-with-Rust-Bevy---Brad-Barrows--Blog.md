<!--yml

category: 未分类

date: 2024-05-27 14:59:43

-->

# 使用 Rust/Bevy 创建蛇游戏克隆 | Brad Barrows' Blog

> 来源：[https://bbarrows.com/posts/bevy-snake](https://bbarrows.com/posts/bevy-snake)

# Rust 实践与 Bevy

Bevy 拥有一个了不起的依赖注入系统，使其“ECS”（实体组件系统）架构非常易于使用和直观。实际上，DI 系统是迄今为止我见过的最令人惊奇的 Rust 功绩之一。

有人花时间记录了如何在[这里基于 Bevy 创建你自己的 DI 系统，这是一篇很棒的阅读](https://promethia-27.github.io/dependency_injection_like_bevy_from_scratch/chapter1/system.html)

也有一本非官方的Bevy书籍对我来说真的很有帮助[这里](https://bevy-cheatbook.github.io/programming/intro-data.html)

## Bevy 和 ECS

ECS 在[这里有文档](https://bevyengine.org/learn/quick-start/getting-started/ecs/)它描述了 Bevy 游戏引擎的一些基本概念。结合强大的依赖注入（DI）系统，Bevy 强大、易于使用且有趣。

对于我的蛇游戏，我实际上只需要三种实体的概念：蛇、食物和蛇的身体。

这由 ECS 的 C 部分完成。我为每个创建了一个组件。这使我能够稍后使用 DI/query 系统在每个游戏循环中执行，查找某一类型的所有实体并对其进行操作。

在设置期间，我使用 spawn 命令创建蛇和食物。它接受一个元组，我可以在其中提供构成我要创建的任何实体的多个组件。

上述描述的每个对象都有一个视觉方面的例子，因此每个对象都有一个 2D 网格（其中包括一个变换/转换，即位置）。

例如，这里是我创建 SnakeHead 的地方：

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

到目前为止，我发现 Bevy 最惊人的部分是它的“查询”/DI 系统。通过类型和特性，我可以查询特定类型的所有实体，并在游戏循环的每个步骤中对它们执行操作。

这是我用于检查碰撞的函数签名的例子：

```
fn check_collisions(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<ColorMaterial>>,
    mut apple_query: Query<(&mut Transform), (With<Apple>, Without<SnakeHead>, Without<SnakeBody>)>,
    mut snake_head_query: Query<(&mut Transform, &mut SnakeHead)>,
    mut snake_body_query: Query<(&mut Transform, Entity), (With<SnakeBody>, Without<SnakeHead>)>, 
```

这个函数签名可以有多长。

DI 系统将为我提供所需的所有对象。命令、网格、材料等。以及解密一个易于使用的类型系统，以提供符合特定查询的所有实体。

例如，对于蛇身体部分。我希望能够移动每个部分，并且如果蛇碰撞则移除它们。要移动它们，我只需要它们的 Transform（在设置期间从 MaterialMesh2dBundle 中提供的一部分）和实体本身，以便我可以发送一个命令来销毁它们或从游戏中移除身体部分。

这一切的 DI 系统对我来说真的很神奇。直到有人从 Bevy Discord 聊天中友好地指引我查看[这篇文档](https://promethia-27.github.io/dependency_injection_like_bevy_from_scratch/chapter1/system.html)，我才意识到在 Rust 中也可以实现这一切。

## 阅读和学习来自上述 DI 系统的复杂宏

如果你从《从零开始学 Bevy》书籍第3页末尾的例子中获取，你可以使用 cargo expand 查看宏的运行情况。

我创建了一个新项目，并将宏添加到 main.rs 文件中。然后我运行了 cargo expand，它显示了扩展的宏生成的未糖化 Rust 代码，这使得理解宏和文档变得更加容易理解。

要安装 cargo expand，请运行：`cargo install cargo-expand`

我运行的代码 `cargo expand > expanded.rs` 是：

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

# 我的 Snake Clone

可在 [此处](https://github.com/bebrws/bevy-snake) 找到

# 并且编译为 WASM 并在此处播放！（仅限支持 WASM 的浏览器 - 因此不支持移动设备）

<canvas id="bevy-portal" tabindex="0" data-raw-handle="1" alt="app" cursor="auto"></canvas>
