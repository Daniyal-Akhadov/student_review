Проект: https://github.com/velz01/tennis-match

[danila]

# Плюсы
1. Отличный и очень симпатичный код. Я рад, что ты мой менти. 
2. Первосходный DAO и его подклассы, замечательная пагинация. 
3. Очень мало недочётов.
4. С архитектурой всё отлично.

# Минусы 
1. Есть ненужные библиотеки в класса, которые подчёркнуты серым цветом, например в классе PlayerDao. Чтобы их удалить с помощью hot-key, нажми "ctrl + alt + O" и все ненужные библиотеки удалятся.
2. Обсудим этот метод

```java
    @Override
    public Optional<Player> findByName(String playerName) {
        String hql = "From Player where name = :playerName";
        try (Session session = sessionFactory.getCurrentSession()) {
            session.beginTransaction();
            Optional<Player> player = session.createQuery(hql, Player.class).setParameter("playerName", playerName).uniqueResultOptional();
            session.getTransaction().commit();
            return player;
        } catch (HibernateException exception) {
            throw new DatabaseException("Database error");
        }
    }
```
2.1. В нём нет отступов перед try. Перед всеми и после блочных сущностей, по типу: for, for-each, try, if, while, do-while, надо ставить отступы.
2.2. Нужно вынести запрос hql в константу. Это позволит переиспользовать данный запрос. Речь об этом:
```sql
String hql = "From Player where name = :playerName";
```
*Таких моменты присутствуют в нескольких классах, так что я не буду более заострять на этом внимание.*

3. Не удалил отступ снизу:

```java
            session.getTransaction().commit();
            return matches;
        } catch (HibernateException exception) {
            throw new DatabaseException("Database error");
        }

    }
```
4. Лучше такие длинные запросы писать через вертикальные отступы:

Было:

```sql
List<Match> matches = session.createQuery("FROM Match where player1.name ILIKE :name or player2.name ILIKE :name", Match.class)
```
Стало: 

```sql
"FROM Match " +
"where player1.name ILIKE :name " +
"or player2.name ILIKE :name", Match.class)
```

5. Слишком длинная строка, снова вертикальные отступы нужны:

```sql
Optional<Player> player = session.createQuery(hql, Player.class).setParameter("playerName", playerName).uniqueResultOptional();
```

6. Снова перестарался:

```java
@Builder
public record MatchDto(Long id, Player player1, Player player2, Player winner, MatchScore matchScore){

}
```
Снова отступы. Давай дальше без меня обдумай это. Не стану повторяться, ибо получится ревью только из отступов))

7. Лучше назвать PlayerType. Понимаю твою мотивацию, мол, enum и всё такое, но это выглядит не очень:

```java
public enum EPlayer {
```

8. Мог сделать final, но не сделал:

```java
    private int indexPlayer;
```
Зачем это нужно? Затем, чтобы указать другому разработчику, что это константа. 

9. Ты добавил lombok, но не пользуешься им. 

Было: 

```java
public enum EPlayer {
    PLAYER1(0),
    PLAYER2(1);

    private final int indexPlayer;

    EPlayer(int indexPlayer) {
        this.indexPlayer = indexPlayer;
    }

    public int getIndexPlayer() {
        return indexPlayer;
    }
}
```

Стало:

```java
@Getter
public enum EPlayer {
    PLAYER1(0),
    PLAYER2(1);

    private final int indexPlayer;

    EPlayer(int indexPlayer) {
        this.indexPlayer = indexPlayer;
    }
}
```

10. Не забывай о форматирование, нажми: ctrl + K + D и всё само собой отформатируется:

```java
public class DatabaseException extends RuntimeException{
```

11. Лучше назвать CharacterEncodingFilter. Таким образом станет яснее мотивация данного класса:

```java
@WebFilter("/*")
public class BaseFilter implements Filter {

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        servletResponse.setCharacterEncoding(StandardCharsets.UTF_8.name());
        filterChain.doFilter(servletRequest,servletResponse);
    }
}
```

И кстати снова форматирование. Но с этим давай дальше сам.

12. Слишком много добавления атрибутов в контекст. Можно всё это свести к очень короткому варианту. 
Ибо зачем устанавливать все поля по одному? Можно указать один объект и все.

Как было:

```java
@WebListener
public class ApplicationServletContextListener implements ServletContextListener {
    public void contextInitialized(ServletContextEvent event) {
        HibernateUtil.initDatabase();
        ServletContext servletContext = event.getServletContext();

        MatchDao matchDao = new MatchDao();
        PlayerDao playerDao = new PlayerDao();
        servletContext.setAttribute("matchDao", matchDao);
        servletContext.setAttribute("playerDao", playerDao);
        MatchDtoMapper matchDtoMapper = new MatchDtoMapper();
        servletContext.setAttribute("matchDtoMapper", matchDtoMapper);

        servletContext.setAttribute("newMatchService", new NewMatchService(playerDao, matchDtoMapper));
        servletContext.setAttribute("matchScoreCalculationService", new MatchScoreCalculationService());
        servletContext.setAttribute("finishedMatchesService", new FinishedMatchesService(matchDao, matchDtoMapper));
        servletContext.setAttribute("ongoingMatchesService", new OngoingMatchesService());
    }
}
```

Как стало:

```java
@Getter
public class ApplicationServletContextListener {
    private final MatchDao matchDao;
    private final PlayerDao playerDao;
    private final MatchDtoMapper matchDtoMapper;
    private final NewMatchService newMatchService;
    private final MatchScoreCalculationService matchScoreCalculationService;
    private final FinishedMatchesService finishedMatchesService;
    private final OngoingMatchesService ongoingMatchesService;

    public ApplicationServletContextListener() {
        this.matchDao = new MatchDao();
        this.playerDao = new PlayerDao();
        this.matchDtoMapper = new MatchDtoMapper();
        this.newMatchService = new NewMatchService(playerDao, matchDtoMapper);
        this.matchScoreCalculationService = new MatchScoreCalculationService();
        this.finishedMatchesService = new FinishedMatchesService(matchDao, matchDtoMapper);
        this.ongoingMatchesService = new OngoingMatchesService();
    }
}
```
Это один объект который имеет все поля, что дальше?

```java
@WebListener
class ApplicationServletContextListener implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent event) {
        HibernateUtil.initDatabase();
        ApplicationServices applicationServices = new ApplicationServices();
        event.getServletContext().setAttribute("applicationServices", applicationServices);
    }
}
```

Можно сделать ещё лучше. Без конструктора в ApplicationServices. С помощью @Inject возможно создать объекты полей и соответственно выполнить инъекцию.

```java
@ApplicationScoped
@Getter
class ApplicationService {
    @Inject
    private MatchDao matchDao;

    @Inject
    private PlayerDao playerDao;

    @Inject
    private MatchDtoMapper matchDtoMapper;

    @Inject
    private NewMatchService newMatchService;

    @Inject
    private MatchScoreCalculationService matchScoreCalculationService;

    @Inject
    private FinishedMatchesService finishedMatchesService;

    @Inject
    private OngoingMatchesService ongoingMatchesService;
}
```
13. Не понимаю что значат эти вопросы. Удали их и не сбивай с толку других:

```java
    public MatchDto mapFromPlayersToDto(Player player1, Player player2) {
        return MatchDto.builder()
                .player1(player1)
                .player2(player2)
                .matchScore(new MatchScore()) //???
                .build();
    }
```

14. GameScore
Снова зачем - то ты закомментировал и опубликовал в продакшен код, с непонятными для других комментариями:

```java
//        if (isAdvantagePlayer1) {
//            winner = FIRST_PLAYER;
//        }
//
//        if (isAdvantagePlayer2) {
//            winner = SECOND_PLAYER;
//        }
```

15. GameScore
Магические значения по типу "7" надо вынести в константы:

```java
        if ((player1TieBreakPoints >= 7 || player2TieBreakPoints >= 7)
            && Math.abs(player1TieBreakPoints - player2TieBreakPoints) >= 2) {
            winner = player.toString();
        }
```

16. Дублирующиеся условия:

```java
getScorePlayer2() != Points.FORTY
getScorePlayer1() != Points.FORTY
```

Вынеси в отдельный метод.

17. Не понимаю что это значит:

```java
score[INDEX_PLAYER1] = score[INDEX_PLAYER1].next();
```
Вынеси в отдельный метод.

18. Тут очевидный дубляж кода. Вынеси в отдельный метод:

```java
        if (player == EPlayer.PLAYER1) {
            if (getScorePlayer1() != Points.FORTY) {
                score[INDEX_PLAYER1] = score[INDEX_PLAYER1].next();
            } else if (!isTie) {
                winner = FIRST_PLAYER;
            }
        }

        if (player == EPlayer.PLAYER2) {
            if (getScorePlayer2() != Points.FORTY) {
                score[INDEX_PLAYER2] = score[INDEX_PLAYER2].next();
            } else if (!isTie) {
                winner = SECOND_PLAYER;
            }
        }
```

19. Забыл снова отступы:

```java
    @ManyToOne
    @JoinColumn(name = "player1")
    private Player player1;
    @ManyToOne
    @JoinColumn(name = "player2")
    private Player player2;
    @ManyToOne
    @JoinColumn(name = "winner")
    private Player winner;
```

20. Я не понимаю, что такое "-1", вынеси в константу:

```java
    public List<MatchDto> getFinishedMatches(int pageNumber, String playerName, int pageSize) {

        int offset = (pageNumber - 1) * pageSize;
        
        if (playerName == null) {
            return matchDao.findAllPaginated(offset, pageSize)
                    .stream()
                    .map(matchDtoMapper::mapFromMatchToDto).toList();
        }
        
        return matchDao.findByPlayerNamePaginated(offset, pageSize, playerName).
                stream().
                map(matchDtoMapper::mapFromMatchToDto).toList();
    }
```

21. Снова дубляж, ещё и форматирования нет:

```java
        if (optPlayer1.isEmpty()) {
            player1 = Player.builder().name(playerOne).build();
            playerDao.save(player1);
        } else {
            player1 = optPlayer1.get();
        }

        if (optPlayer2.isEmpty()) {
            player2 = Player.builder().name(playerTwo).build();
            playerDao.save(player2);
        }else {
            player2 = optPlayer2.get();
        }
```

22. Не бойся писать полные название, я понимаю, что сам API java EE так делает, но почему мы должны следовать их правилам? Если что, я о req и resp

```java
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
```

23. Удаляй не используемые throws исключения:

```java
    @Override
    public void init(ServletConfig config) throws ServletException {
        this.finishedMatchesService = (FinishedMatchesService) config.getServletContext().getAttribute("finishedMatchesService");
    }
```

24. Всё это помести в DTO. Представь что тебе 1000 атрибутов надо ставить, то что, каждый раз так будешь делать?

```java
     req.setAttribute("matches", finishedMatches);
        req.setAttribute("pageNumber", pageNumber);
        req.setAttribute("filterName", playerName);
        req.setAttribute("pageSize", PAGE_SIZE_BY_DEFAULT);
```
Плюс, из - за этого метод большой. Я бы хотя бы вынес это в метод.
25. Снова забыл о констанах в классе FinishedMatchesServlet.

26. Выноси в константы магические значения:
```java
        if (!playerOne.matches("[а-яА-яёЁa-zA-Z ]+") || !playerTwo.matches("[а-яА-ЯёЁa-zA-Z ]+")) {
```

27. Щас будет долгое объяснение. Посмотрим на этот метод

```java

    public static void initDatabase() {
        try (InputStream resource = HibernateUtil.class.getClassLoader().getResourceAsStream("data.sql")) {
            byte[] bytes = Objects.requireNonNull(resource).readAllBytes();
            String sql = new String(bytes);
            Session session = sessionFactory.getCurrentSession();
            session.beginTransaction();
            session.createNativeQuery(sql).executeUpdate();
            session.getTransaction().commit();
        } catch (HibernateException exception) {
            throw new DatabaseException("Database error");
        } catch (IOException exception) {
            throw new ResourceNotFoundException("sql data is not found");
        }
    }
```

Что тут плохого?
Нет rollback. Нет логов. Слишком здоровый метод. Нет загрузчика ресурсов, хотя стоило бы вынести в отдельную ответственность, ты сразу инициализируешь значения, хотя стоило бы использовать DI, дабы подменять значения, ибо создание объектов это не обязанность данного класса, а ты это делаешь.

Как сделать?

```java
public class DatabaseInitializer {

    private final SessionFactory sessionFactory;
    private final ResourceLoader resourceLoader;

    public DatabaseInitializer(SessionFactory sessionFactory, ResourceLoader resourceLoader) {
        this.sessionFactory = sessionFactory;
        this.resourceLoader = resourceLoader;
    }

    public void initDatabase() {
        String sql = loadSqlData("data.sql");
        executeSql(sql);
    }

    private String loadSqlData(String resourceName) {
        try (InputStream resource = resourceLoader.getResourceAsStream(resourceName)) {
            byte[] bytes = Objects.requireNonNull(resource).readAllBytes();
            return new String(bytes);
        } catch (IOException exception) {
            throw new ResourceNotFoundException("SQL data is not found", exception);
        }
    }

    private void executeSql(String sql) {
        Session session = sessionFactory.getCurrentSession();
        try {
            session.beginTransaction();
            session.createNativeQuery(sql).executeUpdate();
            session.getTransaction().commit();
        } catch (HibernateException exception) {
            session.getTransaction().rollback();
            throw new DatabaseException("Database error", exception);
        }
    }
}

public interface ResourceLoader {
    InputStream getResourceAsStream(String resourceName) throws IOException;
}

public class ClasspathResourceLoader implements ResourceLoader {
    @Override
    public InputStream getResourceAsStream(String resourceName) {
        return getClass().getClassLoader().getResourceAsStream(resourceName);
    }
}

```

На этом всё.
