# Плюсы

1. Всё приложение работает отлично. Включая критически ситуации по типу tie-brake.
2. Проверил тесты и они чекают критические ситуации, что, по своей сути, достаточно.
3. Код очень хорош. Архитектура соответствует MVC. Достаточно чистый код, везде присутствует форматирование.
4. Практически отсутстуют магические значения.
5. Хорошие CRUD реализации, с учётом неподдерживаемых CRUD операций.
6. 

# Минусы

Приложени:

1. Не отображаются картинки, причём во всём приложении.
2. После окончания игры нет кнопки выхода. Возможность только через кнопки вправой верхней части.
3. У тебя список побед идёт от первого к последнему, а надо от последних побед к первой. Инвертируй. 

Код:

Класс MatchDao:

1. Присутсвуют неиспользуемые импорты. Нажимай "ctrl + alt + O" и таким образом все неиспользуемые импорты будут удалены.
2. Стоит вынести sessionFactory в поле, дабы код уменьшился и стал яснее.

```java
try (Session session = HibernateUtil.getSessionFactory().openSession()) {
```

3. Стоит использовать вертикальные отступы в таких ситуациях:

```java
public static final String FROM_MATCH_BY_NAME_HQL = "from Match where player1.name ilike :filterByPlayerName or player2.name ilike :filterByPlayerName";
```

```java
List<Match> finishedMatches = session.createQuery(FROM_MATCH, Match.class).setFirstResult(offset).setMaxResults(maxResults).list();
```

enum PointsForDisplay:

1. Снова длинная строка и снова вертикальные отступы требуются (причём именно так и делается):

Было:

```java
    ZERO("0"), FIFTEEN("15"), THIRTY("30"), FORTY("40"), ADVANTAGE("AD");
```

Стало:

```java
    ZERO("0"), 
    FIFTEEN("15"),
    THIRTY("30"), 
    FORTY("40"), 
    ADVANTAGE("AD");
```

Исключение NotValidNameException:

Если не используешь что - то, то удаляй перед тем как отправить в продакшен. 
Не нужно оставлять что - то на будущее.

Класс Match:

1. У тебя создаётся создаётся в Match сам MatchScore, а Match за это не должен отвечать. Вообще это анти-паттерн, называется: "Control Freak"

```java
    @Transient
    private MatchScore matchScore = new MatchScore();
```

Таким образом получай его через конструктор.

Класс MatchScore:

1. У тебя не используется метод getPointsForDisplay, удаляй.
2. Создай метод для этих операций. Что - то по типу resetScore:

```java
    public MatchScore() {
        this.sets = new int[]{0, 0};
        this.games = new int[]{0, 0};
        this.points = new int[]{0, 0};
        this.isTieBreak = false;
        this.isDeuce = false;
        this.tieBreakPoints = new int[]{0, 0};
    }
```

Класс FinishedMatchService:

1. Используй просто имя INSTANCE, не нужно переписывать имя класса в UPPER_CASE:

```java
    private final MatchDao matchDao = MatchDao.MATCH_DAO;
    private final MatchMapper matchMapper = MatchMapper.MATCH_MAPPER;
```

2. Выдели для всего этого методы. Каждая строка пусть имеет свой метод. Иначе так выходит слишком тяжелая для понимания строчка.

```java
        return filterByPlayerName == null ?
                matchDao.findAllPaginated(offset, PAGE_SIZE).stream().map(matchMapper::matchToFinishedMatchDto).toList() :
                matchDao.findFilteredByPlayerNamePaginated(offset, PAGE_SIZE, filterByPlayerName).stream().map(matchMapper::matchToFinishedMatchDto).toList();
    }
```

Та же история:

```java
    public int getTotalPages(String filterByPlayerName) {
        int totalMatches = filterByPlayerName == null ?
                matchDao.findAllPaginated(0, matchDao.findAll().size()).size() :
                matchDao.findFilteredByPlayerNamePaginated(0, matchDao.findAll().size(), filterByPlayerName).size();
        return (int) Math.ceil((double) totalMatches / PAGE_SIZE);
    }
```

Класс MatchScoreCalculationService:

1. Куча магических значений

```java
        int playerIndex = selectedPlayer.equals("player1") ? 0 : 1;

        if (points[playerIndex] < 3) {
            points[playerIndex]++;
        } else if (points[playerIndex] == 3) {
```
Всё в константы.

2. Стоит вынести эту операцию в метод, так как она повторяется:

```java
points[playerIndex]++;
```

3. У тебя есть вложенный if. Я убрал его за счёт метода:

Было:

```java
  private static void updateRegularPoints(MatchScore matchScore, int playerIndex) {
        int[] points = matchScore.getPoints();
        int opponentIndex = getOpponentIndex(playerIndex);

        if (points[playerIndex] < 3) {
            points[playerIndex]++;
        } else if (points[playerIndex] == 3) {
            if (points[opponentIndex] < 3) {
                regularAddGameToPlayer(matchScore, playerIndex);
            } else if (points[opponentIndex] == 3) {
                points[playerIndex] = 4;
            } else if (points[opponentIndex] == 4) {
                points[opponentIndex] = 3;
            }
        } else if (points[playerIndex] == 4) {
            regularAddGameToPlayer(matchScore, playerIndex);
        }

        checkDeuce(matchScore, points);
    }
```
Стало:

```java
    private static void updateRegularPoints(MatchScore matchScore, int playerIndex) {
        int[] points = matchScore.getPoints();
        int opponentIndex = getOpponentIndex(playerIndex);

        if (points[playerIndex] < 3) {
            incrementPlayerPoints(points, playerIndex);
        } else if (points[playerIndex] == 3) {
            handlePointsEqualToThree(points, playerIndex, opponentIndex, matchScore);
        } else if (points[playerIndex] == 4) {
            regularAddGameToPlayer(matchScore, playerIndex);
        }

        checkDeuce(matchScore, points);
    }
```

Класс MatchService:

1. Тут не rollback. У тебя тут происходит поиск, сохранение двух игроков, что если второй игрок не сохранится? 

```java
    public Match createNewMatch(String playerOneName, String playerTwoName) {
        playerOneName = playerOneName.trim();
        playerTwoName = playerTwoName.trim();

        Match newMatch = new Match();
        Optional<Player> playerOne = playerDao.findByName(playerOneName);
        Optional<Player> playerTwo = playerDao.findByName(playerTwoName);

        if (playerOne.isEmpty()) {
            newMatch.setPlayer1(addNewPlayer(playerOneName));
        } else {
            newMatch.setPlayer1(playerOne.get());
        }

        if (playerTwo.isEmpty()) {
            newMatch.setPlayer2(addNewPlayer(playerTwoName));
        } else {
            newMatch.setPlayer2(playerTwo.get());
        }

        return newMatch;
    }
```

Нужно обработать это в try-catch-finally.

Класс BaseServlet:

1. Стоит его переименовать в ExceptionHandlerServlet.
2. 
