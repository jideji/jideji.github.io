---
layout: post
title: "Using jq To Manipulate JSON"
date: 2014-10-18 20:42:38 +0100
comments: true
categories: [bash,jq,tools]
---

Most of our web services at work speak json. Sometimes it is nice to be able to extract select information from the responses.
I find [jq](http://stedolan.github.io/jq/) to be exceptionally suited for these situations.

The syntax can be perplexing in the beginning, and the tutorial and manual do not (in my opinion) explain in enough
detail how you should think when writing your filters, and how you combine them.

The program takes what is called a filter. A filter basically generates output that is usually based on the input.

## Basic usage

The simplest case is where you just copy the input to output:

{% codeblock lang:bash Simple case %}
$ echo '{"key": "value"}' | jq .
{
  "key": "value"
}
{% endcodeblock %}

Since jq pretty-prints the json by default (can be changed to compact mode by using the `-c` flag) the output is nicely formatted.

The single dot just tells jq to copy the root node. If you just want the value you can type:

{% codeblock lang:bash Extract the value of field 'key' %}
$ echo '{"key": "value"}' | jq .key
"value"
$ echo '{"key": "value"}' | jq -r .key
value
{% endcodeblock %}

As you can see, `-r` (_raw-output_) unwraps the string.

You can use multiple filters on your data. Let us say that you want to count the number of elements in an array. You 
first want to pick out the array, and then run a filter called `length` on it:

{% codeblock lang:bash Extract the value of field 'key' %}
$ echo '{"list": [ "value1", "value2", "value3" ]}' | jq '.list | length'
3
{% endcodeblock %}

In this example `.list` picks the array from the `list` field. This is piped to the `length` filter which returns the
number of elements in the array.

## Advanced usage

Now let us consider a slightly more complicated json object. Spotify has an open API where you can search for
tracks and albums.

Let us assume that we want to find songs from the Book Of Mormons.

We will use this search: https://api.spotify.com/v1/search?type=track&market=gb&q=The+Book+Of+Mormon

The json has the following format (excluding a lot of to us uninteresting data):

{% codeblock lang:json Spotify json formation %}
{
  "tracks": {
    "items": [
      {
        "album": { ... },
        "artists": [
          {
            "name": "artist name 1"
          },
          ...
        ],
        "name": "track name",
        "track_number": 1,
        "duration_ms": 123845
      }
    ]
  }
}
{% endcodeblock %}

Store the search result as a file:

{% codeblock lang:bash Save the search result as json %}
$ curl 'https://api.spotify.com/v1/search?type=track&market=gb&q=The+Book+Of+Mormon' > tracks.json
{% endcodeblock %}

OK now let's figure out who is performing in these songs:

{% codeblock lang:bash Extract artist names %}
$ jq -r '[.tracks.items[].artists[].name] | unique | .[]' tracks.json
Andrew Rannells
Asmeret Gebremichael
Benjamin Schrader
Brian Sears
Brian Tyree Henry
Clark Johnson
...
{% endcodeblock %}

Woah, OK that was a big step! Let us break it down.

- `.tracks.items[].artists[].name` clearly picks all the artist names
- this is then wrapped in an array
- the array is sent to a `unique` filter which sorts the array and removes any duplicates
- `.[]` unwraps the array (we would otherwise see an array being returned by jq)

The second step is hinting at another feature of jq: you can construct json objects, not only pick them out from
an existing json object.

Now let us say that we want to construct new json from the track.json file.
We want to create a simplifed tracklist for the album.. The tracks should be sorted by track index. We start
off by creating the list of tracks.

{% codeblock lang:bash Track information, sorted by index %}
$ jq -r '
> [
>   .tracks.items[] |
>   {
>     index: .track_number,
>     name,
>     album: .album.name,
>   }] | sort_by(.index)' tracks.json
[
  {
    "index": 1,
    "name": "Hello!",
    "album": "The Book of Mormon"
  },
  {
    "index": 1,
    "name": "The Book of Mormon in 3 Minutes",
    "album": "The Book of Mormon in 3 Minutes"
  },
  {
    "index": 2,
    "name": "Book Of Mormon Stories/The Golden Plates",
    "album": "Sharing Time"
  },
  ...
]
{% endcodeblock %}

Kind of works. As we don't specify the source of `name`, jq will map it to the input `name`. 
Unfortunately Spotify doesn't let you search exact album titles, so will have to do the filtering ourselves:

{% codeblock lang:bash Track information only for tracks from the specific album, sorted by index %}
$ jq -r '
> [
>   .tracks.items[] |
>   select(.album.name == "The Book of Mormon") |
>   {
>     index: .track_number,
>     name,
>     album: .album.name,
>     length: (.duration_ms / 1000 / 60)
>   }
> ] | sort_by(.index)' tracks.json
[
  {
    "index": 1,
    "name": "Hello!",
    "album": "The Book of Mormon",
    "length": 2.8671
  },
  {
    "index": 2,
    "name": "Two By Two",
    "album": "The Book of Mormon",
    "length": 4.5262166666666666
  },
  {
    "index": 3,
    "name": "You And Me (But Mostly Me)",
    "album": "The Book of Mormon",
    "length": 2.720666666666667
  },
  ...
]
{% endcodeblock %}

Much better! `select` only picks the elements that match the given criteria. We also added a `duration` while we were
at it. As you can see simple maths can be applied.

Now that we have the data we want, we can generate the final json we wanted:

{% codeblock lang:bash Album information %}
$ jq -r '
> [
>   .tracks.items[] |
>   select(.album.name == "The Book of Mormon") |
>   {
>     index:.track_number,
>     name,
>     length:(.duration_ms / 1000 / 60)
>   }
> ] | sort_by(.index) |
> {
>   title: "The Book of Mormon",
>   average_length: ([ .[].length ] | add / length),
>   tracks: .
> }' tracks.json
{
  "title": "The Book of Mormon",
  "average_length": 4.2499520833333335,
  "tracks": [
    {
      "index": 1,
      "name": "Hello!",
      "length": 2.8671
    },
    {
      "index": 2,
      "name": "Two By Two",
      "length": 4.5262166666666666
    },
    {
      "index": 3,
      "name": "You And Me (But Mostly Me)",
      "length": 2.720666666666667
    },
    ...
  ]
}
{% endcodeblock %}

We have the same transformation as before, and then we send it to a new filter:

- `title` is a hardcoded string
- `average_length` is a calculation of the average length of the songs
  - we extract the length of each song and put them in an array. We then pipe it to both `add` and `length`. `add` will
   sum up all the values in the array, while length returns the length of the array
- `tracks` is simply the json we generated before

So you can see that jq is incredibly powerful when it comes to manipulating json. Don't worry if you struggle to
understand everything. To be honest I keep forgetting how it works, which is the reason I put this post together.

The best way to learn is to simply experiment with it. There is an online tool where you
can try: [jqplay.org](https://jqplay.org/)

There are more functions available, but these are most of what I use. You can find them all in the
[manual](http://stedolan.github.io/jq/manual/). Just make sure you have the latest version of jq!
