# Equals och arv är problematiskt

För en ännu djupare diskussion av detta ämne titta på punkt 10 i [Effective
Java](https://www.pearson.com/en-us/subject-catalog/p/effective-java/P200000000138/9780134686042)
av Joshua Bloch.

Grunden i att jämföra objekt med varandra i Java bygger på metoden
`boolean equals(Object obj)`. Den bygger på fyra regler som vi **måste** hålla
oss till om systemet som sådant skall fungera.

Bryter vi så mycket som en av reglerna kan vi inte längre förlita oss på att
saker som sökning eller sortering kommer fungera som det är tänkt.

Reglerna är som följer:

* **Reflexivt**: x.equals(x) skall returnera sant för alla x som inte är null.
* **Symmetriskt**: x.equals(y) skall returnera sant endast om y.equals(x) returnerar sant.
* **Transitivt**: om x.equals(y) returnerar sant och y.equals(z) returnerar sant, ska då x.equals(z) returnera sant.
* **Konsekvent**: Alla anrop av x.equals(y) skall konsekvent returnera sant eller
  konsekvent returnera falskt, förutsatt att ingen information som används i
  equals-jämförelser på objekten ändras.

När vi har objekt som innehåller data i en arvshierarki blir detta omöjligt att följa fullt ut!

Säg att vi har följande klass som låter oss specificera en färg:

```Java
import java.util.Objects;

public class Color {
    private byte red;
    private byte green;
    private byte blue;

    public Color(byte red, byte green, byte blue) {
        this.red = red;
        this.green = green;
        this.blue = blue;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Color color)) return false;
        return red == color.red && green == color.green && blue == color.blue;
    }

    @Override
    public int hashCode() {
        return Objects.hash(red, green, blue);
    }
}
```

Vi kan testa att den fungerar t.ex. med nedan kod:

```Java
// These are all the same
Color c1 = new Color((byte) 1, (byte) 2, (byte) 3);
Color c2 = new Color((byte) 1, (byte) 2, (byte) 3);
Color c3 = new Color((byte) 1, (byte) 2, (byte) 3);

// This one is different
Color c4 = new Color((byte) 99, (byte) 2, (byte) 3);

// Reflexive
System.out.println("true = " + c1.equals(c1));
System.out.println("true = " + c2.equals(c2));
System.out.println("true = " + c3.equals(c3));
System.out.println("true = " + c4.equals(c4));

// Symmetric
System.out.println(c1.equals(c2) + " = " + c2.equals(c1));
System.out.println(c1.equals(c3) + " = " + c3.equals(c1));
System.out.println(c3.equals(c2) + " = " + c2.equals(c3));
System.out.println(c1.equals(c4) + " = " + c4.equals(c1));

// Transitive
System.out.println("true = " + c1.equals(c2) + " and true = " + c2.equals(c3) + " then true = " + c1.equals(c3));
```

Inga konstigheter så långt, men när vi nu inför arv blir det problem. Vi vill
kanske ha en färg som också har en alfa-kanal och tycker det är rimligt att en
sådan klass ärver av `Color`.

```Java
import java.util.Objects;

public class AlphaColor extends Color {
    private byte alpha;
    
    public AlphaColor(byte red, byte green, byte blue, byte alpha) {
        super(red, green, blue);
        this.alpha = alpha;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof AlphaColor that)) return false;
        if (!super.equals(o)) return false;
        return super.equals(o) && alpha == that.alpha;
    }

    @Override
    public int hashCode() {
        return Objects.hash(super.hashCode(), alpha);
    }
}
```

På ytan ser ingenting underligt ut och skulle vi köra samma uppsättning tester
med `AlphaColor` som vi gjort med `Color` kommer allt verka bra där med.

```Java
AlphaColor c1 = new AlphaColor((byte) 1, (byte) 2, (byte) 3, (byte)4);
AlphaColor c2 = new AlphaColor((byte) 1, (byte) 2, (byte) 3, (byte)4);
AlphaColor c3 = new AlphaColor((byte) 1, (byte) 2, (byte) 3, (byte)4);

// This one is different
AlphaColor c4 = new AlphaColor((byte) 99, (byte) 2, (byte) 3, (byte)4);

// Reflexive
System.out.println("true = " + c1.equals(c1));
System.out.println("true = " + c2.equals(c2));
System.out.println("true = " + c3.equals(c3));
System.out.println("true = " + c4.equals(c4));

// Symmetric
System.out.println(c1.equals(c2) + " = " + c2.equals(c1));
System.out.println(c1.equals(c3) + " = " + c3.equals(c1));
System.out.println(c3.equals(c2) + " = " + c2.equals(c3));
System.out.println(c1.equals(c4) + " = " + c4.equals(c1));

// Transitive
System.out.println("true = " + c1.equals(c2) + " and true = " + c2.equals(c3) + " then true = " + c1.equals(c3));
```

Problemet dyker upp när vi blandar `Color` och `AlphaColor`. En `AlphaColor`
skall ju vara en `Color`, men så är inte fallet.

```Java
// same "Color"
AlphaColor c1 = new AlphaColor((byte) 1, (byte) 2, (byte) 3, (byte)4);
Color c2 = new Color((byte) 1, (byte) 2, (byte) 3);

// Not symmetric!
System.out.println(c1.equals(c2) + " = " + c2.equals(c1));
```

En `AlphaColor` är lika med en `Color`, men en `Color` är inte lika med en
`AlphaColor`. Symmetrin är bruten, något som kan ge upphov till underliga
buggar.

En del, inte minst många IDE:ers automatgenererade version av `equals()` "löser"
detta genom att använda `getClass()` istället för `instanceof` vilket gör att
endast två objekt av exakt samma typ kan vara lika med varandra. Men i slutänden
är inte det en lösning heller eftersom den bryter mot 
[Liskovs substitutionsprincip](https://en.wikipedia.org/wiki/Liskov_substitution_principle)
som vi förväntar oss skall gälla för våra objekt.

Den tråkiga sanningen är att det inte går att lösa och att det i denna typ av
sammanhang kan vara rimligt att istället använda sig av komposition istället för
arv för att bygga sina objekt.

```Java
import java.util.Objects;

public class AlphaColor {
    private Color color; // composition, not inheritance
    private int alpha;

    public AlphaColor(Color color, int alpha) {
        this.color = color;
        this.alpha = alpha;
    }
    
    public Color getColor() {
        return color;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof AlphaColor that)) return false;
        return alpha == that.alpha && Objects.equals(color, that.color);
    }

    @Override
    public int hashCode() {
        return Objects.hash(color, alpha);
    }
}
```