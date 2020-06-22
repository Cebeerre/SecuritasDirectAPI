# General

Placeholder to document the Verisure Europe (Securitas Direct) Mobile app API Endpoint.

**_NOTE:_** THIS PROJECT IS NOT IN ANY WAY ASSOCIATED WITH OR RELATED TO THE SECURITAS DIRECT-VERISURE GROUP COMPANIES. The information here and online is for educational and resource purposes only and therefore the developers do not endorse or condone any inappropriate use of it, and take no legal responsibility for the functionality or security of your alarms and devices.

Special thanks to the [mitmproxy](https://github.com/mitmproxy/mitmproxy) team as all the reverse engineering work required to document this API was done with this great opensource tool.

Known projects:

* [Command line client](https://github.com/Cebeerre/VerisureAPIClient)
* [Home Assistant Component](https://github.com/segalion/securitasdirect)
* [WEMO/Belkin Emulation](https://github.com/Cebeerre/VerisureEchoDevice)
* [WIP Homebridge integration](https://github.com/ptz0n/node-verisure/pull/41)
* [Jeedom Plugin](https://github.com/Xav-74/verisure)

Verisure does not really use a REST API, it's more a web application that expects variables in a url encoded format, and returns XML documents.

You should maintain a session using a hash, control (some) queries using IDs, and control the execution in the server querying the log on the endpoint periodically for some calls.

```
BASE URL = https://mob2217.securitasdirect.es:12010/WebService/ws.do
```

Known supported countries:
* Spain (Country="ES", lang="es")
* Italy (Country="IT", lang="it")
* France (Country="FR", lang="fr")
* UK (Country="GB", lang="en") (only the alarm kit on the [left](https://www.verisure.co.uk/alarms/customer-area.html))
* Portugal (Country="PT", lang="pt")

Calls and Actions:

* [Login](#login)
* [Installation Details](#installation-details)
* [SIM Number and INSTIBS](#sim-number-and-instibs)
* [Get Panel Status](#get-panel-status)
* [Arm Alarm System (inside, all sensors)](#arm-alarm-system--inside--all-sensors-)
* [Arm Alarm System (outside, perimeter, all sensors)](#arm-alarm-system--outside--perimeter--all-sensors-)
* [Arm Alarm Nigh/Partial Mode](#arm-alarm-nigh-partial-mode)
* [Arm Alarm Day/Partial Mode](#arm-alarm-day-partial-mode)
* [Disarm Alarm (everything)](#disarm-alarm--everything-)
* [Photo Requests](#photo-requests)
* [Logout](#logout)

## Login

**Call**: `LOGIN`

**Method**: `POST`

**Data constraints**

You should provide:

```
{
    'request': 'LOGIN',
    'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]',
    'Country': '[Checked with `ES` for Spain]',
    'lang': '[Checked with `es` for Spain]',
    'user': '[The webpage/app login]',
    'pwd': '[The webpage/app password]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=LOGIN&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass
```

**Success Response**

Code: 200
Example:

```
<?xml version='1.0' encoding='UTF-8'?>
<PET>
    <RES>OK</RES>
    <MSG>Usuario validado correctamente</MSG>
    <LANG>ES</LANG>
    <HASH>11111111111</HASH>
    <MOBILE newversion='0'/>
    <BLOQ remotereqactive='1' >Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```

The content of the HASH tag should be used for any further requests, as it's used as a token.

## Installation Details

Usefull to get the sensor IDs and other info related to your installation.

**Call**: `MYINSTALLATION`

**Method**: `POST`

**Data constraints**

You should provide:

```
{
    'request': 'MYINSTALLATION',
    'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]',
    'Country': '[Checked with `es` for Spain]',
    'lang': '[Checked with `ES` for Spain]',
    'user': '[The webpage/app login]',
    'pwd': '[The webpage/app password]',
    'numinst': '[Your installation ID, you can get this from the App/WebPage]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=MYINSTALLATION&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass
```

**Success Response**

Code: 200
Example:

```
<?xml version='1.0' encoding='UTF-8'?>
<PET>
  <RES>OK</RES>
  <MSG>Petición procesada correctamente</MSG>
  <INSTALLATION alias=" ">
    <DEVICES>
      <DEVICE idDev="1" code="0" alias="Entrada" type="YR" aliasType="Foto detector"/>
      <DEVICE idDev="2" code="2" alias="Comedor" type="YR" aliasType="Foto detector"/>
      <DEVICE idDev="3" code="3" alias="Cocina" type="YR" aliasType="Foto detector"/>
      <DEVICE idDev="4" code="4" alias="Habitacion pepe" type="YR" aliasType="Foto detector"/>
      <DEVICE idDev="5" code="5" alias="Sala de Tortura" type="YR" aliasType="Foto detector"/>
      <DEVICE idDev="6" code="6" alias="Cuarto de limpieza" type="YR" aliasType="Foto detector"/>
    </DEVICES>
    <CODEWORDS securitas="XXXXX" customer="XXXXX" coercion="XXXXXX"/>
    <EMAILS>
      <EMAIL address="XXXXX@XXXXXX.COM" alias="SERVICIOS" language="es_ES" idService="0"/>
    </EMAILS>
    <PHONES>
      <PHONE number="XXXXXXXXX" alias="SERVICIOS" language="es_ES" idService="0"/>
    </PHONES>
    <SERVICESATTRIBUTES>
      <EMAIL>
        <SERVICE description="Petición de imagen" idservice="1"/>
        <SERVICE description="Entradas y salidas" idservice="2"/>
        <SERVICE description="Video petición" idservice="17"/>
        <SERVICE description="Corte de corriente" idservice="23"/>
      </EMAIL>
      <PHONE>
        <SERVICE description="Accesos SMS" idservice="5"/>
      </PHONE>
    </SERVICESATTRIBUTES>
  </INSTALLATION>
</PET>
```
## SIM Number and INSTIBS

This is usefull in order to get the SIM number. You can straight call this number as long as you're doing it from the allowed numbers to make "silent calls" and listen from the alarm panel.

You need this call to get the tag value `INSTIBS`. I haven't been able to understand the meaning of this tag, but is mandatory to do photo requests from the sensors.

**Call**: `SVR`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'SVR',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```
**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=SVR&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

I've removed the response here, as it generates a huge XML with a lot of garbage.

Basically, parse the response to get the values from the tags `SIM` and `INSTIBS`

## Get Panel Status

In order to get the panel status, you should first do an `EST1` call, and then loop on `EST2`, till the communication with the panel has finished and you get a `<RES>OK</RES>`

**Call**: `EST1`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'EST1',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=EST1&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

```
<?xml version="1.0" encoding="UTF-8"?>
<PET>
  <RES>OK</RES>
  <MSG>Su solicitud ha sido enviada</MSG>
  <BLOQ remotereqactive="1" >Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```

**Call**: `EST2`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'EST1',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=EST2&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

```
<?xml version="1.0" encoding="UTF-8"?>
<PET>
  <RES>OK</RES>
  <STATUS>0</STATUS>
  <MSG>Tu Alarma está desconectada</MSG>
  <NUMINST>2423443</NUMINST>
  <BLOQ remotereqactive="1" >Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```

## Arm Alarm System (inside, all sensors)

In order to get arm the panel, you should first do an `ARM1` call, and then loop on `ARM2`, till the communication with the panel has finished and you get a `<RES>OK</RES>`

**Call**: `ARM1`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'ARM1',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=ARM1&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

```
<?xml version="1.0" encoding="UTF-8"?>
<PET>
  <RES>OK</RES>
  <MSG>Su solicitud ha sido enviada</MSG>
  <BLOQ remotereqactive="1" >Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```

**Call**: `ARM2`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'ARM2',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=ARM1&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

```
<?xml version="1.0" encoding="UTF-8"?>
<PET>
  <RES>OK</RES>
  <STATUS>4</STATUS>
  <MSG>Tu Alarma está conectada en modo Total y en el exterior</MSG>
  <NUMINST>2423443</NUMINST>
  <BLOQ remotereqactive="1" >Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```

## Arm Alarm System (outside, perimeter, all sensors)

In order to get arm the panel, you should first do an `PERI1` call, and then loop on `PERI2`, till the communication with the panel has finished and you get a `<RES>OK</RES>`

**Call**: `PERI1`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'PERI1',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=PERI1&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

```
<?xml version="1.0" encoding="UTF-8"?>
<PET>
  <RES>OK</RES>
  <MSG>Su solicitud ha sido enviada</MSG>
  <BLOQ remotereqactive="1" >Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```

**Call**: `PERI2`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'PERI2',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=PERI2&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

```
<?xml version="1.0" encoding="UTF-8"?>
<PET>
  <RES>OK</RES>
  <STATUS>0</STATUS>
  <MSG>Tu Alarma está conectada en modo Total y en el exterior</MSG>
  <NUMINST>2423443</NUMINST>
  <BLOQ remotereqactive="1" >Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```

## Arm Alarm Nigh/Partial Mode

In order to get arm the panel, you should first do an `ARMNIGHT1` call, and then loop on `ARMNIGHT2`, till the communication with the panel has finished and you get a `<RES>OK</RES>`

**Call**: `ARMNIGHT1`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'ARMNIGHT1',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=ARMNIGHT1&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

```
<?xml version="1.0" encoding="UTF-8"?>
<PET>
  <RES>OK</RES>
  <MSG>Su solicitud ha sido enviada</MSG>
  <BLOQ remotereqactive="1" >Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```

**Call**: `ARMNIGHT2`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'ARMNIGHT2',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=ARMNIGHT2&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

```
<?xml version="1.0" encoding="UTF-8"?>
<PET>
  <RES>OK</RES>
  <STATUS>0</STATUS>
  <MSG>Tu Alarma está conectada en modo Noche</MSG>
  <NUMINST>2423443</NUMINST>
  <BLOQ remotereqactive="1" >Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```

## Arm Alarm Day/Partial Mode

In order to get arm the panel, you should first do an `ARMDAY1` call, and then loop on `ARMDAY2`, till the communication with the panel has finished and you get a `<RES>OK</RES>`

**Call**: `ARMDAY1`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'ARMDAY1',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=ARMDAY1&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

```
<?xml version="1.0" encoding="UTF-8"?>
<PET>
  <RES>OK</RES>
  <MSG>Su solicitud ha sido enviada</MSG>
  <BLOQ remotereqactive="1" >Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```

**Call**: `ARMDAY2`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'ARMDAY2',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=ARMDAY2&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

```
<?xml version="1.0" encoding="UTF-8"?>
<PET>
  <RES>OK</RES>
  <STATUS>0</STATUS>
  <MSG>Tu Alarma está conectada en modo Noche</MSG>
  <NUMINST>2423443</NUMINST>
  <BLOQ remotereqactive="1" >Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```

## Disarm Alarm (everything)

In order to get arm the panel, you should first do an `DARM1` call, and then loop on `DARM2`, till the communication with the panel has finished and you get a `<RES>OK</RES>`

**Call**: `DARM1`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'DARM1',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=DARM1&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

```
<?xml version="1.0" encoding="UTF-8"?>
<PET>
  <RES>OK</RES>
  <MSG>Su solicitud ha sido enviada</MSG>
  <BLOQ remotereqactive="1" >Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```

**Call**: `DARM2`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'DARM2',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=DARM2&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

```
<?xml version="1.0" encoding="UTF-8"?>
<PET>
  <RES>OK</RES>
  <STATUS>0</STATUS>
  <MSG>Tu Alarma ha sido desconectada</MSG>
  <NUMINST>2423443</NUMINST>
  <BLOQ remotereqactive="1" >Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```

## Photo Requests

In order to make photo requests and download the pictures, you should first call `IMG1` and as the previous panel interactions, loop through `IMG2` till you get a successful response. Once confirmed, you can get the pictures calling `INF` which will return 3 pictures inside 3 XML tags encoded in BASE64.

Before calling `INF`, you should call `ACT_V2` till you see an entry saying that the pictures are available. From this response, you should get the values in the tags `idsignal` and `signaltype` as you'll need them then to call `INF`.

**Call**: `IMG1`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'IMG1',
  'idservice': '[Seems a constant, in my case is '1' got from MYINSTALLATION],
  'instibs': '[You get this from the SVR call as mentioned previously]',
  'device': '[ID from the device (idDev), you get this calling MYINSTALLATION]',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=IMG1&idservice=1&instibs=34342342&device=1&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

```
text<?xml version='1.0' encoding='UTF-8'?>
text<PET>
text  <RES>OK</RES>
text  <STATUS>0</STATUS>
text  <MSG>La secuencia de imágenes ha sido tomada. En unos minutos podrás visualizarlas desde tu web de cliente, en tu correo electrónico o en la aplicación My Verisure.</MSG>
text  <BLOQ remotereqactive="1">Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
text</PET>
```

**Call**: `IMG2`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'IMG2',
  'idservice': '[Seems a constant, in my case is '1' got from MYINSTALLATION],
  'instibs': '[You get this from the SVR call as mentioned previously]',
  'device': '[ID from the device (idDev), you get this calling MYINSTALLATION]',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=IMG2&idservice=1&instibs=34342342&device=1&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

```
<?xml version='1.0' encoding='UTF-8'?>
<PET>
  <RES>OK</RES>
  <STATUS>0</STATUS>
  <MSG>La secuencia de imágenes ha sido tomada. En unos minutos podrás visualizarlas desde tu web de cliente, en tu correo electrónico o en la aplicación My Verisure.</MSG>
  <BLOQ remotereqactive="1">Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```

**Call**: `INF`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'panel': '[This is your alarm panel, tipically SDVFAST]',
  'numinst': '[Your installation ID, you can get this from the App/WebPage]',
  'request': 'INF',
  'idsignal': '[You can get this from ACT_V2 after a successful IMG2],
  'signaltype': '[You can get this from ACT_V2 after a successful IMG2]',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```
**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=INF&idsignal=1234&signaltype=1234&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&panel=SDVFAST&numinst=2423443
```

**Success Response**

Code: 200
Example:

```
<?xml version='1.0' encoding='UTF-8'?>
<PET>
  <RES>OK</RES>
  <DEVICES numdevices="1">
    <DEVICE id="0" code="6" name="Cuarto de limpieza" numimg="3">
      <IMG id="1">BASE64</IMG>
      <IMG id="2">BASE64</IMG>
      <IMG id="3">BASE64</IMG>
    </DEVICE>
  </DEVICES>
  <BLOQ remotereqactive="1">Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```

## Logout

**Call**: `CLS`

**Method**: `GET`

**Data constraints**

You should provide:

```
{
  'Country': '[Country]',
  'user': '[The webpage/app login]',
  'pwd': '[The webpage/app password]',
  'lang': '[Language]',
  'numinst': '(null)',
  'request': 'CLS',
  'hash': '[The hash/token that you get back in the XML from doing the Login call]',
  'ID': '[Concatenated string with the device type, the username and a time token, should persist along all the session: IPH_________________________vericlient20191029123621]'
}
```

**Data example**

```
https://mob2217.securitasdirect.es:12010/WebService/ws.do?request=CLS&ID=IPH_________________________vericlient20191029123621&Country=ES&lang=es&user=vericlient&pwd=veripass&hash=11111111111&numinst=
```

**Success Response**

Code: 200
Example:

```
<?xml version="1.0" encoding="UTF-8"?>
<PET>
  <RES>OK</RES>
  <MSG>Mensaje de cierre de session</MSG>
  <BLOQ remotereqactive="1" >Estamos mejorando nuestros servicios. Por favor intentelo de nuevo mas tarde. Gracias por confiar en Securitas Direct</BLOQ>
</PET>
```
