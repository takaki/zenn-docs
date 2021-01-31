意外にもサンプルプログラムが簡単に見つからなかったのでメモ。

```java:
package example.xml;

import org.w3c.dom.Document;
import org.xml.sax.SAXException;

import javax.xml.bind.JAXBContext;
import javax.xml.bind.JAXBElement;
import javax.xml.bind.JAXBException;
import javax.xml.bind.Unmarshaller;
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import java.io.ByteArrayInputStream;
import java.io.IOException;

public class DomJaxb {
    public static void main(
            final String[] args) throws ParserConfigurationException, IOException, SAXException, JAXBException {
        final String xml = "<person><name>hoge</name><age>35</age></person>";
        final DocumentBuilderFactory documentBuilderFactory = DocumentBuilderFactory
                .newInstance();
        final DocumentBuilder documentBuilder = documentBuilderFactory
                .newDocumentBuilder();
        final Document document = documentBuilder
                .parse(new ByteArrayInputStream(xml.getBytes()));

        final JAXBContext jc = JAXBContext.newInstance(Person.class);
        final Unmarshaller unmarshaller = jc.createUnmarshaller();

        final JAXBElement<Person> jaxbElement = unmarshaller
                .unmarshal(document, Person.class);
        System.out.println("JAXBElement: " + jaxbElement.getValue());

        final Person person = (Person) unmarshaller.unmarshal(document);
        System.out.println("unmarshal: " + person);
    }

    @XmlRootElement
    public static class Person {
        @XmlElement
        private String name;
        @XmlElement
        private int age;

        @Override
        public String toString() {
            return String.join("", name + ":" + Integer.toString(age));
        }

    }
}
```

何がポイントかというと@XmlRootElementのアノテーションをつけるところだった。DOMにしないでstreamなどからそのままunmarshalするときには @XmlRootElement を付けないでも大丈夫だった。
