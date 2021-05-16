---
title: "基于Java Swing的飞机大战设计思路"
date: "2018-09-15 17:58:25"
---

## 游戏中的对象

1. 玩家飞机（一架）
2. 敌人飞机（多架）
3. 玩家子弹
4. 敌人子弹
5. 宝物（用于改变玩家子弹类型以及玩家飞机外形）

## 实体类设计

思路：游戏中所有的对象都包括属性：X 轴，Y 轴坐标、长度、宽度以及对应的图片素材，方法：移动（move）、绘图（draw）。因此我们可以设计一个 `GameObject` 抽象类，让其他的实体来继承`GameObject`类。部分代码如下

```java
public abstract class GameObject {
    /**
     * x coordinate
     */
    private int x;
    /**
     * y coordinate
     */
    private int y;

    /**
     * width
     */
    private int width;

    /**
     * height
     */
    private int height;

    /**
     * image
     */
    private BufferedImage image;

    public GameObject() {
    }

    public GameObject(int x, int y, int width, int height) {
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
    }
    // ...get set
    // ...
```

实体层的类层次结构如下图
![](/images/aircraft/entity_hierarchy_2.png)

## 碰撞检测

思路：所有对象都有 X、Y、宽度以及长度，因此我们可以借助 Java 原生的`Rectangle`类的`intersects`方法来完成碰撞检测。具体方法为给`GameObject`添加一个`getRectangle`方法

```java
    public Rectangle getRectangle() {
        return new Rectangle(x, y, width, height);
    }
```

该方法会生成一个坐标为`(x, y)`，宽度值为`width`，高度值为`height`的`Rectangle`类对象。我们可以用以下形式来判定是否发生碰撞

```java
instaneOfGameObject1.getRectangle().intersects(instaneOfGameObject2.getRectangle())
```

若发生碰撞则返回`true`，反之则`false`
关于`Rectangle`类可以查阅文档，获取详细信息。

## 数据的传输

`DTO`(Data Transfer Object)，即数据传输对象。我们设计一个`dto`层来实现各层的数据传输，也就是说`dto`充当了信使的角色，我们在`dto`中存储游戏是否开始、是否暂停、玩家分数、难度相关系数、玩家飞机、敌人飞机、子弹、宝物等游戏相关的数据。

部分代码如下：

```java
public class GameDTO {

    private boolean isStart;

    private boolean isPause;

    private boolean isBoss;

    private int score;

    private Difficulty difficulty;

    private BasePlane playerPlane;

    private BossPlane bossPlane;

    private final List<BaseBullet> playerBullets;

    private final List<BasePlane> enemyPlanes;

    private final List<BaseBullet> enemyBullets;

    private final List<BaseItem> items;

    public GameDTO() {
        isStart = false;
        isPause = false;
        isBoss = false;
        score = 0;
        playerPlane = new PlayerPlane();
        playerBullets = new LinkedList<>();
        enemyPlanes = new LinkedList<>();
        enemyBullets = new LinkedList<>();
        items = new LinkedList<>();
    }
```

由于游戏体量不大，所以我们只设计`GameDTO`一个类就可以了。
出于性能的考虑，我们给`GameDTO`加上单例，使整个程序只有一个`GameDTO`对象，具体实现方式为

```java
    /**
     * single instance dto
     */
    private static GameDTO dto;

    public static GameDTO getDto() {
        if (dto == null)
            dto = new GameDTO();
        return dto;
    }
```

当有对象需要 DTO 对象时，通过`GameDTO.getDto()`就可以获取 dto 对象。

## 游戏中的配置

为了防止出现过多的硬编码，我们需要将硬编码项相关的数据全部写入配置，通过读取配置来获取相关数据。
配置可以有很多形式，比如`XML`、`JSON`、`YAML`等。
这里我们为了方便直接将配置以`static`变量的形式写入到 Java 类中。
部分代码如下

```java
public class GameConfig {

    private GameConfig() {

    }

    /**
     * game name
     */
    private static final String GAME_NAME = "Aircraft";
    /**
     * window width
     */
    private static final int WINDOW_WIDTH = 520;
    /**
     * window height
     */
    private static final int WINDOW_HEIGHT = 700;

    ...
    ...
```

将构造方法私有化是为了使其无法实例化。当需要某个配置数据时使用`GameConfig.getXXX()`的方式获取，如此我们便完成了硬编码的去除。

## 控制器的设计

控制器分为玩家控制器（主要处理玩家的按键操作），游戏控制器和登录控制器（未完成）
游戏控制器主要负责与上层界面的交互，具体为创建界面，处理界面的跳转、随机产生敌人飞机、随机产生宝物、刷新界面、监听用户的操作并将其交给用户控制器处理。

具体实现见代码

## 界面层的设计

1. 游戏对象的绘制

设计图如下

![](/images/aircraft/view_hierarchy_1.png)

其中`Frame`主要是起到容器的作用，实际内容在`Panel`中绘制。

`LoginFrame`为游戏登录界面，`GameFrame`为游戏主体界面，游戏启动时`Panel`设为`LaunchPanel`，选择难度后进入主游戏界面，此时将由`GameController`将`LaunchPanel`变换为`GamePanel`。

`Panel`中绘制界面的方法为`paintComponent(Graphics g)`，我们通过`dto`来获取游戏对象及其相关的数据，由于每个游戏实体对象都有`draw(g)`方法，所以我们只要调用`draw(g)`方法就可以完成游戏对象的绘制。

2. 界面层需要自己处理的逻辑

对于一些背景音乐、按钮、地图的移动，交由界面层处理，而不由控制器和实体方法处理。

## 线程的设计

1. 实体层实体类中的线程设计

游戏中敌人飞机的移动、子弹的移动（包括玩家和敌机的）、宝物的移动、敌机的自动发射子弹，都需要使用线程。

所以我们在`BaseBullet`和`BaseItem`都加入一个线程

```java
private Thread thread;
// get
// ...
// set
// ...
```

由于敌机需要两个线程（移动，射击），所以在`EnemyPlane`添加两个线程

```java
private Thread moveThread;
private Thread shootThread;
```

2. 实体类中线程的启动

子弹的线程由飞机在调用射击`shoot`方法时启动。

敌机的移动线程由游戏控制器在随机生成时启动。

3. 游戏控制器中的线程

游戏控制器由三条线程，处理多按键事件的线程、随机产生宝物的线程、随机产生敌机的线程。

4. 线程锁

 在对实体类对象操作时有可能出现冲突，所以我们需要在`dto`以及其他相关地方中加上关键字`synchronized`，完成线程锁的设定。

`dto`中部分代码如下

```java
    public void addPlayerBullet(BaseBullet bullet) {
        synchronized (playerBullets) {
            playerBullets.add(bullet);
        }
    }

    public void removePlayerBullet(BaseBullet bullet) {
        synchronized (playerBullets) {
            playerBullets.remove(bullet);
        }
    }

    public void addEnemyPlane(BasePlane plane) {
        synchronized (enemyPlanes) {
            enemyPlanes.add(plane);
        }
    }

    public void removeEnemyPlane(BasePlane plane) {
        synchronized (enemyPlanes) {
            enemyPlanes.remove(plane);
        }
    }

    public void addEnemyBullet(BaseBullet bullet) {
        synchronized (enemyBullets) {
            enemyBullets.add(bullet);
        }
    }

    public void removeEnemyBullet(BaseBullet bullet) {
        synchronized (enemyBullets) {
            enemyBullets.remove(bullet);
        }
    }

    public void addItem(BaseItem item) {
        synchronized (items) {
            items.add(item);
        }
    }

    public void removeItem(BaseItem item) {
        synchronized (items) {
            items.remove(item);
        }
    }
```

## 碰撞检测的时机

碰撞检测可以有两种方案：

1. 在控制器中加一条线程，不停的循环检测
2. 在实体对象的每次调用`move`方法时检测

这里，我选择第二种方案，较之于第一种方案更高效。

具体实现为，在每个实体类中添加一个`collisionDetect`方法，下面是玩家子弹的实现，其他类与之类似

```java
    @Override
    public void collisionDetect() {
        synchronized (GameDTO.getDto().getEnemyPlanes()) {
            for (BasePlane enemyPlane : GameDTO.getDto().getEnemyPlanes()) {
                if (!enemyPlane.isDead() && getRectangle().intersects(enemyPlane.getRectangle())) {
                    setHit(true);
                    enemyPlane.setDead(true);
                }
            }
        }
    }
```

## 总结

至此，飞机大战游戏的主要设计就阐述完了。其中还有许多尚未完成的部分：大招、血条、BOSS、散弹、相同子弹的加强。散弹的实现是比较复杂的，因为散弹不同与其他子弹，它不是一颗子弹，而是多颗子弹同时出去，而且有斜向移动。具体的实现方法读者可自行斟酌。

附源码：https://github.com/WhiteVermouth/Aircraft
