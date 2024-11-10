---
title: "Hacked by a bug?"
date: "2016-09-13"
---

Early this afternoon, I had just recorded a wide-ranging episode of [Arrested DevOps](https://www.arresteddevops.com/) with the incomparable [Bridget Kromhout](https://twitter.com/bridgetkromhout) and noticed that I had a flurry of Twitter mentions, all in reaction to [this tweet of mine](https://twitter.com/bcantrill/status/775392664602554369). There was just one problem: **I didn't tweet it**. With my account obviously hacked, I went into fight-or-flight mode and (thanks in no small part to Bridget's calm presence) did the obvious things: I changed my Twitter password, revoked the privileges of all applications, and tried to assess the damage...

Other than the tweet, I (thankfully!) didn't see any obvious additional damage: no crazy DMs or random follows or unfollows. In terms of figuring out where the malicious tweet had come from, the source of the tweet was "Twitter for Android" -- but according to my login history, the last Twitter for Android login was from me during my morning commute about two-and-a-half hours before the tweet. (And according to Twitter, I have only used the one device to access my account.) The only intervening logins were two from Quora about an hour prior to the tweet. (Aside: WTF, Quora?! Revoked!)

Then there was the oddity of the tweet itself. There was no caption -- just the two images from what I gathered to be Germany. Looking at the raw tweet, however, cleared up its source:

```
{
  "created_at": "Mon Sep 12 17:56:31 +0000 2016",
  "id": 775392664602554400,
  "id_str": "775392664602554369",
  "text": "https://t.co/pYKRhaAdvC",
  "truncated": false,
  "entities": {
    "hashtags": [],
    "symbols": [],
    "user_mentions": [],
    "urls": [],
    "media": [
      {
        "id": 775378240244449300,
        "id_str": "775378240244449280",
        "indices": [
          0,
          23
        ],
        "media_url": "http://pbs.twimg.com/media/CsKyZsBWgAAHgVq.jpg",
        "media_url_https": "https://pbs.twimg.com/media/CsKyZsBWgAAHgVq.jpg",
        "url": "https://t.co/pYKRhaAdvC",
        "display_url": "pic.twitter.com/pYKRhaAdvC",
        "expanded_url": "https://twitter.com/MattAndersonBBC/status/775378264772775936/photo/1",
        "type": "photo",
        "sizes": {
          "medium": {
            "w": 1200,
            "h": 1200,
            "resize": "fit"
          },
          "large": {
            "w": 2048,
            "h": 2048,
            "resize": "fit"
          },
          "thumb": {
            "w": 150,
            "h": 150,
            "resize": "crop"
          },
          "small": {
            "w": 680,
            "h": 680,
            "resize": "fit"
          }
        },
        "source_status_id": 775378264772776000,
        "source_status_id_str": "775378264772775936",
        "source_user_id": 1193503572,
        "source_user_id_str": "1193503572"
      }
    ]
  },
  "extended_entities": {
    "media": [
      {
        "id": 775378240244449300,
        "id_str": "775378240244449280",
        "indices": [
          0,
          23
        ],
        "media_url": "http://pbs.twimg.com/media/CsKyZsBWgAAHgVq.jpg",
        "media_url_https": "https://pbs.twimg.com/media/CsKyZsBWgAAHgVq.jpg",
        "url": "https://t.co/pYKRhaAdvC",
        "display_url": "pic.twitter.com/pYKRhaAdvC",
        "expanded_url": "https://twitter.com/MattAndersonBBC/status/775378264772775936/photo/1",
        "type": "photo",
        "sizes": {
          "medium": {
            "w": 1200,
            "h": 1200,
            "resize": "fit"
          },
          "large": {
            "w": 2048,
            "h": 2048,
            "resize": "fit"
          },
          "thumb": {
            "w": 150,
            "h": 150,
            "resize": "crop"
          },
          "small": {
            "w": 680,
            "h": 680,
            "resize": "fit"
          }
        },
        "source_status_id": 775378264772776000,
        "source_status_id_str": "775378264772775936",
        "source_user_id": 1193503572,
        "source_user_id_str": "1193503572"
      },
      {
        "id": 775378240248614900,
        "id_str": "775378240248614912",
        "indices": [
          0,
          23
        ],
        "media_url": "http://pbs.twimg.com/media/CsKyZsCWEAA4oOp.jpg",
        "media_url_https": "https://pbs.twimg.com/media/CsKyZsCWEAA4oOp.jpg",
        "url": "https://t.co/pYKRhaAdvC",
        "display_url": "pic.twitter.com/pYKRhaAdvC",
        "expanded_url": "https://twitter.com/MattAndersonBBC/status/775378264772775936/photo/1",
        "type": "photo",
        "sizes": {
          "small": {
            "w": 680,
            "h": 680,
            "resize": "fit"
          },
          "thumb": {
            "w": 150,
            "h": 150,
            "resize": "crop"
          },
          "medium": {
            "w": 1200,
            "h": 1200,
            "resize": "fit"
          },
          "large": {
            "w": 2048,
            "h": 2048,
            "resize": "fit"
          }
        },
        "source_status_id": 775378264772776000,
        "source_status_id_str": "775378264772775936",
        "source_user_id": 1193503572,
        "source_user_id_str": "1193503572"
      }
    ]
  },
  "source": "Twitter for Android",
  "in_reply_to_status_id": null,
  "in_reply_to_status_id_str": null,
  "in_reply_to_user_id": null,
  "in_reply_to_user_id_str": null,
  "in_reply_to_screen_name": null,
  "user": {
    "id": 173630577,
    "id_str": "173630577",
    "name": "Bryan Cantrill",
    "screen_name": "bcantrill",
    "location": "",
    "description": "Nom de guerre: Colonel Data Corruption",
    "url": "http://t.co/VyAyIJP8vR",
    "entities": {
      "url": {
        "urls": [
          {
            "url": "http://t.co/VyAyIJP8vR",
            "expanded_url": "http://dtrace.org/blogs/bmc",
            "display_url": "dtrace.org/blogs/bmc",
            "indices": [
              0,
              22
            ]
          }
        ]
      },
      "description": {
        "urls": []
      }
    },
    "protected": false,
    "followers_count": 10407,
    "friends_count": 1557,
    "listed_count": 434,
    "created_at": "Sun Aug 01 23:51:44 +0000 2010",
    "favourites_count": 2431,
    "utc_offset": -25200,
    "time_zone": "Pacific Time (US & Canada)",
    "geo_enabled": true,
    "verified": false,
    "statuses_count": 4808,
    "lang": "en",
    "contributors_enabled": false,
    "is_translator": false,
    "is_translation_enabled": false,
    "profile_background_color": "C0DEED",
    "profile_background_image_url": "http://abs.twimg.com/images/themes/theme1/bg.png",
    "profile_background_image_url_https": "https://abs.twimg.com/images/themes/theme1/bg.png",
    "profile_background_tile": false,
    "profile_image_url": "http://pbs.twimg.com/profile_images/618537697670397952/gW9iQsvF_normal.jpg",
    "profile_image_url_https": "https://pbs.twimg.com/profile_images/618537697670397952/gW9iQsvF_normal.jpg",
    "profile_link_color": "0084B4",
    "profile_sidebar_border_color": "C0DEED",
    "profile_sidebar_fill_color": "DDEEF6",
    "profile_text_color": "333333",
    "profile_use_background_image": true,
    "has_extended_profile": false,
    "default_profile": true,
    "default_profile_image": false,
    "following": false,
    "follow_request_sent": false,
    "notifications": false
  },
  "geo": null,
  "coordinates": null,
  "place": {
    "id": "5a110d312052166f",
    "url": "https://api.twitter.com/1.1/geo/id/5a110d312052166f.json",
    "place_type": "city",
    "name": "San Francisco",
    "full_name": "San Francisco, CA",
    "country_code": "US",
    "country": "United States",
    "contained_within": [],
    "bounding_box": {
      "type": "Polygon",
      "coordinates": [
        [
          [
            -122.514926,
            37.708075
          ],
          [
            -122.357031,
            37.708075
          ],
          [
            -122.357031,
            37.833238
          ],
          [
            -122.514926,
            37.833238
          ]
        ]
      ]
    },
    "attributes": {}
  },
  "contributors": null,
  "is_quote_status": false,
  "retweet_count": 2,
  "favorite_count": 9,
  "favorited": false,
  "retweeted": false,
  "possibly_sensitive": false,
  "possibly_sensitive_appealable": false,
  "lang": "und"
}

```

Note in particular that the media has a `source_status_id_str` of 775378264772775936; it's from [this tweet](https://twitter.com/MattAndersonBBC/status/775378264772775936/) roughly an hour before mine from [Matt Anderson](https://twitter.com/MattAndersonBBC), the BBC Culture editor who (I gather) is Berlin-based.

Why would someone who had just hacked my account burn it by tweeting an innocuous (if idiosyncratic) photo of campaign posters on the streets of Berlin?! Suddenly this is feeling less like I've been hacked, and more like I'm the victim of data corruption.

Some questions I have, that I don't know enough about the Twitter API to answer: first, how are tweets created that refer to media entities from other tweets? i.e., is there something about that tweet that can give a better clue as to how it was generated? Does the fact that it's geolocated to San Francisco (albeit with the broadest possible coordinates) indicate that it might have come from the Twitter client misbehaving on my phone? (I didn't follow Matthew Anderson and my phone was on my desk when this was tweeted -- so this would be the app going seriously loco.) And what I'm most dying to know: what other tweets refer to the photos from the tweet from Matthew? (I gather that [DataSift can answer this question](http://dev.datasift.com/docs/products/stream/features/sources/public-sources/twitter/twitter-media-sourcestatusid), but I'm not a DataSift customer and they don't appear to have a free tier.) If there's a server-side bug afoot here, it wouldn't be surprising if I'm not the only one affected.

I'm not sure I'm ever going to know the answers to these questions, but I'm leaving the tweet up there in hopes that it will provide some clues -- and with the belief that the villain in the story, if ever brought to justice, will be a member of the shadowy cabal that I have fought my entire career: busted software.
