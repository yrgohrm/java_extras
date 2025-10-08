# Använda listor som inre tillstånd i Java

Att använda listor som en del av klassers inre tillstånd är mycket vanligt, men kommer med ett
antal subtila problem då listor är föränderliga objekt. Det är viktigt att inte missa dessa och 
behandla listor på rätt sätt som vi skall se nedan.

Grunden till problemen är **inkapsling**. Ett av de primära målen vi har när vi skriver våra program
är att skydda objektens interna data (dess tillstånd) från att ändras oväntat utifrån. När det 
interna tillståndet består av primitiva datatyper som int och double är detta inte något problem.
Däremot när vi har ändringsbara objekt som t.ex. List är det en annan femma. Risken är stor att
du ger kod utanför klassen nycklarna till din privata data.
private data.

## Problemet: Läckande referenser

Variabler för objekt, som listor, innehåller inte objektet självt utan en referens till objektet. Vi
kan se detta lite som en fjärrkontroll till objektet i fråga.

**Fel sätt:** Om vi enbart kopierar fjärrkontrollen (referensen) så innehåller både originalet och 
kopian en fjärrkontroll till *exakt samma* lista.

Det betyder att om du tar in en lista i din konstruktor och enbart tilldelar den, eller om du
returnerar en privat listvariabel från en metod, så ger du inte ut en kopia av listan, utan en kopia
av fjärrkontrollen (referensen). Det gör det möjligt att ändra det inre tillståndet utifrån, man
bara använder fjärrkontrollen!

### Det naiva och farliga sättet

Här tar `Team`-klassen helt enkelt den externa listan `initialMembers` och bara kopierar referensen
till listan in i `members`. Detta bryter inkapslingen.

```java
// DANGEROUS EXAMPLE - DO NOT DO THIS
import java.util.List;
import java.util.ArrayList;

class Team {
    private List<String> members;

    public Team(List<String> initialMembers) {
        // This is the problem! We are just copying the "remote control".
        this.members = initialMembers;
    }

    public void printMembers() {
        System.out.println("Team members: " + this.members);
    }
}
```

När man sedan använder `Team`-klassen blir det lätt att ändra på dess privata tillstånd utifrån:

```Java
// Main program
List<String> players = new ArrayList<>();
players.add("Alice");
players.add("Bob");

Team myTeam = new Team(players);
myTeam.printMembers(); // Output: Team members: [Alice, Bob]

// Now, the external list is modified...
System.out.println("Adding Charlie to the original list...");
players.add("Charlie");

// ...and it has changed our object's private state!
myTeam.printMembers(); // Output: Team members: [Alice, Bob, Charlie] <-- Oops!
```

Som man kan se har det interna tillståndet hos `myTeam` ändrats från utanför `Team`-klassen och
har därmed helt sönder objektets inkapsling.

### Lösningen: Defensiv kopiering

För att lösa det här behöver vi skapa en **ny** lista inuti konstruktorn för `Team` och kopiera
elementen från listan vi fått som argument. Det här kallar man för **defensiv kopiering**.

```java
// CORRECT EXAMPLE
import java.util.List;
import java.util.ArrayList;

class Team {
    private List<String> members;

    public Team(List<String> initialMembers) {
        // The fix: create a NEW list that is a copy of the original.
        // Now, our internal state is completely independent.
        this.members = new ArrayList<>(initialMembers);
    }

    public void printMembers() {
        System.out.println("Team members: " + this.members);
    }
}
```

Nu är problemet löst och vi kan inte längre ändra på det privata inre tillståndet i `myTeam`:

```Java
// Main program
List<String> players = new ArrayList<>();
players.add("Alice");
players.add("Bob");

Team myTeam = new Team(players);
myTeam.printMembers(); // Output: Team members: [Alice, Bob]

// Now, the external list is modified...
System.out.println("Adding Charlie to the original list...");
players.add("Charlie");

// ...but our object's state is safe and unchanged!
myTeam.printMembers(); // Output: Team members: [Alice, Bob] <-- Correct!
```

## Att returnera listor: problemet med getters

Samma problem existerar, fast omvänt när vi har en metod som returnerar den privata listan direkt.
Även här lämnar vi över "fjärkontrollen" till den anropande koden som kan ändra det inre
tillståndet.

```java
// DANGEROUS GETTER EXAMPLE
class SafeTeam {
    private List<String> members;

    public SafeTeam(List<String> initialMembers) {
        this.members = new ArrayList<>(initialMembers); // Constructor is safe
    }

    // This getter is leaky! It returns a direct reference.
    public List<String> getMembers() {
        return this.members;
    }
}
```

```Java
// Main program
SafeTeam myTeam = new SafeTeam(List.of("Alice", "Bob"));
System.out.println("Initial members: " + myTeam.getMembers());

// Get the "leaked" reference
List<String> teamMembersRef = myTeam.getMembers();

// And use it to modify the private state from the outside
System.out.println("Adding a hacker to the team externally...");
teamMembersRef.add("Hacker");

System.out.println("Final members: " + myTeam.getMembers()); // Output: [Alice, Bob, Hacker] <-- Corrupted!
```

### Lösningar

Det finns två utmärkta sätt att lösa det här, med lite olika för- och nackdelar.

#### Variant 1: returnera en kopia

Precis som för konstruktorn kan vi skapa en kopia av listan:

```java
public List<String> getMembers() {
    // Return a new copy each time.
    return new ArrayList<>(this.members);
}
```

  * **Bra:** Det är enkelt att förstå och helt säkert. Den anropande koden får sin helt egna
    lista och kan göra vad de vill med den utan att vi riskerar att ändra vårt interna tillstånd.
  * **Dåligt:** Det kan bli ineffektivt. Om listan är stor och/eller den här metoden anropas ofta
    kommer många och stora objekt att skapas vilket tar mycket minne och processortid.
  
#### Variant 2: Returnera en omodifierbar vy (vanligtvis att föredra)

Java har en trevlig liten metod för exakt det här scenariot. Man kan returnera en "wrapper" eller
"vy" av vår lista som förhindrar alla modifikationer:

```java
import java.util.Collections;

public List<String> getMembers() {
    // Return a read-only "view" of the internal list.
    return Collections.unmodifiableList(this.members);
}
```

Om den anropande koden försöker ändra listan (t.ex. `myTeam.getMembers().add("Hacker");`), kommer
programmet att krascha med ett `UnsupportedOperationException`. Detta är jättebra eftersom det är en
tydlig och högljudd signal om att de inte får göra det.

* **Bra:** Extremt effektivt. Ingen ny lista skapas, så det går väldigt snabbt och använder minimalt
  med minne. Det kommunicerar tydligt get-metodens avsikt att endast tillåta läsning. Detta är det
  vanligaste och rekommenderade tillvägagångssättet.
* **Dåligt:** Det är en vy som är "live". Om din klass internt ändrar sin lista efter att ha
  returnerat vyn, kommer den ändringen att vara synlig för den som har vyn. Detta är oftast vad man
  vill ha från en get-metod, men det är viktigt att komma ihåg.

## Prestandapåverkan vid kopiering

Att kopiera stora listor ger en påverkan på programmets prestanda. De främsta prestandakostnaderna
kommer från två saker:

1.  **Minnesallokering:** Varje gång du skriver `new ArrayList<>(...)` skapas ett helt nytt objekt
    i minnet (heapen).
2.  **Processortid (CPU):** Datorn måste iterera över alla element i den ursprungliga listan och
    kopiera varje referens till den nya listan.

Hur stor påverkan detta har beror på **var** och **hur ofta** du gör kopian och så klart hur stort
det du kopierar faktiskt är.

#### I konstruktorn

Att göra en defensiv kopia i en konstruktor är oftast en **låg och acceptabel kostnad**. Det sker
bara en gång när objektet skapas. För de flesta applikationer är denna engångskostnad försumbar
jämfört med vinsten i form av robust och säker kod. Problemet uppstår endast om du skapar extremt
många objekt med väldigt stora listor i en prestandakritisk del av din applikation.

#### I getters

Här kan prestandapåverkan bli **betydande**. Om du returnerar en ny kopia från en getter-metod
(`return new ArrayList<>(listan)`) och den metoden anropas ofta (till exempel inuti en loop), kommer
du att:

* **Skapa många objekt:** Detta leder till ökat minnesanvändande.
* **Belasta skräpsamlaren:** Alla dessa kortlivade kopior måste städas upp,
  vilket tar extra processorkraft och kan orsaka små pauser i programmet.

Det är just därför att returnera en icke-modifierbar vy med `Collections.unmodifiableList(listan)`
är den överlägset bästa metoden för getters. Den skapar inget nytt listobjekt och kopierar inga
element, utan skapar bara ett litet, lättviktigt "omslagsobjekt" (wrapper). Det ger samma säkerhet
som en kopia men med mycket bättre prestanda. ✨

Observera att det alltid är en avvägning i slutänden. Om användningsmönstret av en getter innebär
att man på den anropande sidan ändå alltid skapar en ny lista från returvärdet kan det vara värt
att ändå skapa en kopia direkt i gettern för att undvika att det lättviktiga, men ändå onödiga,
vyn skapas.

Ser merparten av vår kod ut på det här viset kan vi lika gärna skapa en kopia direkt:

```Java
// Main program
SafeTeam myTeam = new SafeTeam(List.of("Alice", "Bob"));

// Get the unmodifiable list
List<String> teamMembersRef = myTeam.getMembers();

// Directly copy it into a new list, making the unmodifiable list basically useless
List<String> modifiableMembersRef = new ArrayList<>(teamMembersRef);

// Then add/remove/modify the new list
```
