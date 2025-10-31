# Öppen källkod, fri mjukvara och licenser.

Som programmerare badar man i kod från mängder av olika källor och termer som öppen källkod (open
source), fri mjukvara (free software) och licenser kan vara termer man stöter på. Dessa är alla
mycket viktiga att man har lite koll på, inte minst när man utvecklar programvara för företag.

## Öppen källkod och fri mjukvara

Termerna "öppen källkod" (open source) och "fri mjukvara" (free software) används ofta synonymt, men
de har faktiskt olika ursprung och betoningar. Rent juridiskt är de väsentligen samma sak men
ideologiskt är skillnaden milsvid. På senare år har även "källkod tillgänglig" (source available)
blivit allt vanligare men skall inte blandas ihop med vare sig öppen källkod eller fri mjukvara.

### Fri mjukvara (free software)

Begreppet myntades av Richard Stallman och Free Software Foundation (FSF). Fokus ligger på
**frihet**, att kunna göra vad man vill med mjukvaran, inte priset för produkten. Man brukar säga
"free as in speech, not as in beer". Målet är att användaren ska ha kontroll över programvaran.
Denna frihet definieras av "de fyra friheterna":

- **Frihet 0:** Friheten att köra programmet som du önskar, för ett godtyckligt syfte.
- **Frihet 1:** Friheten att studera hur programmet fungerar och att anpassa det för sina behov.
  Tillgång till källkoden är ett villkor för detta.
- **Frihet 2:** Friheten att vidaredistribuera kopior så att användaren kan hjälpa sin nästa.
- **Frihet 3:** Friheten att förbättra programmet och att ge sina förbättringar till allmänheten så
  att hela samhället drar nytta. Tillgång till källkoden är ett villkor för detta.

Filosofin bakom fri mjukvara handlar alltså om att programvara skall finnas tillgänglig för alla och
att förändringar också skall finnas tillgängliga och förbättra för alla.

### Öppen källkod (open source)

Termen "öppen källkod" introducerades senare och syftade till att göra konceptet mer affärsvänligt
och mindre ideologiskt laddat än "fri mjukvara". Allmänt är det definitionen från Open Source
Initiative (OSI) som är den som avses när man pratar om öppen källkod. OSI är mer fokuserad på
effektiv utveckling ur företags perspektiv och mindre på att enskildas rättigheter skall bevaras.

OSI har en lista på 10 punkter som de anser är definitionen av öppen källkod, där grunden så klart
är att källkoden finns tillgänglig.

1. **Fri vidare­distribution:** Programmet får fritt delas eller säljas utan avgifter.
2. **Källkod:** Källkoden ska vara tillgänglig och lätt att ändra.
3. **Härledda verk:** Modifiering och vidareutveckling ska vara tillåten.
4. **Integritet för författarens källkod:** Licensen kan skydda originalkoden men måste tillåta
   modifieringar via patchar.
5. **Ingen diskriminering av personer eller grupper:** Alla ska ha lika rätt att använda programmet.
6. **Ingen diskriminering av användningsområden:**: Programmet får användas inom alla
   verksamhetsområden.
7. **Vidare­distribution av licensen:** Licensen (rättigheterna) följer programmet till alla nya
   användare.
8. **Licensen får inte vara produkt­specifik:** Rättigheterna gäller oberoende av var programmet
   används.
9. **Licensen får inte begränsa annan programvara:** Licensen får inte ställa krav på annan
   programvara som distribueras tillsammans med den licensierade programvaran.
10. **Teknologineutralitet:** Licensen ska vara oberoende av teknik eller plattform.

### Källkod tillgänglig (source available)

Vad "källkod tillgänglig" innebär varierar stort, men innebär i grunden att källkoden finns att
läsa, men användningen, ändringen eller distributionen kan vara begränsad av juridiska villkor.

Med andra ord: du kan läsa koden, men du kanske inte får:
- Använda den kommersiellt.
- Ändra och distribuera den fritt.
- Skapa härledda verk utan tillstånd.

Mjukvara som är "källkod tillgänglig" är *inte* vare sig öppen källkod eller fri mjukvara och bör
i grunden betraktas på samma vis som klassisk proprietär stängd mjukvara.

En del projekt som är "källkod tillgänglig" har klausuler där mjukvaran övergår till öppen källkod
eller fri programvara efter ett antal år.

## Licenser

Idéerna bakom öppen källkod och fri mjukvara genomförs i praktiken med hjälp av licenser. En licens
är ett juridiskt avtal som anger vilka rättigheter och skyldigheter som gäller när någon använder,
ändrar eller distribuerar ett program eller dess mjukvara.

Det är väsentligen samma typ av juridiskt dokument som du godkänner när du installerar en mjukvara
som Office-paketet eller Windows, men som i det här fallet istället för att reglera vad du får och
inte får göra med programmet, reglerar vad du får och inte får göra med programmet och källkoden.

OSI har i runda slängar 100 olika licenser som de räknar som öppen källkod-licenser, medan 
Free Software Foundation listar en handfull med licenser för fri mjukvara.

## Viktiga licenser

En licens är det juridiska dokumentet som ger dig rätten att använda, ändra och distribuera
mjukvaran under specifika villkor. Att förstå dem och använda dem korrekt är kritiskt för att
undvika rättsliga processer.

De flesta open source-licenser faller inom två huvudkategorier: **copyleft** (starka) och
**permissive** (svaga).

### Copyleft-licenser

Dessa är "starka" licenser som har ett bra skydd för individens rättigheter. Om du använder
copyleft-licensierad kod i din produkt, måste hela den nya produkten också licensieras under samma
copyleft-licens. Syftet är att se till att de "Fyra Friheterna" bevaras för alla framtida
användare.

Företag är ofta mycket skeptiska till copyleft-licenser eftersom de kräver att företagens egna
produkter skulle behöva släppas med copyleft-licens de med. Många företag har ett uttalat förbud
mot att inkludera copyleft-licensierad mjukvara i sina produkter.

Urval av copyleft-licenser:
- **GNU General Public License (GPL):** Mycket populär, används bl.a. för Linux och GNU-verktygen.
- **GNU Affero General Public License (GNU AGPL):** Som GPL men med starkt skydd även vid användning
  i SaaS.
- **Mozilla Public License (MPL):** Används för Mozillas produkter, så som Firefox.
- **European Union Public Licence (EUPL):** Framtagen och godkänd av EU-kommissionen.

### Permissive-licenser

Dessa är "svaga" licenser som tillåter dig att använda, modifiera och distribuera koden utan att du
behöver släppa din egen källkod (tillåter att göra din kod till "stängd källkod"). Dessa är därför
extremt populära i företagsvärlden.

Dessa licenser är främst till för att skydda upphovspersonen från att hamna i juridiska tvistemål
och innehåller normalt friskrivningar från det mesta och att källkoden tillhandahålls "i befintligt
skick och utan några uttryckliga eller underförstådda garantier".

Deras tillåtande natur ger dock ett företag möjlighet att själva ta över och bygga en egen produkt
utifrån den öppna källkoden utan på något vis bidra tillbaka till den ursprungliga skaparen.

Urval av permissive-licenser:
- **Massachusetts Institute of Technology (MIT):** Extremt populär på grund av sin enkelhet. Nästan
  inga restriktioner.
- **Apache License 2.0:** Populär i stora företagsmiljöer. Ger bättre skydd mot patenttvister än
  MIT.
- **Berkeley Software Distribution (BSD):** En familj av licenser som liknar MIT. Kräver erkännande
  av upphovsrätt.
- **Internet Systems Consortium (ISC):** En modern, minimalistisk licens. I praktiken är
  funktionellt identisk med MIT/BSD.
  
## Använda och välja licenser

### Licens för egna projekt

När du skall välja licens för ett eget projekt (hobby eller kommersiellt) behöver du först fråga
dig om du är intresserad av att din kod får maximal spridning och är enkel för alla att använda,
eller om du vill värna om dina (och andras) friheter?

Är du ute efter maximal spridning är det en permissive-licens du bör välja. Är du intresserad av att
dina friheter bevaras och att framtida tillägg också blir fritt tillgängliga bör du välja en 
copyleft-licens.

När du bestämt dig om detta är det mer specifika detaljer som får styra och du bör jämföra olika
licenser för att hitta den som bäst matchar dina behov. Rent generellt är det rimligt att utgå från
ISC-licensen för permissive-licenser och GPL v3 för copyleft-licenser.

Observera att om du väljer en permissive-licens kan detta begränsa dina möjligheter att inkludera
bibliotek som är copyleft. Väljer du en copyleft-licens kan detta begränsa dina möjligheter att
inkludera bibliotek som är permissive.

Den licens du har valt bör du inkludera i roten av ditt projekt i filen `LICENSE`. I vissa system
noterar man även licensinformation i andra filer så som i `package.json` för Node-projekt.

Observera att om du inte väljer att inkludera en licens tillsammans med din kod gäller upphovsrätten
och du behåller fullständiga rättigheter. Det är dock väldigt otydligt gentemot andra och bör
undvikas.

### Licenser att inkludera i kommersiell produkt

När du utvecklar en produkt för ett företag och vill inkludera ett bibliotek *måste* du verifiera
att dess licens är kompatibel med produktens. Ofta hittar man bibliotekens licens i `README.md`
eller `LICENSE` i bibliotekets repository.

Många företag har en policy för licenser som bör läsas och efterlevas. Inte sällan kan man behöva be
företagets juridiska avdelning att godkänna biblioteket innan man kan använda det. Observera även
att det är ditt (företagets) ansvar att säkerställa att bibliotek som inkluderas har korrekt
licensierade beroenden i sin tur.

Genrellt sätt om du utvecklar en produkt med stängd källkod är det för det mesta inga problem att
inkludera bibliotek med permissive-licenser, men inte de med copyleft-licenser.

