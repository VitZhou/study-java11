

# XML Catalog API

使用XML Catalog API实现本地XML Catalog。

Java SE 9引入了一个新的XML Catalog API，以支持结构化信息标准促进组织（OASIS）的[XML catalog，OASIS标准V1.1](https://www.oasis-open.org/committees/download.php/14809/xml-catalogs.html)。 “Oracle JDK 9核心库指南”的这一章描述了API，Java XML处理器的支持以及使用模式。

XML Catalog API是用于实现本地catalog的简单API，JDK XML处理器的支持使您可以更轻松地配置处理器或整个环境以利用该功能。

## XML Catalog API的目的

XML Catalog API和Java XML处理器为开发人员和系统管理员提供了一个更好地管理外部资源的选项。

XML Catalog API提供了OASIS XML Catalogs v1.1的实现，该标准旨在解决由外部资源引起的问题。

### 外部资源引发的问题

XML，XSD和XSL文档可能包含对Java XML处理器检索以处理文档所需的外部资源的引用。 外部资源可能会导致应用程序或系统出现问题。 Catalog API和Java XML处理器为开发人员和系统管理员提供了更好地管理这些外部资源的选项。

外部资源可能会导致这些范围中的应用程序或系统出现问题：

- 可用性。当资源是远程的时，XML处理器必须能够连接到远程服务器。尽管连接很少成为问题，但它仍然是应用程序稳定性的一个因素。太多的连接可能会对容纳资源的服务器造成危害（例如，记录良好的案例涉及指向W3C服务器的过多DTD流量），这反过来可能会影响您的应用程序。有关使用XML Catalog API解决此问题的示例，请参阅将目录与XML处理器一起使用。
- 性能。虽然在大多数情况下连接不是问题，但远程提取仍然可能导致应用程序出现性能问题。此外，在同一系统上可能存在多个试图解析相同源的应用程序，这将浪费系统资源。
- 安全。如果应用程序处理不受信任的XML源，则允许远程连接可能会带来安全风险。
- 可管理性。如果系统处理大量XML文档，那么外部引用的文档（无论是本地文档还是远程文档）都可能成为维护麻烦。

### XML Catalog API如何解决外部资源引起的问题

XML Catalog API和Java XML处理器为开发人员和系统管理员提供了更好地管理外部资源的选项。

应用程序开发人员 - 您可以为应用程序创建所有外部引用的本地catalog，并让Catalog API为应用程序解析它们。 这不仅可以避免远程连接，还可以更轻松地管理这些资源.

系统管理员 - 您可以为系统建立本地catalog，并将Java VM配置为指向catalog。 然后，系统上的所有应用程序可以共享相同的catalog而不对应用程序进行任何代码更改，假设它们与Java SE 9兼容。要建立catalog，您可以利用现有catalog，例如某些catalog中包含的catalog。

## XML Catalog API接口

通过其接口访问XML Catalog API。

XML Catalog API定义了以下接口：

- Catalog接口表示由[XML Catalogs，OASIS Standard V1.1](https://www.oasis-open.org/committees/download.php/14809/xml-catalogs.html)，2005年10月7日定义的实体catalog.Target对象是不可变的。 创建后，Catalog对象可用于查找system，public或uri条目中的匹配项。 自定义解析程序实现可能会发现通过catalog查找本地资源很有用。
- CatalogFeatures类包含Catalog API支持的所有功能和属性，包括javax.xml.catalog.files，javax.xml.catalog.defer，javax.xml.catalog.prefer和javax.xml.catalog.resolve。
- CatalogManager类管理XML catalog和catalog解析器的创建。
- CatalogResolver接口是一个catalog解析器，它实现了SAX EntityResolver，StAX XMLResolver，模式验证使用的DOM LS LSResourceResolver，以及转换URIResolver。 此接口使用catalogs解析外部引用。

## CatalogFeatures类的详细信息

catalog功能在CatalogFeatures类中共同定义。 这些功能在API和系统级别定义，这意味着可以通过API，系统属性和JAXP属性进行设置。 要通过API设置功能，请使用CatalogFeatures类。

以下代码将javax.xml.catalog.resolve设置为“continue”，以便即使CatalogResolver找不到匹配项，该过程也会继续：

```java
CatalogFeatures f = CatalogFeatures.builder().with(Feature.RESOLVE, "continue").build();
```

要在系统范围内设置此“continue”功能，请使用Java命令行或System.setProperty方法：

```java
System.setProperty(Feature.RESOLVE.getPropertyName(), "continue");
```

要为整个JVM实例设置此“continue”功能，请在jaxp.properties文件中输入一行：

```java
javax.xml.catalog.resolve = "continue"
```

esolve属性以及prefer和defer属性可以设置为catalog文件中catalog或条目组的属性。 例如，在以下catalog中，resolve属性在entry上设置为值“continue”，指示处理器在通过此catalog找不到匹配项时继续。 也可以在group entry上设置该属性，如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<catalog
  xmlns="urn:oasis:names:tc:entity:xmlns:xml:catalog"
  resolve="continue"
  xml:base="http://local/base/dtd/">
  <group resolve="continue">
    <system
      systemId="http://remote/dtd/alice/docAlice.dtd"
      uri="http://local/dtd/docAliceSys.dtd"/>     
  </group> 
</catalog>
```

在较窄范围内设置的属性会覆盖在较宽范围内设置的属性。 因此，通过API设置的属性始终优先。

## 使用XML Catalog API

使用XML Catalog标准的各种条目类型解析XML源文档中的DTD，实体和备用URI引用。

XML Catalog Standard定义了许多条目类型。 其中，system条目（包括system，rewriteSystem和systemSuffix条目）用于解析XML源文档中的DTD和实体引用，而uri条目用于备用URI引用。

### System Reference

使用CatalogResolver对象查找本地资源。

#### 查找本地资源

以下示例演示了CatalogResolver对象如何使用system条目来查找本地资源，给定包含对example.dtd属性的引用的XML文件：

```xml
<?xml version="1.0"?> 
<!DOCTYPE catalogtest PUBLIC "-//OPENJDK//XML CATALOG DTD//1.0" 
  "http://openjdk.java.net/xml/catalog/dtd/example.dtd">

<catalogtest>
  Test &example; entry
</catalogtest>
```

example.dtd定义了一个实体“example”：

```xml
<!ENTITY example "system">
```

XML中example.dtd的URI不需要存在。 目的是为CatalogResolver对象提供唯一标识符以查找本地资源。 为此，请使用system条目创建名为catalog.xml的条目文件，以引用本地资源：

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<catalog xmlns="urn:oasis:names:tc:entity:xmlns:xml:catalog">
  <system
    systemId="http://openjdk.java.net/xml/catalog/dtd/example.dtd"
    uri="example.dtd"/>
</catalog>
```

使用此catalog和系统条目，您需要做的就是获取默认的CatalogFeatures对象，并将URI设置为catalog文件以创建CatalogResolver对象：

```java
CatalogResolver cr =
  CatalogManager.catalogResolver(CatalogFeatures.defaults(), catalogUri);
```

catalogUri必须是有效的URI。 例如：

```java
URI.create("file:///users/auser/catalog/catalog.xml")
```

CatalogResolver对象现在可以用作JDK XML解析程序。 在以下示例中，它用作SAX EntityResolver：

```java
SAXParserFactory factory = SAXParserFactory.newInstance();
factory.setNamespaceAware(true);
XMLReader reader = factory.newSAXParser().getXMLReader();
reader.setEntityResolver(cr);
```

请注意，在示例中，系统标识符被赋予绝对URI。 这使得解析器可以轻松地在catalog的system条目中找到与完全相同的systemId匹配。

如果XML中的系统标识符是相对的，那么它可能使匹配过程复杂化，因为XML处理器可能使其具有指定的基URI或源文件的URI的绝对值。 在这种情况下，system条目的systemId需要匹配预期的绝对URI。 更简单的解决方案是使用systemSuffix条目，例如：

```xml
<systemSuffix systemIdSuffix="example.dtd" uri="example.dtd"/>
```

systemSuffix条目匹配任何以XML源代码中的example.dtd结尾的引用，并将其解析为uri属性中指定的本地example.dtd文件。您可以向systemId添加更多内容，以确保它是唯一的或正确的引用。例如，您可以将systemIdSuffix设置为xml/catalog/dtd/example.dtd，或者重命名XML源文件和systemSuffix条目中的id以使其成为唯一匹配，例如my_example.dtd。

System条目的URI可以是绝对的或相对的。如果外部资源具有固定位置，则绝对URI更可能保证唯一性。如果外部资源是相对于您的应用程序或catalog entry文件放置的，那么相对URI可能更有效，允许部署您的应用程序而无需知道它的安装位置。如果未指定基URI，则使用基URI或catalog文件的URI解析此类相对URI。在前面的示例中，假定example.dtd已与catalog文件放在同一catalog中。

### Public Reference

使用public条目而不是system条目来查找所需的资源。

如果没有system条目与所需资源匹配，并且指定了PREFER属性以匹配public，则public条目可以与system条目相同。 请注意，public是PREFER属性的默认设置。

#### 使用public引用

当解析的XML文件中的DTD引用包含public标识符（例如“ -//OPENJDK//XML CATALOG DTD//1.0”）时，可以在entry catalog中按如下方式编写public条目：

```xml
<public publicId="-//OPENJDK//XML CATALOG DTD//1.0" uri="example.dtd"/>
```

使用此条目文件创建和使用CatalogResolver对象时，example.dtd将通过publicId属性进行解析。 有关创建CatalogResolver对象的示例，请参见System Reference。

### URI Reference

使用uri条目查找所需的资源。

URI类型条目（包括uri，rewriteURI和uriSuffix）可以与系统类型条目类似的方式使用。

#### 使用uri 条目

虽然标准的XML Catalog优先考虑用于解析DTD引用的System类型条目，并且uri类型条目用于其他所有条目，但Java XML Catalog API没有进行区分。 这是因为现有Java XML解析器的规范（例如XMLResolver和LSResourceResolver）没有给出优先权。 uri类型条目（包括uri，rewriteURI和uriSuffix）可以与系统类型条目类似的方式使用。 uri元素被定义为将备用URI引用与URI引用相关联。 在系统引用的情况下，这是systemId属性。

因此，您可以在以下示例中使用uri条目替换系统条目，尽管system条目更常用于DTD引用。

```xml
<system
  systemId="http://openjdk.java.net/xml/catalog/dtd/example.dtd"
  uri="example.dtd"/>
```

uri条目如下所示：

```xml
<uri name="http://openjdk.java.net/xml/catalog/dtd/example.dtd" uri="example.dtd"/>
```

虽然系统条目经常用于DTD，但uri条目是URI引用的首选，例如XSD和XSL import and include。 下一个示例使用uri条目来解析XSL导入。

如[XML Catalog API接口](https://docs.oracle.com/en/java/javase/11/core/xml-catalog-api1.html#GUID-D9216F5E-8759-4286-95CA-E1D041DD72AF)中所述，XML Catalog API定义了扩展Java XML解析器的CatalogResolver接口，包括EntityResolver，XMLResolver，URIResolver和LSResolver。 因此，SAX，DOM，StAX，Schema Validation以及XSLT Transform可以使用CatalogResolver对象。 以下代码使用默认功能设置创建CatalogResolver对象：

```java
CatalogResolver cr =
  CatalogManager.catalogResolver(CatalogFeatures.defaults(), catalogUri);
```

然后代码在需要URIResolver对象的TransformerFactory类上注册此CatalogResolver对象：

```java
TransformerFactory factory = TransformerFactory.newInstance（）;
factory.setURIResolver（CR）;
```

或者，代码可以在Transformer对象上注册CatalogResolver对象：

```java
Transformer transformer = factory.newTransformer(xslSource); 
transformer.setURIResolver(cur);
```

假设XSL源文件包含一个import元素，用于将xslImport.xsl文件导入XSL源：

```xml
<xsl:import href="pathto/xslImport.xsl"/>
```

要解析导入文件实际所在位置的导入引用，应在创建Transformer对象之前在TransformerFactory类上设置CatalogResolver对象，并且必须将以下uri条目添加到catalog条目：

```xml
<uri name="pathto/xslImport.xsl" uri="xslImport.xsl"/>
```

关于绝对或相对URI的讨论以及systemSuffix或uriSuffix条目与系统引用的使用也适用于uri条目。

## Java XML处理器支持

将XML Catalogs功能与标准Java XML处理器一起使用。

整个Java XML处理器支持XML Catalogs功能，包括SAX和DOM（javax.xml.parsers），StAX解析器（javax.xml.stream），模式验证（javax.xml.validation）和XML转换（javax）.xml.transform）。

这意味着您不需要在XML处理器之外创建CatalogResolver对象。 catalog文件可以直接注册到Java XML处理器，也可以通过系统属性或jaxp.properties文件指定。 XML处理器自动通过catalog执行映射。

### 启用Catalog支持

要在处理器上启用对XML Catalogs功能的支持，必须将USE_CATALOG功能设置为true，并且至少指定一个catalog entry文件。

#### USE_CATALOG

Java XML处理器根据USE_CATALOG功能的值确定是否支持XML Catalogs功能。 默认情况下，所有JDK XML处理器的USE_CATALOG都设置为true。 Java XML处理器进一步检查catalog文件的可用性，并仅在USE_CATALOG功能为true且catalog可用时才尝试使用XML Catalog API。

XML Catalog API，system属性和jaxp.properties文件支持USE_CATALOG功能。 例如，如果将USE_CATALOG设置为true并且需要禁用特定处理器的catalog支持，则可以通过处理器的setFeature方法将USE_CATALOG功能设置为false来完成此操作。 以下代码将USE_CATALOG功能设置为XMLReader对象的指定值useCatalog：

```java
SAXParserFactory spf = SAXParserFactory.newInstance();
spf.setNamespaceAware(true);
XMLReader reader = spf.newSAXParser().getXMLReader();
if (setUseCatalog) {
   reader.setFeature(XMLConstants.USE_CATALOG, useCatalog); 
}
```

另一方面，如果整个环境必须关闭catalog，则可以通过使用以下行配置jaxp.properties文件来完成此操作：

```xml
javax.xml.useCatalog = false;
```

##### javax.xml.catalog.files

javax.xml.catalog.files属性由XML Catalog API定义，并由JDK XML处理器以及其他catalog功能支持。 要在解析，验证或转换过程中使用catalog功能，所需的只是在处理器上，通过其系统属性或使用jaxp.properties文件设置FILES属性。

#### Catalog URI

catalog文件引用必须是有效的URI，例如file:///users/auser/catalog/catalog.xml。

系统中的URI引用或目录文件中的URI条目可以是绝对的或相对的。 如果它们是相对的，则使用catalog文件的URI或基URI（如果指定）解析它们。

##### 使用system或uri条目

直接使用XML Catalog API时，system和uri条目在使用JDK XML Processors对CatalogFeatures类的本机支持时都有效。 通常，首先搜索system条目，然后搜索public条目，如果没有找到匹配，则处理器继续搜索uri条目。 由于系统和uri条目都受支持，因此建议您在使用system或uri条目之间进行选择时遵循XML规范的自定义。 例如，DTD使用systemId定义，因此system条目更可取。

#### 使用Catalog与XML处理器

将XML Catalog API与各种Java XML处理器一起使用。

JDK XML处理器支持XML Catalog API。 以下部分描述了如何为特定类型的处理器启用它。

##### 使用带DOM的catalog

要使用带有DOM的catalog，请在DocumentBuilderFactory实例上设置FILES属性，如以下代码所示：

```java
static final String CATALOG_FILE = CatalogFeatures.Feature.FILES.getPropertyName();
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setNamespaceAware(true);
if (catalog != null) {
   dbf.setAttribute(CATALOG_FILE, catalog);
}
```

> catalog是catalog文件的URI。 例如，它可能类似于“file:///users/auser/catalog/catalog.xml”。

最好将解析catalog文件与entry文件一起部署，以便可以相对于catalog文件解析文件。 例如，如果以下是目录文件中的uri条目，则XSLImport_html.xsl文件将位于/users/auser/catalog/XSLImport_html.xsl中。

```xml
<uri name="pathto/XSLImport_html.xsl" uri="XSLImport_html.xsl"/>
```

#### 使用SAX的catalog

要在SAX解析器上使用catalog功能，请将catalog文件设置为SAXParser实例：

```java
SAXParserFactory spf = SAXParserFactory.newInstance();
spf.setNamespaceAware(true);
spf.setXIncludeAware(true);
SAXParser parser = spf.newSAXParser();
parser.setProperty(CATALOG_FILE, catalog);
```

在前面的示例代码中，请注意语句spf.setXIncludeAware（true）。 启用此选项后，也会使用catalog解析任何XInclude。

给定一个XML文件XI_simple.xml：

```xml
<simple> 
  <test xmlns:xinclude="http://www.w3.org/2001/XInclude">   
    <latin1>
      <firstElement/>
      <xinclude:include href="pathto/XI_text.xml" parse="text"/>
      <insideChildren/>
      <another>
        <deeper>text</deeper>
      </another>
    </latin1>
    <test2>
      <xinclude:include href="pathto/XI_test2.xml"/>   
    </test2>
  </test>
</simple>
```

另外，给定另一个XML文件XI_test2.xml：

```xml
<?xml version="1.0"?>
<!-- comment before root -->
<!DOCTYPE red SYSTEM "pathto/XI_red.dtd">
<red xmlns:xinclude="http://www.w3.org/2001/XInclude">
  <blue>
    <xinclude:include href="pathto/XI_text.xml" parse="text"/>
  </blue>
</red>
```

假设另一个文本文件XI_text.xml包含一个简单的字符串，文件XI_red.dtd如下所示：

```xml
 <!ENTITY red "it is read">
```

在这些XML文件中，XInclude元素中有一个XInclude元素，以及对DTD的引用。 假设它们与catalog文件CatalogSupport.xml一起位于同一文件夹中，请添加以下目录条目以映射它们：

```xml
<uri name="pathto/XI_text.xml" uri="XI_text.xml"/>
<uri name="pathto/XI_test2.xml" uri="XI_test2.xml"/> 
<system systemId="pathto/XI_red.dtd" uri="XI_red.dtd"/>
```

#### 使用StAX的目录

要将目录功能与StAX解析器一起使用，请在创建XMLStreamReader对象之前在XMLInputFactory实例上设置目录文件：

```java
XMLInputFactory factory = XMLInputFactory.newInstance();
factory.setProperty(CatalogFeatures.Feature.FILES.getPropertyName(), catalog);
XMLStreamReader streamReader =
  factory.createXMLStreamReader(xml, new FileInputStream(xml));
```

当XMLStreamReader streamReader对象用于解析XML源时，源中的外部引用将根据目录中的指定条目进行解析。

请注意，与同时具有setFeature和setAttribute方法的DocumentBuilderFactory类不同，XMLInputFactory类仅定义setProperty方法。 包含XMLConstants.USE_CATALOG的XML Catalog API功能都通过此setProperty方法设置。 例如，要在XMLStreamReader对象上禁用USE_CATALOG，可以执行以下操作：

```java
factory.setProperty(XMLConstants.USE_CATALOG, false);
```

#### 使用Schema验证的Catalog

要使用catalog解析模式中的任何外部资源（例如XSD导入和包含），请在SchemaFactory对象上设置catalog：

```java
SchemaFactory factory =
  SchemaFactory.newInstance(XMLConstants.W3C_XML_SCHEMA_NS_URI);
factory.setProperty(CatalogFeatures.Feature.FILES.getPropertyName(), catalog);
Schema schema = factory.newSchema(schemaFile);
```

XMLSchema schema文档包含对外部DTD的引用：

```xml
<!DOCTYPE xs:schema PUBLIC "-//W3C//DTD XMLSCHEMA 200102//EN" "pathto/XMLSchema.dtd" [
   ... 
]>
```

并且到xsd导入：

```xml
<xs:import
  namespace="http://www.w3.org/XML/1998/namespace"
  schemaLocation="http://www.w3.org/2001/pathto/xml.xsd">
  <xs:annotation>
    <xs:documentation>
      Get access to the xml: attribute groups for xml:lang
      as declared on 'schema' and 'documentation' below
    </xs:documentation>
  </xs:annotation>
</xs:import>
```

下面是本示例，通过减少对W3C服务器的调用，使用本地资源来提高应用程序性能：

- 将这些条目包含在SchemaFactory对象的目录集中：

```java
<public publicId="-//W3C//DTD XMLSCHEMA 200102//EN" uri="XMLSchema.dtd"/>
<!-- XMLSchema.dtd refers to datatypes.dtd --> 
<systemSuffix systemIdSuffix="datatypes.dtd" uri="datatypes.dtd"/>
<uri name="http://www.w3.org/2001/pathto/xml.xsd" uri="xml.xsd"/>
```

- 下载源文件XMLSchema.dtd，datatypes.dtd和xml.xsd，并将它们与目录文件一起保存。

  如前所述，XML Catalog API允许您使用您喜欢的任何条目类型。 在先前的情况下，您还可以使用以下任一项代替uri条目:

- public条目，因为import元素中的namespace属性被视为publicId元素：

  ```xml
  <public publicId="http://www.w3.org/XML/1998/namespace" uri="xml.xsd"/>
  ```

- system条目

  ```xml
  <system systemId="http://www.w3.org/2001/pathto/xml.xsd" uri="xml.xsd"/>
  ```

  > 在尝试使用XML Catalog API时，确保示例文件中使用的URI或系统ID都不会指向Internet上的任何实际资源（尤其是W3C服务器）可能会很有用。 这可以让您在catalog解析失败时尽早发现错误，并避免给W3C服务器带来负担，从而将它们从任何不必要的连接中解放出来。 本主题中的所有示例以及有关XML Catalog API的其他相关主题都为此目的在任何URI中添加了任意字符串“pathto”，因此没有URI可能解析为外部W3C资源。

  要使用目录解析要验证的XML源中的任何外部资源，请在Validator对象上设置目录：

  ```java
  SchemaFactory schemaFactory =
    SchemaFactory.newInstance(XMLConstants.W3C_XML_SCHEMA_NS_URI);
  Schema schema = schemaFactory.newSchema();
  Validator validator = schema.newValidator();
  validator.setProperty(CatalogFeatures.Feature.FILES.getPropertyName(), catalog);
  StreamSource source = new StreamSource(new File(xml));
  validator.validate(source);
  ```

  #### 使用Transform Catalog 

  要在XSLT转换过程中使用XML Catalog API，请在TransformerFactory对象上设置目录文件。

  ```java
  TransformerFactory factory = TransformerFactory.newInstance();
  factory.setAttribute(CatalogFeatures.Feature.FILES.getPropertyName(), catalog);
  Transformer transformer = factory.newTransformer(xslSource);
  ```

  如果工厂用于创建Transformer对象的XSL源包含与以下类似的DTD，import和include语句：

  ```xml
  <!DOCTYPE HTMLlat1 SYSTEM "http://openjdk.java.net/xml/catalog/dtd/XSLDTD.dtd">
  <xsl:import href="pathto/XSLImport_html.xsl"/>
  <xsl:include href="pathto/XSLInclude_header.xsl"/>
  ```

  然后，可以使用以下catalog条目来解析这些引用：

  ```xml
  <system
    systemId="http://openjdk.java.net/xml/catalog/dtd/XSLDTD.dtd"
    uri="XSLDTD.dtd"/>
  <uri name="pathto/XSLImport_html.xsl" uri="XSLImport_html.xsl"/>
  <uri name="pathto/XSLInclude_header.xsl" uri="XSLInclude_header.xsl"/>
  ```

  ## 解析器的调用顺序

  JDK XML处理器在catalog解析器之前调用自定义解析程序

  Catalog解析程序（由CatalogResolver接口定义）可用于解析已设置catalog文件的JDK XML处理器的外部引用。但是，如果还提供了自定义解析程序，则它始终位于catalog解析程序之前。这意味着JDK XML处理器首先调用自定义解析程序以尝试解析外部资源。如果解析成功，则处理器会跳过catalog解析程序并继续。只有当没有自定义解析程序或自定义解析程序的解析返回null时，处理器才会调用catalog解析程序。

  由于可以使用自定义解析程序的应用程序，因此可以安全地设置其他目录以解析自定义解析程序无法处理的任何资源。对于现有应用程序，如果更改代码不可行，则可以通过system属性或jaxp.properties文件设置目录，以将外部引用重定向到本地资源，因为这样的设置不会通过自定义解析器干扰处理的现有进程。

  ## 错误检查

  通过隔离问题来检测配置问题。

  XML Catalogs Standard要求处理器从任何资源故障中恢复并继续，因此XML Catalog API会忽略任何失败的目录条目文件而不会发出错误，这使得检测配置问题变得更加困难。

  ### 检测配置问题

  要检测配置问题，请通过一次设置一个catalog，将RESOLVE值设置为strict以及在未找到匹配项时检查CatalogException异常来隔离问题。

  | RESOLVE Value      | CatalogResolver Behavior                             | 描述                                                         |
  | ------------------ | ---------------------------------------------------- | ------------------------------------------------------------ |
  | `strict` (default) | 如果未找到与指定引用的匹配项，则抛出CatalogException | 不匹配的引用可能表示catalog中或设置的catalog中可能存在错误。 |
  | continue           |                                                      | 这在您希望XML处理器继续解析catalog未涵盖的任何外部引用的生产环境中非常有用。 |
  | ignore             |                                                      | 对于允许跳过外部引用的SAX等处理器，ignore值指示CatalogResolver对象返回空的InputSource对象，从而跳过外部引用。 |


