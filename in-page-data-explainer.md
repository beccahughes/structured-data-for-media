# Schema.org Data for Media Explainer

Author: beccahughes@chromium.org

Last modified: 2020-01-29

## Objectives

User agents are adding much more functionality around media on the web. For example, Chromium browsers added [media controls](https://blog.google/products/chrome/manage-audio-and-video-in-chrome/) to the browser UI to control playback on the web.

To improve the functionality of such features we want to be able to add support for sites to provide more structured data to the user agent about the media on a page. Search engines currently use [schema.org](https://schema.org) data embedded into pages using [JSON+LD in script tags](https://w3c.github.io/json-ld-syntax/#embedding-json-ld-in-html-documents) as well as [microdata](https://www.w3.org/TR/microdata/).

### Goals

-   **A site should be able to provide structured data about media in a familiar way.** Sites are already familiar with how they provide structured data to search engines so as a user agent we should support the same format.
    
-   **A site should be able to add a more descriptive media type to the page.** Sites should be able to tell a user agent whether the playing content is a Movie or TV show and this allows user agents to provide a better experience when playing different content types.

### Non-goals

-   Non-media types (e.g. [Event](https://schema.org/Event)) are out of scope for now. However, we hope that this could pave the way for user agents to make more use of structured data in the future.

## API Design

### A site provides data about some video content

If a site is playing videos then they can embed a [VideoObject](https://schema.org/VideoObject) entity. If the site is playing a music video then they should use a [MusicVideoObject](https://schema.org/MusicVideoObject) entity instead.

```html
<script type="application/ld+json">
{
  "@context": "http://schema.org/",
  "@type": "VideoObject",
  "@id": "https://example.org/video",
  "author": {
    "@type": "Person",
    "name": "Test User",
    "url": "https://example.org/testuser"
  },
  "datePublished": "2020-01-27",
  "description": "This is a video that is about a bunny."
  "duration": "PT5M49S",
  "genre": "Animated Shorts",
  "interactionStatistic": {
    "@type": "InteractionCounter",
    "interactionType": "http://schema.org/WatchAction",
    "userInteractionCount": "4356"
  },
  "isFamilyFriendly": "http://schema.org/True",
  "mainEntityOfPage": "https://example.org/video",
  "name": "Big Buck Bunny",
  "potentialAction": {
    "@type": "WatchAction",
    "actionStatus": "http://schema.org/ActiveActionStatus",
    "startTime": "00:00:10",
    "target": "https://example.org/video?time=10s"
  },
  "thumbnail": {
    "@type": "ImageObject",
    "width": 360,
    "height": 480,
    "url": "https://example.org/video_thumbnail.png"
  } 
}
</script>
```

If a site supports resuming playback at the current position they can include a potentialAction that is a [WatchAction](https://schema.org/WatchAction) that has both an [ActiveActionStatus](http://schema.org/ActiveActionStatus) and a startTime (the time to resume playback). This can be used by a user agent for resuming playback later by navigating to the target URL of the watch action (if specified) or the URL of the page.

### A site provides data about a TV Episode

If a site is playing a TV episode then they can embed a [TVEpisode](https://schema.org/TVEpisode) entity. This entity can contain season information (through a [TVSeason](https://schema.org/TVSeason) stored in partOfSeason) as well as series information (though a [TVSeries](https://schema.org/TVSeries) stored in partOfSeries).

```html
<script type="application/ld+json">
{
  "@context": "http://schema.org/",
  "@type": "TVEpisode",
  "@id": "https://example.org/tv-epsiode",
  "author": {
    "@type": "Person",
    "name": "Test User",
    "url": "https://example.org/testuser"
  },
  "datePublished": "2020-01-27",
  "description": "This is some TV episode."
  "duration": "PT5M49S",
  "episodeNumber": 1,
  "genre": "Documentary",
  "partOfSeason": {
    "@type": "TVSeason",
    "seasonNumber": 1,
    "partOfSeries": {
      "@type": "TVSeries",
      "name": "TV Series Name",
      "thumbnail": {
        "@type": "ImageObject",
        "width": 360,
        "height": 480,
        "url": "https://example.org/music_thumbnail.png"
      } 
    }
  },
  "mainEntityOfPage": "https://example.org/tv-episode",
  "name": "Name of Episode"
}
</script>
```

A [TVSeries](https://schema.org/TVSeries) could be embedded directly in the page. However, this is less desirable since it does not provide information about the currently playing episode itself.

### A site provides data about a TV episode to watch next

A site can also provide a TV episode to watch next by embedded the next episode with a [PendingActionStatus](https://schema.org/PendingActionStatus) WatchAction. The user agent can then use the episodeNumber of the current episode to find the next episode in the current season.

```html
<script type="application/ld+json">
{
  "@context": "http://schema.org/",
  "@type": "TVEpisode",
  "@id": "https://example.org/tv-epsiode",
  "author": {
    "@type": "Person",
    "name": "Test User",
    "url": "https://example.org/testuser"
  },
  "datePublished": "2020-01-27",
  "description": "This is some TV episode."
  "duration": "PT5M49S",
  "episodeNumber": 1,
  "genre": "Documentary",
  "partOfSeason": {
    "@type": "TVSeason",
    "seasonNumber": 1,
    "episode": {
      "@type": "TVEpisode",
      "@id": "https://example.org/example-tv-series?e=2&s=1",
      "episodeNumber": 2,
      "potentialAction": {
        "@type": "WatchAction",
        "actionStatus": "http://schema.org/PendingActionStatus"
      },
      "name": "TV Episode 2 Name"
    },
    "partOfSeries": {
      "@type": "TVSeries",
      "name": "TV Series Name",
      "thumbnail": {
        "@type": "ImageObject",
        "width": 360,
        "height": 480,
        "url": "https://example.org/music_thumbnail.png"
      } 
    }
  },
  "mainEntityOfPage": "https://example.org/tv-episode",
  "name": "Name of Episode",
  "potentialAction": {
    "@type": "WatchAction",
    "actionStatus": "http://schema.org/ActiveActionStatus",
    "startTime": "00:00:10",
    "target": "https://example.org/video?time=10s"
  }
}
</script>
```

A site can also provide the first episode of the next season and the user agent can use that as a suggestion to watch next.

```html
<script type="application/ld+json">
{
  "@context": "http://schema.org/",
  "@type": "TVEpisode",
  "@id": "https://example.org/tv-epsiode",
  "author": {
    "@type": "Person",
    "name": "Test User",
    "url": "https://example.org/testuser"
  },
  "datePublished": "2020-01-27",
  "description": "This is some TV episode."
  "duration": "PT5M49S",
  "episodeNumber": 10,
  "genre": "Documentary",
  "partOfSeason": {
    "@type": "TVSeason",
    "seasonNumber": 1,
    "numberOfEpisodes": 10,
    "partOfSeries": {
      "@type": "TVSeries",
      "containsSeason": {
       "@type": "TVSeason",
       "seasonNumber": 2,
       "episode": {
         "@type": "TVEpisode",
         "@id": "https://example.org/example-tv-series?e=1&s=2",
         "episodeNumber": 1,
         "potentialAction": {
           "@type": "WatchAction",
           "actionStatus": "http://schema.org/PendingActionStatus"
         },
         "name": "TV Episode 1 Name"
        },
      },
      "name": "TV Series Name",
      "thumbnail": {
        "@type": "ImageObject",
        "width": 360,
        "height": 480,
        "url": "https://example.org/music_thumbnail.png"
      } 
    }
  },
  "mainEntityOfPage": "https://example.org/tv-episode",
  "name": "Name of Episode",
  "potentialAction": {
    "@type": "WatchAction",
    "actionStatus": "http://schema.org/ActiveActionStatus",
    "startTime": "00:00:10",
    "target": "https://example.org/video?time=10s"
  }
}
</script>
```

### A site provides data about a live video stream

If a site is playing a live video stream they can add a [BroadcastEvent](https://schema.org/BroadcastEvent) in the publication property (this is also on the TVEpisode and Movie media item types as well). The event should have the isLiveBroadcast property set to true and could include the startDate and endDate for the broadcast (if appropriate).

```html
<script type="application/ld+json">
{
  "@context": "http://schema.org/",
  "@type": "VideoObject",
  "@id": "https://example.org/video",
  "author": {
    "@type": "Person",
    "name": "Test User",
    "url": "https://example.org/testuser"
  },
  "datePublished": "2020-01-27",
  "description": "This is a video that is about a bunny."
  "duration": "PT5M49S",
  "genre": "Animated Shorts",
  "interactionStatistic": {
    "@type": "InteractionCounter",
    "interactionType": "http://schema.org/WatchAction",
    "userInteractionCount": "4356"
  },
  "isFamilyFriendly": "http://schema.org/True",
  "mainEntityOfPage": "https://example.org/video",
  "name": "Big Buck Bunny",
  "publication": {
    "@type": "BroadcastEvent",
    "isLiveBroadcast": "http://schema.org/True",
    "startDate": "2020-01-28T06:00:00+0000",
    "endDate": "2020-01-28T07:00:00+0000"
  },
  "thumbnail": {
    "@type": "ImageObject",
    "width": 360,
    "height": 480,
    "url": "https://example.org/video_thumbnail.png"
  } 
}
</script>
```

### A site provides data about a movie

If a site is playing movies then they can embed a [Movie](https://schema.org/Movie) entity.

```html
<script type="application/ld+json">
{
  "@context": "http://schema.org/",
  "@type": "Movie",
  "@id": "https://example.org/dream",
  "contentRating": {
    "@type": "Rating",
    "author": "MPAA",
    "ratingValue": "PG-13"
  }
  "datePublished": "2020-01-27",
  "description": "This is a movie about a dream."
  "duration": "PT5M49S",
  "genre": "Animated",
  "isFamilyFriendly": "http://schema.org/True",
  "mainEntityOfPage": "https://example.org/dream",
  "name": "Dream",
  "thumbnail": {
    "@type": "ImageObject",
    "width": 360,
    "height": 480,
    "url": "https://example.org/movie_thumbnail.png",
  } 
}
</script>
```

### A site provides data about some audio content

If a site is playing audio content then they can embed an [AudioObject](https://schema.org/AudioObject) entity.

```html
<script type="application/ld+json">
{
  "@context": "http://schema.org/",
  "@type": "AudioObject",
  "@id": "https://example.org/audio",
  "author": {
    "@type": "Person",
    "name": "Test User",
    "url": "https://example.org/testuser"
  },
  "datePublished": "2020-01-27",
  "description": "This is some audio."
  "duration": "PT5M49S",
  "genre": "Pop",
  "isFamilyFriendly": "http://schema.org/True",
  "mainEntityOfPage": "https://example.org/audio",
  "name": "Test Pop",
  "thumbnail": {
    "@type": "ImageObject",
    "width": 360,
    "height": 480,
    "url": "https://example.org/music_thumbnail.png"
  } 
}
</script>
```

### A site provides data about some music

If a site is playing music then they can embed a [MusicRecording](https://schema.org/MusicRecording) entity. This entity can contain playlist information (through a [MusicPlaylist](https://schema.org/MusicPlaylist) stored in inPlaylist) as well as album information (though a [MusicAlbum](https://schema.org/MusicAlbum) stored in inAlbum) and artist information (through a [Person](https://schema.org/Person) or [MusicGroup](https://schema.org/MusicGroup) stored in byArtist).

```html
<script type="application/ld+json">
{
  "@context": "http://schema.org/",
  "@type": "MusicRecording",
  "@id": "https://example.org/music-recording",
  "author": {
    "@type": "Person",
    "name": "Test User",
    "url": "https://example.org/testuser"
  },
  "datePublished": "2020-01-27",
  "description": "This is some audio."
  "duration": "PT5M49S",
  "genre": "Pop",
  "inPlaylist": {
    "@type": "MusicPlaylist",
    "name": "Becca’s Pop Playlist"
  },
  "inAlbum": {
    "@type": "MusicAlbum",
    "name": "Album Name",
    "byArtist": {
      "@type": "MusicGroup",
      "name": "Band Name"
    }
  },
  "mainEntityOfPage": "https://example.org/music-recording",
  "name": "Track Name",
  "thumbnail": {
    "@type": "ImageObject",
    "width": 360,
    "height": 480,
    "url": "https://example.org/music_thumbnail.png"
  } 
}
</script>
```

A [MusicAlbum](https://schema.org/MusicAlbum), [MusicGroup](https://schema.org/MusicGroup) or [MusicPlaylist](https://schema.org/MusicPlaylist) could be embedded directly in the page. However, they are less desirable since they do not provide information about the currently playing track itself.

### Recommendations around properties

It is **highly recommended** that all media items should have the following properties:

1.  name - the name of the content.
    
2.  author or creator - the author / creator of the content.
    
3.  datePublished - the timestamp the content was published.
    
4.  duration - the duration of the content.
    
5.  isFamilyFriendly - if the content is family friendly.
    
6.  thumbnail or image - artwork to be displayed by the user agent. We recommend that these are [ImageObject](https://schema.org/ImageObject) with the url, width and height. This allows the user agent to pick the right artwork based on the size.
    
7.  mainEntityOfPage - contains the current URL of the page. This tells the user agent that this entity should be used if there are other entities on the page.

It is suggested that the media items have the following properties:

1.  interactionStatistic- this count of any interactions on the content. The recommended interactions to support are [WatchAction](https://schema.org/WatchAction), [LikeAction](https://schema.org/LikeAction) and [DislikeAction](https://schema.org/DislikeAction).
    
2.  genre - the genre of the content.
    
3.  potentialAction - any potential action for continue watching and play next. These should have an actionStatus and if needed a target URL. If the actionStatus is ActiveActionStatus the action should also have a start time.

6.  contentRating - the content rating of the content (e.g. PG-18).
    
7.  name + thumbnail/image (for TVSeries) - the name and artwork for the TV series.
    
8.  seasonNumber (for TVSeason) - the number of the season in the TV series, should be sequential.
    
9.  episodeNumber (for TVEpisode) - the number of the episode in the current season, should be sequential.
    
10.  numberOfEpisodes (for TVSeries or TVSeason) - the total number of episodes has series / season has.
    
11.  numberOfSeasons (for TVSeries) - the total number of seasons a series has.
