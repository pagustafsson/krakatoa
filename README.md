# Krakatoa Smoke

Färgad partikelrök i realtid som följer pekaren när du håller nere musknappen. Inspirerad av partikelrenderaren Krakatoa. Allt ligger i en enda HTML-fil i ren WebGL2, utan bibliotek eller byggsteg.

## Kom igång

Öppna `index.html` i en aktuell version av Chrome, Safari, Edge eller Firefox. Håll nere musknappen och dra. På mobil och surfplatta fungerar det med fingret, även med flera fingrar samtidigt.

Om webbläsaren vägrar köra sidan direkt från disken kan du starta en lokal server:

```bash
python3 -m http.server 8000
```

Gå sedan till `http://localhost:8000`.

## Styrning

| Tangent | Funktion |
| --- | --- |
| Space | Pausa och fortsätt |
| C | Rensa röken |
| H | Visa och dölj panelen |
| A | Slå på och av autopiloten |
| 1–5 | Byt palett: Aurora, Nebula, Magma, Spektrum, Aska |

I panelen ställer du in kvalitet (262 000 till 4,2 miljoner partiklar), pensel, flöde, ljus, skuggor och glöd.

## Så fungerar det

1. **Vätskesimulering.** En Navier–Stokes-lösare på grafikkortet, i stil med Stams *Stable Fluids*. Den har virvelförstärkning och turbulens från curl-brus. Penseldrag läggs in som utsmetade Gauss-fläckar längs pekarens väg, så att snabba drag inte lämnar luckor.
2. **Partiklar.** Positionerna lagras i en flyttalstextur och flyttas av hastighetsfältet. Ovanpå det läggs finare curl-brus för tunna strängar. Nya partiklar föds ur en ringbuffert längs penseldraget.
3. **Färg.** Röken föds i en varm färg som skiftar medan du ritar och svalnar sedan mot paletten kalla färg.
4. **Ljus.** En ljuskälla vid pekaren kastar skuggor genom röken med en strålmarsch i skärmrymden. Rökens täthet ger ytor med form, och ett svagt toppljus lyser upp resten.
5. **Efterbehandling.** HDR-bild med glöd, ACES-tonmappning, vinjett och dithering.

## Krav

WebGL2 med tillägget `EXT_color_buffer_float`. Det finns i alla moderna skrivbordswebbläsare och i Safari på iOS 15 och senare.
