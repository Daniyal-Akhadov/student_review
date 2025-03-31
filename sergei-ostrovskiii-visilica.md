Проект: https://github.com/sergei-ostrovskiii/visilica

[sergei ostravskiii]

# Привет!

## Плюсы
Постарался написать в ООП стиле.
Сканируешь словарь с файла.

## Минусы
1. После правильно введённого слова игра не заканчивается, а просто бесконечный цикл происходит, куда можно писать бесконечно буквы. По-хорошему, сделать так, чтобы после окончания было окончание игры, где пользователю приходит сообщение о победе, либо поражении.
2. Очень трудные слова для угадывания
3. Плохая структура папок. Стоит исходить из контекста сущностей.

Класс Game: 
1. В Game есть комментарии, притом ничем не обоснованные. Прочти главу 4 "Чистый код"
```java
    // массив для правильных букв
    char[] correctLetter = new char[word.length()];
```

2. У тебя есть неиспользуемые импорты. 

```java
import java.util.Arrays;
import java.util.HashSet;
import java.util.Scanner;
import java.util.Set;
```
Чтобы удалить их все сразу, то нажми ctrl + alt + o

3. Изучи суть инкапсуляции: сокрытие данных извне, а также интегрирование сущностей по одной ответственности.

В данном случае не происходит никакого сокрытия данных:
```java
    public int countErrors;
```

4. Все поля должны быть приватными. У тебя все они открыты. Прочти про инкапсуляцию ещё раз.

5. Обёртки вместо примитивов:

```java
    //игровой цикл
    public void startGameLoop() {
        Boolean maxCountError = true;
        while (maxCountError) {
            if (countErrors>= 7) {
                maxCountError = false;
            }else {
                showWord();
                System.out.println("Введите букву");
                String letter = validatorUtils.userInput();
                validatorUtils.volidate(letter);
                System.out.println("Количество ошибок: " + countErrors);
                Gallows.showVisilica(countErrors);
            }
        }
    }
```

Зачем тебе тут Boolean, когда можешь использовать boolean? Всегда используй примитивы, когда это возможно. Они быстрее. 

6. Отсутствует форматирование строк. Прочти про Java Convension.

7. Магические значения. Вынеси их в константы. Прочти главу 2 "Чистый код". Там об этом написано.

Вот о чём я:

```java
    private boolean isMaxErrorReached() {
        return countErrors >= 7;
    }
```

8. Множество ошибок в словах. Можно использовать переводчик если не знаешь как правильно пишется слово. Также ты сам проект назвал `viselica` в то время как для этого есть перевод `hangman`. Таких ошибок у тебя очень много. Поправляй.
9. Вынеси блок while в отдельный метод. В блоке while должен быть один метод. Таким образом вынеси if в отдельный метод. Посмотри как красиво могло бы быть решение:

```java
   public void startGameLoop() {
        maxCountError = false;
        gameLoop();
    }

    private void gameLoop() {
        while (!isMaxErrorReached()) {
            showWord();
            System.out.println("Введите букву");
            String letter = validatorUtils.userInput();
            validatorUtils.volidate(letter);
            System.out.println("Количество ошибок: " + countErrors);
            Gallows.showVisilica(countErrors);

            if (isMaxErrorReached()) {
                this.maxCountError = true;
            }
        }
    }
```
11. Чем меньше параметров в методе, тем лучше. Избегай этого. 
Ты мог бы вынести в поля класса это и всё было бы хорошо. Суть класса читалась бы лучше.
Было:

```java
    private void gameLoop(boolean maxCountError) {
        while (maxCountError) {
            if (isMaxErrorReached()) {
                maxCountError = false;
            } else {
                showWord();
                System.out.println("Введите букву");
                String letter = validatorUtils.userInput();
                validatorUtils.volidate(letter);
                System.out.println("Количество ошибок: " + countErrors);
                Gallows.showVisilica(countErrors);
            }
        }
    }
```
Стало:

```java
    private void gameLoop() {
        while (maxCountError) {
            if (isMaxErrorReached()) {
                this.maxCountError = false;
            } else {
                showWord();
                System.out.println("Введите букву");
                String letter = validatorUtils.userInput();
                validatorUtils.volidate(letter);
                System.out.println("Количество ошибок: " + countErrors);
                Gallows.showVisilica(countErrors);
            }
        }
    }
```
12. Логика `VIEW` должна находиться во View, а не в самом классе игры. Назови, например, GameView. И туда выноси все эти выводы на экран, речь об этом:

```java
System.out.println("Введите букву");
```

13. Отсутствует вертикальный отступ:

```java
    public void showWord() {
        for (char ch : correctLetter) {
            System.out.print(ch);
        }
        System.out.println();
    }
```

Как надо:

```java
    public void showWord() {
        for (char ch : correctLetter) {
            System.out.print(ch);
        }

        System.out.println();
    }
```
В каких случаях применять: do-while, while, for, for-each, if, try-catch-finally.

14.  Ввод пользователя следут вынести в класс UserInput, а не validatorUtils.

Речь об этом:

```java
String letter = validatorUtils.userInput();
```

15. ValidatorUtils лучше назвать ValidationUtils.

16. Как можно меньше статики:

Речь об этом:

```java
public static void showVisilica(int numberErrors) {
```

Уменьшай их количество. 
Создавай экземпляры классов, а далее используй их методы, через объект.

17. Прочти по паттерн команда. 
У тебя уменьшатся такие длинные блоки if.
Речь о методе `Gallows.showVisilica(int numberErrors)`.

18. Класс FileManager:

Этот метод следует отнести в класс WordRandomizer:

```java
    // получить рэндомное слово
    public static String getRandomWord(ArrayList<String> collekcia){
        int l = (int) (Math.random() * collekcia.size()) + 1;
        String stroka = collekcia.get(l);
        String slovo = stroka.toLowerCase();
        return slovo;
    }
```

Также не сокращай слова на одну букву. Это плохо читается. Тут также присутствует все проблемные пункты что я описал выше.

Ещё слово collekcia. Это совсем неправильно. Следует использовать англоязычные слова, а не руссификация на английском.

19. Ещё раз, не сокращай слова до одной, двух и т.д. букв. Это плохо читается. 

20. try/catch надо изолировать в другие методы. То что внутри try/catch должно быть одной строкой, а не несколько. Try-catch уродливы сами по себе и их трудно читать. 

![image](https://github.com/user-attachments/assets/490ca24b-349a-4a26-a1d8-d13febb1d9a3)

Таким образом следует этот метод изменить:
```java
  // получить коллекцию слов из файла
    public static ArrayList<String> getCollectionWordsFromFile() {
        ArrayList<String> collekcia = new ArrayList<>();

        try {
            FileReader fr = new FileReader("src/main/resources/slovar.txt");
            Scanner sc = new Scanner(fr);

            while (sc.hasNextLine()) {
                String st = sc.nextLine();
                collekcia.add(st);
            }
            return collekcia;

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        return collekcia;
    }
```

21. Класс ValidatorUtils:

```java
    // введенные символы
    private Set<String> enteredLetters = new HashSet<>();

    private Scanner scanner = new Scanner(System.in);
```

Данные поля не `final`, хотя могли ими быть. Делай final, когда видишь, что у тебя не меняются поля. Вообще, есть такая методология, как писать все классы неизменяемые. Таким образом ты сможешь не менять менять объект класса вовсе, и при каждом изменений объекта, создаётся новый объект. String как раз отличный пример того как это работает. Всё это дело называется Immutable. 

22. `volidate` должен выполнять одну функцию, а тут выполняет две:

```java
    public void volidate(String input) {
        if (enteredLetters.contains(input)) {
            System.out.println("Буква уже была введенна");
        } else {
            checkLetterInWord(input);
        }
    }
```

23. Этот код делает слишком много всего: проверяет является ли буква корректной в контексте, добавляет букву, выводи сообщение о том, правильная ли буква, увеличивает количество ошибок, снова вывод сообщение. Это всё надо выносить в отдельные методы и классы. Читай чистый код: глава 3. Для скрытого слова стоит сделать класс: Mask. Там уже открывать буквы, проверять открытые и прочее

```java
    public void checkLetterInWord(String letter) {
        int index = game.word.indexOf(letter);

        if (index >= 0) {
            for (int i = 0; i < game.arrayWord.length; i++) {
                if (game.arrayWord[i] == game.word.charAt(index) && game.correctLetter[i] != game.arrayWord[i]) {
                    game.correctLetter[i] = game.arrayWord[i];
                }
                enteredLetters.add(letter);
            }
            System.out.println("Правильная буква");
        } else {
            enteredLetters.add(letter);
            game.countErrors++;
            System.out.println("Неправильная буква");
        }
    }
```

24. Класса: VisilicaApplication

В методе main, должен быть лишь один запуск процесса игры. Тут же есть вообще всё. Вывод сообщений, цикл while, который играет роль меню (К слову надо выносить эту логику в Menu), получает ответ от пользователя. В общем снова декомпозиция, снова hardcore.

25. Файл: slovar
Проблема в названии. Следует назвать его Dictionary. 



