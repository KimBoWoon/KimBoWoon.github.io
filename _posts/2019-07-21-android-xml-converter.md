---
title: "[Android] Retrofit을 위한 xml converter"
date: 2019-07-21 15:48:41
categories:
- Android
---

# xml parsing
![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/retrofit-converter.PNG)

위 사진과 같이 retrofit에는 다양한 컨버터가 존재합니다. 다만 simple xml converter는 deprecated가 되었습니다.

![그림](https://raw.githubusercontent.com/KimBoWoon/KimBoWoon.github.io/master/_img/android/simple-xml-converter-deprecated.PNG)

JAXB converter라는 다른 방법을 소개하기는 하지만 안드로이드에서는 작동을 안한다고 합니다.

그래서 더 많은 정보를 찾아보았고 JakeWharton이 말하는 tikxml을 사용했습니다.

# TikXml
사용법은 간단(?)합니다. 기존에 사용하던 것처럼 retrofit에 컨버터를 추가하면 됩니다. 다만 필요하다면 사전에 TikXml 인스턴스를 만들어서 넘겨줘야합니다.
저는 rss를 파싱해서 제가 사용한 방법을 토대로 보여드리겠습니다.

```gradle
implementation 'com.tickaroo.tikxml:retrofit-converter:0.8.13'
```

일단 gradle에 라이브러리를 추가합니다.

```kotlin
val tikXml: TikXml = TikXml.Builder()
    .addTypeConverter(Date::class.java, MyDateConverter())
    .addTypeAdapter(Rss::class.java, RssTypeAdapter())
    .addTypeAdapter(Channel::class.java, ChannelTypeAdapter())
    .addTypeAdapter(Image::class.java, ImageTypeAdapter())
    .addTypeAdapter(Item::class.java, ItemTypeAdapter())
    .build()

val client: Retrofit = Retrofit.Builder()
    .baseUrl(BASE_URL)
    .addConverterFactory(TikXmlConverterFactory.create(tikXml))
    .client(createOkHttpClient())
    .build()
```

우선 TikXml 인스턴트를 만드는데 두 가지 옵션을 더 추가합니다. addTypeConverter는 파싱하기위해 변환이 필요한것들을 위한 컨버터를 추가하는 것이고 addTypeAdapter은 해당하는 xml에 대해 어떤 엘리먼트들을 파싱할건지 추가하는겁니다.

```kotlin
@Xml
data class Rss(
    @Attribute
    var version: String,
    @Element
    var channel: Channel
) {
    constructor() : this("", Channel())
}

@Xml
data class Channel(
    @PropertyElement
    var title: String,
    @PropertyElement
    var link: String,
    @PropertyElement
    var language: String,
    @PropertyElement
    var copyright: String,
    @PropertyElement
    var pubDate: String,
    @PropertyElement
    var lastBuildDate: String,
    @PropertyElement
    var description: String,
    @Element
    var image: Image,
    @PropertyElement
    var item: ArrayList<Item>
) {
    constructor() : this("", "", "", "", "", "", "", Image(), ArrayList<Item>())
}

@Xml
data class Image(
    @PropertyElement
    var title: String,
    @PropertyElement
    var url: String,
    @PropertyElement
    var link: String
) {
    constructor() : this("", "", "")
}

data class Item(
    @PropertyElement
    var title: String,
    @PropertyElement
    var link: String,
    @PropertyElement
    var description: String,
    @PropertyElement
    var author: String,
    @PropertyElement
    var pubDate: String,
    var ogTag: OGTag = OGTag()
) {
    constructor() : this("", "", "", "", "", OGTag())
}
```

@Attribute는 엘리먼트들이 가진 속성을 추출하는것이고 @PropertyElement는 엘리먼트를 추출하는 어노테이션입니다.

```kotlin
class RssTypeAdapter : TypeAdapter<Rss> {
    override fun fromXml(reader: XmlReader?, config: TikXmlConfig?): Rss {
        val rss = Rss()
        reader!!.nextAttributeName()
        rss.version = reader.nextAttributeValue()
        if (reader.hasElement()) {
            reader.beginElement()
            reader.nextElementName()
            val type = config!!.getTypeAdapter(Channel::class.java)
            rss.channel = type.fromXml(reader, config)
            reader.endElement()
        }
        return rss
    }

    override fun toXml(writer: XmlWriter?, config: TikXmlConfig?, value: Rss?, overridingXmlElementTagName: String?) {

    }
}

class ChannelTypeAdapter : TypeAdapter<Channel> {
    override fun fromXml(reader: XmlReader?, config: TikXmlConfig?): Channel {
        val channel: Channel = Channel()
        var type: TypeAdapter<*>

        while (reader!!.hasElement()) {
            reader.beginElement()

            when (reader.nextElementName()) {
                "title" -> channel.title = reader.nextTextContent()
                "link" -> channel.link = reader.nextTextContent()
                "language" -> channel.language = reader.nextTextContent()
                "copyright" -> channel.copyright = reader.nextTextContent()
                "pubDate" -> channel.pubDate = reader.nextTextContent()
                "lastBuildDate" -> channel.lastBuildDate = reader.nextTextContent()
                "description" -> channel.description = reader.nextTextContent()
                "image" -> {
                    type = config!!.getTypeAdapter(Image::class.java)
                    channel.image = type.fromXml(reader, config)
                }
                "item" -> {
                    type = config!!.getTypeAdapter(Item::class.java)
                    channel.item.add(type.fromXml(reader, config))
                }
                else -> {
                    Log.i("Channel", "not found xml element")
                }
            }
            reader.endElement()
        }

        return channel
    }

    override fun toXml(
        writer: XmlWriter?,
        config: TikXmlConfig?,
        value: Channel?,
        overridingXmlElementTagName: String?
    ) {

    }
}

class ImageTypeAdapter : TypeAdapter<Image> {
    override fun fromXml(reader: XmlReader?, config: TikXmlConfig?): Image {
        val image: Image = Image()
        while (reader!!.hasElement()) {
            reader.beginElement()
            when (reader.nextElementName()) {
                "title" -> image.title = reader.nextTextContent()
                "url" -> image.url = reader.nextTextContent()
                "link" -> image.link = reader.nextTextContent()
                else -> {
                    Log.i("Image", "not found xml element")
                }
            }
            reader.endElement()
        }
        return image
    }

    override fun toXml(writer: XmlWriter?, config: TikXmlConfig?, value: Image?, overridingXmlElementTagName: String?) {

    }
}

class ItemTypeAdapter : TypeAdapter<Item> {
    override fun fromXml(reader: XmlReader?, config: TikXmlConfig?): Item {
        val item = Item()
        while (reader!!.hasElement()) {
            reader.beginElement()
            when (reader.nextElementName()) {
                "title" -> item.title = reader.nextTextContent()
                "link" -> item.link = reader.nextTextContent()
                "description" -> item.description = reader.nextTextContent()
                "author" -> item.author = reader.nextTextContent()
                "pubDate" -> item.pubDate = reader.nextTextContent()
                else -> {
                    Log.i("Image", "not found xml element")
                }
            }
            reader.endElement()
        }
        return item
    }

    override fun toXml(
        writer: XmlWriter?,
        config: TikXmlConfig?,
        value: Item?,
        overridingXmlElementTagName: String?
    ) {

    }
}

class MyDateConverter : TypeConverter<Date> {
    private val formatter = SimpleDateFormat("yyyy.MM.dd")

    @Throws(Exception::class)
    override fun read(value: String): Date {
        return formatter.parse(value)
    }

    @Throws(Exception::class)
    override fun write(value: Date): String {
        return formatter.format(value)
    }
}
```

어댑터에서도 알수있듯이 fromXml에서 원하는 엘리먼트들을 파싱합니다. toXml에서는 xml을 만드는것 같은데 아직 사용해보지 않아서 모르겠습니다.

# 주의점
TikXml은 현재 0.8.15버전이 최신버전인데 이 버전에서 오류가 있어서 0.8.15를 사용할 수 없습니다. 그래서 0.8.13버전을 사용해야합니다. 오류 내용은 com.tickaroo.tikxml:retrofit-converter:0.8.15 이곳에 어노테이션 라이브러리가 첨부되지 않았다고 합니다. 해당 라이브러리에 이슈를 참조하면 자기들도 왜 포함이  잘 모른다고 서술하고 있습니다.

# 참조
* TikXml : https://github.com/Tickaroo/tikxml
* TikXml 오류 이슈 : https://github.com/Tickaroo/tikxml/issues/115
* simple xml converter deprecated : https://github.com/square/retrofit/issues/2733
* TikXml Docment : https://github.com/Tickaroo/tikxml/blob/master/docs/AnnotatingModelClasses.md
