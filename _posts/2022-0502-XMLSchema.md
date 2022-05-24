---
layout: post
title:  XML Validation
date:   2020-05-02 00:00:00 +0800
categories: xml
tags:
	- xml
	- java
---

# XML Validation with Java

[Reference](http://www.edankert.com/validate.html)

## dom4j with external schema

```java
String schemaFile = "schema.xsd";

SAXParserFactory factory = SAXParserFactory.newInstance();
factory.setNamespaceAware(true);
SAXParser parser = factory.newSAXParser();
parser.setProperty("http://java.sun.com/xml/jaxp/properties/schemaLanguage",
		"http://www.w3.org/2001/XMLSchema");
parser.setProperty("http://java.sun.com/xml/jaxp/properties/schemaSource", "file:" + schemaFile);

XMLErrorHandler errorHandler = new XMLErrorHandler();
SAXValidator validator = new SAXValidator(parser.getXMLReader());
validator.setErrorHandler(errorHandler);

SAXReader reader = new SAXReader();
Document xmlDocument = reader.read(file);

validator.validate(xmlDocument);
```

* issues

```
schema_reference.4: Failed to read schema document 'schema.xsd', because 1) could not find the do
cument; 2) the document could not be read; 3) the root element of the document is not <xsd:schema>.
```

如果资源文件`schema.xsd`打包到jar包中，且其中引用了其他的schema文件，则会遇到上述问题。
解决方法是使用`SchemaFactory`

```java
String schemaFile = "schema.xsd";

SchemaFactory factory = SchemaFactory.newInstance(XMLConstants.W3C_XML_SCHEMA_NS_URI);
SchemaFactory schemaFactory = SchemaFactory.newInstance(XMLConstants.W3C_XML_SCHEMA_NS_URI);
Schema schema = schemaFactory.newSchema(schemaFile);

SAXParserFactory factory = SAXParserFactory.newInstance();
factory.setNamespaceAware(true);
factory.setSchema(schema);

SAXParser parser = factory.newSAXParser();
XMLErrorHandler errorHandler = new XMLErrorHandler();
SAXValidator validator = new SAXValidator(parser.getXMLReader());
validator.setErrorHandler(errorHandler);

SAXReader reader = new SAXReader();
Document xmlDocument = reader.read(file);

validator.validate(xmlDocument);
```

* issues

```
Document is invalid: no grammar found schema
```

在使用Schema文件校验时，如果将`SAXParseFactory`的`setValidating`设为`true`，则会出现上述问题。
因为这里的`setValidating`是指是否启用DTD校验，而不是是否启用Schema校验。
