# Att skriva bra dokumentation för kod

Här kommer en kort sammanfattning om vad som utgör bra dokumentation av kod, framför allt med
fokus på hur man skriver bra javadoc. Det är så klart även applicerbart i andra sammanhang
som när man använder JSDoc eller Doxygen.

Bra dokumentation är avgörande eftersom den förvandlar din källkod till en **tydlig
specifikation och guide**, vilket gör att både du och dina kollegor snabbt kan förstå hur koden
fungerar.

## Grundprincipen: skriv för människor

Dokumentation handlar om att **skriva för människor först, datorer i andra hand**. Syftet är att
skapa ett tydligt **kontrakt** som beskriver exakt hur en metod eller klass ska bete sig. Utan att
man skall behöva läsa något av koden i sig.

## Format och struktur

Javadoc använder speciella blockkommentarer som börjar med `/**` och slutar med `*/`. Inuti dessa
kommentarer används vanligtvis HTML-formatering (eller från Java 23 kan även Markdown användas).

För att använda Markdown kommenterar man istället varje rad med `///`.

Exempel:
```Java
/**
 * Classic Javadoc comment.
 * 
 * @since 1.0
 */

/// Modern Markdown comment.
///
/// @since 1.0
```

### Den sammanfattande första meningen

**Den första meningen är viktigast.** Den ska vara en koncis men komplett sammanfattning av vad
klassen, metoden eller fältet gör. Detta beror på att Javadoc-verktyget automatiskt kopierar den
första meningen till sammanfattningstabellerna i den genererade dokumentationen.

Använd aldrig en hyperlänk (`{@link}`) i den första meningen, eftersom det kan göra den genererade
sammanfattningen förvirrande. Använd istället `{@code}`.

Exempel från verkligheten:
```Java
/**
 * This class consists exclusively of static methods for obtaining
 * encoders and decoders for the Base64 encoding scheme.
 * 
 * ...
 */
public final class Base64 {
    // ...
}
```

### Utförligare beskrivning

Efter den första meningen följer en mer detaljerad beskrivning. Om du har flera stycken, bör du
separera dem med en `<p>`-tagg (i traditionell Javadoc) eller en tom rad (i Markdown-format).

Exempel:
```Java
/**
 * This class provides a skeletal implementation of the {@code Set}
 * interface to minimize the effort required to implement this
 * interface.
 *
 * <p>
 * The process of implementing a set by extending this class is identical
 * to that of implementing a Collection by extending AbstractCollection,
 * except that all of the methods and constructors in subclasses of this
 * class must obey the additional constraints imposed by the {@code Set}
 * interface (for instance, the add method must not permit addition of
 * multiple instances of an object to a set).
 *
 * <p>
 * Note that this class does not override any of the implementations from
 * the {@code AbstractCollection} class.  It merely adds implementations
 * for {@code equals} and {@code hashCode}.<p>
 * 
 * ...
 */
public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E> {
    // ...
}
```

### Användning av kodterminologi

När du refererar till kodnyckelord, klassnamn, metodnamn eller variabler i beskrivningstexten,
använd den inbäddade taggen `{@code ...}`. Detta gör texten lättare att läsa i källkoden och
säkerställer att texten visas i ett *monospace*-typsnitt utan att tolkas som HTML.

Exempel:
```Java
/**
 * ...
 * 
 * Note that this class does not override any of the implementations from
 * the {@code AbstractCollection} class. It merely adds implementations
 * for {@code equals} and {@code hashCode}.<p>
 * 
 * ...
 */
public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E> {
    // ...
}
```

### Viktiga taggar

Javadoc stödjer ett flertal olika taggar (som börjar med `@`) vilka ger strukturerad information om
metoder och klasser. De som är absolut viktigast, för metoder, är följande:

- **`@param`:** Beskriver varje enskild parameter. Du måste tydligt förklara vad parametern
  representerar och, om tillämpligt, definiera om den kan vara `null` eller vilka gränser den har.
  `@param` är i princip obligatorisk för alla parametrar.
- **`@return`:** Beskriver returvärdet. Detta gäller även om beskrivningen känns uppenbar. Var noga
  med att specificera vad som returneras i specialfall (t.ex. `null` om ingen matchning hittas). 
- **`@throws:`** Dokumenterar undantag som metoden kan kasta, inklusive *kontrollerade undantag*
  (som måste deklareras i metodsignaturen) och de *okontrollerade undantag* som användaren rimligen
  kan vilja hantera.

Taggen **`@author`** kan man gärna undvika då den lätt blir utdaterad och kan ge en missvisande bild
av kodägande. Vem som har gjort vad ser vi ändå i vårt program för versionshantering.

Exempel:
```Java
/**
 * ...
 * 
 * @param  c collection containing elements to be removed from this set
 * @return {@code true} if this set changed as a result of the call
 * @throws UnsupportedOperationException if the {@code removeAll} operation
 *         is not supported by this set
 * ...
 */
public boolean removeAll(Collection<?> c) {
    // ...
}
```

Exempel:
```Java
/**
 * Calculates the final discount amount applicable to the provided order value
 * based on the supplied coupon code.
 * <p>
 * The returned discount amount is guaranteed not to exceed 50% of the
 * total order value.
 *
 * @param orderValue the total monetary value of the order, must be positive
 *                   and within the maximum supported limit (e.g., $100,000). Not null.
 * @param couponCode the discount code string. Null treated as no coupon applied.
 * @return the calculated discount amount as a {@link java.math.BigDecimal},
 *         or zero if the coupon is invalid, expired, or null.
 * @throws OrderProcessingException if an external service failure prevents 
 *                                  the verification or calculation of the discount.
 * @throws IllegalArgumentException if the {@code orderValue} is negative or 
 *                                  exceeds the maximum supported order amount.
 *
 */
public BigDecimal calculateOrderDiscount(
        BigDecimal orderValue, 
        String couponCode) throws OrderProcessingException {
    // ...
}
```

### Länkning och referenser

Du bör inkludera länkar till relaterade delar av koden för att göra navigeringen enklare:

- **`{@link}`:** Används för att skapa en inbäddad hyperlänk till en annan del av API:et (klass,
  metod, fält) direkt i textbeskrivningen.
- **`@see`:** Används ofta sist i dokumentationsblocket för att lista relaterade klasser eller
  metoder som är relevanta att känna till.

Exempel:
```Java
/**
 * ...
 *
 * This ensures that {@code s1.equals(s2)} implies that
 * {@code s1.hashCode()==s2.hashCode()} for any two sets {@code s1}
 * and {@code s2}, as required by the general contract of
 * {@link Object#hashCode}.
 *
 * ...
 * @return the hash code value for this set
 * @see Object#equals(Object)
 * @see Set#equals(Object)
 */
public int hashCode() {
        // ...
}
```

## Best practice för dokumentation

Best practice för dokumentationens innehåll handlar om att etablera ett **tydligt kontrakt** för hur
koden ska bete sig och att alltid skriva med den framtida underhållaren eller användaren i åtanke. I
fallet med koddokumentation är den framtida användaren så klart en programmerare, inte en
slutanvändare.

### Underhåll

Det är väldigt viktigt för bra dokumentation att den är **korrekt och aktuell**.

Den gyllene regeln är att **ändra dokumentationen i samma *commit* som kodändringen**. Föråldrad
dokumentation kan vara värre än ingen dokumentation alls.

### Stil och perspektiv

För att dokumentationen ska vara professionell är det viktigt att vara konsekvent och tänka på sitt
språkbruk:

#### Använd tredje person

Dokumentationen ska skrivas i tredje person (deklarativt) istället för andra person (imperativt, som
en instruktion).

- **Föredra:** "Gets the label."
- **Undvik:** "Get the label."

#### Fokusera på objektet

Metodbeskrivningar bör börja med en verbfras (eftersom en metod implementerar en operation). För
klass-, gränssnitts- och fältbeskrivningar kan du utelämna subjektet och bara ange objektet.

**Metodexempel:**
- Bra: "Gets the label for this button."
- Dålig: "This method gets the label for this button."

**Klass/fältexempel:**
- Bra: "A button label."
- Dålig: "This field is a button label."

#### Använd konsekventa referenser

När du hänvisar till en instans som skapats från den aktuella klassen, använd ordet **this**
istället för "the".

- **Föredra:** "Gets the toolkit for this component."
- **Undvik:** "Get the toolkit for the component."

#### Var kortfattad men specifik

Det är acceptabelt att använda fraser istället för fullständiga meningar i beskrivningar, särskilt i
sammanfattningar och `@param`-taggar, i syfte att vara kortfattad. Använd dock inte förkortningar
som "e.g." eller "i.e."; använd istället "for example" eller "that is".

## Detaljnivå

Dokumentationen är kodens kontrakt. Den ska beskriva allt som en anropare kan förlita sig på om
metodens beteende. Dokumentationen bör vara såpass detaljerad och tydlig att man inte behöver läsa
koden för att förstå.

### Skriv dokumentation som tillför värde

Undvik redundans. De bästa namnen i kod är självdokumenterande. Om dokumentationen bara upprepar de
ord som finns i metodnamnet eller parametrarna tillför den inget värde. Den ideala kommentaren ska
alltid belöna läsaren med information som inte var uppenbar från namnen.

Variera gärna hur du uttrycker dig. Beskriv beteendet på ett sätt i huvudbeskrivningen och med andra
ord i `@param` och `@return`.

Var noga med att dokumentera eventuella **sidoeffekter** av metoden, de måste framgå tydligt.

Dåligt exempel:
```Java
/**
 * Obtains a locale.
 *
 * @param language The language.
 * @param country The country.
 * @param variant The variant.
 * @throws NullPointerException thrown if something is null.
 * @return the object
 */
public static Locale of(String language, String country, String variant) {
    // ...
}
```

Bra exempel:
```Java
/**
 * Obtains a locale from language, country and variant.
 * This method normalizes the language value to lowercase and
 * the country value to uppercase.
 * 
 * ...
 *
 * @param language A language code. See the {@code Locale} class description of
 * {@linkplain ##def_language language} values.
 * @param country A country code. See the {@code Locale} class description of
 * {@linkplain ##def_region country} values.
 * @param variant Any arbitrary value used to indicate a variation of a {@code Locale}.
 * See the {@code Locale} class description of {@linkplain ##def_variant
 * variant} values.
 * @throws    NullPointerException thrown if any argument is null.
 * @return A {@code Locale} object
 */
public static Locale of(String language, String country, String variant) {
    // ...
}
```

### Skilj på "vad/hur" och "varför"

Dokumentationen (metod- och klasskommentarer) ska svara på **vad** koden gör och **hur** man
använder den. Förklaringar om **varför** koden beter sig på ett visst sätt eller varför ett
designval gjordes hör hemma i inline-kommentarer, inte i dokumentationen.

Det är viktigt att dokumentationen inte beskriver i detalj hur beteendet är implementerat i själva
koden, utan vad **beteendet** är av koden som helhet. Om du måste dokumentera
implementationsspecifikt beteende, sätt det i ett separat stycke med en tydlig inledande fras, till
exempel: "Implementation specific:" eller "On Windows-systems...".

Dåligt exempel:
```Java
/**
 * Calculates a simple checksum for the input data set.
 * 
 * We have chosen to use a simple 32-bit XOR-based algorithm because 
 * a previous version (v1.5) had critical performance issues when we used 
 * the full CRC32 implementation. The main goal of this lightweight
 * implementation, as determined in the architecture meeting 2023-Q4, is to 
 * minimize CPU load. 
 * 
 * <p>
 * This method performs its operation by iterating over the internal 
 * data structure 'currentDataArray', which is a {@code byte[]}. It is vital
 * to note that, due to our platform-specific JNI bridge requiring 
 * the internal array to be pre-allocated, the calculation will halt 
 * if the list {@code data} has more than 1024 elements, instead of dynamically
 * expanding. This is a temporary workaround to avoid a memory leak 
 * discovered in Q2.
 *
 * @param data The list of integers to calculate the sum for. (not null)
 * @return The integer representing the XOR checksum.
 * @throws java.lang.NullPointerException If data is null.
 *
 */
public int calculateChecksum(List<Integer> data) { 
    // ...
}
```

### Markera terminologi

Använd inbäddade taggar för att markera tekniska termer och kodfragment, vilket gör dokumentationen
lättare att läsa i både källkod och genererad HTML.

- Använd `{@code ...}` (eller backticks i Markdown) för nyckelord, paketnamn, klassnamn,
  metodnamn, fältnamn, argumentnamn och kodexempel.
- Använd `{@link}` **sparsamt**. Länka endast till något annat om användaren faktiskt kan tänkas
  vilja klicka på den, och endast vid den **första förekomsten** av det namnet i kommentaren.

### Null-hantering

Att definiera hur `null` hanteras är en kritisk information för att bygga stora system och minimera
`NullPointerException`.

Alla icke-primitiva parametrar och returtyper **ska** definiera sin null-tolerans i `@param` eller
`@return`. Detta är viktigare än att dokumentera i klass- eller paketdokumentation, eftersom
utvecklare oftast bara ser Javadoc på metodnivå i sina IDE:er.

Använd standardiserade formuleringar, såsom:
- "not null" - `null` accepteras inte och kastar troligen ett undantag.
- "may be null" - `null` accepteras och dess beteende måste definieras.
- "null treated as x" - `null` hanteras på exakt samma sätt som värdet "x".
- "null returns x" - `null` kommer alltid att returnera "x".

Om null-hantering är definierad på detta sätt, bör du inte inkludera en `@throws NullPointerException`.

Exempel:
```Java
/**
 * Converts and formats a raw user name string based on an optional format preference.
 * <p>
 * The formatting process first removes all leading and trailing whitespace from the 
 * {@code rawName}. If the name is empty after trimming, an empty string is returned.
 *
 * @param rawName          the unprocessed name of the user, not null
 * @param preferredFormat  the desired output format string. Null treated as the default 
 *                         (mixed-case, e.g., "John Smith"). Accepts: {@code "UPPER"} 
 *                         or {@code "LOWER"}.
 * @return the formatted display name string, returns null if internal 
 *         processing fails unexpectedly due to resource constraints.
 * @throws IllegalArgumentException if the {@code preferredFormat} is provided 
 *                                  (is not null) but contains an unrecognized value.
 */
public static String formatDisplayName(String rawName, String preferredFormat) 
        throws IllegalArgumentException {
    // ...
}
```

## Referenser

- https://www.oracle.com/technical-resources/articles/java/javadoc-tool.html
- https://openjdk.org/jeps/467
- https://blog.joda.org/2012/11/javadoc-coding-standards.html
- https://developer.atlassian.com/server/confluence/javadoc-standards/
- https://www.jrebel.com/blog/tips-and-tricks-for-better-java-documentation
- https://google.github.io/styleguide/docguide/best_practices.html
