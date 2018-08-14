---
title: "1С магия XDTO-пакетов на примере интеграций с ГИС ЖКХ"
date: 2017-02-13
description : ""
tags : [
    "1c",
    "XML",
    "XDTO"
]
image : ""
categories : []
---
Есть очень много статей о том, как работать с XSL/XSD из 1С, но все они в стиле: возьмем нашу XSD схему (простую и удбоную) или наш web-сервис и смотрите, как все легко экспортировать или импортировать. А что делать, если нам дали пачку XSD-схем со сложным взаимосвязями и изменять мы них не можем, а работать и поддерживать актуальность схем надо? 

<!--more-->

{{< figure src="https://habrastorage.org/files/9dc/a04/d67/9dca04d67c744c61851fe829e8a5fed9.JPG"  class="float-right" >}}

**Сразу скажу, вопросы шифрования/подписи по ГОСТУ при работе с ГИС ЖКХ за рамками этой статьи. Хотя без подписей запросы выполнить не удастся.**

Начнем с простого - скачаем пакет форматов <a href="https://dom.gosuslugi.ru/#!/regulations">по интеграционному взаимодействию  с ГИС ЖКХ</a>, импортируем все xsd схемы из пакета интеграций, наведем порядок переименуем все как нам удобно. В итоге получим как показано на картинке: 

Ну а теперь приступим к магии. Попробуем запросить данные из справочника организаций по ОРГН. Это подсистема organizations-registry-common метод exportOrgRegist.

В hcs-organizations-registry-common-service.wsdl указано:


```xml
...
<wsdl:message name="exportOrgRegistryRequest">
  <wsdl:part name="exportOrgRegistryRequest" element="ro:exportOrgRegistryRequest"/>
</wsdl:message>
<wsdl:message name="exportOrgRegistryResult">
  <wsdl:part name="exportOrgRegistryResult" element="ro:exportOrgRegistryResult"/>
</wsdl:message>
...
<wsdl:message name="ISRequestHeader">
  <wsdl:part name="Header" element="ISRequestHeader"/>
</wsdl:message>

...

<wsdl:operation name="exportOrgRegistry">
  <wsdl:documentation>экспорт сведений об организациях</wsdl:documentation>
  <wsdl:input message="tns:exportOrgRegistryRequest"/>
  <wsdl:output message="tns:exportOrgRegistryResult"/>
  <wsdl:fault name="InvalidRequest" message="tns:Fault"/>
</wsdl:operation>

...

<wsdl:operation name="exportOrgRegistry">
  <soap:operation soapAction="urn:exportOrgRegistry"/>
    <wsdl:input>
      <soap:body use="literal"/>
      <soap:header message="tns:ISRequestHeader" part="Header" use="literal"/>
    </wsdl:input>
    <wsdl:output>
      <soap:body use="literal"/>
      <soap:header message="tns:ResultHeader" part="Header" use="literal"/>
    </wsdl:output>
    <wsdl:fault name="InvalidRequest">
      <soap:fault name="InvalidRequest" use="literal"/>
    </wsdl:fault>
</wsdl:operation>
```


Надо собрать SOAP пакет из заголовка ISRequestHeader, тела exportOrgRegistryRequest. Посмотрим их в xsd схемах  спецификаций по интеграций.

<details>
```xml
...
  <xs:element name="ISRequestHeader" type="tns:HeaderType">
    <xs:annotation>
      <xs:documentation>Заголовок запроса</xs:documentation>
    </xs:annotation>
    </xs:element> 
...
   <xs:complexType name="HeaderType">
      <xs:annotation>
         <xs:documentation>Базовый тип заголовка</xs:documentation>
      </xs:annotation>
   <xs:sequence>
      <xs:element name="Date" type="xs:dateTime">
         <xs:annotation>
            <xs:documentation>Дата отправки пакета</xs:documentation>
         </xs:annotation>
      </xs:element>
      <xs:element name="MessageGUID" type="tns:GUIDType">
         <xs:annotation>
            <xs:documentation>Идентификатор сообщения</xs:documentation>
         </xs:annotation>
      </xs:element>
      </xs:sequence>
   </xs:complexType>    
...
   <xs:simpleType name="GUIDType">
      <xs:annotation>
         <xs:documentation>GUID-тип.</xs:documentation>
      </xs:annotation>
      <xs:restriction base="xs:string">
         <xs:pattern value="([0-9a-fA-F]){8}-([0-9a-fA-F]){4}-([0-9a-fA-F]){4}-([0-9a-fA-F]){4}-([0-9a-fA-F]){12}"/>
      </xs:restriction>
   </xs:simpleType>
...
</source></spoiler>
<spoiler title="hcs-organizations-base.xsd"><source lang="xml">
...
   <xs:element name="OGRN" type="tns:OGRNType">
      <xs:annotation>
         <xs:documentation>ОГРН</xs:documentation>
      </xs:annotation>
   </xs:element>
   <xs:simpleType name="OGRNType">
      <xs:restriction base="xs:string">
         <xs:length value="13"/>
      </xs:restriction>
   </xs:simpleType>
...
</source></spoiler>
<spoiler title="hcs-organizations-registry-common-types.xsd"><source lang="xml">
...
<!--Экспорт из реестра организаций-->
  <xs:element name="exportOrgRegistryRequest">
    <xs:annotation>
      <xs:documentation>Экспорт сведений из реестра организаций</xs:documentation>
    </xs:annotation>
    <xs:complexType>
      <xs:complexContent>
        <xs:extension base="base:BaseType">
          <xs:sequence>
            <xs:element name="SearchCriteria" maxOccurs="100">
              <xs:annotation>
                <xs:documentation>Критерий поиска организаций.</xs:documentation>
              </xs:annotation>
              <xs:complexType>
                <xs:sequence>
                  <xs:choice>
                    <xs:choice>
                      <xs:annotation>
                        <xs:documentation>Поиск по реквизитам.</xs:documentation>
                      </xs:annotation>
                      <xs:element ref="organizations-base:OGRNIP"/>
                      <xs:sequence>
                        <xs:element ref="organizations-base:OGRN"/>
                        <xs:element ref="organizations-base:KPP" minOccurs="0"/>
                      </xs:sequence>
                      <xs:element ref="organizations-base:NZA"/>
                    </xs:choice>
                    <xs:element ref="organizations-registry-base:orgVersionGUID"/>
                    <xs:element ref="organizations-registry-base:orgRootEntityGUID"/>
                  </xs:choice>
                  <xs:element name="isRegistered" type="xs:boolean" fixed="true" minOccurs="0">
                    <xs:annotation>
                      <xs:documentation>Поиск среди организаций, имеющих личных кабинет</xs:documentation>
                    </xs:annotation>
                  </xs:element>
                </xs:sequence>
              </xs:complexType>
            </xs:element>
            <xs:element name="lastEditingDateFrom" type="xs:date" minOccurs="0">
              <xs:annotation>
                <xs:documentation>Время последнего изменения (от)</xs:documentation>
              </xs:annotation>
            </xs:element>
          </xs:sequence>
          <xs:attribute ref="base:version" use="required" fixed="10.0.2.1"/>
        </xs:extension>
      </xs:complexContent>
    </xs:complexType>
  </xs:element>
  <xs:element name="exportOrgRegistryResult">
    <xs:complexType>
      <xs:complexContent>
        <xs:extension base="base:BaseType">
          <xs:choice>
            <xs:element ref="base:ErrorMessage"/>
            <xs:element name="OrgData" type="tns:exportOrgRegistryResultType" minOccurs="0" maxOccurs="unbounded">
              <xs:annotation>
                <xs:documentation>Найденная организация.</xs:documentation>
              </xs:annotation>
            </xs:element>
          </xs:choice>
          <xs:attribute ref="base:version" use="required" fixed="10.0.2.1"/>
        </xs:extension>
      </xs:complexContent>
    </xs:complexType>
  </xs:element>
  <xs:complexType name="exportOrgRegistryResultType">
    <xs:sequence>
      <xs:element ref="organizations-registry-base:orgRootEntityGUID"/>
      <xs:element name="OrgVersion">
        <xs:annotation>
          <xs:documentation>Версия организации в реестре организаций</xs:documentation>
        </xs:annotation>
        <xs:complexType>
          <xs:sequence>
            <xs:element ref="organizations-registry-base:orgVersionGUID"/>
            <xs:element name="lastEditingDate" type="xs:date">
              <xs:annotation>
                <xs:documentation>Время последнего изменения</xs:documentation>
              </xs:annotation>
            </xs:element>
            <xs:element name="IsActual" type="xs:boolean">
              <xs:annotation>
                <xs:documentation>Признак актуальности записи</xs:documentation>
              </xs:annotation>
            </xs:element>
            <xs:choice>
              <xs:element name="Legal" type="organizations-registry-base:LegalType">
                <xs:annotation>
                  <xs:documentation>Юридическое лицо</xs:documentation>
                </xs:annotation>
              </xs:element>
              <xs:element name="Subsidiary">
                <xs:annotation>
                  <xs:documentation>Обособленное подразделение</xs:documentation>
                </xs:annotation>
                <xs:complexType>
                  <xs:complexContent>
                    <xs:extension base="organizations-registry-base:SubsidiaryType">
                      <xs:sequence>
                        <xs:element name="StatusVersion">
                          <xs:annotation>
                            <xs:documentation>Статус версии </xs:documentation>
                          </xs:annotation>
                          <xs:simpleType>
                            <xs:restriction base="xs:string"/>
                          </xs:simpleType>
                        </xs:element>
                        <xs:element name="ParentOrg">
                          <xs:annotation>
                            <xs:documentation>Информация о головной организации</xs:documentation>
                          </xs:annotation>
                          <xs:complexType>
                            <xs:sequence>
                              <xs:element ref="organizations-registry-base:RegOrgVersion"/>
                            </xs:sequence>
                          </xs:complexType>
                        </xs:element>
                      </xs:sequence>
                    </xs:extension>
                  </xs:complexContent>
                </xs:complexType>
              </xs:element>
              <xs:element name="Entrp" type="organizations-registry-base:EntpsType">
                <xs:annotation>
                  <xs:documentation>Индивидуальный предприниматель</xs:documentation>
                </xs:annotation>
              </xs:element>
              <xs:element name="ForeignBranch" type="organizations-registry-base:ForeignBranchType">
                <xs:annotation>
                  <xs:documentation>ФПИЮЛ (Филиал или представительство иностранного юридического лица)</xs:documentation>
                </xs:annotation>
              </xs:element>
            </xs:choice>
            <xs:element name="registryOrganizationStatus" minOccurs="0">
              <xs:annotation>
                <xs:documentation>Статус:
(P)UBLISHED - опубликована в одном из документов в рамках раскрытия</xs:documentation>
              </xs:annotation>
              <xs:simpleType>
                <xs:restriction base="xs:string">
                  <xs:enumeration value="P"/>
                </xs:restriction>
              </xs:simpleType>
            </xs:element>
          </xs:sequence>
        </xs:complexType>
      </xs:element>
      <xs:element ref="base:orgPPAGUID" minOccurs="0"/>
      <xs:element name="organizationRoles" type="nsi-base:nsiRef" minOccurs="0" maxOccurs="unbounded">
        <xs:annotation>
          <xs:documentation>Полномочие организации (НСИ №20)</xs:documentation>
        </xs:annotation>
      </xs:element>
      <xs:element name="isRegistered" type="xs:boolean" fixed="true" minOccurs="0">
        <xs:annotation>
          <xs:documentation>Зарегистрирована в ГИС ЖКХ</xs:documentation>
        </xs:annotation>
      </xs:element>
    </xs:sequence>
  </xs:complexType>
...
```
</details>

Ну приступим, откроем нужные нам пакеты XDTO. Оказывается, нужные сущности являются не типами, а свойствами, как с этим работать в документации на XDTO в статьях, которые я находил, не описано, поэтому воспользуемся урокам магии:

* Создадим объекты из свойств
* Создадим объекты из внутренних типов свойств или тип объектов
* Создадим объекты из Типов значений


<img src="https://habrastorage.org/files/67c/639/535/67c639535ea749f5b72eee461fd05a5b.JPG" alt="image"/>


Начнем с тела exportOrgRegistryRequest.


```
&НаСервереБезКонтекста
Функция ПолучитьДанныеОрганизацияПоОГРН(ОГРН)
  //Получим нужные пакеты
  ПакетOrgRegCom = ФабрикаXDTO.Пакеты.Получить("http://dom.gosuslugi.ru/schema/integration/organizations-registry-common/");
  ПакетOrgBase = ФабрикаXDTO.Пакеты.Получить("http://dom.gosuslugi.ru/schema/integration/organizations-base/");

  //Магия:выйдем на нужный тип через свойство, этот метод нам очень часто пригодится:
  СвойствоXDTO = ПакетOrgRegCom.КорневыеСвойства.Получить("exportOrgRegistryRequest");
  //Создадим сам объект для тела запроса
  ОбъектXDTO = ФабрикаXDTO.Создать(СвойствоXDTO.Тип);
  //признак что это элемент надо подписать сертификатом
  ОбъектXDTO.Id = "signed-data-container";
  //требования стандарта, импорт XDTO не обрабатывает аттрибут fixed 
  ОбъектXDTO.version = "10.0.2.1";

  //критерий поиска
  //Магия:опять выйдем на строенный тип, через свойства объекта
  SearchCriteriaЗапись = ФабрикаXDTO.Создать(ОбъектXDTO.SearchCriteria.ВладеющееСвойство.Тип);
	
  //огрн надо ставить через типизированное значение
  OGRN = ФабрикаXDTO.Создать(ПакетOrgBase.КорневыеСвойства.Получить("OGRN").Тип, Строка(ОГРН));
  SearchCriteriaЗапись.OGRN = OGRN;
  ОбъектXDTO.SearchCriteria.Добавить(SearchCriteriaЗапись);
	
  //сохраним тело
  Запрос = Новый Структура("ОбъектXDTO,СвойствоXDTO,Имя", ОбъектXDTO, СвойствоXDTO);
  //сохраним тип ответа, нам по надобиться для десериализаций ответа
  ОтветСвойствоXDTOResult = ПакетOrgRegCom.КорневыеСвойства.Получить("exportOrgRegistryResult");
	
  Возврат СформироватьXMLЗапрос(Новый УникальныйИдентификатор, Запрос, ОтветСвойствоXDTOResult );
КонецФункции
```

Напишем функцию для сбора XML-запроса:


```
&НаСервереБезКонтекста
Функция СформироватьXMLЗапрос(GuidЗаголовка, ОтправкаXDTO, ОтветXDTO)
  ПакетBase = ФабрикаXDTO.Пакеты.Получить("http://dom.gosuslugi.ru/schema/integration/base/");
  ПространствоИменSOAP = "http://schemas.xmlsoap.org/soap/envelope/";
  //XML файл	
  ЗаписьXML = Новый ЗаписьXML;
  ПараметрыЗаписиXML = Новый ПараметрыЗаписиXML("UTF-8");
  ЗаписьXML.УстановитьСтроку(ПараметрыЗаписиXML);
  ЗаписьXML.ЗаписатьОбъявлениеXML();
    ЗаписьXML.ЗаписатьНачалоЭлемента("Envelope", ПространствоИменSOAP);
    ЗаписьXML.ЗаписатьСоответствиеПространстваИмен("soap", ПространствоИменSOAP);
    ЗаписьXML.ЗаписатьСоответствиеПространстваИмен("xsi", "http://www.w3.org/2001/XMLSchema-instance");
    ЗаписьXML.ЗаписатьСоответствиеПространстваИмен("xs", "http://www.w3.org/2001/XMLSchema");
		
    ЗаписьXML.ЗаписатьСоответствиеПространстваИмен("base", "http://dom.gosuslugi.ru/schema/integration/base/");
    ЗаписьXML.ЗаписатьСоответствиеПространстваИмен("organizations-registry-base", "http://dom.gosuslugi.ru/schema/integration/organizations-registry-base/"); 
    ЗаписьXML.ЗаписатьСоответствиеПространстваИмен("ns", "http://www.w3.org/2000/09/xmldsig#");
    ЗаписьXML.ЗаписатьСоответствиеПространстваИмен("ro", "http://dom.gosuslugi.ru/schema/integration/organizations-registry-common/");
		
     //Заголовок ISRequestHeader
    ЗаписьXML.ЗаписатьНачалоЭлемента("Header", ПространствоИменSOAP);
      //снова метод через свойства
      ЗаголовокСвойствоXDTO = ПакетBase.КорневыеСвойства.Получить("ISRequestHeader");
      ЗаголовокЗапроса = ФабрикаXDTO.Создать(ЗаголовокСвойствоXDTO.Тип);
      ЗаголовокЗапроса.Date = ТекущаяДата();
      ЗаголовокЗапроса.MessageGUID = ФабрикаXDTO.Создать(ФабрикаXDTO.Тип("http://dom.gosuslugi.ru/schema/integration/base/", "GUIDType"), Строка(GuidЗаголовка));
      ФабрикаXDTO.ЗаписатьXML(ЗаписьXML, ЗаголовокЗапроса, ЗаголовокСвойствоXDTO.ЛокальноеИмя);
    ЗаписьXML.ЗаписатьКонецЭлемента();	
    //Тело у нас будет подготовленый exportOrgRegistryRequest
    ЗаписьXML.ЗаписатьНачалоЭлемента("Body", ПространствоИменSOAP);
      ФабрикаXDTO.ЗаписатьXML(ЗаписьXML, ОтправкаXDTO.ОбъектXDTO, ОтправкаXDTO.СвойствоXDTO.ЛокальноеИмя, ОтправкаXDTO.СвойствоXDTO.URIПространстваИмен);
      ЗаписьXML.ЗаписатьКонецЭлемента();
  ЗаписьXML.ЗаписатьКонецЭлемента();
  //отдаем результат для отправки
  XMLЗапрос = Новый Структура();
  XMLЗапрос.Вставить("XMLTекст", ЗаписьXML.Закрыть());
  XMLЗапрос.Вставить("ОтветXDTO", ОтветXDTO);
  Возврат XMLЗапрос;
КонецФункции
```

В итоге получим запрос:


```xml
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:base="http://dom.gosuslugi.ru/schema/integration/base/" xmlns:hm="http://dom.gosuslugi.ru/schema/integration/house-management/" xmlns:ns="http://www.w3.org/2000/09/xmldsig#" xmlns:nsi-base="http://dom.gosuslugi.ru/schema/integration/nsi-base/" xmlns:nsi-common="http://dom.gosuslugi.ru/schema/integration/nsi-common/" xmlns:organizations-registry-base="http://dom.gosuslugi.ru/schema/integration/organizations-registry-base/" xmlns:ro="http://dom.gosuslugi.ru/schema/integration/organizations-registry-common/" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:tns="http://dom.gosuslugi.ru/schema/integration/house-management-service/" xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <soap:Header>
    <base:ISRequestHeader>
      <base:Date>2016-10-29T20:06:35</base:Date>
      <base:MessageGUID>8120c453-d4ee-4918-84f8-276257bb2c84</base:MessageGUID>
      </base:ISRequestHeader>
  </soap:Header>
  <soap:Body>
    <ro:exportOrgRegistryRequest Id="signed-data-container" base:version="10.0.2.1">
      <!--в боевых запросах тут большой кусок подписи запроса, куски сертификатов и прочие-->
      <ro:SearchCriteria>
        <OGRN xmlns="http://dom.gosuslugi.ru/schema/integration/organizations-base/">1027700132195</OGRN>
      </ro:SearchCriteria>
    </ro:exportOrgRegistryRequest>
  </soap:Body>
</soap:Envelope>
</source>
```


Отправим запрос:


```
&НаСервере
Процедура ТестоваяОтправкаНаСервере()
  //Узнаем данные по Сбербанку
  XMLЗапрос = ПолучитьДанныеОрганизацияПоОГРН("1027700132195");
  //Это тема для другой статья
  //тут мы подписываем пакет данных
  //и отправляем Post запрос на сервис 
  XMLОтвет = ОтправкаXMLЗапроса(XMLЗапрос);
  Если XMLОтвет.КодОтвета < 299 Тогда
    //запрос успешен
    ОбъектXDTO = ДесериализацияОтвета(XMLОтвет.ИмяФайлРезультата, XMLЗапрос.ОтветXDTO)
  КонецЕсли; 
КонецПроцедуры
```


Ответ от серверов ГИС ЖКХ (СИТ-1):


```xml
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:ns10="http://dom.gosuslugi.ru/schema/integration/organizations-base/" xmlns:ns11="http://dom.gosuslugi.ru/schema/integration/payments-base/" xmlns:ns12="http://dom.gosuslugi.ru/schema/integration/bills-base/" xmlns:ns13="http://dom.gosuslugi.ru/schema/integration/organizations-registry-common/" xmlns:ns3="http://www.w3.org/2000/09/xmldsig#" xmlns:ns4="http://dom.gosuslugi.ru/schema/integration/base/" xmlns:ns5="http://dom.gosuslugi.ru/schema/integration/account-base/" xmlns:ns6="http://dom.gosuslugi.ru/schema/integration/nsi-base/" xmlns:ns7="http://dom.gosuslugi.ru/schema/integration/individual-registry-base/" xmlns:ns8="http://dom.gosuslugi.ru/schema/integration/metering-device-base/" xmlns:ns9="http://dom.gosuslugi.ru/schema/integration/organizations-registry-base/" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Header>
    <ns4:ResultHeader>
    <ns4:Date>2016-10-29T20:06:37.185+03:00</ns4:Date>
      <ns4:MessageGUID>8120c453-d4ee-4918-84f8-276257bb2c84</ns4:MessageGUID>
    </ns4:ResultHeader>
    </soap:Header>
    <soap:Body>
        <ns13:exportOrgRegistryResult Id="signed-data-container" ns4:version="10.0.2.1">
            <!--опистул кусок подписи пакета он не особо интересен-->
            <ns13:OrgData>
                <ns9:orgRootEntityGUID>50a8619b-2d27-4f20-8233-eab1ccf9dffe</ns9:orgRootEntityGUID>
                <ns13:OrgVersion>
                    <ns9:orgVersionGUID>50a8619b-2d27-4f20-8233-eab1ccf9dffe</ns9:orgVersionGUID>
                    <ns13:lastEditingDate>2015-08-10+03:00</ns13:lastEditingDate>
                    <ns13:IsActual>true</ns13:IsActual>
                    <ns13:Legal>
                        <ns9:ShortName>ОАО «Сбербанк России»</ns9:ShortName>
                        <ns9:FullName>ОАО «Сбербанк России»</ns9:FullName>
                        <ns10:OGRN>1027700132195</ns10:OGRN>
                        <ns9:StateRegistrationDate>2015-08-10+03:00</ns9:StateRegistrationDate>
                        <ns10:INN>7707083893</ns10:INN>
                        <ns10:KPP>775001001</ns10:KPP>
                        <ns10:OKOPF>12247</ns10:OKOPF>
                        <ns9:Address>г. Москва, ул. Вавилова, д. 19</ns9:Address>
                        <ns9:FIASHouseGuid>93409d8c-d8d4-4491-838f-f9aa1678b5e6</ns9:FIASHouseGuid>
                    </ns13:Legal>
                    <ns13:registryOrganizationStatus>P</ns13:registryOrganizationStatus>
                </ns13:OrgVersion>
                <ns4:orgPPAGUID>2c57ed5e-583a-4471-839e-776250bdde50</ns4:orgPPAGUID>
                <ns13:organizationRoles>
                    <ns6:Code>1</ns6:Code>
                    <ns6:GUID>9875cc2e-73f9-41d6-bceb-47b48ed23395</ns6:GUID>
                    <ns6:Name>Управляющая организация</ns6:Name>
                </ns13:organizationRoles>
                <ns13:isRegistered>true</ns13:isRegistered>
            </ns13:OrgData>
        </ns13:exportOrgRegistryResult>
    </soap:Body>
</soap:Envelope>
```

Как мы видим, ответ напрямую десериализовать не получится, потому что нет такого типа в предложенных xsd схемах. Нам надо как-то пропустить часть тэгов и обработать только область ответа. На эту тему я тоже не нашел информации, но методом проб и ошибок приходим к кусочку магий:
	

```
&НаСервереБезКонтекста
Функция ДесериализацияОтвета(ПутьКФайлу, ОтветXDTO)
  //Откроем файл
  Чтение = новый ЧтениеXML;
  Чтение.ОткрытьФайл(ПутьКФайлу, , , "UTF-8"); 
  //Начинаем пропускать все служебное заголовки, тело ищем наш тэг ответа
  Пока Чтение.Прочитать() Цикл
    Если Чтение.ЛокальноеИмя = ОтветXDTO.СвойствоXDTO.Имя Тогда
      //т.к. вы спозиционировались на нужном место можно десериализовать данные. Магия
      ОбъектXDTO = ФабрикаXDTO.ПрочитатьXML(Чтение, ОтветXDTO.СвойствоXDTO.Тип); 
      КонецЕсли; 
  КонецЦикла;
  Возврат ОбъектXDTO;
КонецФункции
```

В итоге работать можно с очень сложными xsd схемами через стандартные инструменты платформы. В целом 1С контролируют типизацию и заполнения, бывает чересчур излишне, особенно когда внутри свойства пакета используется базовый тип другого пакета, но в любом случае тип нужно привести к локальному из-за другого пространства URI. Удобно работать с десериализоваными данными, так как там всю работу на себя берет платформа. Но проверки происходят на этапе выполнения, а при написания кода платформа 1С не предоставляет никаких подсказок и проходится пользоваться сторонними утилитами, и даже при выполнении большая часть элементов находится в состоянии "Неопределено" и даже тип или его свойство можно увидеть только в спецификации.

[habrahabr.ru](https://habrahabr.ru/post/313910/)