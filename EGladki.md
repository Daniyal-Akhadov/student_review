Проект: https://github.com/EGladki/Simulation/tree/master

[EGladki]

# Плюсы

1. Игра работает!
2. Проект имеет аж 4 режима. Каждый уникален
3. Все режимы работают.
4. Иконки травоядных отображаются разными цветом, в зависимости от ситуации на поле.

# Минусы

## Вне кода
1. Нет возможности, в самом начале выбора режимов, выйти из меню и закрыть приложение.
2. Карта не является прямоугольной в силу тайлов. Стоило подобрать такие, чтобы всё было выравнено. 
![image](https://github.com/user-attachments/assets/86052766-0df7-46f3-8157-cbdcb089dd8e)
3. Ты запушил папку .idea в репозиторий. Эта папка генерируется каждый раз при запуске проекта у каждого из пользователей твоей репы. Также она немало весит, потому это избыточная папка, которую стоит добавить в .gitignore.
![image](https://github.com/user-attachments/assets/56785b6e-82e5-4c19-8eea-4138bf56601a)

## По коду

### Main

1. Лишний отступ внизу. Перед окончанием класса не должно быть вертикальных отступов. Глава 5 "Чистый код", привожу подробное объяснение, когда вертикальные отступы нужны: Каждая пустая строка становится зрительной подсказкой, указывающей на начало новой самостоятельной концепции.

На текущий момент:
```java
public class Main {
    public static void main(String[] args) {
        Simulation simulation = new Simulation();
        simulation.start();
    }

}
```

Как должно быть: 
```java
public class Main {
    public static void main(String[] args) {
        Simulation simulation = new Simulation();
        simulation.start();
    }
}
```

### Simulation

1. Класс не совсем правильно назван. Simulation --- это прежде всего название нашего проекта. Данный класс отвечает за gameloop. Так что корректней было бы назвать его GameLoop (игровой цикл).
2. Мы можем обратить внимание на то, что класс занимается не только игровым процессом, но и созданием сущностей внутри себя, речь об этом
```java
public class Simulation {

    private final Scanner scanner = new Scanner(System.in);
    private WorldMap worldMap;
    private final WorldMapFactory worldMapFactory = new WorldMapFactory();
    private final Menu menu = new Menu();
    private final Actions moveAction = new MoveAction();
    private final Actions renderAction = new RenderAction();
    private final Actions bornAction = new BornAction();
    private volatile boolean running = true;
    private volatile boolean pause = false;
```
Ты можешь видеть здесь, что очень большое количество созданий различных объектов. Это антипаттерн. Назвается:  «Control Freak». Руководитель-наркоман. Прочесть: https://habr.com/ru/articles/166287/. Как справиться? Все зависимости получать в конструкторе класса.

3. Слишком много полей в этом классе. Я бы подумал о том как уменьшить их количество. Почему большое и почему стоит убрать? Дело в том, что чем больше полей в нашем классе, тем больше вероятность того, что у него много различных ответственностей, что нарушает `SRP принцип SOLID`. Также это трудно читать. А также, в дальнейшем, когда познакомишься с тестированием, поймёшь, что тут очень много зависимостей, которые нужно будет мокать, дабы проверить работоспособность этого класса. Это дело будет явно нелёгким

4. switch-case стоит вынести в отдельный метод для лучше читаемости:
```java
    public void start() {
        int mode = menu.start();
        switch (mode) {
            case DEFAULT_SETTINGS -> startDefaultSimulation();
            case CUSTOM_SETTINGS -> startCustomSimulation();
            case ENDLESS_SIMULATION -> startEndlessSimulation();
            case CUSTOM_ENDLESS_SIMULATION -> startCustomEndlessSimulation();
        }
    }
```

Правильно:

```java
    public void start() {
        int mode = menu.start();
        chooseMode(mode);
    }

    private void chooseMode(int mode) {
        switch (mode) {
            case DEFAULT_SETTINGS -> startDefaultSimulation();
            case CUSTOM_SETTINGS -> startCustomSimulation();
            case ENDLESS_SIMULATION -> startEndlessSimulation();
            case CUSTOM_ENDLESS_SIMULATION -> startCustomEndlessSimulation();
        }
    }
```

5. Вместо Settings лучше назвать Mode: DEFAULT_MODE.
CUSTOM_SETTINGS --- CUSTOM_MODE. И т.д.

6. Стоит погуглить на тему паттерна Command и узнать как уйти от switch - case. Прочти главу 3 от Дяди Боба "Чистый код" 

```java
    private void chooseMode(int mode) {
        switch (mode) {
            case DEFAULT_SETTINGS -> startDefaultSimulation();
            case CUSTOM_SETTINGS -> startCustomSimulation();
            case ENDLESS_SIMULATION -> startEndlessSimulation();
            case CUSTOM_ENDLESS_SIMULATION -> startCustomEndlessSimulation();
        }
    }
```

Вот как он мог бы сократиться: 

```java
        if (chooseAction(actionId)) {
            action.execute();
        }
```
7. Всегда ставь отступы между блоками if-else, switch-case, try-catch, while, do-while:

```java
    private boolean containsHerbivoresAndPredators(WorldMap worldMap) {
        boolean hasHerbivore = false;
        boolean hasPredator = false;
        for (Entity entity : worldMap.getAll()) {
            if (entity instanceof Herbivore) {
                hasHerbivore = true;
            } else if (entity instanceof Predator) {
                hasPredator = true;
            }
            if (hasHerbivore && hasPredator) {
                return true;
            }
        }
        return false;
    }
```

8. Измени имя. Если у тебя добавятся сущности, то ты каждый раз имя метода будешь менять?

```java
    private boolean containsHerbivoresAndPredators(WorldMap worldMap) { // плохое имя
        boolean hasHerbivore = false;
        boolean hasPredator = false;
        for (Entity entity : worldMap.getAll()) {
            if (entity instanceof Herbivore) {
                hasHerbivore = true;
            } else if (entity instanceof Predator) {
                hasPredator = true;
            }
            if (hasHerbivore && hasPredator) {
                return true;
            }
        }
        return false;
    }
```

9. Метод должен либо собирать entity (в данном случае животных), либо давать информацию о том, есть ли травоядные и хищники (речь о том, что метод выполняет действия, а также возвращает результат)

```java
    private boolean containsHerbivoresAndPredators(WorldMap worldMap) {
        boolean hasHerbivore = false;
        boolean hasPredator = false;
        for (Entity entity : worldMap.getAll()) {
            if (entity instanceof Herbivore) {
                hasHerbivore = true;
            } else if (entity instanceof Predator) {
                hasPredator = true;
            }
            if (hasHerbivore && hasPredator) {
                return true;
            }
        }
        return false;
    }
```

10. Обсуждение метода `listenForUserInput`

 ```java
  private void listenForUserInput() {
        while (running) {
            String input = scanner.nextLine();
            while (!menu.isInteger(input)) {
                menu.incorrectInputMessage();
                input = scanner.nextLine();
            }
            int command = Integer.parseInt(input);
            if (command == Menu.EXIT) {
                running = false;
                break;
            } else if (command == PAUSE) {
                synchronized (this) {
                    pause = !pause;
                    if (!pause) {
                        notifyAll();
                    }
                }
            } else {
                menu.incorrectInputMessage();
            }
        }
        scanner.close();
    }
```

Тут слишком много ответственности. Во-первых, сам метод не должен находиться в Simlation. Он должен быть в UserInput классе. Также он выполняет слишком много всего. Сканирует значение, проверяет на корректность значение пользователя, проверяет является ли оно EXIT или PAUSE, потом ещё и выкидывает сообщение о том, что некорректный ввод. 

11. Обсуждение метода: `runSimulation`
Всё что я сейчас буду объяснять взято из "Чистый код" 3-я глава, которую настоятельно рекоммендую прочесть.

```java
 private void runSimulation(WorldMap worldMap, boolean isEndless) {
        this.worldMap = worldMap;
        this.worldMap = worldMapFactory.createWorldMap(DEFAULT_WIDTH, DEFAULT_HEIGHT);
        renderAction.execute(worldMap);
        Thread inputThread = new Thread(this::listenForUserInput);
        inputThread.start();

        while (running && (isEndless || containsHerbivoresAndPredators(worldMap))) {
            synchronized (this) {
                while (pause) {
                    try {
                        wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException("Exception from pause");
                    }
                }
            }

            sleepThreadForMillis(800);
            moveAction.execute(worldMap);
            renderAction.execute(worldMap);
            menu.pauseResumeMessage();
            menu.exitMessage();

            if (isEndless) {
                bornAction.execute(worldMap);
            }
        }
```

Начнём с самого начала:
Слишком длинное и непонятное условие. Вытащи в методе и дай ему более нормальное название:
```java
 while (running && (isEndless || containsHerbivoresAndPredators(worldMap))) {
```

Выдели while в отдельный метод.

В методе while должна быть 1 - 2 строки. Не более. Выдели всё в отдельный метод. 
try - catch должен быть в отдельном методе. Внутри него должна быть одна строка, как и во всех других блочных выражениях.

12. Класс WorldMapFactory:

```java
    public void placeNewEntityOnMap(WorldMap worldMap, Entity entity, int quantity) {
        for (int i = 0; i < quantity; i++) {
            Coordinates coordinates;
            do {
                coordinates = getRandomCoordinates(worldMap);
            } while (!worldMap.isValid(coordinates) || !worldMap.isEmpty(coordinates));
            try {
                Entity newEntity = entity.getClass().getDeclaredConstructor().newInstance();
                worldMap.put(coordinates, newEntity);
            } catch (Exception e) {
                throw new RuntimeException("Couldn't create new instance of " + entity.getClass().getSimpleName(), e);
            }
        }
    }
```

Также вынеси блочные выражения в один метод. Помни, что должна быть одна - две строки внутри них.

13. Плохое название метод

```java
 public void populateWithEntities(WorldMap worldMap) {
        placeNewEntityOnMap(worldMap, new Herbivore(), DEFAULT_HERBIVORE_QUANTITY);
        placeNewEntityOnMap(worldMap, new Predator(), DEFAULT_PREDATOR_QUANTITY);
        placeNewEntityOnMap(worldMap, new Grass(), DEFAULT_GRASS_QUANTITY);
        placeNewEntityOnMap(worldMap, new Tree(), DEFAULT_TREE_QUANTITY);
        placeNewEntityOnMap(worldMap, new Rock(), DEFAULT_ROCK_QUANTITY);
    }
```
Достаточно назвать `setupEntities`

14. Вынеси настройки в отдельный класс, либо даже файл:

```java
    public static final int DEFAULT_WIDTH = 10;
    public static final int DEFAULT_HEIGHT = 10;
    public static final int DEFAULT_HERBIVORE_QUANTITY = 6;
    public static final int DEFAULT_PREDATOR_QUANTITY = 2;
    public static final int DEFAULT_GRASS_QUANTITY = 25;
    private static final int DEFAULT_TREE_QUANTITY = 10;
    private static final int DEFAULT_ROCK_QUANTITY = 10;
```

15. Класс WorldMap

Не опускай фигурные скобки:

```java
  public void put(Coordinates coordinates, Entity entity) {
        if (isValid(coordinates)) {
            entities.put(coordinates, entity);
        } else
            throw new IllegalArgumentException("Invalid coordinates");
    }
```

16. Валидацию нужно вынести в отдельный класс: 

    public boolean isValid(Coordinates coordinates) {
        return coordinates.width() >= 0 && coordinates.width() < width &&
                coordinates.height() >= 0 && coordinates.height() < height;
    }

17. Класс Renderer. 
По-хорошему, стоит назвать его ConsoleRenderer и реализовать интерфейс Renderer, который позволит указать, что 
есть контракт, который скажет о том, что есть возможность разных реализаций UI.

Слишком много вложенных блоков. Декомпозируй и доведи всё до того, что один блок хранил в себе одну строку.

```java
    public void render(WorldMap worldMap) {
        for (int height = worldMap.getHeight() - 1; height >= 0; height--) {
            StringBuilder line = new StringBuilder();
            for (int width = 0; width < worldMap.getWidth(); width++) {
                Coordinates coordinates = new Coordinates(width, height);
                if (worldMap.isEmpty(coordinates)) {
                    line.append(EMPTY_CELL_SPRITE);
                } else {
                    Entity entity = worldMap.getEntity(coordinates);
                    String background = getBackground(entity);
                    String sprite = getSprite(entity);
                    line.append(background).append(sprite).append(RESET);
                }
            }
            System.out.println(line);
        }
        System.out.println();
    }
```

Используй паттерн команда:

```java
    private String getSprite(Entity entity) {
        return switch (entity.getClass().getSimpleName()) {
            case "Herbivore" -> HERBIVORE_SPRITE;
            case "Predator" -> PREDATOR_SPRITE;
            case "Grass" -> GRASS_SPRITE;
            case "Tree" -> TREE_SPRITE;
            case "Rock" -> ROCK_SPRITE;
            default -> UNKNOWN_ENTITY_SPRITE;
        };
    }
```

Класс BornAction:

1. Вновь антипаттерн про наркомана:

```java
    private final WorldMapFactory worldMapFactory = new WorldMapFactory();
```

2. Слишком много условий. Стоит использовать паттерн команда. А также вынести всё в отдельные методы, притом объединить их исходя из их уровня абстракциии:

```java
    public void bornMissingEntities(WorldMap worldMap) {
        List<Entity> entities = worldMap.getAll();
        int herbivoreCount = 0;
        int predatorCount = 0;
        int grassCount = 0;

        for (Entity entity : entities) {
            if (entity instanceof Herbivore) {
                herbivoreCount++;
            } else if (entity instanceof Predator) {
                predatorCount++;
            } else if (entity instanceof Grass) {
                grassCount++;
            }
        }
        if (herbivoreCount < WorldMapFactory.DEFAULT_HERBIVORE_QUANTITY) {
            worldMapFactory.placeNewEntityOnMap(worldMap, new Herbivore(), WorldMapFactory.DEFAULT_HERBIVORE_QUANTITY - herbivoreCount);
        }
        if (predatorCount < WorldMapFactory.DEFAULT_PREDATOR_QUANTITY) {
            worldMapFactory.placeNewEntityOnMap(worldMap, new Predator(), WorldMapFactory.DEFAULT_PREDATOR_QUANTITY - predatorCount);
        }
        if (grassCount < WorldMapFactory.DEFAULT_GRASS_QUANTITY) {
            worldMapFactory.placeNewEntityOnMap(worldMap, new Grass(), WorldMapFactory.DEFAULT_GRASS_QUANTITY - grassCount);
        }
    }
```

