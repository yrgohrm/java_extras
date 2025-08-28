# Tips för att jobba effektivt med filer

## Använd alltid try-with-resources

### Problem

Filer, nätverksanslutningar och andra resurser som operativsystemet måste hålla reda på tar upp
minne som inte Javas skräpsamlare kan ta hand om effektivt själv (eftersom det inte finns några
garantier om exakt när skräpsamlaren körs). De är dessutom ofta begränsade i antal av andra
anledningar så de är en resurs som kan ta slut även om det finns gott om minne kvar.

Därför är det av yttersta vikt att de stängs korrekt när de inte behövs längre. På så vis kan
resurserna i operativsystemet rensas så fort som möjligt och problem undviks.

Det bör göras med hjälp av try-with-resources så ofta som möjligt, inte några andra varianter om
det inte absolut behövs. Introducerades med Java 7 så om man tittar på *riktigt* gammal kod kan
det hända att den hanterar det hela annorlunda.

Det här är mycket dålig kod:

```Java
public void processFile(Path inputPath, Path weightsPath) throws Exception {
    BufferedReader input = Files.newBufferedReader(inputPath);
    List<DataRecord> records = parseInput(input);
    input.close();

    return calculationOfResult(records);
}
```

Om det skulle uppstå något undantag i parseInput() kommer filen inte att stängas korrekt.

Att försöka fixa det med något annat än try-with-resources gör koden väldigt besvärlig. Om man
dessutom vill öppna flera filer så blir det ännu värre.

```Java
public void processFile(Path inputPath, Path weightsPath) throws Exception {
    BufferedReader input;
    try {
      input = Files.newBufferedReader(inputPath);
      List<DataRecord> records = parseInput(input);
      return calculationOfResult(records);
    }
    finally {
        // since there could have been an exception when creating input it could be null
        if (input != null) {
            input.close();
        }
    }
}
```

### Lösning

Använd try-with-resources. Det är inte svårare än så. När koden i try-blocket körts klart, eller
ett undantag kastas kommer filen automatiskt att stängas under ordnade former.

public void processFile(Path inputPath, Path weightsPath) throws Exception {
    try (BufferedReader input = Files.newBufferedReader(inputPath)) {
      List<DataRecord> records = parseInput(input);
      return calculationOfResult(records);
    }
}

## Gör inte try-with-resources blocket större än nödvändigt

### Problem

Man vill stänga filer så snart det bara går för att frigöra resurserna snabbt. Man behöver för
det mesta inte sträva efter maximal prestanda, men det kan vara lätt att göra lite för mycket.

Man bör se till att try-with-resources-blocken inte innehåller för mycket annat som gör att filerna
hålls öppna längre än nödvändigt.

```Java
public void processFile(Path inputPath, Path weightsPath) throws Exception {
    try (BufferedReader weights = Files.newBufferedReader(weightsPath);
         BufferedReader input = Files.newBufferedReader(inputPath)) {
        
        double weight = heavyCalculattionOfWeightFromFile(weights);

        List<DataRecord> records = parseInput(input);

        return heavyCalculationOfResult(records, weight);
    }
}
```

### Lösning

Använd flera try-with-resources om möjligt och strukturera om koden så filerna hålls öppna så kort
som möjligt. Det finns ingen anledning att sträva efter maximal prestanda alltid, men om man har kod
som öppnar många filer och processar data i långa sjok (sekunder/minuter) kan det vara bra att tänka
lite.

```Java
public void processFile(Path inputPath, Path weightsPath) throws Exception {
    double weight;
    try (BufferedReader weights = Files.newBufferedReader(weightsPath)) {
        weight = heavyCalculattionOfWeightFromFile(weights);
    }

    List<DataRecord> records;
    try (BufferedReader input = Files.newBufferedReader(inputPath)) {
        records = parseInput(input);
    }

    return heavyCalculationOfResult(records, weight);
}
```

## Läsa rader enkelt med `Files`

### Problem

Det är förhållandevis ofta man vill läsa filer rad för rad och göra något med varje rad. Den
klassiska loopen är inte speciellt svår, men ändå ofta mer än vad man behöver. Man vill ha
så enkel kod som möjligt för det allra mesta.

```Java
try (BufferedReader reader = Files.newBufferedReader(Path.of("file.txt")))
{
    String line;
    while ((line = reader.readLine()) != null) {
        doSomeStuff(line);
    }
}
```

### Lösning

Som tur var innehåller klassen `Files` massor med smarta metoder och en av dem är väldigt bra i det
här läget. `Files.readAllLines()` som returnerar en lista innehållande alla rader i filen. Inget
krångel alls behövs.

När man lär sig mer och förstår klassen `Stream` kan man dessutom använda metoden `Files.lines()` 
som är ännu bättre.

```Java
for (String line : Files.readAllLines(Path.of("file.txt"))) {
    doSomeStuff(line);
}
```

Man behöver inte ens en try-with-resources eftersom den används inuti `readAllLines()`. Observera
dock att den returnerar en lista med alla rader så är det en stor fil kommer det här ta mycket
minne på en och samma gång.

## Skriva och läsa data enkelt med `Files`

### Problem

Man vill läsa lite text från en fil eller skriva lite text till en fil och man vill ha det så enkelt
som möjligt. Även med try-with-resources blir det en del att hantera.

## Lösning

Använd `Files.readString()` och `Files.writeString()` så blir det jättenkelt.

```Java
// read everything in the file
String wholeFile = Files.readString(Path.of("input.txt"));

// write the whole string to the file
Files.writeString(Path.of("output.txt"), wholeFile.toUpperCase());
```
