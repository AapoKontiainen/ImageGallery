# Kysymykset — Osa 2: Azure-julkaisu

Vastaa kysymyksiin omin sanoin. Lyhyet, selkeät vastaukset riittävät.

---

## Azure Blob Storage

**1.** Mitä eroa on `LocalStorageService.UploadAsync`:n ja `AzureBlobStorageService.UploadAsync`:n palauttamilla URL-arvoilla? Miksi ne eroavat?

> Vastauksesi: LocalStorageService palauttaa suhteellisen URL:n, kun taas AzureBlobStorageService palauttaa täyden Blob-URL:n. Ne eroavat, koska Local palvelee tiedostot Appin omasta wwwroot-kansiosta ja toinen Azuren Blob Storagesta.

---

**2.** `AzureBlobStorageService` luo `BlobServiceClient`:n käyttäen `DefaultAzureCredential()` eikä yhteysmerkkijonoa. Mitä etua tästä on? Mitä `DefaultAzureCredential` tekee eri ympäristöissä?

> Vastauksesi: Default poistaa tarpeen kovakoodatuille avaimille/yhteysmerkkijonoille ja tekee samasta koodista toimivan sekä lokaalisti että Azuressa. Se käyttää ympäristön mukaan esim Azure CLI-kirjautumista lokaalisti ja Managed Identityä App Servicessä.

---

**3.** Blob Container luodaan `--public-access blob` -asetuksella. Mitä tämä tarkoittaa: mitä pystyy tekemään ilman tunnistautumista, ja mikä vaatii Managed Identityn?

> Vastauksesi: --public-access blob tarkoittaa, että blobit voi lukea suoraan URL:lla ilman kirjautumista. Kirjoitus, poisto ja muut muokkaukset vaativat edelleen tunnistautumisen (esim. Managed Identity + RBAC)

---

## Application Settings

**4.** Application Settings ylikirjoittavat `appsettings.json`:n arvot. Selitä tämä mekanismi: miten se toimii ja miksi se on hyödyllistä eri ympäristöjä varten?

> Vastauksesi: ASP.NET Core yhdistää konfiguraation useista lähteistä, ja App Servicen Application Settings yliajaa AppSettings.json-arvot samalle avaimelle. Tämä on hyödyllistä, koska voit käyttää eru arvoja dev/test/prod ympäristöissä ilman koodimuutoksia.

---

**5.** Application Settingsissa käytetään `Storage__Provider` (kaksi alaviivaa), mutta koodissa luetaan `configuration["Storage:Provider"]` (kaksoispiste). Miksi?

> Vastauksesi: Azure App Settings ei käytä kaksoispistettä avainnimissä, joten hierarkia kirjoitetaan muodossa __, ASP.NET Core muuntaa automaattisesti Storage__Provider = Storage:Provider

---

**6.** Mitkä konfiguraatioarvot soveltuvat Application Settingsiin, ja mitkä eivät? Anna esimerkki kummastakin tässä tehtävässä.

> Vastauksesi: Application Settingsiin sopivat, ei-salaiset ympäristökohtaiset arvot, kuten Storage__Provider=azure. Itse salaisuudet, kuten API-avaimet, eivät kuulu sinne vaan Key Vaultiin. Esimerkkinä tässä tehtävässä esim ModerationService:ApiKey.

---

## Managed Identity ja RBAC

**7.** Selitä omin sanoin: mitä tarkoittaa "System-assigned Managed Identity"? Mitä tapahtuu tälle identiteetille, jos App Service poistetaan?

> Vastauksesi: System-assigned Managed Identity on App Servicen oma automaattisesti luotu Azure-identiteetti, jota sovellus käyttää palveluihin kirjautumiseen ilman salasanoja. Jos App Service poistetaan, myös tämä identiteetti poistuu automaattisesti.

---

**8.** App Servicelle annettiin `Storage Blob Data Contributor` -rooli Storage Accountin tasolle — ei koko subscriptionin tasolle. Miksi tämä on parempi tapa? Mikä periaate tähän liittyy?

> Vastauksesi: Storage Account- tasoinen roolitus rajaa oikeudet vain tarvittuun resurssiin eikä koko subscriptioniin. Tämä noudattaa least privilege - periaatetta (minimioikeudet), mikä parantaa tietoturvaa.

---


