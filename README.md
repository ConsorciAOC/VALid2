# VALid 2.0

## **DI – VALId 2.0 (Validador d'identitats 2.0)**

**Índex**

- [1 Introducció 1](#1)
    * [1.1 Registre de l'aplicació client](#1.1)
    * [1.2 Entorns ](#2)
- [2 Integració de l'aplicació client](#2)
    * [2.1 Construcció de la URL](#2.1)
    * [2.2 Tractament de la resposta](#2.2)
        * [2.2.1 Generació d'un nou accés token a partir del refresh Token.](#2.2.1)
    * [2.3 Revocació d'un token d'accés](#2.3)
    * [2.4 Logout programàtic ](#2.4)
- [3 Serveis de dades de suport a les aplicacions](#3)
    * [3.1 Dades de l'usuari validat ](#3.1)
    * [3.2 Evidències d'autenticació ](#3.2)
- [4 Operacions de signatura ordinària ](#4)
    * [4.1 Signatura ordinària a partir de l'accés token ](#4.1)
    * [4.2 Signatura ordinària a partir de l'accés token i acció d'autenticació addicional ](#4.2)
    * [4.3 Consideracions sobre el resum criptogràfic](#4.3)
    * [4.4 Missatge de signatura ordinària](#4.4)
- [Annex - evidències del procés de validació ](#5)
    * [Evidències generades en la consulta a la Base de Dades de la Seu ](#6)
    * [Evidències generades en la validació amb certificat digital](#7)
    * [Evidències generades en la validació amb contrasenya al mòbil (SMS)](#8)
    * [Evidències generades en la validació amb Cl@ve](#10)

# 1 Introducció <a name="1"></a>

Aquest document detalla el procediment a seguir per a integrar-se mitjançant el protocol OAuth 2.0 amb el Validador d'Identitats del Consorci AOC (en endavant VALId).

## 1.1 Registre de l'aplicació client <a name="1.1"></a>

Abans de poder realitzar la integració amb VALId és necessari fer el registre de la aplicació client per tal que el validador la reconegui com a un client autoritzat.

Per tal de registrar una aplicació client cal posar-se en contacte amb el Consorci AOC per tal de proporcionar les següents dades:

| _Dada_ | _Descripció_ |
| --- | --- |
| Nom curt | Nom curt de la aplicació client que es vol integrar amb VALId (p.e. "eNOTUM"). |
| Descripció | Descripció de la aplicació client (p.e. "Servei de notificacions electròniques del Consorci AOC"). |
| Redirect URI | URL a la qual VALId haurà d'enviar el resultat de la autenticació (p.e. https://enotum.aoc.cat/code).  Es pot passar una llista amb més d'una URL de redirecció, tot i que no és el cas més habitual. |
| Mètodes d'autenticació | Mètodes de validació que es vol emprar per a les autenticacions a la aplicació web client.Actualment es suporten els següents mecanismes:<br>- idCAT Mòbil (contrasenya SMS al mòbil).<br>- Certificat digital.<br>- [Cl@ve](#cl@ve)  del Ministerio de Hacienda y Administraciones Públicas. |
| Mètodes de signatura ordinària | Mètodes de validació que es vol emprar per a les signatures ordinàries que requereixen una autenticació addicional per part de l'usuari.Actualment es suporten els següents mecanismes:<br>- un senzill CAPTCHA. |
| Recursos de personalització | CSSs i logos amb el que es presentaran les pantalles de validació d'identitats. |
| IPs de les aplicacions clients | IPs de les aplicacions clients per tal d'habilitar l'accés als serveis de dades REST i els contexts de bescanvi de _tokens_ OAuth. |

<br> - Cl@ve del Ministerio de Hacienda y Administraciones Públicas: [http://clave.gob.es](http://clave.gob.es/).<a name="cl@ve"></a>

Un cop la aplicació client hagi estat registrada, es proporcionarà als desenvolupadors de la integració una sèrie de codis que s'hauran d'usar en la invocació a VALId. Aquestes dades són:

- **client-id**: identificador únic de la aplicació dins de l'àmbit del VALId.
- **client-secret**: secret compartit que s'haurà d'usar a la operació de negociació del token d'accés.

Amb aquests codis els integradors podran configurar qualsevol dels clients OAuth que hi ha disponibles (existeixen implementacions per a múltiples llenguatges com Java, .NET o PHP) per protegir les seves aplicacions delegant la autenticació al VALId.

Un cop el client OAuth estigui configurat és molt possible que sigui necessari fer uns ajustos la lògica de control de la sessió d'usuari per aconseguir la aplicació client funcioni segons aquest nou model d'autenticació.

## 1.2 Entorns <a name="1.2"></a>

Es disposa de dos entorns:

- Pre-producció (entorn per realitzar integracions i proves): https://valid-pre.aoc.cat/o/oauth2
- Producció: https://valid.aoc.cat/o/oauth2

---

![image](https://user-images.githubusercontent.com/32306731/137281698-9dfc2044-94f7-487f-a7d6-9a4e0707feb3.png) Recordeu que els entorns de DEV i PRE estan destinats únicament per a l'ús de proves i que d'acord amb el Supervisor Europeo de Protecció de Dades (EDPS) no es poden fer servir dades personals reals. Per això, als entorns de DEV i PRE, VALId es recolza en els serveis de preproducció que cada mecanisme d'identificació integrat a VALId ofereix.

<br>• idCATMòbil: VALId es recolza en l'entorn de preproducció d'idCATMòbil (Base de Dades de la Seu-e de la Generalitat de Catalunya de preproducció). Podeu enregistrar usuaris de a l'entorn de preproducció per realitzar les vostres proves a l'autoregistre de l'idCATMòbil (https://idcatmobil-pre.seu.cat, fora de l'abast de VALId).

<br>• Cl@ve: VALId es recolza en el Clave de l'entorn de Servicios Estables (preproducció) de manera que els usuaris que s'identificaran són els validats per aquest entorn (fora de l'abast de VALId).

<br>• Certificat: VALId es recolza en la plataforma de validació de certificats PSIS de preproducció de manera que també es suporten certificats emesos amb la jerarquia de proves de CATCert, Firmaprofesional i ANCERT.

---

![image](https://user-images.githubusercontent.com/32306731/137281698-9dfc2044-94f7-487f-a7d6-9a4e0707feb3.png) Us suggerim que a l'hora de realitzar la vostra integració amb VALId tingueu present les recomanacions recollides al document [La privacitat des del disseny i la privacitat per defecte - Guia per a desenvolupadors][APDCAT] de l'APDCAT.

[APDCAT]:https://apdcat.gencat.cat/ca/documentacio/guies_basiques/Guies-apdcat/la-privacitat-des-del-disseny-i-la-privacitat-per-defecte.-guia-per-a-desenvolupadors/

---
# 2 Integració de l'aplicació client <a name="2"></a>

VALId, basat en OAuth 2.0, permet que aplicacions desenvolupades en múltiples llenguatges de programació i tecnologies es puguin integrar fàcilment.

A continuació es descriuen les diferents parts implicades en el procés d'autenticació i com hi intervenen.

- _Servidor d'autenticació / VALId_: és l'encarregat d'identificar l'usuari. En el cas del VALId només es realitza l'autenticació, ja que no es gestiona cap recurs de l'usuari.

- _Aplicació client_: és la aplicació que es recolza en VALId per a autenticar els seus usuaris. D'aquesta manera s'estalvia mantenir una base de dades d'usuaris o actualitzar els mecanismes d'autenticació.
- _Usuari_

![1](Captures/1.png)

La seqüència d'autorització s'inicia quan l'aplicació web que s'integra realitza una redirecció cap a la URL del punt d'autenticació del VALId. Aquesta URL inclou una sèrie de paràmetres que indiquen el tipus d'accés que es vol sol·licitar. VALId realitza l'autenticació de l'usuari així com la seva sessió OAuth amb el navegador. El resultat d'aquest procés és un codi d'autorització, que és retornat per VALId cap a la aplicació web a mode de paràmetre dins d'una URL.

Un cop rebut el codi d'autorització, la aplicació web pot bescanviar aquest (junt amb el seu identificador de client i secret compartit) per un codi d'accés (anomenat _access token_) i, en alguns casos, un codi de refresc (_refresh token_).

Un cop obtingut aquest codi d'accés, l'aplicació pot donar l'usuari per autenticat i també el pot usar per consumir altres serveis de dades del Consorci AOC.

Si la aplicació ha demanat un codi de refresc durant la negociació del codi d'accés, llavors el podrà usar per obtenir nous codis d'accés en qualsevol moment. Això s'anomena accés _offline_ ja que en aquest cas no és necessari que un usuari intervingui des del seu navegador per a obtenir un nou codi d'accés.

En aquest cas, la aplicació web envia una sol·licitud de token a VALId, rep un codi d'autorització i, seguidament, bescanvia aquest codi d'autorització per un nou codi d'accés amb el qual pot seguir consumint els serveis de dades de VALId.

## 2.1 Construcció de la URL <a name="2.1"></a>

La URL que s'usa per realitzar la autenticació és: https://valid-pre.aoc.cat/o/oauth2/auth

Aquesta URL és accessible únicament per protocol segur _https_i tots els processos d'autenticació s'iniciaran accedint a aquesta, tot passant una sèrie de paràmetres dins de la _query string_ que enumerem a continuació:

| _Paràmetre_ | _Descripció_ |
| --- | --- |
| response_type | Tipus de resposta a retornar. Actualment només es suporta el valor code que indica que el servidor retornarà a la aplicació un codi d'autorització (a_uthorization_code_) per a poder negociar el token d'accés. |
| client_id | Identificador de la aplicació web que esta realitzant la operació d'autenticació. Aquest identificador és assignat pel Consorci AOC en el moment de fer el registre de l'aplicació al VALId. Aquests identificadors tenen un aspecte similar al que es mostra a continuació: <br><br>_0123456789.serveis.aoc.cat_ </br> |
| redirect_uri | URL a la que VALId haurà de retornar el resultat del procés d'autenticació.<br><br> El resultat de la operació pot ser el codi d'autorització per tal que la aplicació web pugui negociar l'_access token_ definitiu o bé un codi d'error en cas de que la validació no s'hagi pogut realitzar amb èxit.<br><br> Aquesta URL s'haurà de proporcionar en el moment del registre de l'aplicació.<br><br>Una aplicació pot tenir més d'una URI de redirecció, però totes han de constar al registre de l'aplicació del VALId. |
| scope | El paràmetre _scope_ indica una llista de permisos que la aplicació web vol obtenir sobre les dades de l'usuari.<br><br> Ara per ara, VALId només realitza l'autenticació dels usuaris i no gestiona cap autorització, pel que aquest paràmetre haurà de venir informat sempre amb el valor _autenticacio_usuari_ |
| state | Camp lliure que serà retornat a la aplicació web en el moment de fer-li arribar el resultat de la autenticació (ja sigui un _authorization_code_ o un _error_).<br><br>Donades les restriccions que hi ha en la codificació de les cadenes de text a les URL es recomana usar cadenes molt simples, sense caràcters especials (accentuats, dièresi, etc...) per tal d'evitar problemes en el moment de realitzar les redireccions. |
| access_type | Tipus d'accés. Actualment el protocol OAuth 2.0 només admet els valors _online_ i _offline_. Per a la majoria de casos, s'haurà d'informar el valor online. |
| approval_prompt | Aquest paràmetre indica si cal presentar a l'usuari la pantalla de sol·licitud de permisos que vol obtenir l'aplicació web cada cop o només el primer cop que es realitza la autenticació. <br> <br>Donat que VALId no realitza ara per ara tasques d'autorització, aquest camp no es té en compte, tot i que per especificacions del protocol OAuth és necessari informar-lo. |
| login_hint | Aquest paràmetre és opcional i normalment conté dades com l'adreça de correu electrònic de l'usuari (si l'aplicació ja la coneix) o un subidentificador. <br><br>VALId no usa aquest paràmetre pel que es pot ometre. |

A continuació es mostra un exemple de URL d'inici de procés d'autenticació:
```
https://valid-pre.aoc.cat/o/oauth2/auth?scope=autenticacio_usuari&state=codi_estat_propi&redirect_uri=https://enotum.aoc.cat/code&response_type=code&client_id=0123456789.serveis.aoc.cat&approval_prompt=auto
```

## 2.2 Tractament de la resposta <a name="2.2"></a>

La resposta s'enviarà a la URL indicada al paràmetre redirect_uri informat a la URL de petició:

- Si l'usuari es valida correctament, la resposta contindrà un codi d'autorització (i si s'havia informat el paràmetre state, també rebrà aquest valor).

```
https://enotum.aoc.cat/code?code=1/j23a71vICLQm6bTrtp7&state=codi_estat_propi
```

- Si per contra l'autenticació es cancel·lada per l'usuari, l'aplicació client rebrà un codi d'error SESSION_CANCEL.
```
https://enotum.aoc.cat/code?error=SESSION_CANCEL&state=codi_estat_propi
```

Un cop rebut el codi d'autorització, la aplicació client ha de negociar un _token_ d'accés i opcionalment un _token_ de refresc. L'obtenció d'aquest _token_ d'accés finalitza el procés d'autenticació i és llavors quan la aplicació web pot donar l'usuari per autenticat.

La URL que s'usa per negociar el _token_ d'accés és:
```
https://valid-pre.aoc.cat/o/oauth2/token
```

La negociació del _token_ d'accés es realitza mitjançant una crida POST entre el servidor de la aplicació client i el VALId. Aquesta crida ha d'incloure els següents paràmetres:

| _Paràmetre_ | _Descripció_ |
| --- | --- |
| code | Codi d'autorització rebut del VALId. |
| client_id | Identificador de la aplicació client. Ha de coincidir amb el que s'ha enviat per iniciar el procés d'autenticació. |
| client_secret | Cadena de text que fa de secret compartit entre la aplicació client i el VALId. |
| redirect_uri | URL de resposta que ha de constar a la llista de URLs registrades per a la aplicació client al VALId. El més senzill és usar la mateixa URL que s'ha especificat al moment d'iniciar el procés d'autenticació. |
| grant_type | Per especificació OAuth 2.0, el valor d'aquest camp sempre serà authorization_code. |

Un exemple de petició POST seria similar a el que es mostra a continuació:
```
POST /o/oauth2/token HTTP/1.1
Host: accounts-dev.aoc.cat
Content-Type: application/x-www-form-urlencoded

code=1/j23a71vICLQm6bTrtp7&
client_id=0123456789.serveis.aoc.cat&
client_secret=...............&
redirect_uri=https://enotum.aoc.cat/code&
grant_type=authorization_code
```

La resposta a aquesta crida conté els següents camps:

| _Paràmetre_ | _Descripció_ |
| --- | --- |
| access_token | _Token_ d'accés que acredita a autenticació de l'usuari.<br><br>Aquest _token_ es pot emprar per obtenir una sèrie de dades com les evidències d'autenticació o les dades bàsiques de l'usuari (document d'identitat i número de telèfon) invocant una sèrie de serveis REST de dades (vegeu apartat 3 d'aquest document). |
| refresh_token | _Token_ de refresc que pot ser usat per obtenir nous _tokens_ d'accés a mesura que aquests vagin expirant.<br><br>Un _token_ de refresc serà vàlid fins que l'usuari el revoqui i només serà emès pel VALId si a la petició d'autenticació inicial es va especificar el valor offline per al paràmetre access_type. |
| expires_in | Temps de vida restant per al _token_, en segons. |
| token_type | Tipus de token generat. Actualment aquest camp sempre tindrà el valor Bearer. |

La resposta a la crida vindrà donada amb representació JSON.
```json
{
    "access_token":"1/g073bzAr24Fz3Z1e44g73v",
    "expires_in":3600,
    "token_type":"Bearer"
}
```

### 2.2.1 Generació d'un nou accés token a partir del refresh Token. <a name="2.2.1"></a>

Si en la petició d'autenticació inicial s'especifica el valor offline al paràmetre accés\_type, la resposta generada contindria token de refresc assignat per a poder regenerar nous tokens d'autenticació:

```json
{
    "access_token":"1/g073bzAr24Fz3Z1e44g73v",
    "refresh_token":"zZ5c9nbFmaEQJbFaHknXQe4SjyYWwAu1qiw7Ic9_",
    "expires_in":3600,
    "token_type":"Bearer"
}
```

A partir d'aquest moment, es podran demanar nous tokens a partir de la URL que s'usa per negociar el _token_ d'accés:
```
https://valid-pre.aoc.cat/o/oauth2/token
```

A diferència del cas anterior, caldrà especificar el valor refresh_token al paràmetre grant_type i en comptes d'enviar el paràmetre code, caldrà enviar el paràmetre refresh_token amb el valor de la resposta anterior. Per exemple:
```
POST /o/oauth2/token HTTP/1.1
Host: accounts-dev.aoc.cat
Content-Type: application/x-www-form-urlencoded

refresh_token=zZ5c9nbFmaEQJbFaHknXQe4SjyYWwAu1qiw7Ic9_
client_id=0123456789.serveis.aoc.cat&
client_secret=...............&
redirect_uri=https://enotum.aoc.cat/code&
grant_type=refresh_token
```

La resposta a aquesta crida contindria el nou accés token generat:

```json
{
    "access_token":"1/eZU0_DyEUjETrw9B0VIWZvSympnrm-vnKdVzC1xF",
    "refresh_token":"zZ5c9nbFmaEQJbFaHknXQe4SjyYWwAu1qiw7Ic9_",
    "expires_in":3600,
    "token_type":"Bearer"
}
```

## 2.3 Revocació d'un token d'accés <a name="2.3"></a>

Per revocar un _token_ d'accés l'aplicació client pot realitzar una invocació a la següent URL del VALId:
```
https://valid-pre.aoc.cat/o/oauth2/revoke?token=<token_acces>
```
Si la revocació es realitza correctament, l'aplicació rebrà un codi de resposta HTTP 200. En qualsevol altre cas, rebrà un codi HTTP 400 i un missatge d'error.

La revocació del _token_ d'accés només implica que no es podrà invocar els serveis REST oferts per obtenir dades de l'usuari o les evidencies del procés d'autenticació. En cap cas la revocació d'un _token_ implica el tancament de la sessió web establerta entre el navegador de l'usuari i el VALId.

## 2.4 Logout programàtic <a name="2.4"></a>

Normalment els usuaris tanquen les seves sessions a la aplicació client a la qual han accedit, independentment del servei mitjançant el qual s'hagin autenticat. Aquest _logout_ és el que s'anomena _logout local_.

El problema d'aquest escenari és que la sessió d'usuari segueix estant activa entre el VALId i el navegador de l'usuari. Per tant, un cop es torni a intentar accedir a la aplicació web, la autenticació de l'usuari serà directa i es tornarà a iniciar sessió amb la mateixa identitat.

Per evitar aquesta situació s'ha implementat una funcionalitat que permet invalidar la sessió d'usuari al VALId tot invocant una URL, a l'igual que es fa per revocar un _token_ d'accés. Cal aclarir que aquesta funcionalitat no entra dins del protocol OAuth 2.0 i es tracta d'una característica implementada a VALId per requeriments del propi servei.

Així doncs, per tancar la sessió d'usuari a VALId, l'aplicació client pot realitzar una invocació a la següent URL:
```
https://valid-pre.aoc.cat/o/oauth2/logout?token=<token_acces>
```
La invocació retornarà un codi HTTP 200 si la operació s'ha realitzat correctament o un codi HTTP 400 si s'ha produït algun error, junt amb un missatge descriptiu.

# 3 Serveis de dades de suport a les aplicacions <a name="3"></a>

La integració amb VALId abstrau l'aplicació client de totes les tasques relacionades amb la identificació i autenticació de l'usuari.

Degut a això, és necessari oferir serveis que a partir del _token_ d'accés proporcionin informació relativa al procés d'autenticació de l'usuari, per exemple, les credencials presentades o les evidències de l'acte d'autenticació.

A continuació es descriuen els serveis REST que s'han posat a disposició de les aplicacions integrades amb VALId.

## 3.1 Dades de l'usuari validat <a name="3.1"></a>

El servei proporciona la següent informació:

El servei d'obtenció de dades de l'usuari té un únic paràmetre, l'_access token_ obtingut per la aplicació client en el moment de fer la autenticació.

| _Mètode (GET)_ | https://valid-pre.aoc.cat/serveis-rest/getUserInfo |
| --- | --- |
| _Paràmetre_ | AccessToken |
| _Resposta (exemple)_ | {<br>"status":"ok",<br>"identifier":"99999999R",<br>"prefix":"0034",<br>"phone":"609112233",<br>"documentType":"1"<br>} |

La resposta obtinguda, en format JSON, conté les següents dades:

| status | Resultat de l'operació. Cadena de text que pot tenir el valor ok o ko. |
| --- | --- |
| identifier | Document identificador de l'usuari. L'identificador por ser un NIF, un NIE o un número de passaport. |
| prefix | Codi o prefix internacional del telèfon. Per exemple 0034 per al territori espanyol. |
| phone | Número de telèfon mòbil de l'usuari. |
| identifierType | Tipus de document d'identitat. 1=NIF, 2=NIE, 3=Passaport, 4=Altres (targeta de residència comunitària, permís de residència de treball, document identificador d'un país de la CE). Només si l'usuari s'ha autenticat amb idCAT Mòbil o MobileID. |
| name | El nom de l'usuari (en cas que el mecanisme de validació el proporcioni). |
| surnames | Els cognoms de l'usuari (en cas que el mecanisme de validació el proporcioni). |
| surname1 | Primer cognom de l'usuari (en cas que el mecanisme de validació proporcioni els cognoms per separat). |
| surname2 | Segon cognom de l'usuari (en cas que el mecanisme de validació proporcioni els cognoms per separat i el segon cognom estigui informat). |
| countryCode | Codi de país de l'usuari en format ISO 3166-1 (en cas que el mecanisme de validació el proporcioni). |
| email | El correu de l'usuari (en cas que el mecanisme de validació el proporcioni). |
| userCertificate | Certificat digital de l'usuari si aquest s'ha autenticat mitjançant certificat. |
| certificateType | En cas d'autenticació amb certificat digital, tipus de certificat:<br>- 0: Persona física<br>- 1: Persona jurídica<br>- 2: Component SSL<br>- 3: Seu electrònica<br>- 4: Segell electrònic<br>- 5: Empleat públic<br>- 6: Entitat sense personalitat jurídica<br>- 7: Empleat públic amb pseudònim<br>- 8: Qualificat de segell<br>- 9: Qualificat d'autenticació de lloc web<br>- 10: Certificat de segell de temps<br>- 11: Representant de persona jurídica<br>- 12: Representant d'entitat sense personalitat jurídica |
| companyId | En cas d'autenticació amb certificat digital, CIF vinculat al certificat si aquest està informat. |
| companyName | En cas d'autenticació amb certificat digital, nom de l'empresa vinculat al certificat si aquest està informat. |
| method | Mètode d'autenticació emprat per l'usuari (_idcatmobil, certificat, clave_). |
| assuranceLevel | Nivell de seguretat de l'autenticació practicada d'acord amb el ReIdAS (_low, substantial, high_):<br>- Baix: idCAT Mòbil acreditat telemàticament.<br>- Substancial: idCAT Mòbil acreditat amb certificat o presencialment, Cl@ve, certificat qualificat en programari.<br>- Alt: certificat qualificat en targeta. |
| error | En cas d'error, missatge descriptiu de l'error que s'ha produït. |

## 3.2 Evidències d'autenticació <a name="3.2"></a>

Quan l'usuari s'autentica al VALId es generen una sèrie d'evidències de cadascuna de les operacions que el validador realitza per tal d'autenticar l'usuari. Aquestes evidències inclouen:

- Consulta i resposta a la base de dades de la Seu Electrònica del Departament de Presidència i generació i validació de la contrasenya SMS d'un sol ús en cas d'autenticació amb IDCAT-SMS.
- Verificació del certificat d'usuari en el cas d'autenticació amb certificat digital.
- Missatges SAML en el cas d'autenticació amb Cl@ve.

El servei d'obtenció d'evidències té un únic paràmetre, l'_access token_ obtingut per la aplicació client en el moment de fer la autenticació.

| _Mètode (GET)_ | https://valid-pre.aoc.cat/serveis-rest/getAuthenticationEvidence |
| --- | --- |
| _Paràmetre_ | AccessToken (GET) |
| _Resposta (exemple)_ | {<br>&nbsp;&nbsp;&nbsp;&nbsp;"status": "ok",<br>&nbsp;&nbsp;&nbsp;&nbsp;"evidences":<br>&nbsp;&nbsp;&nbsp;&nbsp;[<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"PD94bWwgdmVyc2l(...)RpY2lvPg==",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"PD94bWwgdmVyc2lvb(...)vc3RhPg==",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"PD94bWwgdmVy(...)jaW8+"<br>&nbsp;&nbsp;&nbsp;&nbsp;]<br>} |

La resposta obtinguda, en format JSON, conté les següents dades:

| status | Resultat de l'operació. Cadena de text que pot tenir el valor ok o ko. |
| --- | --- |
| evidences | Llista d'evidències amb les evidències codificades en Base64. <br>Per més detalls sobre les evidències d'autenticació, consulteu l'annex d'aquest mateix document. |

# 4 Operacions de signatura ordinària <a name="4"></a>

VALId ofereix dos mecanismes de signatura ordinària que es detallen a continuació:

- Signatura ordinària a partir de l'_access token_ obtingut en el procés d'autenticació.
- Signatura ordinària a partir de l_'acces token_ obtingut en el procés d'autenticació i una nova acció d'autenticació realitzada per l'usuari (un senzill CAPTCHA).

## 4.1 Signatura ordinària a partir de l'accés token <a name="4.1"></a>

L'autenticació de l'usuari es pot usar com a clau per generar el que s'anomena _signatura ordinària_.

Aquesta signatura consisteix en la generació d'una evidència signada en la que consten els noms i resums criptogràfics dels documents que es volen signar.

| _Mètode (POST)_ | https://valid-pre.aoc.cat/serveis-rest/getBasicSignature |
| --- | --- |
| _Petició (exemple)_ | {<br> &nbsp;&nbsp;&nbsp;&nbsp;"accessToken":"ACCESS-TOKEN",<br> &nbsp;&nbsp;&nbsp;&nbsp;"documents": [<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{ <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"name":"fitxer1.pdf", <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"algorithm":"SHA1",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "hash":"UkVTVU0="<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"metadata":"classificacio=00002;format=PDF" <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},<br> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{ <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"name":"fitxer2.doc",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "algorithm":"SHA1",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "hash":"UkVTVU0=" <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;]<br>} |
| _Resposta (exemple)_ | {<br>&nbsp;&nbsp;&nbsp;&nbsp;"status":"ok",<br>&nbsp;&nbsp;&nbsp;&nbsp;"evidence":"PD94bW0dXJhT (...) 6U2lnbmF0dXJlPg=="<br>} |

El servei d'obtenció de la signatura té com a paràmetres (estructura JSON), l'_access token_ obtingut per la aplicació client en el moment de fer l'autenticació i les dades dels documents :

| accessToken | L'_access token_ obtingut per la aplicació client en el moment de fer la autenticació |
| --- | --- |
| documents | Llista de documents dels quals es vol generar la signatura ordinària. |
| name | Nom del document. |
| algorithm | Algorisme emprat per calcular el resum criptogràfic del document. |
| hash | Resum criptogràfic del document. |
| metadata | Dades addicionals –text lliure- que s'incorporaran a l'evidència generada, vinculada al document. |

La resposta obtinguda, en format JSON, conté les següents dades:

| status | Resultat de l'operació. Cadena de text que pot tenir el valor ok o ko. |
| --- | --- |
| evidence | Signatura XAdES-T amb l'evidència de la signatura ordinària codificada en Base64.<br> Per més detalls sobre l'evidència de la signatura consulteu l'apartat 4.3 d'aquest mateix document. |
| error | En cas d'error, descripció de l'error que s'ha produït. |

## 4.2 Signatura ordinària a partir de l'accés token i acció d'autenticació addicional <a name="4.2"></a>

La seqüència de signatura s'inicia quan l'aplicació web que s'integra desitja generar una signatura ordinària però, a diferència del cas anterior, es vol que l'usuari realitzi una nova acció d'autenticació (un senzill CAPTCHA).

Per fer-ho, l'aplicació web ha de realitzar una operació REST _initBasicSignature_ tot indicant l'_access token_ associat a l'usuari autenticat, una URL de redirecció de l'aplicació client on rebre el resultat de la signatura i les dades dels documents a signar:

![2](Captures/2.png)

| _Mètode (POST)_ | https://valid-pre.aoc.cat/serveis-rest/initBasicSignature |
| --- | --- |
| _Petició (exemple)_ | {<br>&nbsp;&nbsp;&nbsp;&nbsp; "accessToken":"ACCESS-TOKEN",<br>&nbsp;&nbsp;&nbsp;&nbsp;"redirectUri":"URL-REDIRECT",<br>&nbsp;&nbsp;&nbsp;&nbsp; "documents": [<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{ <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"name":"fitxer1.pdf",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "algorithm":"SHA1", <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"hash":"UkVTVU0=""metadata":"classificacio=00002;format=PDF" <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; { <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"name":"fitxer2.doc", <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"algorithm":"SHA1",<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "hash":"UkVTVU0=" <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br>&nbsp;&nbsp;&nbsp;&nbsp;]<br>} |
| _Resposta (exemple)_ | {<br>&nbsp;&nbsp;&nbsp;&nbsp;"status":"ok",<br>&nbsp;&nbsp;&nbsp;&nbsp;"signatureCode":"XXXXXXXXXXXXXXXXXXX"<br>} |

El servei d'inici de signatura té com a paràmetres (estructura JSON), l'_access token_ obtingut per la aplicació client en el moment de fer l'autenticació i les dades dels documents:

| accessToken | L'_access token_ obtingut per la aplicació client en el moment de fer la autenticació |
| --- | --- |
| redirectUri | URL de redirecció de l'aplicació client que rebrà el resultat de la signatura. |
| documents | Llista de documents dels quals es vol generar la signatura ordinària. |
| name | Nom del document. |
| algorithm | Algorisme emprat per calcular el resum criptogràfic del document. |
| hash | Resum criptogràfic del document. |
| metadata | Dades addicionals –text lliure- que s'incorporaran a l'evidència generada, vinculada al document. |

La resposta obtinguda, en format JSON, conté les següents dades:

| status | Resultat de l'operació. Cadena de text que pot tenir el valor ok o ko. |
| --- | --- |
| signatureCode | Codi que identifica el procés de signatura que s'inicia. |
| error | En cas d'error, descripció de l'error que s'ha produït. |

VALId verifica la validesa d'aquest _access token_ i retorna un _signatureCode_ (d'un sòl ús) que l'aplicació client haurà d'informar con a paràmetre en la següent URL del VALId:
```
https://valid-pre.aoc.cat/o/sign?signature_code=<signature code>
```
En aquest punt, VALID presenta a l'usuari la pantalla que li permetrà realitzar la nova autenticació.

![3](Captures/3.png)

Un cop l'usuari ha realitzat la nova acció d'autenticació, VALId realitza una redirecció a la URL de signatura informada per l'aplicació client en la operació _initBasicSignature_ indicant com a paràmetres el _signatureCode_ que identifica el procés de signatura i el codi resultat de l'operació en el paràmetre _status_:

- Si l'usuari ha signat correctament, la resposta contindrà el paràmetre status amb el valor _OK_:
```
https://enotum.aoc.cat/signatura?signatureCode=1/j23a71vICLQm6bTrtp7&state=OK
```

- Si l'usuari ha cancel·lat la signatura, la resposta contindrà el paràmetre status amb el valor _CANCEL_:
```
https://enotum.aoc.cat/signatura?signatureCode=1/j23a71vICLQm6bTrtp7&state=CANCEL
```
- Si per contra hi ha cap error realitzant la signatura, l'aplicació client rebrà un codi d'estat _ERROR_:
```
https://enotum.aoc.cat/signatura?signatureCode=1/j23a71vICLQm6bTrtp7&state=ERROR
```
Si la signatura s'ha realitzat correctament, l'aplicació client pot sol·licitar a VALId la signatura ordinària generada realitzant una operació REST _getBasicSignature_.

| ![image](https://user-images.githubusercontent.com/32306731/137281698-9dfc2044-94f7-487f-a7d6-9a4e0707feb3.png) És important que l'aplicació client controli el número d'accessos a la seva URL de redirecció on se li comunica el resultat de la signatura per evitar descàrregues de signatures ja obtingudes prèviament.
 

| _Mètode (POST)_ | https://valid-pre.aoc.cat/serveis-rest/getBasicSignature |
| --- | --- |
| _Petició (exemple)_ | { <br>&nbsp;&nbsp;&nbsp;&nbsp;"accessToken":"ACCESS-TOKEN", <br>&nbsp;&nbsp;&nbsp;&nbsp;"signatureCode":"SIGNATURE-CODE"<br>} |
| _Resposta (exemple)_ | {<br>&nbsp;&nbsp;&nbsp;&nbsp;"status":"ok",<br>&nbsp;&nbsp;&nbsp;&nbsp;"evidence":"PD94bW0dXJhT (...) 6U2lnbmF0dXJlPg=="<br>} |

El servei d'obtenció del resultat la signatura té com a paràmetres (estructura JSON), l'_access token_ obtingut per la aplicació client en el moment de fer l'autenticació i les dades dels documents :

| accessToken | L'_access token_ obtingut per la aplicació client en el moment de fer la autenticació |
| --- | --- |
| signatureCode | Codi del procés de signatura realitzada correctament del qual es vol recollir l'evidència. |

La resposta obtinguda, en format JSON, conté les següents dades:

| status | Resultat de l'operació. Cadena de text que pot tenir el valor ok o ko. |
| --- | --- |
| evidence | Signatura XAdES-T amb l'evidència de la signatura ordinària codificada en Base64.<br> Per més detalls sobre l'evidència de la signatura consulteu l'apartat 4.3 d'aquest mateix document. |
| error | En cas d'error, descripció de l'error que s'ha produït. |

## 4.3 Consideracions sobre el resum criptogràfic <a name="4.3"></a>

Els mètodes descrits per la obtenció de signatures ordinàries permeten a les aplicacions usuàries vincular la identitat autenticada dels usuaris amb el resum criptogràfic d'un document que la pròpia aplicació envia. L'aplicació ha d'informar, per tant, tan de l'algorisme resum emprat com el valor que pren.

VALId no porta a terme cap comprovació sobre els valors enviats per les aplicacions usuàries. Tot i això es recomana seguir les indicacions estandarditzades a l'hora de definir-los, i que W3C publica al seu document [XML Security Algorithm Cross-Reference][4].

[4]:https://www.w3.org/TR/xmlsec-algorithms/

Per exemple, si l'algorisme que es vol emprar, i que actualment es recomana, és SHA256, l'identificador de l'algorisme hauria de ser la URI http://www.w3.org/2001/04/xmlenc#sha256 i el valor hauria de ser la codificació en base64 de la cadena de bits vista com a un flux de 32 octets.

Un document, per tant, quedaria correctament identificat, per exemple, de la següent manera:

```json
"documents": [
    {
        "name":"fitxer1.pdf",
        "algorithm":"http://www.w3.org/2001/04/xmlenc#sha256",
        "hash":"mx3kGyP73e+5bGsYdLNmKQoy0Wf1aK5lgjh1tU3HWF8="
        "metadata":"classificacio=00002;format=PDF"
    },
...
]
```

## 4.4 Missatge de signatura ordinària <a name="4.4"></a>

A continuació es descriu el format del missatge que forma la signatura ordinària.

La signatura ordinària està formada per una signatura XAdES-T que embolcalla (_enveloping_) un missatge XML que compleix l'schema que es mostra a la il·lustració. A continuació es llisten els camps que formen aquest missatge d'evidència:

| _Element_ | _Descripció_ | _Obligatori_ |
| --- | --- | --- |
| timestamp | Data de l'acte de signatura. | S ||
| identificador | Identificador del procés d'autenticació de l'usuari que realitza l'acte de signatura. | S |
| metode | Mètode amb el que l'usuari s'ha autenticat:<br>- idcatmobil<br>- certificat<br>- clave<br>- mobileid<br>- mobileconnect | S |
| identitat/evidencia | Evidències d'autenticació. Inclou tots els missatges de petició i resposta bescanviats entre el broker d'identitats i els diferents serveis d'autenticació. | S |
| identitat/evidencia@dataGeneracio | Data de generació de la evidència. | S |
| identitat/evidencia@tipus | Tipus d'evidència:<br>- bdseu-peticio / bdseu-resposta: evidències de la consulta de les dades de l'usuari a la BD de la seu<br>- autenticacio-inici-peticio / autenticacio-resposta-peticio: evidències d'autenticació. Vegeu l'annex d'aquest document.<br> | S |
| identitat/document | Document identificatiu de l'usuari que realitza l'acte de signatura (p.e. NIF).<br>En cas d'autenticació amb certificat que té vinculat un CIF, aquest s'informarà en una altra ocurrència de l'element document. | S |
| identitat/nom | Nom l'usuari que realitza l'acte de signatura. | S |
| document/nom | Nom del document a signar tal i qual s'ha informat a la petició. | S |
| document/resum | Resum criptogràfic del document a signar. | S |
| document/algorisme | Algorisme usat per a calcular el resum criptogràfic | S |
| document/metadades | Metadades informades en la petició, codificades en Base64. | N |

| ![image](https://user-images.githubusercontent.com/32306731/137281698-9dfc2044-94f7-487f-a7d6-9a4e0707feb3.png)  L'evidència de signatura ordinària està conformada per l'agregació d'una sèrie d'evidències XML generades per cadascun dels serveis i mòduls que participen en la validació de la identitat d'un usuari en l'acte de la signatura dels documents referenciats (Base de dades de la Seu de la DGACD, servei de contrasenya al mòbil del CAOC o PSIS de CATCert, entre d'altres).<br> Algunes d'aquestes evidències no són autocontingudes -no estan signades digitalment- de manera que per tal de garantir-ne l'autenticitat i integritat, el Consorci AOC guarda traça de totes les accions realitzades en el procés de la validació de la identitat d'un usuari en un sistema de traces [certificades](#11) que podrà ser consultat sota demanda per part de l'organisme requeridor de la mateixa (consultes a la Base de dades de la Seu de la DGACD i crides als diferents serveis de validació d'identitats: contrasenya d'un sol ús SMS al mòbil, o validació de certificat digital contra PSIS de CATCert).

---

<a name="11"></a> El sistema de traces certificades té la particularitat que els seus registres van enllaçats amb una signatura HMAC: cada registre comença amb el hash SHA-256 del registre anterior xifrat amb una clau privada simètrica que només coneix el sistema. D'aquesta manera és impossible afegir o esborrar una traça a posteriori sense trencar la integritat interna del fitxer de traces, és a dir, es pot detectar en quina línia del fitxer de log s'ha realitzat una alteració.<br>Addicionalment, cada nit s'executa un procés de consolidació que afegeix un segell de temps a tot el fitxer de traces de forma que assegurem la integritat de tot el fitxer i permet determinar amb fiabilitat la data de creació del mateix i si s’ha modificat o no des d’aleshores.


![4](Captures/4.png)


# Annex - evidències del procés de validació <a name="5"></a>

A continuació es mostren uns exemples de les evidencies que es generen en cada pas del procés de validació.

### Evidències generades en la consulta a la Base de Dades de la Seu <a name="6"></a>

La comunicació amb el servei de la Base de Dades de la Seu de la Direcció General d'Atenció Ciutadana i Difusió (en endavant DGACD) es realitza via uns serveis REST que intercanvien missatges JSON.

| _Exemple petició_ |
| --- |
```xml
{"documentId":"47778894M","telefon":{"prefix":"0034","numero":"626213433","tipusTelefonid":null}}
```




| _Exemple resposta_ |
| --- |
```xml
{"request":{"documentId":"47778894M","telefon":{"tipusTelefonid":2,"prefix":"0034","numero":"626213433"}},
"response":{"codiMissatge":"01","descMissatge":"Document d'identificació i mòbil relacionats","nom":"Antoni","cognom1":"Llebaria","cognom2":"Seoane","correusElectronics":["allebaria@aoc.cat"],"codiAutenticacio":"0","assuranceLevel":"No informat","paisDocumentISO3166":"ES","canalRelacioPreferent":"D","tipusDocument":"1","autenticacioCodiPublic":"No informat"}}

```

### Evidències generades en la validació amb certificat digital <a name="7"></a>

En cas d'autenticació amb certificat digital s'enregistra com evidència tant el missatge de petició de validació del certificat amb el que l'usuari s'autentica com la resposta obtinguda amb el resultat de la validació generat per la plataforma PSIS de CATCert.

| _Exemple petició_ |
| --- |
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<VerifyRequest>
 <OptionalInputs>
  <ReturnProcessingDetails/>
  <ReturnX509CertificateInfo>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectDistinguishedName:organizationName"/>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectDistinguishedName:commonName"/>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectDistinguishedName:surname"/>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectDistinguishedName:givenName"/>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectDistinguishedName:title"/>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectDistinguishedName:serialNumber"/>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectDistinguishedName:countryName"/>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:IssuerDistinguishedName:commonName"/>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:Version"/>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SerialNumber"/>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:IssuerDistinguishedName"/>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectPublicKeyAlgorithm"/>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectPublicKey"/>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:KeyUsages"/>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectEmail"/>
   <AttributeDesignator Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:CertificatePolicies"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:professionalAssociations:ProfessionalAssociationName"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:professionalAssociations:ProfessionalAssociationInitials"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:professionalAssociations:ProfessionalAssociationNumber"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:professionalAssociations:ProfessionalAssociationZone"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:professionalAssociations:ProfessionalAssociationEmployeeNumber"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:KeyOwnerNIF"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:Title"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:LegalEntityCIF"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:LegalEntityGlobalCIF"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:ClassificationLevel"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:Department"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:issuerCA"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:CertificateType"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:CertificateTypeCode"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:VinculatedCompanyCIF"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:VinculatedCompanyName"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:VinculatedPersonFullName"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:VinculatedPersonName"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:VinculatedPersonNIForNIE"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:VinculatedPersonSurname"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:NaturalPersonIdentityType"/>
   <AttributeDesignator Name="urn:catcert:psis:certificateAttributes:NaturalPersonCountryCode"/>
  </ReturnX509CertificateInfo>
  <ReturnSignedResponse/>
 </OptionalInputs>
 <SignatureObject>
  <Other>
   <X509Data>
     <X509Certificate>MIIHKj (...) </X509Certificate>
   </X509Data>
  </Other>
 </SignatureObject>
</VerifyRequest>

```

| _Exemple resposta_ |
| --- |
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<dss:VerifyResponse Profile="urn:oasis:names:tc:dss:1.0:profiles:XSS" xmlns:dss="urn:oasis:names:tc:dss:1.0:core:schema">
			<dss:Result>
				<dss:ResultMajor>urn:oasis:names:tc:dss:1.0:resultmajor:Success</dss:ResultMajor>
				<dss:ResultMinor>urn:oasis:names:tc:dss:1.0:profiles:XSS:resultminor:valid:certificate:Definitive</dss:ResultMinor>
			</dss:Result>
			<dss:OptionalOutputs>
				<dss:ProcessingDetails>
					<dss:ValidDetail Type="urn:oasis:names:tc:dss:1.0:detail:ValidityInterval">
						<dss:Message xml:lang="en">The signing key is inside its static validity interval.</dss:Message>
					</dss:ValidDetail>
					<dss:ValidDetail Type="urn:oasis:names:tc:dss:1.0:detail:IssuerTrust">
						<dss:Message xml:lang="en">The issuer of the given key is trusted.</dss:Message>
					</dss:ValidDetail>
					<dss:ValidDetail Type="urn:oasis:names:tc:dss:1.0:detail:RevocationStatus">
						<dss:Message xml:lang="en">The signing key is not revoked.</dss:Message>
					</dss:ValidDetail>
				</dss:ProcessingDetails>
				<urn:X509CertificateInfo xmlns:urn="urn:oasis:names:tc:dss:1.0:profiles:XSS">
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectDistinguishedName:organizationName">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">Organització de prova</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectDistinguishedName:commonName">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">Persona de la Peça de Prova - DNI 00000000T (TCAT)</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectDistinguishedName:surname">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">de la Peça de Prova - DNI 00000000T</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectDistinguishedName:givenName">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">Persona</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectDistinguishedName:title">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">Càrrec de prova</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectDistinguishedName:serialNumber">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">IDCES-00000000T</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectDistinguishedName:countryName">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">ES</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:IssuerDistinguishedName:commonName">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">EC-SectorPublic</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:Version">
						<urn1:AttributeValue xsi:type="xs:integer" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">3</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SerialNumber">
						<urn1:AttributeValue xsi:type="xs:integer" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">4663096142786113154784315765348532954</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:IssuerDistinguishedName">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">CN=EC-SectorPublic,OU=Serveis Públics de Certificació,O=CONSORCI ADMINISTRACIO OBERTA DE CATALUNYA,C=ES</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectPublicKeyAlgorithm">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">RSA</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectPublicKey">
						<urn1:AttributeValue xsi:type="xs:base64Binary" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAj5jvzZq50EiXjtu8eKn+1lgiFtOKkYq4RFoiZGWr/CKJqtjwnVBXTlU3vqcIG0yUF6IUEEbgpL8g73nMGdOTN1/SKMFShoNujyMQqu61sjn80paLX43hfJjiNVtxCXlsQ90QhRbemIqL7UhtEOzQbIdbdqVn8+39p1Vy4ZWPGuV1Pp9uJa23uxcjju4ijkErrs2QAk5OrCifDcczIjI8FpmDcTsMBfwl3DtzBdJbTlA1C+Z0onpIWPF+xj+cps+fY6ZLj8VUxom20kGtEHpTWFbnjLtxJES0GrsRdCH23tzpLFiXwFxRrjPkw6485yoeAf6CgU6VPv/sBsXi0TjsaQIDAQAB</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:KeyUsages">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">digitalSignature,nonRepudiation,keyEncipherment</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:SubjectEmail">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">scd@aoc.cat</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:oasis:names:tc:dss:1.0:profiles:XSS:certificateAttributes:CertificatePolicies">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">1.3.6.1.4.1.15096.1.3.2.82.1,0.4.0.194112.1.2</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:professionalAssociations:ProfessionalAssociationName"/>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:professionalAssociations:ProfessionalAssociationInitials"/>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:professionalAssociations:ProfessionalAssociationNumber"/>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:professionalAssociations:ProfessionalAssociationZone"/>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:professionalAssociations:ProfessionalAssociationEmployeeNumber"/>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:KeyOwnerNIF">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">00000000T</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:Title">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">Càrrec de prova</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:LegalEntityCIF">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">Q0000000J</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:LegalEntityGlobalCIF">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">VATES-Q0000000J</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:ClassificationLevel">
						<urn1:AttributeValue xsi:type="xs:integer" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">4</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:Department">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">Persona vinculada de nivell alt</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:issuerCA">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">EC-SectorPublic</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:CertificateType">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">Persona vinculada de nivell alt</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:CertificateTypeCode">
						<urn1:AttributeValue xsi:type="xs:integer" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">0</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:VinculatedCompanyCIF">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">Q0000000J</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:VinculatedCompanyName">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">Organització de prova</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:VinculatedPersonFullName">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">Persona de la Peça de Prova</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:VinculatedPersonName">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">Persona</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:VinculatedPersonNIForNIE">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">00000000T</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:VinculatedPersonSurname">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">de la Peça de Prova</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:NaturalPersonIdentityType">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">IDC</urn1:AttributeValue>
					</urn:Attribute>
					<urn:Attribute Name="urn:catcert:psis:certificateAttributes:NaturalPersonCountryCode">
						<urn1:AttributeValue xsi:type="xs:string" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:urn1="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">ES</urn1:AttributeValue>
					</urn:Attribute>
				</urn:X509CertificateInfo>
				<urn:ResponseSignature xmlns:urn="urn:oasis:names:tc:dss:1.0:profiles:XSS">
					<ds:Signature Id="id-63b37f2c-71fc-4f65-bf1a-d8a11e68e3ae" xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
						<ds:SignedInfo Id="id-e2082c67-7868-4837-99c0-5e942e5f86a6">
							<ds:CanonicalizationMethod Algorithm="http://www.w3.org/TR/2001/REC-xml-c14n-20010315"/>
							<ds:SignatureMethod Algorithm="http://www.w3.org/2000/09/xmldsig#rsa-sha1"/>
							<ds:Reference URI="" Id="id-b646502e-3d4c-48a2-88b1-bff5ba11ae21">
								<ds:Transforms>
									<ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
								</ds:Transforms>
								<ds:DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1"/>
								<ds:DigestValue>8V1vt0GhL36ft5RuSsQqX828Y5Y=</ds:DigestValue>
							</ds:Reference>
							<ds:Reference URI="#id-4c09cc36-ec7b-4e7b-9587-12435296cca5" Id="id-ad1448f5-1ea0-4dc8-97b0-89bfaf218322" Type="http://uri.etsi.org/01903#SignedProperties">
								<ds:Transforms>
									<ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
								</ds:Transforms>
								<ds:DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1"/>
								<ds:DigestValue>ho5zBzZf1VNgffPXZ1ulYYw2KD8=</ds:DigestValue>
							</ds:Reference>
						</ds:SignedInfo>
						<ds:SignatureValue Id="id-831f41b3-a52d-4a53-a126-b889961825fe">mU+9 (...) slg==</ds:SignatureValue>
						<ds:KeyInfo>
							<ds:X509Data>
								<ds:X509Certificate>MIIG (...) lKN00=</ds:X509Certificate>
							</ds:X509Data>
							<ds:KeyValue>
								<ds:RSAKeyValue>
									<ds:Modulus>AOqEXweLeJKYIhYn+XTSsgoZoQm6osSFQTic5wI76/crGdux2Xqbaz4dHz1KMTe2n/VT9hR+FQgFO4VPLiZReR6bJJHO9y5X0o8vQmZ/tLwhGOlR37DM6ugdoZubwXLm9uGKlPI+gpiniX60XeqWh/S8xTmxyvK82Ee/6mnTTDsA50bkCJ3rGMp5ZOHzUXizGTaoj2bx6aJ1k8Wt0e19ZKs8JWBvFyuaBEWdo2zcHPnEG7hnKaB5KgBo9F+T0GhhAUU0+OMyP032KbZpvztcn/NBFfu9qRfxFC4yn+BYiCkvmNK6G9koLjKAw0WqbFp/dd6JR6O9LzRDVLQCK1XdWc0=</ds:Modulus>
									<ds:Exponent>AQAB</ds:Exponent>
								</ds:RSAKeyValue>
							</ds:KeyValue>
						</ds:KeyInfo>
						<ds:Object Id="id-55bed222-336c-4bd7-95cf-258b24cb32c8">
							<xades:QualifyingProperties Target="#id-63b37f2c-71fc-4f65-bf1a-d8a11e68e3ae" xmlns:xades="http://uri.etsi.org/01903/v1.3.2#">
								<xades:SignedProperties Id="id-4c09cc36-ec7b-4e7b-9587-12435296cca5">
									<xades:SignedSignatureProperties>
										<xades:SigningCertificate>
											<xades:Cert>
												<xades:CertDigest>
													<ds:DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1"/>
													<ds:DigestValue>G05OIrUPLjsOdQV2ljdhCMbsQV8=</ds:DigestValue>
												</xades:CertDigest>
												<xades:IssuerSerial>
													<ds:X509IssuerName>CN=EC-SectorPublic,OU=Serveis P\C3\BAblics de Certificaci\C3\B3,O=CONSORCI ADMINISTRACIO OBERTA DE CATALUNYA,C=ES</ds:X509IssuerName>
													<ds:X509SerialNumber>152989983607946174710335949872320871922</ds:X509SerialNumber>
												</xades:IssuerSerial>
											</xades:Cert>
										</xades:SigningCertificate>
									</xades:SignedSignatureProperties>
								</xades:SignedProperties>
							</xades:QualifyingProperties>
						</ds:Object>
					</ds:Signature>
				</urn:ResponseSignature>
			</dss:OptionalOutputs>
		</dss:VerifyResponse>

```

Per més detalls sobre les especificacions de la missatgeria DSS corresponent a les respostes de PSIS podeu adreçar-vos a la pròpia especificació del servei:

[https://www.aoc.cat/Inici/SERVEIS/Signatura-electronica-i-seguretat/Validador/Com-utilitzar-ho](https://www.aoc.cat/Inici/SERVEIS/Signatura-electronica-i-seguretat/Validador/Com-utilitzar-ho).

### Evidència generada en la validació amb contrasenya al mòbil (SMS) <a name="8"></a>

| _Exemple evidència enviament de contrasenya_ |
| --- |
```xml
<?xml version="1.0" encoding="UTF-8"?>
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:adap="http://adaptors.j2ee.limsp.latinia.com">
   <soapenv:Header/>
   <soapenv:Body>
      <adap:putMessage soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
         <string xsi:type="xsd:string">
            <![CDATA[
               <message id="87571A13EC074BAEAEB67E9A076AF738" ts="1698314022606">
                  <head>
                     <type ref="sms">
                        <format>text</format>
                        <mroute>MT</mroute>
                     </type>
                     <info>
                        <gsmDest>0034626213433</gsmDest>
                        <idContract>1165</idContract>
                        <numeroOrigen>idCATMobil</numeroOrigen>
                     </info>
                  </head>
                  <body>
                     <contentOut>Contrasenya : 468775
@valid-pre.aoc.cat #468775</contentOut>
                  </body>
               </message>
            ]]>
         </string>
      </adap:putMessage>
   </soapenv:Body>
</soapenv:Envelope>


```

### Evidències generades en la validació amb Cl@ve <a name="10"></a>

En cas de validació d'identitat amb el sistema Cl@ve s'enregistren com evidència tant el tiquet SAML generat per VALId en el moment d'iniciar l'autenticació com el tiquet SAML signat per Cl@ve un cop l'usuari s'ha autenticat correctament (ambdós tiquets codificats en Base64).

| _Exemple petició – inici d'autenticació (tiquet SAML generat per VALId)_ |
| --- |
```xml
<?xml version="1.0" encoding="UTF-8"?>
<saml2p:AuthnRequest xmlns:saml2p="urn:oasis:names:tc:SAML:2.0:protocol"
xmlns:ds="http://www.w3.org/2000/09/xmldsig#" xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion"
xmlns:stork="urn:eu:stork:names:tc:STORK:1.0:assertion" xmlns:storkp="urn:eu:stork:names:tc:STORK:1.0:protocol"
AssertionConsumerServiceURL="https://accounts-dev.aoc.cat/o/oauth2/auth"
Consent="urn:oasis:names:tc:SAML:2.0:consent:unspecified" ForceAuthn="true" ID="_a5f059d8088e7d4ac0c2f987e81bbc37"
IsPassive="false" IssueInstant="2015-07-07T11:38:06.290Z" ProtocolBinding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
ProviderName="DEMO-SP" Version="2.0">
	<saml2:Issuer Format="urn:oasis:names:tc:SAML:2.0:nameid-format:entity">http://S-PEPS.gov.xx</saml2:Issuer>
	<ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
		<ds:SignedInfo>
			<ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
			<ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"/>
			<ds:Reference URI="#_a5f059d8088e7d4ac0c2f987e81bbc37">
				<ds:Transforms>
					<ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
					<ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
				</ds:Transforms>
				<ds:DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1"/>
				<ds:DigestValue>G/i0BVuAPRgxRQXEKlty5q76G04=</ds:DigestValue>
			</ds:Reference>
		</ds:SignedInfo>
		<ds:SignatureValue>WCJ2v2OQz/MvJSfkV84Hs/h (...) JOBdD/0e6fKrSLw==</ds:SignatureValue>
		<ds:KeyInfo>
			<ds:X509Data>
				<ds:X509Certificate>MIIDezCCAmMC(...)Fd0ug==</ds:X509Certificate>
			</ds:X509Data>
		</ds:KeyInfo>
	</ds:Signature>
	<saml2p:Extensions>
		<stork:QualityAuthenticationAssuranceLevel>3</stork:QualityAuthenticationAssuranceLevel>
		<stork:spSector>DEMO-SP</stork:spSector>
		<stork:spInstitution>DEMO-SP</stork:spInstitution>
		<stork:spApplication>DEMO-SP</stork:spApplication>
		<storkp:eIDSectorShare>true</storkp:eIDSectorShare>
		<storkp:eIDCrossSectorShare>true</storkp:eIDCrossSectorShare>
		<storkp:eIDCrossBorderShare>true</storkp:eIDCrossBorderShare>
		<storkp:RequestedAttributes>
			<stork:RequestedAttribute Name="http://www.stork.gov.eu/1.0/eIdentifier"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" isRequired="true"/>
			<stork:RequestedAttribute Name="http://www.stork.gov.eu/1.0/givenName"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" isRequired="true"/>
			<stork:RequestedAttribute Name="http://www.stork.gov.eu/1.0/dateOfBirth"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" isRequired="false"/>
			<stork:RequestedAttribute Name="http://www.stork.gov.eu/1.0/eMail"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" isRequired="false"/>
			<stork:RequestedAttribute Name="http://www.stork.gov.eu/1.0/citizenQAALevel"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" isRequired="false"/>
			<stork:RequestedAttribute Name="http://www.stork.gov.eu/1.0/fiscalNumber"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" isRequired="false"/>
			<stork:RequestedAttribute Name="http://www.stork.gov.eu/1.0/nationalityCode"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" isRequired="false"/>
			<stork:RequestedAttribute Name="http://www.stork.gov.eu/1.0/surname"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" isRequired="true"/>
			<stork:RequestedAttribute Name="http://www.stork.gov.eu/1.0/canonicalResidenceAddress"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri" isRequired="false"/>
		</storkp:RequestedAttributes>
		<storkp:AuthenticationAttributes>
			<storkp:VIDPAuthenticationAttributes>
				<storkp:SPInformation>
					<storkp:SPID>DEMO-SP</storkp:SPID>
				</storkp:SPInformation>
			</storkp:VIDPAuthenticationAttributes>
		</storkp:AuthenticationAttributes>
	</saml2p:Extensions>
</saml2p:AuthnRequest>

```

| _Exemple resposta – resultat d'autenticació (tiquet SAML generat per Cl@ve)_ |
| --- |
```xml
<?xml version="1.0" encoding="UTF-8"?>
<saml2p:Response xmlns:saml2p="urn:oasis:names:tc:SAML:2.0:protocol"
xmlns:ds="http://www.w3.org/2000/09/xmldsig#" xmlns:saml2="urn:oasis:names:tc:SAML:2.0:assertion"
xmlns:stork="urn:eu:stork:names:tc:STORK:1.0:assertion"
xmlns:storkp="urn:eu:stork:names:tc:STORK:1.0:protocol" xmlns:xs="http://www.w3.org/2001/XMLSchema"
Consent="urn:oasis:names:tc:SAML:2.0:consent:obtained"
Destination="https://accounts-dev.aoc.cat/o/oauth2/auth"
ID="_e6f0344f0780971784c8f8129b05241e" InResponseTo="_a5f059d8088e7d4ac0c2f987e81bbc37"
IssueInstant="2015-07-07T11:39:12.495Z" Version="2.0">
	<saml2:Issuer Format="urn:oasis:names:tc:SAML:2.0:nameid-format:entity">PIN24H</saml2:Issuer>
	<ds:Signature>
		<ds:SignedInfo>
			<ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
			<ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"/>
			<ds:Reference URI="#_e6f0344f0780971784c8f8129b05241e">
				<ds:Transforms>
					<ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
					<ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#">
						<ec:InclusiveNamespaces xmlns:ec="http://www.w3.org/2001/10/xml-exc-c14n#"
						PrefixList="xs"/>
					</ds:Transform>
				</ds:Transforms>
				<ds:DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1"/>
				<ds:DigestValue>jHyaPRRoamBRcX4VDNNRemqrG2g=</ds:DigestValue>
			</ds:Reference>
		</ds:SignedInfo>
		<ds:SignatureValue>LJjhjYxl4dDH0WRBAT(...)0M/ITwPs=</ds:SignatureValue>
		<ds:KeyInfo>
			<ds:X509Data>
				<ds:X509Certificate>MIIEHTCCA(...)rJ6Xw==</ds:X509Certificate>
			</ds:X509Data>
		</ds:KeyInfo>
	</ds:Signature>
	<saml2p:Status>
		<saml2p:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success"/>
		<saml2p:StatusMessage>urn:oasis:names:tc:SAML:2.0:status:Success</saml2p:StatusMessage>
	</saml2p:Status>
	<saml2:Assertion ID="_5654123081c8637a7790e0adf32e89c0" IssueInstant="2015-07-07T11:39:12.496Z"
	Version="2.0">
		<saml2:Issuer Format="urn:oasis:names:tc:SAML:2.0:nameid-format:entity">PIN24H</saml2:Issuer>
		<ds:Signature>
			<ds:SignedInfo>
				<ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
				<ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"/>
				<ds:Reference URI="#_5654123081c8637a7790e0adf32e89c0">
					<ds:Transforms>
						<ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
						<ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#">
							<ec:InclusiveNamespaces xmlns:ec="http://www.w3.org/2001/10/xml-exc-c14n#"
							PrefixList="xs"/>
						</ds:Transform>
					</ds:Transforms>
					<ds:DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1"/>
					<ds:DigestValue>BBz+aqijg244tgP48tBA95ap5i4=</ds:DigestValue>
				</ds:Reference>
			</ds:SignedInfo>
			<ds:SignatureValue>o7+az5hWA(...)ZcM+tlipr8=</ds:SignatureValue>
			<ds:KeyInfo>
				<ds:X509Data>
					<ds:X509Certificate>MIIEHTCC(...)IMrJ6Xw==</ds:X509Certificate>
				</ds:X509Data>
			</ds:KeyInfo>
		</ds:Signature>
		<saml2:Subject>
			<saml2:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified"
          NameQualifier="http://C-PEPS.gov.xx">
          urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified</saml2:NameID>
			<saml2:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
				<saml2:SubjectConfirmationData Address="https://accounts-dev.aoc.cat/o/oauth2/auth"
				InResponseTo="_a5f059d8088e7d4ac0c2f987e81bbc37" NotOnOrAfter="2015-07-07T11:44:12.496Z"
				Recipient="https://accounts-dev.aoc.cat/o/oauth2/auth"/>
			</saml2:SubjectConfirmation>
		</saml2:Subject>
		<saml2:Conditions NotBefore="2015-07-07T11:39:12.496Z" NotOnOrAfter="2015-07-07T11:44:12.496Z">
			<saml2:AudienceRestriction>
				<saml2:Audience>http://S-PEPS.gov.xx</saml2:Audience>
			</saml2:AudienceRestriction>
			<saml2:OneTimeUse/>
		</saml2:Conditions>
		<saml2:AuthnStatement AuthnInstant="2015-07-07T11:39:12.496Z">
			<saml2:SubjectLocality Address="https://accounts-dev.aoc.cat/o/oauth2/auth"/>
			<saml2:AuthnContext>
				<saml2:AuthnContextDecl/>
			</saml2:AuthnContext>
		</saml2:AuthnStatement>
		<saml2:AttributeStatement>
			<saml2:Attribute Name="http://www.stork.gov.eu/1.0/eIdentifier"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
				<saml2:AttributeValue xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
				xsi:type="xs:anyType">ES/ES/DNI</saml2:AttributeValue>
			</saml2:Attribute>
			<saml2:Attribute Name="http://www.stork.gov.eu/1.0/givenName"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
				<saml2:AttributeValue xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
				xsi:type="xs:anyType">ROGER</saml2:AttributeValue>
			</saml2:Attribute>
			<saml2:Attribute Name="http://www.stork.gov.eu/1.0/dateOfBirth"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri"/>
			<saml2:Attribute Name="http://www.stork.gov.eu/1.0/eMail"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
				<saml2:AttributeValue xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
				xsi:type="xs:anyType">email@aoc.cat</saml2:AttributeValue>
			</saml2:Attribute>
			<saml2:Attribute Name="http://www.stork.gov.eu/1.0/citizenQAALevel"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
				<saml2:AttributeValue xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
				xsi:type="xs:anyType">03</saml2:AttributeValue>
			</saml2:Attribute>
			<saml2:Attribute Name="http://www.stork.gov.eu/1.0/fiscalNumber"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri"/>
			<saml2:Attribute Name="http://www.stork.gov.eu/1.0/nationalityCode"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri"/>
			<saml2:Attribute Name="http://www.stork.gov.eu/1.0/surname"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri">
				<saml2:AttributeValue xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
				xsi:type="xs:anyType">NOGUERA ARNAU</saml2:AttributeValue>
			</saml2:Attribute>
			<saml2:Attribute Name="http://www.stork.gov.eu/1.0/canonicalResidenceAddress"
			NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri"/>
		</saml2:AttributeStatement>
	</saml2:Assertion>
</saml2p:Response>

```
