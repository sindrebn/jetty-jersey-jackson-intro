Vi skal nå sette opp en webserver som tar i mot forespørsler (f. eks. fra en nettleser) og svarer på de fra bunnen av.


Komponentene vi skal bruke er:

* **Jetty**. Selve web-serveren. Denne sørger for at Java-applikasjonen vår kan ta i mot forespørsler og svare på de.
* **Jersey**. Rammeverk for å lage REST-applikasjoner.
* **Jackson**. Bibliotek for å oversette mellom JSON (som egner seg godt for å kommunisere mellom f. eks. nettleser og server) og Java-objekter
* **Maven**. Verktøy for å automatisere bygging av Java-applikasjoner. Her definerer vi hvilke andre biblioteker/rammeverk vi vil bruke og beskriver hvordan Jar-filen skal bygges.

Underveis i teksten lenker jeg til noen nyttige ressurser det kan være lurt å titte på. Jeg oppfordrer naturligvis til å oppsøke dokumentasjon, Stack Overflow og andre ressurser hvis noe er uklart. Lykke til!

## Lag prosjekt

Det er mange måter å lage et nytt Java-prosjekt på, men det er i de fleste tilfeller lurt å begynne med `pom.xml`-fila til Maven. Man for eksempel kopiere en fil fra et annet prosjekt eller bruke Maven sin prosjektgenerator (`mvn archetype:generate`). Men vi går for den litt enkle varianten i dag, og lar IntelliJ gjør jobben.

1. Klikk `File > New Project`, velg `Maven`, huk av for «Create from archetype» og finn `maven-archetype-quickstart` i lista.

1. Gi prosjektet en `groupId` og en `artifactId`, f. eks. `no.bekk` og `fagdag` hhv. Trykk deg igjennom resten av dialogen. Standardvalgene er sannsynligvis OK.

1. Gratulerer, du har akkurat laget en POM-fil 🏆 Ta gjerne en titt på den genererte fila og se om du skjønner hva som er greia her. Hvis ikke kan det være en idé å ta en titt i dokumentasjonen til maven. Som Java-utvikler kommer du til å se mange POM-filer 😅 Du kan forresten se litt bort fra alt som står mellom `<build>` og `</build>` for øyeblikket. Det er ikke så relevant for oss helt enda!

1. Kikk litt rundt i prosjekt-visningen til venstre i IntelliJ for å se hvilke andre filer som har blitt opprettet 👀 Mappestrukturen følger nå standard mappestruktur for Java-prosjekter med Java-kode i `src/main/java`-mappen og tester i `src/test/java`.

## Sett opp Jetty

For å kunne kjøre appen vår, skal vi altså bruke Jetty. Jetty er et open source prosjekt som er fritt tilgjengelig på [GitHub](https://github.com/eclipse/jetty.project). Siden applikasjonen vår skal bruke Jetty, må vi fortelle Maven at vi er avhengig av deres biblioteker.

Legg inn følgende linjer etter `<dependencies>`:

```xml
<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-server</artifactId>
    <version>${jetty.version}</version>
</dependency>
<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-servlet</artifactId>
    <version>${jetty.version}</version>
</dependency>
```

`${jetty.version}` refererer til _propertien_ `jetty.version`. Legg inn følgende linje før `</properties>`:

```xml
<jetty.version>VERSJONSNUMMER</jetty.version>
```

I stedet for «VERSJONSNUMMER» skal du putte inn nyeste versjon av bibliotekene som er publisert til [Maven Central](http://central.maven.org/maven2/). Det står litt om hva Maven Central er her: https://www.quora.com/What-is-the-Maven-central-repository

💡 For å finne nyeste versjon, kan det være lurt å bruke [søkefunksjonen](https://search.maven.org/) til Maven Central.

Du trenger _egentlig_ ikke flere avhengigheter enn Jetty for å ha en kjørende web-server. Men det kan være ganske komplisert å sette opp en stor web-tjeneste med bare Jetty. Vi skal derfor bruke: Jersey 🎉

## Sett opp Jersey

Jersey er et rammeverk som gjør det ganske enkelt å lage REST-tjenester. [Denne artikkelen](https://medium.com/extend/what-is-rest-a-simple-explanation-for-beginners-part-1-introduction-b4a072f8740f) beskriver ganske godt hva en REST-tjeneste er. Det er ikke nødvendig å følge alle disse prinsippene og begrensningene slavisk, men grunntanken om URLer som ressurser og bruken av HTTP-verb er i alle tilfeller sentrale.

Vi begynner med å legge til noen (🙄) avhengigheter:

```xml
<dependency>
    <groupId>org.glassfish.jersey.core</groupId>
    <artifactId>jersey-server</artifactId>
    <version>${jersey.version}</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.containers</groupId>
    <artifactId>jersey-container-servlet-core</artifactId>
    <version>${jersey.version}</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.containers</groupId>
    <artifactId>jersey-container-jetty-http</artifactId>
    <version>${jersey.version}</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.inject</groupId>
    <artifactId>jersey-hk2</artifactId>
    <version>${jersey.version}</version>
</dependency>

<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jaxb</groupId>
    <artifactId>jaxb-runtime</artifactId>
    <version>2.4.0-b180830.0438</version>
</dependency>
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.4.0-b180830.0359</version>
</dependency>
```

Legg til propertien `jersey.version` med nyeste versjonsnummer på samme måte som for Jetty.

## Konfigurer Jetty og Jersey

Legg følgende kode i `main`-metoden i `App`-klassen som har blitt laget for deg:


```java
ResourceConfig config = new ResourceConfig();
config.packages("no.bekk");
ServletHolder servlet = new ServletHolder(new ServletContainer(config));

Server server = new Server(8080);
ServletContextHandler context = new ServletContextHandler(server, "/*");
context.addServlet(servlet, "/*");

try {
    server.start();
    server.join();
} catch (Exception e) {
    e.printStackTrace();
} finally {
    server.destroy();
}
```

Her lager vi først en `ResourceConfig`. Dette er konfigurasjonen til Jersey, og forteller Jersey hvor den finner ressurser. Hvis du skrev inn noe annet enn «no.bekk» i `groupId`-feltet da du lagde prosjektet må du bytte ut det i koden her også.

Videre definerer vi at serveren vår skal kjøre på port `8080`. Dette tallet kan være hva som helst, så lenge det er under [65 535](https://www.google.no/search?q=max+port+number) 🤷‍♀️ 80 og 8080 er vanlige porter å kjøre webservere på.

Videre sier vi at våre ressurser skal være tilgjengelig under `/*`, dvs. at hvis vi har en ressurs som svarer på path «hello», er den tilgjengelig på https://localhost:8080/hello . Hadde vi skrevet `/api/*`, ville samme ressurs vært tilgjengelig på https://localhost:8080/api/hello .

Til slutt starter vi serveren. Vi må kalle på `join` for at [appen skal kjøre helt til vi avslutter den manuelt](https://stackoverflow.com/questions/15924874/embedded-jetty-why-to-use-join#comment22687651_15925026).

Du kan nå kjøre appen fra IntelliJ ved å trykke på den grønne «Play»-symbolet i den høyre «skulderen» ved main-metoden.

Nå kjører serveren vår, og du kan besøke den på http://localhost:8080. Siden vi ikke har laget noen ressurser enda, får vi bare [404 Not Found](https://en.wikipedia.org/wiki/HTTP_404) 😱, men det løser vi straks!

## Vår første ressurs

Nå skal vi lage vår første ressurs. Den skal ikke være superspennende, men bare vise litt tekst i nettleseren når vi går på en viss URL.

Først lager vi en ny klasse i samme package som `App`. Kall den f.eks. `Resource`:

```java
@Path("/")
public class Resource {

}
```

Vi legger på annoteringen `@Path` på klassen, med verdien `"/"`. Dette betyr at endepunktene (dvs. metodene) i denne klassen er tilgjengelig rett under pathen `/`. I praksis http://localhost:8080/.


Legg inn denne metoden i klassen:

```java
@GET
@Path("hello")
@Produces(MediaType.TEXT_PLAIN)
public String helloWorld() {
    return "Hello, world!";
}
```

Annotasjonene på denne metoden angir følgende egenskaper:

* Den skal svare på verbet `GET`. Det er verbet som brukes når du taster inn en URL i nettleseren og betyr at klienten spør om å få data fra serveren.
* Den skal svare på pathen `/hello`.
* Den produserer innhold av typen `text/plain`. Serveren forteller her hvordan klienten skal behandle responsen. Hvis _media typen_ er `text/plain` vil de aller fleste nettlesere bare vise responsen som sort tekst på hvit bakgrunn. Andre vanlige mediatyper kan f. eks. være `application/json` eller `application/pdf`.

Videre sier vi at metoden skal returnerer en `String` og den skal være `Hello, world!`. Metodenavnet spiller ingen rolle.

Kjør appen og gå til http://localhost:8080/hello 👊

## Mer avanserte ressurser

Nå skal du få leke litt på egen hånd. Når du er ferdig med dette avsnittet skal det gå an å gå til følgende URLer i nettleseren:

1. http://localhost:8080/hello/[navn]
1. http://localhost:8080/hello?name=[navn]

I begge tilfellene skal det stå «Hello, [navn]!» i nettleseren når man åpner de. `[navn]` kan selvfølgelig variere her, så hvis jeg går til http://localhost:8080/hello/Sindre skal det stå «Hello, Sindre!» 😉

Les mer om hvordan du kan få til dette i [dokumentasjonen til Jersey](https://jersey.github.io/documentation/latest/jaxrs-resources.html#d0e2040) 💪

## Sending av data fra klient til server

Frem til nå har vi bare _hentet_ data fra serveren. En web-server er ikke spesielt nyttig hvis vi ikke kan _sende_ data dit også.

Først skal vi bare lage oss en bitteliten klient, en enkelt HTML-side som bare har et skjema som vi sender til serveren vår. Legg denne filen hvor du vil på datamaskinenen din:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Simple POST example</title>
</head>
<body>
<form action="http://localhost:8080/post" method="post">
    <p>Name: <input type="text" name="name"></p>
</form>
</body>
</html>
```

💡 Legg merke til at `action` er «http://localhost:8080/post» og at input-feltets navn er «name».

Lag en ny metode i ressursen som:

1. Tar _i mot_ data på pathen `/post` (dvs. at vi ikke kan bruke verbet `GET`)
1. Aksepterer data fra et HTML-form (`@Consumes(MediaType.APPLICATION_FORM_URLENCODED)`)
1. Aksepterer ett parameter fra et HTML-form med navnet «name» (https://jersey.github.io/documentation/latest/jaxrs-resources.html#d0e2271)
1. Responderer med HTML. Det skal stå «Hello, [navn]» i fet tekst (`<strong>`) i nettleseren.

Åpne HTML-filen i en nettleser, fyll ut skjemaet og trykk enter for å teste.

## JSON JSON JSON

Det er ganske vanlig å lage klienter i JavaScript, og for å kommunisere med servere overføres data veldig ofte på JSON-format. JSON brukes fordi det egner seg godt i JavaScript-applikasjoner, det står tross alt for _JavaScript Object Notation_. Et alternativ til JSON er XML, men det skal vi ikke begi oss ut på i dag 🙃

Vi ønsker imidlertid ikke å jobbe med JSON i Java-koden vår, fordi det ikke er spesielt godt støttet. Derfor bruker vi et eget bibliotek for å oversette mellom Java-objekter og JSON. Vi skal nå bruke Jackson, men det finnes flere alternativer, f. eks. GSON.

Etter dette avsnittet skal vi kunne åpne http://localhost:8080/jsonGreeting?name=[navn] og få responsen:

```json
{
	"hello": "[navn]"
}
```

Først må vi legge til et par avhengigheter til:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.9.6</version>
</dependency>

<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-jackson</artifactId>
    <scope>runtime</scope>
    <version>${jersey.version}</version>
</dependency>
```

Den siste der er for at Jersey automatisk skal bruke Jackson for å _serialisere_ Java-objekter til JSON når det er nødvendig.

Så må vi lage oss en Java-klasse som skal oversettes til JSON. La oss kalle den `Greeting`:

```java
public class Greeting {

    private String hello;

    public Greeting() {
    }

    public Greeting(String hello) {
        this.hello = hello;
    }

    public String getHello() {
        return hello;
    }

    public void setHello(String hello) {
        this.hello = hello;
    }
}
```

Jackson funker sånn at den ser etter metoder som begynner med `get` når den skal lage JSON. Når vi har en metode som heter `getHello`, vil JSON-strukturen vi får ut ha elementet `hello: <verdien av hello>`.

IntelliJ kommer til å gi oss en advarsel fordi konstruktøren _uten_ argumenter og `set`-metoden ikke er i bruk, men den må likevel være der. Når Jackson skal oversette _fra_ JSON til Java bruker den nemlig den tomme konstruktøren for å instansiere klassen, og `set`-metodene for å sette verdier.

Nå må vi bare få laga en siste metode i ressursen vår. Den skal gjøre følgende:

* Svare på pathen `jsonGreeting?name=<name>`
* Returnere en instans av typen `Greeting` med `<name>` som verdi
* Returnere data med media type `application/json`

## Oppstart av applikasjonen

Litt ukult å starte appen fra IntelliJ, eller? 🤓 De mest hipstrete folka bruker selvfølgelig kommandolinja til alt! Helt til slutt skal vi ta i bruk en veldig praktisk maven-plugin som gjør dette veldig enkelt.

Maven har nemlig et helt lass av plugins for å automatisere så og si alle aspekter ved bygging og kjøring av en Java-applikasjon. Vi skal ikke gå i dybden på dette nå, men bare kjapt kaste et blikk på `exec-maven-plugin`.

Inne i den svære `<build>…</build>`-blokka som jeg sa vi ikke skulle tenke så mye på, legger du inn:


```xml
<plugins>
    <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>1.6.0</version>
        <executions>
            <execution>
                <goals>
                    <goal>java</goal>
                </goals>
            </execution>
        </executions>
        <configuration>
            <mainClass>no.bekk.App</mainClass>
        </configuration>
    </plugin>
</plugins>
```

Her har vi definert at vi skal bruke pluginen `exec-maven-plugin`, som lar oss kjøre f. eks. Java-kode via maven. Vi konfigurerer hvilken klasse som  har main-metoden vår, fordi det klarer den ikke gjette selv.

Vi kan nå ganske enkelt kjøre `mvn exec:java` for å kjøre appen 😏

## Hva nå?

Se om du får til å lage et endepunkt som _tar i mot_ JSON. Det kan for eksempel være på samme format som vi brukte tidligere:

```json
{
	"hello": "[navn]"
}
```

Metoden kan ganske enkelt returnere teksten «Hello, [navn]!» slik som de aller første metodene vi lagde.

For å sende JSON til serveren kan du for eksempel bruke `curl` ➰

```sh
curl --header "Content-Type: application/json" \
  --request POST \
  --data '{"hello":"[navn]"}' \
  http://localhost:8080/<path>
```
