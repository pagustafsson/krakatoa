# Krakatoa Smoke

Färgad partikelrök i realtid som följer pekaren när du håller nere musknappen. Inspirerad av partikelrenderaren Krakatoa. Allt ligger i en enda HTML-fil i ren WebGL2, utan bibliotek eller byggsteg.

## Kom igång

Öppna `index.html` i en aktuell version av Chrome, Safari, Edge eller Firefox. Håll nere musknappen och dra. På mobil och surfplatta fungerar det med fingret, även med flera fingrar samtidigt.

Om webbläsaren vägrar köra sidan direkt från disken kan du starta en lokal server:

```bash
python3 -m http.server 8000
```

Gå sedan till `http://localhost:8000`.

## Lägen

- **Partiklar**: miljontals fina partiklar som lyser ihop, med strängar och slöjor av rök.
- **Volym**: fluffig, bollig rök där det som sprutar ut glöder gulgrönt och snabbt åldras till matt, upplyst blått. Puffarna skuggar varandra och lyses upp av glöden.

Byt läge med knappen överst i panelen eller med M. Valt läge sparas till nästa besök. Lägg till `#volym` eller `#partiklar` sist i adressen för att öppna ett visst läge direkt.

## Styrning

| Tangent | Funktion |
| --- | --- |
| Space | Pausa och fortsätt |
| C | Rensa röken |
| H | Visa och dölj panelen |
| A | Slå på och av autopiloten |
| M | Byt mellan Partiklar och Volym |
| 1–5 | Byt palett: Aurora, Nebula, Magma, Spektrum, Aska |

I panelen ställer du in kvalitet (262 000 till 4,2 miljoner partiklar), pensel, flöde, ljus, skuggor och glöd.

## Så fungerar det

1. **Vätskesimulering.** En Navier–Stokes-lösare på grafikkortet, i stil med Stams *Stable Fluids*. Den har virvelförstärkning och turbulens från curl-brus. Penseldrag läggs in som utsmetade Gauss-fläckar längs pekarens väg, så att snabba drag inte lämnar luckor.
2. **Partiklar.** Positionerna lagras i en flyttalstextur och flyttas av hastighetsfältet. Ovanpå det läggs finare curl-brus för tunna strängar. Nya partiklar föds ur en ringbuffert längs penseldraget.
3. **Färg.** Röken föds i en varm färg som skiftar medan du ritar och svalnar sedan mot palettens kalla färg.
4. **Ljus.** En ljuskälla vid pekaren kastar skuggor genom röken med en strålmarsch i skärmrymden. Rökens täthet ger ytor med form, och ett svagt toppljus lyser upp resten.
5. **Volymläget.** En egen population på 6 000–41 000 rökpuffar, beroende på kvalitet, flyttas av samma strömning. Varje puff ritas som en klumpig, belyst sfär, de äldsta först så att den glödande nya röken hamnar överst. En täthetskarta över puffarna ger skuggor mot ljuset vid pekaren. En suddig karta över den unga röken fungerar som ljuskälla och ger blå puffar nära huvudet grönt ljus.
6. **Efterbehandling.** HDR-bild med glöd, ACES-tonmappning, vinjett och dithering.

I Volym styr **Avsvalning** hur snabbt den glödande röken övergår till den åldrade färgen, och **Puffstorlek** hur stora bollarna blir.

## Krav

WebGL2 med tillägget `EXT_color_buffer_float`. Det finns i alla moderna skrivbordswebbläsare och i Safari på iOS 15 och senare.
