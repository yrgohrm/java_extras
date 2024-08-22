# Best Practices för undantag

Nedan följer en liten neddykning i vad jag anser är vettigt att tänka på
gällande hantering av undantag.

## Fånga aldrig ett undantag utan att hantera det

### Problem

Det finns lägen där man tvingas fånga undantag som man "vet" aldrig kommer att
inträffa men som man ändå tvingas fånga för att de är kontrollerade undantag.
Man kan då känna sig sugen på att bara fånga dem och inte göra något med dem.

Som t.ex. i följande kod:

```Java
try {
    URI dataUri = new URI("https://example.com/foobar");
    // then do something with dataUri
}
catch (URISyntaxException ex) {
    // do nothing, this will never fail!
}
```

Detta är väldigt dumt att göra av två anledningar. Det är inte sällan att man
faktiskt tänkt fel och det finns lägen där undantaget ändå kan uppstå, men
framför allt är det sannolikt att koden ändras utan att man lägger märke till
den tomma catch-satsen.

Detta kan då leda till att applikationen inte fungerar som det är tänkt, men man
får ingen vettig information om vad det faktiskt är som går fel. Det gör det
otroligt mycket svårare att hitta och åtgärda felet.

### Lösning

Lösnignen är enkel: fånga aldrig ett undantag utan att göra något med det.

I de flesta riktiga applikationer finns ett loggningsramverk och det är enkelt
att helt enkelt logga undantaget om det skulle uppstå. Om det faktiskt aldrig
inträffar så kommer ingenting att loggas och ingen skada är skedd. Om man
faktiskt har fel och det uppstår, då finns det i loggen och felsökningen blir
mycket enklare.

Om det är väldigt tidigt i ett projekt och man mot förmodan inte har valt ett
loggningsramverk ännu, se ändå till att gör någonting som `ex.printStackTrace()`
eller kasta igen som ett okontrollerat undantag.

```Java
try {
    URI dataUri = new URI("https://example.com/foobar");
    // then do something with dataUri
}
catch (URISyntaxException ex) {
    log.error("This should never occur.", ex);
}
```

## Kasta så specifikt undantag som möjligt

Ju mer specifikt undantag du kastar, desto bättre. Kodning är ett grupparbete
och ju tydligare det är vad varje bit kod gör, desto enklare är det för alla
inblandade att förstå och använda den koden korrekt.

Därför är det viktigt att även se till att undantag din kod kastar är så
informativ som möjligt.

Försök därför alltid att hitta det undantag som passar bäst. T.ex. kan det vara
lämpligt att använda `NumberFormatException` istället för ett
`IllegalArgumentException` om ett argument inte är ett korrekt nummer när det
borde vara det.

Undvik även att kasta helt generiska undantag som `RuntimeException` och
`Exception`. Var inte rädd för att skapa egna undantag vid behov.

## Fånga så specifika undantag som möjligt

För det mesta vill man undvika att fånga väldigt generiska undantag så som
`RuntimeException` eller `Exception`. Ju längre ner i anropshierarkin du
befinner dig desto mer specifik bör du sannolikt vara när du fångar undantag.

Om du fångar väldigt generiska undantag finns det stor risk att du fångar
undantag som du inte visste kunde uppstå och att du sedan hanterar dem på ett
inkorrekt sett.

Om du skriver kod för att hantera att en fil inte finns när du försöker läsa den,
se då till att fånga `FileNotFoundException` inte `IOException` eller ännu värre
`Exception`.

## Fånga aldrig Throwable.

Den värsta formen av att fånga för generiska undantag är att man försöker fånga
`Throwable`. Detta undantag omfattar både `Exception` och `Error` vilket kan ge
upphov till väldigt intressanta problem när något verkligen går snett.

Undantag av typen `Error` är fel som är av så allvarlig natur att det inte finns
någon anledning att fånga dem. Som t.ex. `OutOfMemoryError`. Om vi fångar dessa
riskerar vi att skapa ännu värre problem som gör att felsökningen i efterhand blir
mycket svårare att genomföra.

## När du kastar ett nytt undantag skicka alltid med det gamla

### Problem

Det är förhållandevis ofta vi vill ändra ett kastat undantag till ett annat,
t.ex. för att göra om det till ett mer specifikt undantag och få med mer
information.

När vi gör detta är det av yttersta vikt att vi skickar med det gamla undantaget
som `cause` annars tappar vi en väsentlig del av stack tracet som skall hjälpa
oss att felsöka problemet.

```Java
try {
    var img = readThePictureFromFile();
    // do some more stuff
}
catch (EOFException ex) {
    // This will remove any information about where in readThePictureFromFile
    // the EOFException really happened
    throw new PictureFormatException("EOF reached early while reading image.");
}
```

### Lösning

Lösningen är enkel. Se tilltid till att skicka med originalet som `cause`.
Därmed måste man även se till att egna undantag alltid har en konstruktor som
accepterar och hanterar den parametern korrekt.

```Java
try {
    var picture = readThePictureFromFile();
    // do some more stuff
}
catch (EOFException ex) {
    throw new PictureFormatException("EOF reached early while reading image.", ex);
}
```

## Inkludera relevant information i undantagets meddelande

Meddelandet som skickar med när ett undantag kastas är det första man tittar på
när man felsöker. Det är därmed av stor vikt att det meddelandet är tydligt och
förklarar vad det faktiskt är som gått fel.

"Something went wrong" är inte speciellt hjälpsamt och ger ingen indikation om
vad som faktiskt hänt. Att något gått snett vet vi redan eftersom det har
kastats ett undantag!

Försök skriva en kort och koncis beskrivning av vad som faktiskt gått fel och
inkludera gärna exakt den data som gett upphov till felet. T.ex. när man
försöker läsa från en fil som inte finns får man ett specifikt undantag som
berättar detta, men i meddelandet finner man även själva filnamnet på filen som
inte gått att öppna.

Man måste dock tänka till lite i hur man hanterar meddelanden i undantag så man
inte inkluderar känslig information som t.ex. personlig information eftersom
den informationen kommer att lagras på ställen där man sannolikt inte har riktigt
det skydd man borde ha.

## “throw early, catch late”

Uttrycket "throw early" innebär att man vill kasta undantag så tidigt som
möjligt. Har du en metod som kräver att ett antal villkor skall uppfyllas
kontrollera dessa direkt och kasta ett undantag om något är fel. Undvik att göra
halva arbetet och sedan inse att något är fel så du inte kan fortsätta. På så
vis undviker du även att undantaget som faktiskt kastas är et följdfel av någon
annat, vilket kör felet svårare att diagnosticera.

"Catch late" bör egentligen vara "catch as early as you can actually handle the
problem but not too early" men det är inte lika slagfärdigt.

När man är långt ner i anropsstacken är det sällan man kan hantera undantaget på
ett vettigt vis. Normalt sett är det högre upp i koden som det går att hantera
felen på ett bra vis, t.ex. genom att visa ett felmeddelande till användaren.

Att ha felhanteringen högre upp i anropsstacken gör dessutom att vi ofta kan
undvika att duplicera kod. Om vi har ett ställe där vi hanterar alla fel behöver
vi enbart ha koden där.

Som grund finns det två tillfällen där vi fångar undantag:
* När vi vill kasta ett annat mer specifikt undantag för tydlighets skull.
* När vi kan hantera felet på ett rimligt vis.

## Använd aldrig undantag för flödesstyrning.

Undantag är för fel som uppstår i koden och ett sätt att meddela anropande kod
att något gått snett. Det är inte ett substitut för if-satser eller annan logik.

Följande kod är inte bra kod:

```Java
public void bookSpa() {
    Scanner input = new Scanner(System.in);

    try {
        System.out.println("How old are you?");
        int n = input.nextInt();
        if (n < 18) {
            throw new IllegalArgumentException();
        }

        doSpaBooking(n);
    }
    catch (IllegalArgumentException ex) {
        System.err.println("You must be over 18 years old to book a spa");
    }
}
```

Att man fångar ett undantag man kastar i samma metod är ett tecken på att koden
bör skrivas om. Undantag är inte till för flödesstyrning och bör därför inte
användas till det.

## Återställ alltid interrupt när du fångar InterruptedException.

### Problem

`InterruptedException` är ett undantag som kastas av ett antal olika
tråd-relaterade metoder när en annan tråd begär att den tråden man befinner sig
i skall avsluta det den håller på med. För att hålla reda på om en tråd bör
avsluta sitt arbete finns det normalt en flagga, en interrupt-status, som man
kan kontrollera vid behov. Denna flagga nollställs dock när ett
`InterruptedException` kastas. Detta innebär att om vi fångar
`InterruptedException` är det mycket viktigt att vi återställer statusen så
denna information inte tappas bort.

Detta bör vi alltid göra, även om det inte spelar någon roll så som koden ser ut
för tillfället eftersom det är mycket stor risk att man missar att lägga till detta
när det väl behövs.

### Lösning

Lösningen är väldigt enkel. När man fångar `InterruptedException` bör man alltid
återställa interrupt-statusen med `Thread.currentThread().interrupt();` det
första man gör i catch-satsen.

```Java
try {
    // do stuff
}
catch (InterruptedException ex) {
    Thread.currentThread().interrupt();
    // handle exception
}
```

## Dokumentera vilka undantag dina metoder kastar.

Använd `@throws` i din javadoc för att tydligt dokumentera de undantag som dina
metoder kan kasta. Dokumentera både de kontrollerade och de okontrollerade
undantagen din metod kan kasta.

Men, försök inte leta upp alla potentiella undantag som i något specialfall
skulle kunna kastas. Om du har en metod som tar en int som argument som måste
vara positiv och du kastar `IllegalArgumentException` som så inte är fallet,
dokumentera det.

Däremot finns det ingen anledning att alltid deklarera en `@throws` för varje
metod som tar ett objekt som argument där anroparen skulle kunna skicka in null
istället för ett objekt.
