# Hilfreiche Informationen 🙂

 - Listen initialisieren (LinkedList)
 - JsonBTransient

# DB - Connection
![Wenn des Bild ned geht hast a Pech ghabt](/img/db_connection.PNG)

# Panache - Entity

 - PanacheEntityBase: Bei eigener ID (extends)
 - PanacheEntity: Bei generated ID (extends)
 - implements Serializible
 - @Id --> @GeneratedValue --> Long
 - @Column, @JoinColumn, @JoinTable (Attribute wie name, length, ... vergeben)
 - @OneToMany, @ManyToOne, @ManyToMany --> mappedBy
 - @NamedQuery(name = Klasse.Name, query = "...")


# Repository

 - ApplicationScoped
 - implements PanacheRepository\<Type>
 - @Transactional
 - Panache.getEntityManager() --> merge, persist, ...
 - Panache-Methoden, wie find, findById, findAll, --> .list() nicht vergessen

```java
TypedQuery<Genre> query = Panache.getEntityManager().createQuery( "...", Genre.class);

Panache.getEntityManager().createNamedQuery(Genre.FINDALL, Genre.class).getResultList();

query.getResultList().stream().map(entity -> new EntityDTO(entity.bezeichnung, entity.id)).collect(Collectors.toList());
```

# Boundary

 - ApplicationScoped
 - Path(...) --> Klasse und Methoden
 - @GET, @POST, ...
 - @Inject
 - Response.ok().build()


# Achja, DTO kommt übrigens auch 😄 🙂
LG 


# EmbeddedID
## Entity
```java
@Entity()
public class Entity {
    @EmbeddedId
    EntityEmbeddedId id;
    ...
```

## EntityEmbeddedId
```java
@Embeddable
public class EntityEmbeddedId implements Serializable {

    //Kann aus ids oder ganzen Entities bestehen, wie zB:
    @ManyToOne
    private Hall hall;
    
    private int genreId;

   
    // btw vergessts in hashcode und des equals ned haha
}
```


# IdClass
## Entity
```java
@Entity
@IdClass(ProjectId.class)
public class Project {
    @Id
    private int departmentId;
    @Id
    private int projectNo;
    ...
}
```

## ProjectId
```java
public class ProjectId implements Serializable {
    private int departmentId;
    private int projectNo;

    //Konstruktoren, equals, hashcode
}
```

# Socket
```java
@ServerEndpoint("/...")
@ApplicationScoped
public class Socket {
    HashMap<Long, List<Session>> sessions = new HashMap<>();
}
```
Events:
- onOpen
- onMessage
- onError
- onClose
```java
@OnOpen
public void onOpen(Session session, @PathParam("id") long productId) {
    if (sessions.get(productId) == null) {
        List<Session> dummy = new LinkedList<Session>();
        dummy.add(session);
        sessions.put(productId, dummy);
    } else {
        sessions.get(productId).add(session);
    }
}

@OnClose
    public void onClose(Session session, @PathParam("id") long productId) {
        // session entfernen
    sessions.get(productId).remove(session);
}

@OnError
public void onError(Session session, Throwable throwable, @PathParam("id") long productId) {
    // session entfernen
    sessions.get(productId).remove(session);
}


@OnMessage
public void onMessage(String message, @PathParam("id") long productId) {
    send()
}

public void send(long productId, int amount) {
    sessions.get(productId).forEach(session -> {
        session.getAsyncRemote().sendObject(amount, sendResult -> {
            if (sendResult.getException() != null) {
                System.out.println("Unable to send message: " + sendResult.getException());
            }
        });
    });
}
```


# Hilfreiche SQL-Methoden
- null value handling = COALESCE(value, default)
- Cast datatypes = cast(ivd.trnDatetime as LocalDate) (LocalDateTime to LocalDate)
- eliminate duplicates = select DISTINCT c from...
- count, sum, avg, max, min,
- Document AS d WHERE :kauf MEMBER OF b.kaeufe
- SELECT new htl.model.Kauf()


# Datumsgschichtl
```java
LocalDateTime.parse(
    dateString,
    DateTimeFormatter.ofPattern("dd.MM.yyyy HH:mm:ss")
)
```

```java
LocalDateTime jetzt = LocalDateTime.now();
LocalDate datum = jetzt.toLocalDate();
```

```java
String s = neich.format(DateTimeFormatter.ofPattern("yyyy.MM.dd"));
```