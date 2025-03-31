Проект: https://github.com/youlovehamit/Hangman

[khamit]

# Плюсы

1. Игра запускается
2. Игра работает
3. Есть форматирование в коде. Отлично! Если что, можно использовать ctrl + k + d и всё само собой поправится.
4. Есть хорошее отображение текущего положения пользователя.
5. Код написан в процедурном стиле, есть методы, но их, по-хорошему, разбить на более мелкие. Но это внизу.
6. Слова берутся из текстового файла.

# Минусы

1. Нет вертикальных отступов. Они должны быть под блочными сущностями, по типу: for, for-each, while, if и т.д. А также под ними (если только самим блоком метод не оканчивается). 

Было:

```java
        Scanner scanner = new Scanner(System.in);
        List<String> words;
        try {
            words = Files.readAllLines(Paths.get("src/resources/words.txt"));
        } catch (IOException e) {
            System.out.println("Ошибка чтения файла: " + e.getMessage());
            return;
        }
```

Стало: 

```java
        Scanner scanner = new Scanner(System.in);
        List<String> words;

        try {
            words = Files.readAllLines(Paths.get("src/resources/words.txt"));
        } catch (IOException e) {
            System.out.println("Ошибка чтения файла: " + e.getMessage());
            return;
        }
```

2. По - хорошему прокинуть ошибку в блоке catch и в другом месте её обработать. Иначе в другом месте не обработать (по-мимо текущего места).

Надо:

```java
        try {
            words = Files.readAllLines(Paths.get("src/resources/words.txt"));
        } catch (IOException e) {
            throw new RuntimeException(e);
            return;
        }
```
И вот дальше прокидываешь эту ошибку и обрабатываешь её в другом методе. Так можно обработать эту ошибку по - разному, а щас у тебя это делается только одним способом.

3. Хорошо бы вынести путь к файлу в константу. Таким образом можно её переиспользовать, а также если потребуется изменить значение, то сделаешь это в одном месте.

Я об этом: 

```java
words = Files.readAllLines(Paths.get("src/resources/words.txt"));
```

4. Слишком длинные строки - это плохо. Надо их разбивать на строчки.

Было:
```java
String randomWord = words.get(new Random().nextInt(words.size()));
```

Стало:
```java
Random random = new Random();
int index = random.nextInt(words.size());
String randomWord = words.get(index);
```
Теперь видно, что можно выделить в отдельный метод. Наприме из этих строк можно выделить метод:

```java
public static String getRandomWord() {
  ...
}
```

5. Учись выделять методы. Имеется в виду, что тут много различных операций и все они находятся в одном большом God - методе. Прочти по SRP в статейках.

Например, можно было выделить метод из этого:

```java
        try {
            words = Files.readAllLines(Paths.get("src/resources/words.txt"));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
```

И вот это вот:

```java
        while (game) {
            System.out.println("1. Новая игра");
            System.out.println("2. Выход");
            System.out.println("Введите цифру");
            int choice = scanner.nextInt();

            switch (choice) {
                case 1:
                    System.out.println("Загаданное слово: ");
                    Random random = new Random();
                    int index = random.nextInt(words.size());
                    String randomWord = words.get(index);
                    playGame(randomWord);
                    break;
                case 2:
                    System.out.println("До свидания!");
                    game = false;
                    break;
                default:
                    System.out.println("Некорректный ввод");
                    break;
            }
        }
```

Чем меньше метод будет содержать строк, тем легче будет дать ему название.

6. Очень сложно понять картинку. 

```java
    public static final String[] HANGMAN = new String[]{
            "______\n     |\n     |\n     |\n     |\n======",
            "______\n |   |\n     |\n     |\n     |\n     |\n======",
            "______\n |   |\n 0   |\n     |\n     |\n     |\n======",
            "______\n |   |\n 0   |\n |   |\n     |\n     |\n======",
            "______\n |   |\n 0   |\n/|   |\n     |\n     |\n======",
            "______\n |   |\n 0   |\n/|\\  |\n     |\n     |\n======",
            "______\n |   |\n 0   |\n/|\\  |\n/    |\n     |\n======",
            "______\n |   |\n 0   |\n/|\\  |\n/ \\  |\n     |\n======",
    };
```

Я покажу щас как бы сделал для одной из них:

```java
            "______" +
            "|     " +
            "|     " +
            "|     |" +
            "======",
```

Вот для первого индекса. Согласись, что так проще внедрять изменения?

7. В Java по конвенции принято давать имена boolean переменным со слова: has или is. А у тебя просто found.

Мы как бы ставим вопрос, is found? Найден ли? 

```java
        boolean found;
```
Не надо так делать

Надо так:
```java
        boolean isFound;
```

8. Не используй магические значения, по типу "8". Что такое 8? Это количество жизней? Ошибок? Или это секунды? Я могу понять из контекста, прочитав код, но хорошо было бы узнать это по имени. Выведи константу в таких ситуациях.

9. Метод playGame выполняет слишком много задач: управляет игрой, обрабатывает ввод пользователя, обновляет состояние игры и выводит результаты. Это нарушает принцип единой ответственности (Single Responsibility Principle). Лучше разделить код на несколько методов.

Вот как можно было бы его улучшить (я разбил на несколько методов его и дал ясные названия):

```java
public static void playGame(String word) {
    char[] guessedLetters = initializeGuessedLetters(word.length());
    int mistakes = 0;

    while (true) {
        printGameState(guessedLetters, mistakes);
        char guess = getPlayerGuess();
        boolean isCorrect = updateGuessedLetters(word, guessedLetters, guess);

        if (!isCorrect) {
            mistakes++;
            if (mistakes == 8) {
                System.out.println("Вы проиграли. Загаданное слово: " + word);
                break;
            }
        }

        if (isWordGuessed(guessedLetters, word)) {
            System.out.println("\nПоздравляю, вы угадали слово: " + word);
            break;
        }
    }
}

private static char[] initializeGuessedLetters(int length) {
    char[] guessedLetters = new char[length];
    Arrays.fill(guessedLetters, '_');
    return guessedLetters;
}

private static void printGameState(char[] guessedLetters, int mistakes) {
    System.out.println("\nТекущее состояние: " + Arrays.toString(guessedLetters));
    System.out.println("Ошибок: " + mistakes);
    if (mistakes > 0) {
        System.out.println(HANGMAN[mistakes - 1]);
    }
}

private static char getPlayerGuess() {
    Scanner scanner = new Scanner(System.in);
    System.out.println("Введите букву:");
    return Character.toLowerCase(scanner.next().charAt(0));
}

private static boolean updateGuessedLetters(String word, char[] guessedLetters, char guess) {
    boolean isCorrect = false;
    for (int i = 0; i < word.length(); i++) {
        if (Character.toLowerCase(word.charAt(i)) == guess) {
            guessedLetters[i] = word.charAt(i);
            isCorrect = true;
        }
    }
    return isCorrect;
}

private static boolean isWordGuessed(char[] guessedLetters, String word) {
    return new String(guessedLetters).equals(word);
}
```

В остальном всё хорошо!
Молодец, что решил задачу. Продолжай развиваться и скоро выйдешь на работу.
