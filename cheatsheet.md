# Hilfreiche Informationen :)

# DB - Connection
![Wenn des Bild ned geht hast a Pech ghabt](/img/db_connection.PNG)

# Panache - Entity
```java
@NamedQuery(name = Showing.FINDALL, query = "SELECT s FROM Showing s")
@Entity
public class Showing extends PanacheEntityBase implements Serializable {
    @Id
    @GeneratedValue
    Long id;

    public Date start;

    public static final String FINDALL = "FINDALLSHOWINGS";


    @Column(name="vorname", length=50, nullable=false)
    public String name;

    @ManyToOne
    public Movie movie;

    @ManyToOne
    public Hall hall;


    @OneToMany(mappedBy = "showing")
    public List<Reservation> reservations;

}

```
PanacheEntityBase: Bei eigener ID

PanacheEntity: Bei generated ID





Wichtig:
Panache hängt bei Beziehung defaultmässig _id an den Attributnamen dazu (aufpassen wegen import.sql!)

Overwrite mit @JoinColumn(name=)


# Repository Beispiel
```java
@ApplicationScoped
public class GenreRepository implements PanacheRepository<Genre> {

    @Transactional
    public Genre update(Genre g) {
        return Panache.getEntityManager().merge(g);
    }

    public List<Genre> findGenresWithName(String s) {
        TypedQuery<Genre> query = Panache.getEntityManager().createQuery(
                "SELECT g FROM Genre g WHERE g.name like :s", Genre.class
        );

        query.setParameter("s", "%" + s + "%");

        return query.getResultList();
    }

    public List<Genre> getGenres(Genre g) {
        return find("name", g.name).list();
    }

      public List<Genre> getTest() {
        return Panache.getEntityManager().createNamedQuery(Genre.FINDALL, Genre.class).getResultList();
    }

}
```

# Boundary Beispiel
```java
@ApplicationScoped
@Path("g")
public class GenreBoundary {
    @Inject
    GenreRepository repo;


    @Path("/getGenres")
    @GET
    public Response getGenres() {
        return Response.ok(repo.findGenresWithName("H")).build();
    }

    @Path("/getTest")
    @GET
    public List<Genre> getGenresTest() {
        return repo.find("name", "Horror").list();
    }
}
```


# Achja, DTO kommt übrigens auch :D :)
LG 


# EmbeddedID
## Project
```java
@Entity(name="ProjectEmbedded")
public class Project {
    @EmbeddedId
    ProjectId id;

    public Project() {

    }

    public Project(ProjectId id, String projectName) {
        this.id = id;
        this.projectName = projectName;
    }

    private String projectName;
}
```

## ProjectId
```java
@Embeddable
public class ProjectId implements Serializable {



    @ManyToOne
    private Hall hall;


    @ManyToOne
    private Genre genre;


    public ProjectId() {

    }

    public ProjectId(Hall hall, Genre g) {
        this.hall = hall;
        this.genre = g;
    }
   
    // btw vergessts in hashcode und des equals ned haha
}
```


# IdClass
## Project
```java
@Entity
@IdClass(ProjectId.class)
public class Project {

    @Id
    private int departmentId;

    @Id
    private int projectNo;

    private String projectName;
}
```

## ProjectId
```java
public class ProjectId implements Serializable {
    private int departmentId;
    private int projectNo;


    public ProjectId() {

    }

    public ProjectId(int departmentId, int projectNo) {
        this.departmentId = departmentId;
        this.projectNo = projectNo;
    }
}
```

# Socket


Events:
- onOpen
- onMessage
- onError
- onClose

```java
@ServerEndpoint("/...")
@ApplicationScoped
public class Socket {

    Set<Session> sessions = new ConcurrentHashSet<>();

    @OnOpen
    public void onOpen(Session session) {
        sessions.add(session);
        send(...);
    }

    @OnClose
    public void onClose(Session session) {
        sessions.remove(session);
    }

    @OnError
    public void onError(Session session, Throwable throwable) {
        sessions.remove(session);
    }

    
    @OnMessage
    public void onMessage(String message, @PathParam("username") String username) {
        send()
    }

    public void send(Session session, Survey survey) {
        session.getAsyncRemote().sendObject(surv, sendResult -> {
        });

    }
}
```