# Vaihe 3: Service-kerros, Repository, Result Pattern ja API-dokumentaatio — Teoriakysymykset

Vastaa alla oleviin kysymyksiin omin sanoin. Kirjoita vastauksesi kysymysten alle.

> **Vinkki:** Jos jokin kysymys tuntuu vaikealta, palaa lukemaan teoriamateriaalit:
> - [Service-kerros ja DI](https://github.com/xamk-mire/Xamk-wiki/blob/main/C%23/fin/04-Advanced/WebAPI/Services-and-DI.md)
> - [Repository Pattern](https://github.com/xamk-mire/Xamk-wiki/blob/main/C%23/fin/04-Advanced/Patterns/Repository-Pattern.md)
> - [Result Pattern](https://github.com/xamk-mire/Xamk-wiki/blob/main/C%23/fin/04-Advanced/Patterns/Result-Pattern.md)

---

## Osa 1: Service-kerros

### Kysymys 1: Fat Controller -ongelma

Miksi on ongelma jos controller sisältää kaiken logiikan (tietokantakyselyt, muunnokset, validoinnin)? Anna vähintään kaksi konkreettista haittaa.

**Vastaus:**
Koodi voi kasvaa todella pitkäksi ja vaikeuttaa testailua.

---

### Kysymys 2: Vastuunjako

Miten vastuut jakautuvat controller:n, service:n ja repository:n välillä tässä harjoituksessa? Kirjoita lyhyt kuvaus kunkin kerroksen tehtävästä.

**Controller vastaa:**
Controllerin tehtävä on määrittää mitä HTTP-pyynnöillä halutaan tehdä.

**Service vastaa:**
Service määrittää mitä itse sovellus tekee.

**Repository vastaa:**
Repository liittyy datan hakemiseen.

---

### Kysymys 3: DTO-muunnokset servicessä

Miksi DTO ↔ Entity -muunnokset kuuluvat serviceen eikä controlleriin? Mitä hyötyä siitä on, että controller ei tunne `Product`-entiteettiä lainkaan?

**Vastaus:**
Nämä muunnokset kuuluvat serviceen, sillä se sisältää liiketoimintalogiikan. Controller ei tarvitse tietää, miten dataa tallennetaan tai haetaan, vaan se voi keskittyä HTTP-pyyntöjen käsittelyyn.

---

## Osa 2: Interface ja Dependency Injection

### Kysymys 4: Interface vs. konkreettinen luokka

Miksi controller injektoi `IProductService`-interfacen eikä suoraan `ProductService`-luokkaa? Mitä hyötyä tästä on?

**Vastaus:**
Controller täten riippuu vain rajapinnasta. Hyöty tästä näkyy siinä, että voimme tehdä testejä mock-toteutuksella ja vaihtaa toteutusta ilman, että controllerin koodia tarvitsee muuttaa.

---

### Kysymys 5: DI-elinkaaret

Selitä ero näiden kolmen elinkaaren välillä ja anna esimerkki milloin kutakin käytetään:

- **AddScoped:** Luodaan kerran per HTTP-pyyntö. Esim. tietokannat.
- **AddSingleton:** Luodaan kerran per sovellus. Esim. konfiguraatioasetukset.
- **AddTransient:** Luodaan joka kerta kun sitä pyydetään. Esim. kevyet palvelut. (Laskurit)

Miksi `AddScoped` on oikea valinta `ProductService`:lle?

`ProductService` käyttää `DbContext`:ia, joten `AddScoped` on oikea valinta.

---

### Kysymys 6: DI-kontti

Selitä omin sanoin mitä DI-kontti tekee kun HTTP-pyyntö saapuu ja `ProductsController` tarvitsee `IProductService`:ä. Mitä tapahtuu vaihe vaiheelta?

**Vastaus:**
Ensin DI-kontti löytää `IProductService`:n. Se luo instanssin `ProductService`:sta ja injektoi sen `ProductsController`:iin.

---

### Kysymys 7: Rekisteröinnin unohtaminen

Mitä tapahtuu jos unohdat rekisteröidä `IProductService`:n `Program.cs`:ssä? Milloin virhe ilmenee ja miltä se näyttää?

**Vastaus:**
Virhe ilmenee kun HTTP-pyyntö saapuu ja `ProductsController`:iin. 

Virhe näyttää:
"System.InvalidOperationException: Unable to resolve service for type 'ProductApi.Services.IProductService' while attempting to activate 'ProductApi.Controllers.ProductsController'."

---

## Osa 3: Repository-kerros

### Kysymys 8: Miksi repository?

`ProductService` käytti aluksi `AppDbContext`:ia suoraan. Miksi se refaktoroitiin käyttämään `IProductRepository`:a? Anna vähintään kaksi syytä.

**Vastaus:**
Muuten Servicen yksikkötestaus vaatii oikean tietokannan ja Service tuntee EF Core:n ja tietokantateknologian.

---

### Kysymys 9: Service vs. Repository

Mikä on `IProductService`:n ja `IProductRepository`:n välinen ero? Mitä tietotyyppejä kumpikin käsittelee (DTO vai Entity)?

**IProductService:** Käsittelee DTO:ita ja sisältää liiketoimintalogiikan.

**IProductRepository:** Käsittelee Entity:itä ja vastaa datan hakemisesta ja tallentamisesta.


---

### Kysymys 10: Controllerin muuttumattomuus

Kun Vaihe 7:ssä lisättiin repository-kerros, `ProductsController` ei muuttunut lainkaan. Miksi? Mitä tämä kertoo rajapintojen (interface) hyödystä?

**Vastaus:** Tämä kertoo rajapintojen hyödystä, sillä controller ei tarvitse tietää, miten dataa haetaan tai tallennetaan. `ProductsController` ei muuttunut, koska se riippuu vain `IProductService`:sta, ei sen toteutuksesta.

---

## Osa 4: Exception-käsittely ja lokitus

### Kysymys 11: ILogger

Mikä on `ILogger` ja miksi sitä tarvitaan? Mistä lokit näkee kehitysympäristössä?

**Vastaus:**
`ILogger` on lokitusrajapinta, jonka avulla voidaan lokittaa sovelluksen tietoja ja tapahtumia. Sen voi esim nähdä konsolissa.

---

### Kysymys 12: Odotetut vs. odottamattomat virheet

Selitä ero "odotetun" ja "odottamattoman" virheen välillä. Anna esimerkki kummastakin ja kerro miten ne käsitellään eri tavalla servicessä.

**Odotettu virhe (esimerkki + käsittely):** Esim haetaan tuotetta id:llä jota ei ole. Käsitellään palauttamalla 404 Not Found.

**Odottamaton virhe (esimerkki + käsittely):** Esim tietokantayhteys epäonnistuu. Käsitellään palauttamalla 500 Internal Server Error.


---

## Osa 5: Result Pattern

### Kysymys 13: Miksi null ja bool eivät riitä?

Alla on kaksi esimerkkiä. Selitä miksi ensimmäinen tapa on ongelmallinen ja miten toinen ratkaisee ongelman:

```csharp
// Tapa 1: null
ProductResponse? product = await _service.GetByIdAsync(id);
if (product == null)
    return NotFound();

// Tapa 2: Result
Result<ProductResponse> result = await _service.GetByIdAsync(id);
if (result.IsFailure)
    return NotFound(new { error = result.Error });
```

**Vastaus:**
Ensimmäisessä ei kerrota, miksi product on null. Toinen ratkaisee ongelman Result Patternin avulla, jolloin virheen syyn saa selville.

---

### Kysymys 14: Result.Success vs. Result.Failure

Miten `Result Pattern` muutti virheiden käsittelyä servicessä? Vertaa Vaihe 8:n `throw;`-tapaa Vaihe 9:n `Result.Failure`-tapaan: mitä eroa niillä on asiakkaan (API:n kutsuja) näkökulmasta?

**Vastaus:**
Asiakas näkee Vaihe 8:ssa vain 500 -virheen, eikä hän saa tietoa siitä, mikä meni pieleen. Vaihe 9:ssä asiakas saa selkeän virheilmoituksen, joka kertoo miksi pyyntö epäonnistui.

---

## Osa 6: API-dokumentaatio

### Kysymys 15: IActionResult vs. ActionResult\<T\>

Miksi `ActionResult<ProductResponse>` on parempi kuin `IActionResult`? Anna vähintään kaksi syytä.

**Vastaus:**
`ActionResult<ProductResponse>`:sta näkee suoraan, mitä se palauttaa. Lisäksi se mahdollistaa erilaisten HTTP-vastaustyyppien palauttamisen. `IActionResult` ei kerro tarkasti, mitä palautetaan.

---

### Kysymys 16: ProducesResponseType

Mitä `[ProducesResponseType]`-attribuutti tekee? Miten se näkyy Swagger UI:ssa?

**Vastaus:**
`[ProducesResponseType]`-attribuutti määrittää, mitä HTTP-statuskoodeja ja vastaustyyppejä endpoint voi palauttaa. Swagger UI:ssa näkyy täten, mitä statuskoodeja ja vastaustyyppejä voidaan odottaa kullekin endpointille.

---

### Kysymys 18: Refaktorointi

Sovelluksen toiminnallisuus pysyi täysin samana koko harjoituksen ajan — samat endpointit, samat vastaukset. Mitä refaktorointi tarkoittaa ja miksi se kannattaa, vaikka käyttäjä ei huomaa eroa?

**Vastaus:**
Refaktorointi tarkoittaa, että koodin toiminnallisuus on sama, mutta rakenne paranee. Se on kannattavaa, sillä se helpottaa koodin ylläpitoa, parantaa luettavuutta ja mahdollistaa uusien ominaisuuksien lisäämisen ilman suuria muutoksia olemassa olevaan koodiin.

---
