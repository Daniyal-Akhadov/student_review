Проект: https://github.com/vltolstov/Hangman-OOP

[vltolstov]


# Плюсы:
1. Игра запускается.
2. Хорошая попытка в ООП стиле написать приложение.

# Минусы: 
Класс Hangman:
1. Забыл указать модификаторы доступа как private в классе.

```java
    private static final int ERROR_COUNT = 6;
    static Scanner scanner = new Scanner(System.in);
    static Notifier notifier = new Notifier();
    static Status status = new Status();
    static Result result = new Result();
```

В чём минус? 
Минус в том, что есть доступ извне к данным полям. 
Что приведёт к тому, что будут side-effects, а это приведёт к неожиданным последствиям. 
Также я и другие разработчики, не будут видеть зависимости, которые используются для класса. 

2. Слишком большой метод main. Хорошо было бы вынести всё это в отдельный класс Gameloop. main тяжело читать. Слишком много всего в нём происходит, хотя нужно было бы ограничиться запуском игры. Щас же он реализует логику Menu, логику GameLoop, логику WordMask.
3. Класс Hangman, я бы переименовал в класса Launcher. Дело в том, что Hangman ни о чём не говорит. А Launcher подразумевает под собой запуск приложения.
4. Нет отступов перед if. Глава 5 "Чистый код", привожу подробное объяснение, когда вертикальные отступы нужны: Каждая пустая строка становится зрительной подсказкой, указывающей на начало новой самостоятельной концепции.
```java
    public static void initialization() {
        System.out.println("Для старта новой игры введите 'Старт'. Для выхода из игры введите 'Стоп'");
        String commandFromUserInput = scanner.next().toLowerCase();
        if (commandFromUserInput.equals("старт")) {
            status.setGameStatus(true);
            status.setGameLoopStatus(true);
        } else if (commandFromUserInput.equals("стоп")) {
            status.setGameStatus(false);
        } else {
            System.out.println("Некорректный ввод");
            status.setGameStatus(true);
        }
    }
```
Такая же история в main.

5. Это View. И оно должно быть вынесено в отдельный класс:

```java
        System.out.println("Для старта новой игры введите 'Старт'. Для выхода из игры введите 'Стоп'");
```

Так будет легче читаться код, за счёт того что всё вынесено в отдельный класс. Также возможно дублирование, а вынесение в отдельный класс позволит его убрать.

6. Это должно быть вынесено в отдельный класс

```java
String commandFromUserInput = scanner.next().toLowerCase();
```
Данная логика может быть переиспользована.

7. Данный блок стоит вынести в класс Menu. 

```java
    public static void initialization() {
        System.out.println("Для старта новой игры введите 'Старт'. Для выхода из игры введите 'Стоп'");
        String commandFromUserInput = scanner.next().toLowerCase();
        if (commandFromUserInput.equals("старт")) {
            status.setGameStatus(true);
            status.setGameLoopStatus(true);
        } else if (commandFromUserInput.equals("стоп")) {
            status.setGameStatus(false);
        } else {
            System.out.println("Некорректный ввод");
            status.setGameStatus(true);
        }
    }
```
8. В методе result.check мы выполняем две операции: проверка, а также возврат значения boolean. Стоит выполнить что - то одно. Читай 3-ю главу "Чистый код"

9. status - плохое название, ибо непонятно статус чего. Я бы переименовал как GameState, либо пусть будет GameStatus.

Класс Status:
1. Класс, в целом, хороший, однако стоит выносить магические значения в константы: Чистый код, глава 2.
```java
    public void checkGameLoopStatus(GameField gameField, Mistake mistake) {
        if (!gameField.getField().contains("-") || mistake.getMistakesCount() == 0) {
            this.gameLoopStatus = false;
        }
    }
```

Класс Scene.

1. Слишком большой switch. Я бы реализовал паттерн "Команда" и тем самым перестал бы добавлять каждый раз нечитаемый switch-case. Таких у тебя будет больше и больше. А так ты красиво реализуешь эту логику в маленькие команды.

Класс Result:

1. Во - первых, можно использовать тернарный оператор:

```java
    public boolean check(GameField gameField, Mistake mistake) {
        return !gameField.getField().contains("-") && mistake.getMistakesCount() != 0;
    }
```
А также слишком долгое условие, которое сложно читать. 

```java
    public boolean check(GameField gameField, Mistake mistake) {
        return !isContains(gameField) && isMistakeCountFinished(mistake);
    }

    private static boolean isMistakeCountFinished(Mistake mistake) {
        return mistake.getMistakesCount() != 0;
    }

    private static boolean isContains(GameField gameField) {
        return gameField.getField().contains("-");
    }
```

Класс Mistake:

1. Когда поле не меняется, стоит использовать final

```java
    private final List<String> duplicateWrongLetters;
```
2. Снова магические значения (я про -1)

3. Лучше использовать Set, и в целом переписать класс Mistake так, чтобы он смотрел на значения, которые уже были вставлены. А тут сложная и нечитаемая логика, которая подразумевает постоянное сравнение и странное добавление дублирующихся значений.

Класс GameField:

1. Сложно понять за что отвечает данный класс. Я даже сам не пойму. Переименуй.
2. В данном случае как раз используется View в неположенном месте.
3. SecretWord, стоит вынести в класс и назватье его Mask, либо WordMask.
4. Очень длинные методы. Стоит их декомпозировать.

Класс Dictionary:

1. Опять магическое значение в виде пути. Надо вынести в отдельную константу.
2. Данную конструкцию можно сократить до:

Было:
```java
this.dictionary = new ArrayList<String>();
```
Стало:
```java
this.dictionary = new ArrayList<>();
```
3. У тебя рандомное слово получается в этом классе. Я бы предложил данную логику вынести в класс WordRandomizer. Прочти про SRP.
4. Пробрасывай исключения:

```java
    private void readAllWords() {
        try {
            this.dictionary = Files.readAllLines(Paths.get("src/main/resources/words.txt"));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```
Так ты просто показываешь stack trace. А так мог бы его обработать где - то сверху.

Класс Loop:

1. Очень здоровые методы. Трудно читать
2. Очень большое количество параметров входит в play. Надо либо создать объект отдельный для того чтобы вмещать лишь одно значение в метод, благодаря тому, что у тебя в одном объекте будут все эти параметры.
3. Долгие условия (о них я писал выше), подумай как их вынести в метод и дать нормальное название.
4. Снова View в неположенном месте.
5. Снова не хватает отступов (читай чистый код, глава 5)
6. Декомпозируй метод в несколько и дай хорошие название для этого.
7. Сам класс назван так, что он не позволяет понять за что отвечает. Таким образом придётся вчитываться в логику класса, что плохо. Стоит дать подходящее название исходя из того что делает сам класс.

Класс Notifier:

1. Неплохой класс, который уведомляет об ошибках. Интересное решение
