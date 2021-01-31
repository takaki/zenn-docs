---
title: JavaでXML DOMを処理するいくつかの方法
tags: Java XML dom xpath
author: takaki@github
slide: false
---
# 環境
* Java 8
* Joox 1.3.0
* Jsoup 1.8.3

いくつか XML のパースを Java で実験してみた。CSSをつかったセレクタだとテキストノードの選択がちょっと難しい。

```java:XMLParse
package example.xml;

import org.joox.Match;
import org.jsoup.Jsoup;
import org.w3c.dom.Document;
import org.w3c.dom.Element;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.xpath.XPath;
import javax.xml.xpath.XPathConstants;
import javax.xml.xpath.XPathFactory;
import java.nio.charset.StandardCharsets;
import java.nio.file.Path;
import java.nio.file.Paths;

import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.core.Is.is;
import static org.joox.JOOX.$;
import static org.joox.JOOX.attr;

public class XMLParse {
    public static void main(final String[] args) throws Exception {
        final Path path = Paths.get("data/sample.xml");
        useDOM(path);
        useXPath(path);
        useJSoup(path);
        useJoox(path);
    }


    private static void useDOM(final Path path) throws Exception {
        final DocumentBuilderFactory documentBuilderFactory = DocumentBuilderFactory
                .newInstance();
        final DocumentBuilder documentBuilder = documentBuilderFactory
                .newDocumentBuilder();
        final Document document = documentBuilder.parse(path.toFile());

        assertThat(document.getElementsByTagName("h1").item(0).getTextContent(),
                is("header text"));
        final Element html = document.getDocumentElement();

        // h1のテキスト取得
        assertThat(html.getElementsByTagName("h1").item(0).getTextContent(),
                is("header text"));
        // h1のclass属性を取得
        assertThat(html.getElementsByTagName("h1").item(0).getAttributes()
                .getNamedItem("class").getNodeValue(), is("header1"));

        // 2番目のdivのテキスト取得
        assertThat(
                html.getElementsByTagName("div").item(1).getChildNodes().item(0)
                        .getNodeValue().trim(), is("div2"));
        // 2番目のdivのulのliを取得
        assertThat(
                html.getElementsByTagName("ul").item(1).getChildNodes().item(1)
                        .getChildNodes().item(0).getTextContent(), is("111"));
        // 同じノード
        assertThat(
                html.getElementsByTagName("div").item(1).getChildNodes().item(1)
                        .getChildNodes().item(1).getTextContent(), is("111"));
        // 次のli
        assertThat(
                html.getElementsByTagName("div").item(1).getChildNodes().item(1)
                        .getChildNodes().item(3).getTextContent(), is("222"));
    }

    private static void useXPath(final Path path) throws Exception {
        final DocumentBuilderFactory documentBuilderFactory = DocumentBuilderFactory
                .newInstance();
        final DocumentBuilder documentBuilder = documentBuilderFactory
                .newDocumentBuilder();
        final Document document = documentBuilder.parse(path.toFile());

        final XPath xpath = XPathFactory.newInstance().newXPath();

        // h1のテキスト取得
        final String h1 = (String) xpath
                .evaluate("/html/body/div/h1/text()", document,
                        XPathConstants.STRING);
        assertThat(h1, is("header text"));

        // h1のclass属性を取得
        final String h1c = (String) xpath
                .evaluate("/html/body/div/h1/@class", document,
                        XPathConstants.STRING);
        assertThat(h1c, is("header1"));

        // 2番目のdivのテキスト取得
        final String div = (String) xpath
                .evaluate("//div[2]/text()", document, XPathConstants.STRING);
        assertThat(div.trim(), is("div2"));

        // 2番目のdivのulのliを取得
        final String li = (String) xpath
                .evaluate("//div[2]/ul[1]/li[1]/text()", document,
                        XPathConstants.STRING);
        assertThat(li, is("111"));

        // 次のliの中身
        final String li3 = (String) xpath
                .evaluate("//div[2]/ul[1]/li[2]/text()", document,
                        XPathConstants.STRING);
        assertThat(li3, is("222"));

    }

    private static void useJSoup(final Path path) throws Exception {
        final org.jsoup.nodes.Document document = Jsoup
                .parse(path.toFile(), StandardCharsets.UTF_8.toString());


        // h1のテキスト取得
        final String h1 = document.select("html > body > div:nth-child(1) > h1")
                .text();
        assertThat(h1, is("header text"));

        // h1のclass属性を取得

        final String h1c = document
                .select("html > body > div:nth-child(1) > h1").attr("class");
        assertThat(h1c, is("header1"));

//        // 2番目のdivのテキスト取得
//        final String div = document.select("div:nth-child(2) :be")
//        assertThat(div.trim(), is("div2"));

        // 2番目のdivのulのliを取得
        final String li = document
                .select("div:nth-child(2) > ul:nth-child(1) > li:nth-child(1)")
                .text();
        assertThat(li, is("111"));


        // 次のliの中身
        final String li3 = document
                .select("div:nth-child(2) > ul:nth-child(1) > li:nth-child(2)")
                .text();
        assertThat(li3, is("222"));


    }

    private static void useJoox(final Path path) throws Exception {
        final DocumentBuilderFactory documentBuilderFactory = DocumentBuilderFactory
                .newInstance();
        final DocumentBuilder documentBuilder = documentBuilderFactory
                .newDocumentBuilder();
        final Document document = documentBuilder.parse(path.toFile());

        final Match match = $(document);

        // h1のテキスト取得
        final String h1 = match.find("html body").find("div").find("h1").eq(0).text();
        assertThat(h1, is("header text"));

        // h1のclass属性を取得

        final String h1c = match.find("h1").filter(attr("class")).attr("class");
        assertThat(h1c, is("header1"));

//        // 2番目のdivのテキスト取得
//        final String div = match.xpath("//div[2]").content(1);
//        assertThat(div.trim(), is("div2"));

        // 2番目のdivのulのliを取得
        final String li = match
                .find("div:nth-child(2) > ul:nth-child(1) > li:nth-child(1)")
                .text();
        assertThat(li, is("111"));

        // 同じ
        final String li4 = match.find("div").eq(1).find("ul").eq(0).find("li")
                .eq(0).text();
        assertThat(li4, is("111"));

        // 次のliの中身
        final String li3 = match.find("div").eq(1).find("ul").eq(0).find("li")
                .eq(1).text();
        assertThat(li3, is("222"));

    }
}
```
最低でもXPathはやらないとDOMを直接はめんどくさい。

サンプルに使った XML は以下。

```xml:sample.xml
<html>
    <head>
        <title>the title</title>
    </head>
    <body>
        <div>
            div1
            <h1 class="header1">header text</h1>
            body text
            <ul>
                <li>aaa</li>
            </ul>
        </div>
        <div>
            div2
            <ul>
                <li>111</li>

                <li>222</li>
            </ul>
        </div>
        <div>
            <h1>second text</h1>
        </div>
    </body>
</html>
```

