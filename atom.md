---
---
<?xml version="1.0" encoding="utf-8"?>
<feed xmlns="http://www.w3.org/2005/Atom">

 <title>Schrodinator's Blag</title>
 <link href="http://schrodinator.github.io/atom.xml" rel="self"/>
 <link href="http://schrodinator.github.io/"/>
 <updated>{{ site.time | date_to_xmlschema }}</updated>
 <id>https://schrodinator.github.io/</id>
 <author>
   <name>schrodinator</name>
 </author>

 {% for post in site.posts %}
 <entry>
   <title>{{ post.title }}</title>
   <link href="https://schrodinator.github.io{{ post.url }}"/>
   <updated>{{ post.date | date_to_xmlschema }}</updated>
   <id>https://schrodinator.github.io{{ post.id }}</id>
   <content type="html">{{ post.content | xml_escape }}</content>
 </entry>
 {% endfor %}

</feed>
