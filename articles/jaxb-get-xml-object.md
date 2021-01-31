---
emoji: "💻"
type: "tech"
published: true
title: JAXBで他のXMLの要素のオブジェクトを参照する
topics: ["Java","XML","JAXB"]
author: takaki@github
slide: false
---
JAXB を使っていて他のXMLで変換されたオブジェクトを参照したい場合がある。外部キーとかそういう奴である。
この場合は `@XmlID` と `@XmlIDREF` を使う。

```java:DomIdRefJaxb.java
package example.xml;

import org.w3c.dom.Document;
import org.xml.sax.SAXException;

import javax.xml.bind.JAXBContext;
import javax.xml.bind.JAXBElement;
import javax.xml.bind.JAXBException;
import javax.xml.bind.Unmarshaller;
import javax.xml.bind.annotation.*;
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import java.io.ByteArrayInputStream;
import java.io.IOException;

public class DomIdRefJaxb {
    public static void main(
            final String[] args) throws ParserConfigurationException, IOException, SAXException, JAXBException {
        final String xml = "<info><person><name>hoge</name><age>35</age><addr>12</addr></person><city id='12' name='Nagoya'/></info>";
        final DocumentBuilderFactory documentBuilderFactory = DocumentBuilderFactory
                .newInstance();
        final DocumentBuilder documentBuilder = documentBuilderFactory
                .newDocumentBuilder();
        final Document document = documentBuilder
                .parse(new ByteArrayInputStream(xml.getBytes()));

        final JAXBContext jc = JAXBContext.newInstance(Info.class);
        final Unmarshaller unmarshaller = jc.createUnmarshaller();
        final JAXBElement<Info> jaxbElement = unmarshaller
                .unmarshal(document, Info.class);
        System.out.println("JAXBElement: " + jaxbElement.getValue());

        final Info info = (Info) unmarshaller.unmarshal(document);
        System.out.println("unmarshal: " + info);
    }

    @XmlRootElement
    public static class Info {
        @XmlElement(name = "person")
        private Person person;
        @XmlElement(name = "city")
        private City city;

        @Override
        public String toString() {
            return String.format("%s", person);
        }

    }

    @XmlRootElement
    public static class City {
        @XmlAttribute(name = "id")
        @XmlID
        private String id;
        @XmlAttribute(name = "name")
        private String name;
    }

    @XmlRootElement
    public static class Person {
        @XmlElement
        private String name;
        @XmlElement
        private int age;

        @XmlElement(name = "addr")
        @XmlIDREF
        private City city;

        @Override
        public String toString() {
            return String.format("%s:%s:%s", name, Integer.toString(age), city.name);
        }

    }
}
```

`Person`クラスの中で `@XmlIDREF` を使って参照するという記述をつけておけば 対象のクラスの `@XmlID` を探してきて結合してくれる。

