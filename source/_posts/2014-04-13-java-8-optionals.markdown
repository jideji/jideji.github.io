---
layout: post
title: "Java 8 Optionals"
date: 2014-04-13 09:01:47 +0100
comments: true
published: false
categories: [java8, lambda, optional, programming, functional programming]
---
When I first heard that java8 will include Optionals as part of the standard library, I wasn't too excited.

Guava Optional had already been out there for a long time, and so we had tried to use it at work in places where it makes sense. It turned out that it was mostly useful for its .or(value) method, returning a default value. In any other case you would have to create a Supplier, and you would have to call the .get() call in the end anyway to get the actual value out.

What I didn't think about was the fact that the problems with creating Suppliers go away. Also, in the java8 version they have introduced a new method .ifPresent(Consumer).

This means that you can use it in cases where you want to build something.

Say you want to build a string based on a bean that has some optional data.

{% codeblock lang:java %}
class Song {
    private final Artist artist;
    private final String title;
    private final Optional<Artist> singer;

    Song(Artist artist, String title, Artist singer) {
        this.artist = artist;
        this.title = title;
        this.singer = Optional.of(singer);
    }

    Song(Artist artist, String title) {
        this.artist = artist;
        this.title = title;
        this.singer = Optional.empty();
    }

    Artist artist() { return artist; }
    String title() { return title; }
    Optional<Artist> singer() { return singer; }
}
class Artist {
    private final String name;

    Artist(String name) {
        this.name = name;
    }

    static Artist artist(String name) {
        return new Artist(name);
    }

    String name() { return name; }
}
{% endcodeblock %}

You have a class that wants to print out the song details:

{% codeblock lang:java %}
public class SongInfoPrinter {
    private static final Song song1 = new Song(
            artist("Stereophonics"),
            "She's Alright",
            artist("Kelly Jones"));
    private static final Song song2 = new Song(
            artist("Beegie Adair"),
            "Manhattan");

    public static void main(String[] args) {
        print(song1);
        print(song2);
    }
    
    ...
}
{% endcodeblock %}

You only want to print out the Lead singer if there is one. You can then do the following:

{% codeblock lang:java %}
    private static void print(Song song) {
        StringBuilder sb = new StringBuilder();
        sb
            .append("Artist: ").append(song.artist().name()).append("\n")
            .append("Title: ").append(song.title()).append("\n");
        // singer is optional
        song.singer().ifPresent(singer ->
                        sb.append("Lead singer: ").append(singer.name()).append("\n")
        );
        System.out.println(sb);
    }
{% endcodeblock %}

Optionals also have a mapping function, meaning it can transform the data if available.

{% codeblock lang:java %}
    private static void printLengthOfSingerName(Song song) {
        song.singer()
                .map(singer -> singer.name().length())
                .ifPresent(l -> System.out.println("Length of singer name: " + l));
    }
{% endcodeblock %}

As with the guava version of Optional, even if an object is not explicitly an Optional but a possible null then you can turn it into an Optional. Also, you can use multiple map functions to protect yourself against any null pointer exceptions:

{% codeblock lang:java %}
    private static void printLengthOfArtistName(Song song) {
        Optional.ofNullable(song.artist())
                .map(artist -> artist.name()) // will not be invoked if artist is null
                .map(name -> name.length()) // will not be invoked if name is null
                .ifPresent(l -> System.out.println("Length of singer name: " + l));
    }
{% endcodeblock %}


This is great for the [builder pattern](http://en.wikipedia.org/wiki/Builder_pattern) in general!

I do wish they would have added an .ifAbsent(Runnable) method though, as you sometimes might want to perform an operation if the optional is empty. Also, it would have been great if you could have chained .ifPresent() calls. Then you could have called

{% codeblock lang:java %}
// not possible
optional
    .ifPresent(e -> someSometing(e))
    .ifAbsent(() -> doSomethingElse());
{% endcodeblock %}
