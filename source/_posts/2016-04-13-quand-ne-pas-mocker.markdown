---
layout: post
title: "Quand ne pas mocker"
date: 2016-04-13 20:55:56 +0200
comments: true
categories: [Test, Craftsmanship]
---

En tant que _Test Driven Developer_ et plutôt adepte de l'école de TDD de Londres, j'ai tendance à utiliser un nombre assez important de _mocks_ quand je teste mon code. Cependant, il y a des cas où je trouve plus utile de ne **pas** en utiliser.

L'objectif de cet article est de décrire ces cas ainsi que la façon de tester le code sans utiliser de _mocks_.

<!-- more -->

## Bases de données

Quand je traite directement avec une base de données, la seule chose qui m'importe est si le code interagit correctement avec elle ou non. A part quelques cas particuliers d'optimisation ou de protection contre les injections SQL, la requête elle-même n'a que très peu d'importance.  
Je préfère donc utiliser une vraie base de données et faire des assertions directement dessus plutôt que de vérifier la requête générée.

De plus, il est plus simple de monter une base de données embarquée que de capturer la requête. En effet, il serait nécessaire de connaître l'implémentation utilisée pour accéder à la base de données et ceci est un signe que nos tests sont trop couplés à notre implémentation. Ils tomberaient donc en échec si l'on venait à en changer (passage de requêtes JDBC à un ORM, ou l'inverse par exemple).

L'exemple suivant utilise les classes `EmbeddedDatabaseBuilder` et `JdbcTemplate` de Spring pour créer et accéder à une base de données embarquée.

```java
public class UserRepositoryTest {

    @Test
    public void should_save_user_to_database() {
        DataSource ds = dataSource();
        JdbcUserRepository userRepository = new JdbcUserRepository(ds);

        userRepository.save(new User("John", "Doe"));

        assertThat(count(ds, "John", "Doe")).isEqualTo(1);
    }

    private int count(DataSource ds, String firstName, String lastName) {
        return new JdbcTemplate(ds).queryForObject(
                "select count(*) from user where first_name = ? and last_name = ?",
                Integer.class,
                firstName, lastName
        );
    }

    private DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.HSQL)
                .addScript("db/sql/create-db.sql")
                .build();
    }
}
```

Ce test ne tombera jamais en échec tant que le *comportement* de la classe ne change pas, même si son implémentation change totalement.

## Réseau

Le code qui communique à travers le réseau peut être difficile à tester en isolation. Selon moi, ceci n'est surtout pas forcément utile. En effet, le but principal est de vérifier le bon transfert d'informations à travers le réseau, quelle que soit la librairie utilisée pour effectuer ce transfert.

Dans ce genre de cas, je prends souvent le parti de créer un faux serveur qui sera lancé durant mes tests afin de communiquer avec le code à tester.

L'exemple suivant décrit le test d'un client TCP qui va communiquer avec un serveur qui se contente de renvoyer ce qu'on lui envoie. Le faux serveur est représenté par la classe `FakeEchoServer` :

```java
public class FakeEchoServer {
    private final int port;
    private volatile boolean running;

    public FakeEchoServer(int port) {
        this.port = port;
    }

    public void run() throws IOException {
        ServerSocket serverSocket = new ServerSocket(port);
        running = true;

        new Thread(() -> {
            while (running) {
                try {
                    Socket socket = serverSocket.accept();
                    try (DataInputStream reader = getReader(socket); DataOutputStream writer = getWriter(socket)) {
                        writer.writeUTF(reader.readUTF());
                        writer.flush();
                    }
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }).start();
    }

    private DataOutputStream getWriter(Socket socket) throws IOException {
        return new DataOutputStream(socket.getOutputStream());
    }

    private DataInputStream getReader(Socket socket) throws IOException {
        return new DataInputStream(socket.getInputStream());
    }

    public void stop() {
        running = false;
    }
}
```

Le test, quant à lui, va instancier ce faux serveur et appeler la méthode à tester du client.

```java
public class EchoClientTest {

    private static final int PORT = 9999;

    @Test
    public void should_send_message_and_get_response() throws IOException {
        FakeEchoServer server = new FakeEchoServer(PORT);
        server.run();

        String echoResult = new EchoClient("localhost", PORT).sendAndReceive("Hello");
        assertThat(echoResult).isEqualTo("Hello");

        server.stop();
    }
}
```

Le faux serveur devra évoluer au fur et à mesure que le *comportement* du client changera mais ne sera pas modifié par un changement de librairie ou de façon d'utiliser le réseau.


## Système de fichiers

Lorsque l'on doit interagir avec un système de fichiers, on le fait souvent via des appels statiques à des classes comme `FileUtils` de `commons-io` ou `Files` du JDK ou bien des appels à `new File()`. Quoi qu'il en soit, il est difficile de _mocker_ ce genre d'appels pour faire des assertions pertinentes.

Une solution pourrait être de ne pas _mocker_ du tout et de récupérer le fichier manipulé directement depuis le système de fichiers mais il faudrait alors penser à bien nettoyer le disque avant chaque test et faire aussi attention aux droits dans le dossier à utiliser.

Fort heureusement, JUnit propose une `Rule` dédiée à ce genre de cas. Il s'agit de `TemporaryFolder` qui va créer un dossier temporaire dans lequel il sera possible de travailler. A la fin du test, ce dossier sera supprimé.

Le test d'une classe manipulant des fichiers se présente alors sous la forme suivante :

```java
public class FileStoreTest {

    @Rule
    public TemporaryFolder temporaryFolder = new TemporaryFolder();

    @Test
    public void should_store_file_in_the_user_folder() throws IOException, URISyntaxException {

        File rootFolder = temporaryFolder.newFolder();
        byte[] content = Files.readAllBytes(Paths.get(ClassLoader.getSystemResource("files/avatar.png").toURI()));

        new FileStore(rootFolder).store(new UserFile("avatar.png", content, "john"));

        File expectedFile = new File(rootFolder, "john/files/avatar.png");
        assertThat(expectedFile).exists().hasBinaryContent(content);
    }
}
```

Cette technique a pour inconvénient de supprimer le dossier temporaire après chaque test. Ainsi, si le test est en échec il ne sera pas forcément aisé de comprendre pourquoi car le dossier aura disparu. Si ce besoin se fait sentir, il peut être plus pertinent de créer un dossier temporaire à la main dans une méthode de setup (annotée `@Before`) au lieu d'utiliser la `@Rule`.

## Mail

De nombreuses applications envoient des mails. Les stratégies généralement utilisées pour les tester se divisent en deux catégories :

- _Mocker_ la classe responsable de l'envoi des mails.
- Utiliser une adresse mail "poubelle" qui va recevoir tous les mails et les vérifier manuellement.

Aucune de ces deux méthodes n'est optimale car dans le premier cas, on ne sait pas ce qui est réellement envoyé par mail et cela rend le test très couplé à l'implémentation et dans le deuxième cas, le test n'est que très difficilement automatisable.

La solution optimale serait de pouvoir instancier un vrai serveur SMTP et de pouvoir récupérer les mails qu'il a reçu. C'est ce que propose la librairie [SubEtha SMTP](https://github.com/voodoodyne/subethasmtp).

Le test d'une classe permettant d'envoyer des mails va donc ressembler à ceci (on utilise également la classe `JavaMailSender` de Spring pour envoyer les mails) :

```java
public class MailSenderTest {

    private static final int SMTP_PORT = 5555;

    @Test
    public void should_send_mail_to_customer_address() throws MessagingException, IOException {
        Wiser wiser = new Wiser(SMTP_PORT);
        wiser.start();

        new MailSender(getJavaMailSender(), "contact@acme.org").send(
                new CustomerMail("john@acme.org", "Hey you!", "How are you?")
        );

        List<WiserMessage> messages = wiser.getMessages();
        assertThat(messages).hasSize(1);

        MimeMessage mimeMessage = messages.get(0).getMimeMessage();
        assertThat(mimeMessage.getSubject()).isEqualTo("Hey you!");
        assertThat(mimeMessage.getFrom()[0].toString()).isEqualTo("contact@acme.org");
        assertThat(mimeMessage.getRecipients(Message.RecipientType.TO)[0].toString()).isEqualTo("john@acme.org");
        assertThat(mimeMessage.getContent().toString()).isEqualTo("How are you?\r\n");

        wiser.stop();
    }

    private JavaMailSender getJavaMailSender() {
        JavaMailSenderImpl javaMailSender = new JavaMailSenderImpl();
        javaMailSender.setHost("localhost");
        javaMailSender.setPort(SMTP_PORT);
        javaMailSender.setProtocol("smtp");
        return javaMailSender;
    }
}
```

## Conclusion

Cette liste n'est bien sûr pas exhaustive est peut être complétée à l'infini. 
Cependant, une tendance émerge. En effet, on se rend compte que les zones où le _mock_ peut être compliqué, voire contre productif, se situent au niveau des interfaces avec le monde extérieur. Il n'est pas toujours important de tester la manière dont le code va communiquer avec l'extérieur, ni même le format exact des données envoyées. 
La plupart du temps, il est seulement important de savoir si le système avec lequel le code interagit a correctement compris le message ou non. Dans les autres cas (vérification de protocole, optimisations de requêtes, etc.), il est plus pertinent d'utiliser des _mocks_ pour analyser le contenu exact de ce qui est envoyé.
