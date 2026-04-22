# Kysymykset — Osa 1: Lokaali kehitys

Vastaa kysymyksiin omin sanoin. Lyhyet, selkeät vastaukset riittävät — tarkoitus on osoittaa, että olet ymmärtänyt konseptit.

---

## Clean Architecture

**1.** Selitä omin sanoin: mitä tarkoittaa, että `UploadPhotoUseCase` "ei tiedä" tallennetaanko kuva paikalliselle levylle vai Azureen? Näytä koodirivit, jotka osoittavat tämän.

> Vastauksesi: UploadPhotoUseCase käyttää vain rajapintaa IStorageService eikä mitään paikallisen levyn tai Azuren SDK-koodia, joten käyttötapaus ei tiedä toteutuksesta. 

    private readonly IStorageService _storageService;
    
        imageUrl = await _storageService.UploadAsync(
            request.FileStream, request.FileName, request.ContentType, request.AlbumId);


---

**2.** Miksi `IStorageService`-rajapinta on määritelty `GalleryApi.Domain`-kerroksessa, mutta `LocalStorageService` on `GalleryApi.Infrastructure`-kerroksessa? Mitä hyötyä tästä jaosta on? 

> Vastauksesi: Rajapinta on Domainissa, koska se kuvaa liiketoiminnan tarvitsemaa sopimusta, mutta LocalStorageService on Infrastructuressa, koska se on tekninen toteutus. Toteutuksen voi vaihtaa ilman muutoksia sovelluslogiikkaan.

---

**3.** Testit käyttävät `Mock<IAlbumRepository>`. Mitä mock-objekti tarkoittaa, ja miksi Clean Architecture tekee tämän testaustavan mahdolliseksi?

> Vastauksesi: Mock-objekti on testitupla, joka jäljittelee riippuvuuden toimintaa ilman oikeaa tietokantaa tai ulkoista palvelua. Clean Architecture mahdollistaa tämän, koska käyttötapaukset riippuvat rajapinnoista eikä toteutuksista.

---

## Salaisuuksien hallinta

**4.** Kovakoodattu API-avain on ongelma, vaikka repositorio olisi yksityinen. Selitä kaksi eri syytä miksi.

> Vastauksesi: Yksityisessäkin repositoriossa avain voi vuotaa esim. väärien käyttöoikeuksien, varmuuskopioiden tai forkkausten kautta. Avain voi myös helposti päätyä lokiin, CI-ajoon tai kehittäjän koneelle, jolloin hallinta ja auditointi heikkenee.

---

**5.** Riittääkö se, että poistat kovakoodatun avaimen myöhemmässä commitissa? Perustele vastauksesi.

> Vastauksesi: Ei, koska avain jää git-historiaan vanhoihin committeihin vaikka se poistettaisiin uusimmasta versiosta. Avain täytyy siis vaihtaa ja siirtää heti salaisuushallintaan.

---

**6.** Minne User Secrets tallennetaan käyttöjärjestelmässä? (Mainitse sekä Windows- että Linux/macOS-polut.) Miksi tämä sijainti on turvallinen?

> Vastauksesi: Windowsissa User Secrets on polussa %APPDATA%/Microsoft/UserSecrets/jne ja Linux/macOS ~/.microsoft/usersecrets/jne. Sijainti on turvallisempi, koska tiedosto on käyttäjäkohtainen eikä kuulu projektikansioon tai versiohallintaan.

---

## Options Pattern ja konfiguraatio

**7.** Mitä hyötyä on `IOptions<ModerationServiceOptions>`:n käyttämisestä verrattuna siihen, että luetaan arvo suoraan `IConfiguration`-rajapinnalta (`configuration["ModerationService:ApiKey"]`)?

> Vastauksesi: IOptions antaa tyypitetyn konfiguraation, jolloin saat IntelliSensen ja vähemmän kirjoitusvirheitä kuin merkkijonoavaimilla luettaessa. Se myös helpottaa testausta, koska testissä voi syöttää suoraan options-olioon.

---

**8.** ASP.NET Core lukee konfiguraation useista lähteistä prioriteettijärjestyksessä. Listaa lähteet korkeimmasta matalimpaan ja selitä, mikä arvo lopulta käytetään, kun sama avain on sekä `appsettings.json`:ssa että User Secretsissä.

> Vastauksesi: Prioriteettijärjestys korkeimmasta alimpaan. User Secrets, appsettings.Development.json, appsettings.json. Jos sama avain on sekä appsettings että User Secrets, User Secrets arvo voittaa.

---

**9.** `DependencyInjection.cs`:ssä valitaan tallennustoteutus näin:

```csharp
var provider = configuration["Storage:Provider"] ?? "local";
if (provider == "azure")
    services.AddScoped<IStorageService, AzureBlobStorageService>();
else
    services.AddScoped<IStorageService, LocalStorageService>();
```

Miksi käytetään konfiguraatioarvoa `env.IsDevelopment()`-tarkistuksen sijaan? Mitä haittaa olisi `if (env.IsDevelopment()) { käytä lokaalia }`-lähestymistavassa?

> Vastauksesi: Konfiguraatioarvo erottaa selvästi tallennusvalinnan ympäristöstä, joten voit käyttää esim. Azure-tallennusta myös kehityksessä tai lokaalitallennusta testiympäristössä. 

---

## Tiedostotallennus

**10.** Kun lataat kuvan, `imageUrl`-kentän arvo on `/uploads/abc123-..../photo.jpg`. Miten tähän URL:iin pääsee selaimella? Mihin koodiin tämä perustuu?

> Vastauksesi: Selainosoitteeseen laitetaan palvelimen perusosoite + imageUrl, esim. https://localhost:1234/uploads/albumId/tiedosto.png. Tämä toimii, koska staattiset tiedostot on otettu käyttöön program.cs tiedostossa. 

app.UseStaticFiles();

---

**11.** Mitä tapahtuu jos yrität ladata tiedoston jonka MIME-tyyppi on `application/pdf`? Missä tiedostossa ja millä koodirivillä tämä käyttäytyminen on määritelty?

> Vastauksesi: application/pdf hylätään BadRequest-virheenä, koska MIME-tyyppi ei ole sallituissa tyypeissä ehto on määritelty UploadPhotoUseCase.cs tiedostossa.


    if (!AllowedContentTypes.Contains(request.ContentType))
        return Result<PhotoDto>.Failure(
            $"Tiedostotyyppi '{request.ContentType}' ei ole sallittu. " +
            $"Sallitut tyypit: {string.Join(", ", AllowedContentTypes)}");
---

**12.** `DeletePhotoUseCase` poistaa tiedoston kutsumalla `_storageService.DeleteAsync(photo.FileName, photo.AlbumId)` — ei `photo.ImageUrl`:lla. Miksi?

> Vastauksesi: Poistossa käytetään FileName + AlbumId, koska ne vastaavat tallennuspalvelun sisäistä polkuavainta myös silloin kun URL ei suoraan vastaa fyysistä tiedostonimeä. Tämä tulee ilmi DeletePhotoUseCase.cs Tiedostossa.

await _storageService.DeleteAsync(photo.FileName, photo.AlbumId);
