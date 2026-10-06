# Krakatoa Smoke

Färgad rök i realtid som följer pekaren när du håller nere musknappen, antingen som miljontals fina partiklar eller som en riktig 3D-simulering av rök. Inspirerad av partikelrenderaren Krakatoa. Allt ligger i en enda HTML-fil i ren WebGL2, utan bibliotek eller byggsteg.

## Kom igång

Öppna `index.html` i en aktuell version av Chrome, Safari, Edge eller Firefox. Håll nere musknappen och dra. På mobil och surfplatta fungerar det med fingret, även med flera fingrar samtidigt.

Om webbläsaren vägrar köra sidan direkt från disken kan du starta en lokal server:

```bash
python3 -m http.server 8000
```

Gå sedan till `http://localhost:8000`.

## Lägen

- **Partiklar**: miljontals fina partiklar som lyser ihop, med strängar och slöjor av rök.
- **Volym**: rök i tre dimensioner. Gasen vid penseln är het och glöder, stiger av sig själv, svalnar och blir till rök som virvlar, skuggar sig själv och lyses upp av glöden. Med paletten Aurora glöder den gulgrönt och blir sedan upplyst blå.

Byt läge med knappen överst i panelen eller med M. Första gången startar sidan i Partiklar, sedan sparas valt läge till nästa besök. Lägg till `#volym` eller `#partiklar` sist i adressen för att öppna ett visst läge direkt. Varje läge simulerar bara sin egen rök, så den andra står kvar tills du byter tillbaka. Rensa (C) tömmer dock båda lägena, och byter du kvalitet föds partiklarna i Partiklar på nytt, medan röken där står kvar. Palettrutorna visar färgerna i det läge du är i.

## Styrning

| Tangent | Funktion |
| --- | --- |
| Space | Pausa och fortsätt |
| C | Rensa röken |
| H | Visa och dölj panelen |
| A | Slå på och av autopiloten |
| M | Byt mellan Partiklar och Volym |
| 1–5 | Byt palett: Aurora, Nebula, Magma, Spektrum, Aska |

I panelen ställer du in kvalitet, pensel, flöde, ljus, skuggor och glöd. Kvaliteten bestämmer antalet partiklar i Partiklar och storleken på 3D-gridet i Volym. Gridet följer fönstrets form. I Volym visar menyn vilket grid varje kvalitet ger i fönstret som det ser ut just nu, och listan uppdateras när du ändrar fönstrets storlek. Så här blir det i ett fönster på 1920 × 1080:

| Kvalitet | Partiklar | Volym |
| --- | --- | --- |
| Låg | 262 000 | 100 × 56 × 36 |
| Medel | 1 miljon | 136 × 76 × 52 |
| Hög | 2,1 miljoner | 200 × 112 × 76 |
| Ultra | 4,2 miljoner | 228 × 128 × 84 |

Volym har två egna reglage, och Livslängd betyder något annat där:

- **Avsvalning** styr hur snabbt den heta gasen svalnar. Ett kort värde ger en liten glödande låga. Ett långt värde låter gasen glöda och stiga längre.
- **Lyftkraft** styr hur starkt het gas stiger, från 0 till 3 gånger det normala. På 0 stiger röken inte av sig själv.
- **Livslängd** styr hur länge röken finns kvar och hur snabbt den går över i palettens äldsta färg. Värdet visar ungefär hur många sekunder röken syns efter att den lämnat penseln, från cirka 2 till 7,5 sekunder. Det planar ut eftersom gammal rök tunnas ut och stiger ut ur bilden. På lägre kvalitet försvinner röken lite tidigare.

Reglagen Rökfält och Partikeltäthet och de två första brytarna under Lager gäller bara Partiklar och döljs i Volym.

## Så fungerar det

### Partiklar

1. **Vätskesimulering.** En Navier–Stokes-lösare på grafikkortet, i stil med Stams *Stable Fluids*. Den har virvelförstärkning och turbulens från curl-brus. Penseldrag läggs in som utsmetade Gauss-fläckar längs pekarens väg, så att snabba drag inte lämnar luckor.
2. **Partiklar.** Positionerna lagras i en flyttalstextur och flyttas av hastighetsfältet. Ovanpå det läggs finare curl-brus för tunna strängar. Nya partiklar föds ur en ringbuffert längs penseldraget.
3. **Färg.** Röken föds i en varm färg som skiftar medan du ritar och svalnar sedan mot palettens kalla färg.
4. **Ljus.** En ljuskälla vid pekaren kastar skuggor genom röken med en strålmarsch i skärmrymden. Rökens täthet ger ytor med form, och ett svagt toppljus lyser upp resten.

### Volym

1. **3D-grid.** Röken simuleras i en låda bakom skärmen, uppdelad i kubiska celler: 56 till 128 celler på skärmens kortsida beroende på kvalitet, och två tredjedelar så många på djupet. Varje skiva i djupled ligger som en ruta i en stor 2D-textur, så att varje delsteg i lösaren blir ett enda ritanrop. När fönstret ändrar storlek så att gridet måste bytas står röken still tills storleken har stannat, och flyttas sedan över till ett grid som passar det nya fönstret. Röken behåller då sin storlek på skärmen och hamnar i mitten, så att den inte dras ut åt sidorna när ett fönster som har varit smalt blir stort igen, vare sig det sker i ett enda steg eller i flera. Bara när skärmen vrids, så att bredd och höjd ungefär byter plats, formas röken om så att den fyller den vridna skärmen. Blir fönstrets kortsida smalare än 32 pixlar står röken still tills fönstret blir större igen.
2. **Transport.** Täthet, temperatur och färskhet (hur ny röken är) flyttas med strömningen enligt MacCormack-metoden, som behåller skarpare detaljer än en vanlig semi-Lagrange-lösare. Där korrigeringen skjuter över grannvärdena används det vanliga värdet i stället, och där den skjuter under dem används det minsta grannvärdet. Det lyfter ändå tätheten en aning i varje steg, så när ingen pensel är aktiv summeras röken på grafikkortet i varje bildruta och skalas så att den totala mängden aldrig blir större än i bildrutan innan. Röken växer alltså inte av sig själv när du har släppt musknappen. Lådans sidor är öppna, så rök som lämnar den försvinner och ren luft strömmar in.
3. **Krafter.** Het gas stiger (Lyftkraft). Virvelförstärkningen (Virvlar) verkar bara där det finns rök, och ett mjukt turbulensfält (Turbulens) håller röken i rörelse. Penseln är en mjuk sfär i lådans mittplan som värmer gasen, fyller på rök och drar gasen med sig åt det håll du drar.
4. **Tryck.** Luft går inte att pressa ihop. En tryckberäkning med 16 till 30 Jacobi-iterationer per bildruta, beroende på kvalitet, justerar hastigheterna så att lika mycket luft strömmar ut ur varje cell som in i den. Det är den som får röken att välla fram och rulla runt sig själv.
5. **Ljus.** Varje bildruta kopieras tätheten till en 3D-textur. En ljusvolym i halva upplösningen räknar ut hur mycket ljus som når varje punkt från ett ljus i glödens färg vid pekaren och från ett toppljus snett ovanifrån, med skuggor genom röken.
6. **Strålföljning.** En bild i lägre upplösning än skärmen skickar en stråle per pixel genom lådan, en cell i taget. Röken dämpar ljuset bakom sig, sprider ljuset från ljusvolymen mot kameran och glöder där den är het. Bilden skalas sedan mjukt upp till skärmen.
7. **Detaljer finare än gridet.** I stora, svalnade rökmoln förskjuts tätheten med ett brus som följer strömningen, så att detaljerna töjs ut och virvlar med röken. Bruset finns i två omgångar som startar om var 1,2 sekund, förskjutna en halv period, och tonas över i varandra så att det aldrig hinner töjas ut för mycket. Den heta lågan och snabba strålar får inget brus.
8. **Automatisk kvalitet.** Om bildfrekvensen ligger under 50 bilder per sekund i två sekunder byter Volym till ett grövre grid, högst ett steg var tredje sekund och aldrig under Låg. Röken flyttas över till det nya gridet. Rubriken Kvalitet visar då ”sänkt till …” och raden längst ned till vänster visar det grid som körs. Tre sekunder efter ett steg, och tidigast när tre bildrutor har ritats med det nya gridet, kontrolleras att bildfrekvensen har ökat med minst 10 procent jämfört med de två sista sekunderna före steget. Gör den inte det var det inte gridet som begränsade, till exempel när webbläsaren låser sidan till 30 bilder per sekund för att spara ström. Då tas steget tillbaka och Volym sänker inte igen från den nivån på 30 sekunder. Händer det flera gånger i rad från samma nivå fördubblas tiden varje gång, upp till 4 minuter. Det gäller bara den nivån: går Volym upp till en högre nivå som visar sig vara för långsam sänker den därifrån som vanligt. Bildfrekvensen som det ångrade steget inte kunde höja blir ett tak. Så länge taket gäller är bildfrekvensen för låg först under 85 procent av taket och tillräckligt hög från 95 procent av det, så att till exempel 30 bilder per sekund räcker när sidan är låst till 30. Taket gäller lika länge som väntetiden. Sedan prövar Volym att sänka igen, så att den märker att låsningen har släppt även när gridet självt bara hinner med 30 bilder per sekund. Taket släpper tidigare om bildfrekvensen har legat minst 15 procent över det i två sekunder, när fönstret ändrar storlek och när du väljer en kvalitet. Ligger bildfrekvensen på 58 eller mer, eller nära taket, i fem sekunder går Volym upp ett steg igen, men efter en sänkning som hjälpte väntar den först 20 sekunder innan den går tillbaka till nivån den lämnade. Tiden fördubblas för varje ny sänkning från samma nivå, upp till 4 minuter, så att kvaliteten inte pendlar fram och tillbaka. Byter du kvalitet i menyn börjar Volym om från den nivån, och både det och en ny fönsterstorlek nollställer väntetiderna. Partiklar påverkas inte.

Båda lägena avslutas likadant: HDR-bild med glöd, ACES-tonmappning, vinjett och dithering.

## Krav

WebGL2 med tillägget `EXT_color_buffer_float`. Det finns i alla moderna skrivbordswebbläsare och i Safari på iOS 15 och senare.

Volym behöver dessutom kunna rendera till 3D-texturer i flyttalsformat. Om grafikkortet inte klarar det är Volym-knappen avstängd och sidan visar Partiklar.
