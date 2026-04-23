# Kysymykset — Osa 3: Key Vault ja Infrastructure as Code

Vastaa kysymyksiin omin sanoin. Lyhyet, selkeät vastaukset riittävät.

---

## Key Vault

**1.** Miksi `ModerationService:ApiKey` tallennettiin Key Vaultiin eikä Application Settingsiin? Mitä lisäarvoa Key Vault tuo Application Settingsiin verrattuna?

> Vastauksesi: Key Vault tallentaa salaisuudet salattuina ja erillään sovelluskonfiguraatiosta, joten ne eivät näy esim. portalin logeissa tai ympäristömuuttujussa. Application Settings tallentaa arvot selkotekstinä

---

**2.** Key Vault -salaisuuden nimi on `ModerationService--ApiKey` (kaksi väliviivaa), mutta koodissa se luetaan `configuration["ModerationService:ApiKey"]` (kaksoispiste). Miksi käytetään `--`?

> Vastauksesi:Key Vaultissa ei voi käyttää kaksoispistettä salaisuuden nimessä, joten .NET-konfiguraation hierarkiaerotin ( : ) korvataan kahdella väliviivalla ( -- ), jonka konfiguraatioprovider muuttaa automaattisesti takaisin kaksoispisteeksi 

---

**3.** `Program.cs`:ssä Key Vault lisätään konfiguraatiolähteeksi `if (!string.IsNullOrEmpty(keyVaultUrl))`-ehdolla. Miksi tämä ehto on tärkeä? Mitä tapahtuisi ilman sitä?

> Vastauksesi: Ilman ehtoa koodi yrittäisi rekistöröidä Key Vault -providerin tyhjällä URL:lla, mikä kaataisi sovelluksen käynnistyksen poikkeukseen. Ehto mahdollistaa, että sovellus toimii myös lokaalisti ilman Key Vaultia.

---

**4.** Kun sovellus on käynnissä Azuressa, konfiguraation prioriteettijärjestys on: Key Vault → Application Settings → `appsettings.json`. Selitä millä arvolla `ModerationService:ApiKey` lopulta ladataan — ja käy läpi jokainen askel siitä, miten arvo päätyy sovelluksen `IOptions<ModerationServiceOptions>`:iin.

> Vastauksesi: Arvo ladataan Key Vaultista, koska se on korkein prioriteetti. .NET:n konfiguraatiojärjestelmä yhdistää kaikki lähteet ja Key Vault ylikirjoittaa muut. AddAzureKeyVault() rekistöröi salaisuuden ModerationService:ApiKey-avaimelle, jolloin services.Configure sitoo arvon  ModerationServiceOptions.ApiKey-propertyyn, jolloin luokka injektoidaan IOptions<ModerationServiceOptions> -rajapinnan kautta.

---

**5.** Mitä eroa on `Key Vault Secrets User` ja `Key Vault Secrets Officer` -roolien välillä? Miksi annettiin nimenomaan `Secrets User`?

> Vastauksesi: Secrets User voi vain lukea salaisuuksia, Secrets Officer voi myös luoda, päivittää ja poistaa niitä. Sovellus tarvitsee vain lukuoikeuden, joten Secrets User noudattaa least privilege -periaatetta.

---

## Infrastructure as Code (Bicep)

**6.** Bicep-templatessa RBAC-roolimääritykset tehdään suoraan (`storageBlobRole`, `keyVaultSecretsRole`). Mitä etua tällä on verrattuna siihen, että ajat erilliset `az role assignment create` -komennot käsin?

> Vastauksesi: Bicep-templatessa roolimääräykset ovat versiohallinnoituja ja idempotenteja -sama template voidaan ajaa uudelleen ilman duplikaatteja tai inhimillisiä virheitä, toisin kuin manuaaliset komennot jotka pitää muistaa ajaa erikseen oikeassa järjestyksessä.

---

**7.** Bicep-parametritiedostossa `main.bicepparam` on `param moderationApiKey = ''` — arvo jätetään tyhjäksi. Miksi? Miten oikea arvo annetaan?

> Vastauksesi: Arvo jätetään tyhjäksi, koska API-avainta ei haluta tallentaa versiohallintaan. Oikea arvo annetaan deployment-hetkellä esim. az deployment group create -- parameters moderation ApiKey=<arvo> tai CI/CD-pipelinen salaisuusmuuttujana.

---

**8.** Bicep-templatessa `webApp`-resurssin `identity`-lohkossa on `type: 'SystemAssigned'`. Mitä tämä tekee, ja mitä manuaalista komentoa se korvaa?

> Vastauksesi: SystemAssigned luo Web Appille Managed Identityn automaattisesti Azure Entra ID:hen, korvaten manuaalisen az webapp identity assign- komennon. Identiteetti on sidottu resurssiin ja poistuu sen mukana.

---

**9.** RBAC-roolimäärityksen nimi generoidaan `guid()`-funktiolla:

```bicep
name: guid(storageAccount.id, webApp.identity.principalId, 'StorageBlobDataContributor')
```

Miksi nimi generoidaan näin eikä esimerkiksi kovakoodatulla merkkijonolla? Mitä tapahtuisi jos nimi olisi sama kaikissa deploymenteissa?

> Vastauksesi: guid() generoi deterministisen mutta uniikin nimen syöteparametrien perusteella, joten sama roolimääritys ei saa duplikaattia uudelleendeploymentissa — kovakoodattu nimi aiheuttaisi konfliktin tai ylikirjoittaisi väärän roolimäärityksen eri resursseilla.


---

**10.** Olet nyt rakentanut saman infrastruktuurin kahdella tavalla: manuaalisesti (Osat 2 & 3) ja Bicepillä (Osa 3). Kuvaile konkreettisesti yksi tilanne, jossa IaC-lähestymistapa on selvästi manuaalista parempi. Kuvaile myös tilanne, jossa manuaalinen tapa riittää.

> Vastauksesi:IaC on selvästi parempi kun pitää pystyttää sama ympäristö uudelleen eri tiimeille tai staging/prod-ympäristöihin — kaikki resurssit ja oikeudet syntyvät yhdellä komennolla identtisinä. Manuaalinen tapa riittää kun kokeilet kertaluonteisesti uutta palvelua kehitystilauksella etkä aio toistaa konfiguraatiota.
