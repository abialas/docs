---
title: Send WhatsApp Messages
excerpt: Send messages via WhatsApp with Sinch WhatsApp API. Get more information here.
next:
  pages:
    - whatsapp-media-provider
---
The message endpoint is used as the primary endpoint of the API and this is where all the messages are sent through.

## WhatsApp message flow

![image](images\whatsapp-msg-flow.png)

1.  Customer opt-in is essential before sending any messages.
2.  Businesses can only start a conversation with a defined message template.
3.  Once you get a reply from your customer, a *customer care session* starts. You can then send “session” rich content messages for 24 hours.
4.  Every time a customer replies to one of your messages, a new 24-hour cycle starts.
5.  If a “session” expires, you’ll need to re-initiate a conversation, starting with a defined message template again.
6.  Customers can start a rich content conversation with a business at any time  
    -   this opens up a new 24-hour session.


## Send a WhatsApp message

### Request
`POST whatsapp/v1/{bot-id}/messages`

JSON object parameters:

| Name    | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| to      | List of MSISDNs                                                      | String array | N/A        | 1 to 20 elements      | Yes      |
| message | Message object                                                       | Object       | N/A        | Valid Message object  | Yes      |
| callback| Callback URL to overwrite configured callback URL for status updates | String       | N/A        | Valid URL             | No       |

### Response

`201 Created`

The response body is a JSON object with the same format as a [delivery report callback](doc:whatsapp-callback#delivery-report-callback).

```json
{
  "type": "whatsapp",
  "statuses":[
    {
      "message_id":"01DPNXZ0WCF9XD19MH84XD0P62",
      "recipient":"+46732001122",
      "status":"success",
      "state":"queued"
    }
  ]
}
```

`400 Bad Request`

There was an error with your request. The body is a JSON object described in the [introduction](doc:whatsapp-introduction#http-errors).

`401 Unauthorized`

There was an authentication error with your request. Either you're using incorrect credentials or you're attempting to authenticate
in a region where your bot does not reside. The body is a JSON object described in the [introduction](doc:whatsapp-introduction#http-errors).

## Message object types

The types of messages that can be sent are one of the following:

### Template message

Accepted language codes can be found in the [introduction](doc:whatsapp-introduction#supported-language-codes). You can use [media provider](doc:whatsapp-media-provider) feature in template messages.

> 📘 Note
> 
> In the release on **29th June 2021** we are going to deprecate use of the `ttl` parameter when sending template messages. Facebook is not going to support this feature anymore.

JSON object parameters:

| Name          | Description                                                           | JSON Type    | Default    | Constraints           | Required |
| ------------- | --------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| type          | Constant value `template`.                                            | String       | N/A        | N/A                   | Yes      |
| template_name | Name of the template.                                                 | String       | N/A        | N/A                   | Yes      |
| language      | Language to send the template in.                                     | String       | `en`       | Language codes and locales (e.g `en`, `en_us`) | No       |
| header_params | Parameter to inject into the header of the template.                  | String array | N/A        | Can only used when there is a header of type text in the template. Up to 60 characters for all parameters and predefined template header text. | No      |
| body_params   | Parameters to inject into the body of the template.                   | String array | N/A        | Up to 1024 characters for all parameters and predefined template text.        | No      |
| media         | An object describing the document, image or video to include in the header of the template. The objects are the same as described under Document message, Image message and Video message below, except that the `caption` parameter is not allowed. Also see the note below. For a message without media, set the media type to `text`.       | String array | N/A        | N/A                   | Yes     |
| buttons       | A list of buttons to include in the template message.                 | List of button objects | N/A        | N/A                   | Yes, if the template definition includes either at least one quick reply button or a dynamic URL button. |
| ttl           | DEPRECATED. | String       | 30 Days    | See description | No      |

> 📘 Note
>
> The `caption` parameter is not supported for media in template messages. For document media, the `filename` parameter can be used to describe the file. If the `filename` parameter is not explicitly used, it will take the default value "Filename".
> 
> Audio template messages are not supported.

```json
{
    "to": [
        "46732001122"
    ],
    "message": {
        "type": "template",
        "template_name": "test_template",
        "language": "en",
        "body_params": [
            "param here"
        ],
        "media": {
            "type": "text"
        }
    }
}
```

```json
{
    "to": [
    	"46732001122"
    ],
    "message": {
        "type": "template",
		"template_name": "demo_rich_text",
		"language": "en",
		"header_params": [
              "Nick"
            ],
        "body_params": [
          "Swan Lake"
        ],
        "media": {
            "type": "text"
        }
    }
}
```

### Templates with buttons

- Call button

| Name          | Description                                                          | JSON Type    | Constraints           | Required |
| ------------- | -------------------------------------------------------------------- | ------------ | --------------------- | :------: |
| type          | The type of button.                                                  | String       | `call`                | Yes      |

- URL button

| Name          | Description                                                          | JSON Type    | Constraints           | Required |
| ------------- | -------------------------------------------------------------------- | ------------ | --------------------- | :------: |
| type          | The type of button.                                                  | String       | `url`                 | Yes      |
| parameter     | The URL parameter for the variable part of the URL.                  | String       | N/A                   | Yes, if the button is a dynamic URL button. |

- Quick reply button

| Name          | Description                                                          | JSON Type    | Constraints           | Required |
| ------------- | -------------------------------------------------------------------- | ------------ | --------------------- | :------: |
| type          | The type of button.                                                  | String       | `quick_reply`         | Yes      |
| payload       | A payload to return when the recipient presses the button.           | String       | N/A                   | No       |

[Find more button examples here](doc:whatsapp-examples).

```json
{
  "to": [
    "46732001122"
  ],
  "message": {
    "type": "template",
    "template_name": "some_template_name",
    "language": "en",
    "header_params": [
      "a_parameter"
    ],
    "body_params": [
      "some_first_parameter",
      "some_second_parameter"
    ],
    "media" : {
      "type": "image",
      "url": "https://www.example.com/some_image.jpg",
      "provider": "some_provider_name"
    },
    "buttons" : [
      {
        "type": "call"
      },
      {
        "type": "url",
        "parameter": "some_url_parameter"
      }
    ],
    "ttl": "P1D"
  }
}
```

```json
{
  "to": [
    "46732001122"
  ],
  "message": {
    "type": "template",
    "template_name": "some_template_name",
    "language": "en",
    "header_params": [
      "some_parameter"
    ],
    "body_params": [
      "some_first_parameter",
      "some_second_parameter"
    ],
    "media" : {
      "type": "text"
    },
    "buttons" : [
      {
        "type": "quick_reply",
        "payload": "some_quick_reply_payload"
      },
      {
        "type": "quick_reply"
      }
    ]
  }
}
```

### Text message

Available formatting and using emojis for the text message content can be found in the [introduction](doc:whatsapp-introduction#formatting-text-messages).

JSON object parameters:

| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| type        | Constant value `text`                                                | String       | N/A        | N/A                   | Yes      |
| preview_url | Message object                                                       | Boolean      | false      | `true` or `false`     | No       |
| text        | The text message content                                             | String       | N/A        | Up to 4096 characters | Yes      |

```json
{
  "to": [
    "46732001122"
  ],
  "message": {
    "type": "text",
    "preview_url": false,
    "text": "Greetings from Sinch"
  }
}
```

### Image message

> 📘 Note
>
> Any media file sent through the Sinch WhatsApp API can be at most 100.0 MB

Accepted content types can be found in the [introduction](doc:whatsapp-introduction#accepted-media-types).

JSON object parameters:

| Name        | Description                                                               | JSON Type    | Default    | Constraints           | Required |
| ----------- | ------------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| type        | Constant value `image`                                                    | String       | N/A        | N/A                   | Yes      |
| url         | Public url of the image file. Should be either HTTP or HTTPS link.        | String       | N/A        | Accepted Content-Type header | Yes      |
| caption     | Optional caption that will be displayed underneath the image.             | String       | None       | N/A                   | No       |
| provider    | Optional name of a provider to be used when trying to download the file.  | String       | None       | N/A                   | No       |

```json
{
  "to": [
    "46732001122"
  ],
  "message": {
    "type": "image",
    "url": "https://example.com/image.jpg",
    "caption": "Example Image"
  }
}
```

### Video message

> 📘 Note
>
> Any media file sent through the Sinch WhatsApp API can be at most 100.0 MB

Accepted content types can be found in the [introduction](doc:whatsapp-introduction#accepted-media-types).

JSON object parameters:

| Name        | Description                                                               | JSON Type    | Default    | Constraints           | Required |
| ----------- | ------------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| type        | Constant value `video`                                                    | String       | N/A        | N/A                   | Yes      |
| url         | Public url of the video file (mp4). Should be either HTTP or HTTPS link.  | String       | N/A        | Accepted Content-Type header| Yes      |
| caption     | Optional caption that will be displayed underneath the video.             | String       | None       | N/A                   | No       |
| provider    | Optional name of a provider to be used when trying to download the file.  | String       | None       | N/A                   | No       |

```json
{
  "to":[
    "46732001122"
  ],
  "message":{
    "type": "video",
    "url": "https://example.com/video.mp4",
    "caption": "Example Video",
    "provider": "your-bearer-provider"
  }
}
```

### Document message

> 📘 Note
>
> Any media file sent through the Sinch WhatsApp API can be at most 100.0 MB

Accepted content types can be found in the [introduction](doc:whatsapp-introduction#accepted-media-types).

JSON object parameters:

| Name        | Description                                                               | JSON Type    | Default    | Constraints           | Required |
| ----------- | ------------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| type        | Constant value `document`                                                 | String       | N/A        | N/A                   | Yes      |
| url         | Public url of the document file. Should be either HTTP or HTTPS link.     | String       | N/A        | Accepted Content-Type header| Yes      |
| filename    | Optional parameter that describes the filename of the document.           | String       | None       | N/A                   | No       |
| caption     | Optional caption that will be displayed as the document title.            | String       | None       | N/A                   | No       |
| provider    | Optional name of a provider to be used when trying to download the file.  | String       | None       | N/A                   | No       |

```json
{
  "to": [
    "46732001122"
  ],
  "message": {
    "type": "document",
    "url": "https://example.com",
    "caption": "Example study",
    "filename": "document.pdf"
  }
}
```

### Audio message

> 📘 Note
>
> Any media file sent through the Sinch WhatsApp API can be at most 100.0 MB

Accepted content types can be found in the [introduction](doc:whatsapp-introduction#accepted-media-types).

JSON object parameters:

| Name        | Description                                                               | JSON Type    | Default    | Constraints           | Required |
| ----------- | ------------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| type        | Constant value `audio`                                                    | String       | N/A        | N/A                   | Yes      |
| url         | Public url of the audio file. Should be either HTTP or HTTPS link.        | String       | N/A        | Accepted Content-Type header| Yes      |
| provider    | Optional name of a provider to be used when trying to download the file.  | String       | None       | N/A                   | No       |

```json
{
  "to": [
    "46732001122"
  ],
  "message": {
    "type": "audio",
    "url": "https://example.com/song.mp3"
  }
}
```


### Location message

JSON object parameters:

| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| type        | Constant value `location`                                            | String       | N/A        | N/A                   | Yes      |
| lat         | The latitude position as a float number.                             | Number       | N/A        | [-90, 90]             | Yes      |
| lng         | The longitude position as a float number.                            | Number       | N/A        | [-180, 180]           | Yes      |
| name        | The name for the location. Will be displayed in the message.         | String       | N/A        | N/A                   | No       |
| address     | The address for the location. Will be displayed in the message.      | String       | N/A        | N/A                   | No       |

```json
{
  "to": [
    "46732001122"
  ],
  "message": {
    "type": "location",
    "lat": 55.7047,
    "lng": 13.191,
    "name": "Sinch Ideon Lund",
    "address": "Scheelevägen 17"
  }
}
```
### Contacts message

JSON object parameters:

| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| type        | Constant value `contacts`                                            | String       | N/A        | N/A                   | Yes      |
| contacts    | List of contact cards                                                | Object array | N/A        | Valid contact cards   | Yes      |

```json
{
  "to": [
    "46732001122"
  ],
  "message": {
    "type": "contacts",
    "contacts": [
      {
        "addresses": [
          {
            "city": "Menlo Park",
            "country": "United States",
            "country_code": "us",
            "state": "CA",
            "street": "1 Hacker Way",
            "type": "HOME",
            "zip": "94025"
          }
        ],
        "birthday": "2012-08-18",
        "emails": [
          {
            "email": "test@fb.com",
            "type": "WORK"
          }
        ],
        "name": {
          "first_name": "John",
          "formatted_name": "John Smith",
          "last_name": "Smith"
        },
        "org": {
          "company": "WhatsApp",
          "department": "Design",
          "title": "Manager"
        },
        "phones": [
          {
            "phone": "+1 (650) 555-1234",
            "type": "WORK",
            "wa_id": "16505551234"
          }
        ],
        "urls": [
          {
            "url": "https://www.facebook.com",
            "type": "WORK"
          }
        ]
      }
    ]
  }
}
```

### Sticker message

Custom sticker must comply with WhatsApp requirements:
1. Each sticker should have a transparent background.
2. Stickers must be exactly 512x512 pixels.
3. Each sticker must be less than 100 KB.

Please note that WhatsApp does not support animated stickers.

> 📘 Note
>
> For more information on using a custom sticker, please visit [WhatsApp sticker page](https://faq.whatsapp.com/en/general/26000345) 
>

Accepted content types can be found in the [introduction](doc:whatsapp-introduction#accepted-media-types).

JSON object parameters:

| Name        | Description                                                                    | JSON Type    | Default    | Constraints                                                               | Required |
| ----------- | ------------------------------------------------------------------------------ | ------------ | ---------- | ------------------------------------------------------------------------- | :------: |
| type        | Constant value `sticker`.                                                      | String       | N/A        | `sticker`                                                                 | Yes      |
| url         | Public url of the sticker file. Should be either HTTP or HTTPS link.           | String       | N/A        | Accepted Content-Type header. Must not be used in combination with `id`.  | Yes      |
| id          | ID of a sticker. Can be found using the stickerpack management endpoints.      | String       | N/A        | Accepted Content-Type header. Must not be used in combination with `url`. | Yes      |
| provider    | Optional name of a media provider to be used when trying to download the file. | String       | None       | Can only be used in combination with `url`, not with `id`.                | No       |

> 📘 Note
>
> Only one of the parameters `url` and `id` may be used in a single request.

```json
{
  "to": [
    "46732001122"
  ],
  "message": {
    "type": "sticker",
    "url": "https://example.com/sticker.webp"
  }
}
```
 
> 📘 Note
>
> Stickers can be organized in stickerpacks. See [Stickerpack Management](doc:whatsapp-stickerpack-management) for more on this.

### Interactive message

Interactive messages give the recipient options to choose from. The choices are returned in callbacks.

JSON object parameters:

| Name        | Description                                                             | JSON Type    | Default    | Constraints           | Required |
| ----------- | ----------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| type        | Constant value `interactive`                                            | String       | N/A        | N/A                   | Yes      |
| message     | The specific interactive message, button or list                        | Object       | false      | Described below       | Yes      |

#### Button message
This message type provides the recipient with up to three buttons which can be pressed. A button press causes a response to be sent back in a callback. See [Interactive button reply message](doc:whatsapp-callback#interactive-button-reply-message)

| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| type        | Constant value `button`                                              | String       | N/A        | N/A                   | Yes      |
| header      | See decription below                                                 | Object       | false      | Described below       | No       |
| body        | See decription below                                                 | Object       | false      | Described below       | Yes      |
| footer      | See decription below                                                 | Object       | false      | Described below       | No       |
| action      | See decription below                                                 | Object       | false      | Described below       | Yes      |

The button header has the following field:

| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| type        | One of `text`, `document`, `video`, or `image`                       | String       | N/A        | N/A                   | Yes      |

The rest of the header fields are different for different header types.

- Text header

| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| text        | The header text                                                      | String       | N/A        | Max 60 characters     | Yes      |


- Document header

| Name        | Description                                                            | JSON Type    | Default    | Constraints           | Required |
| ----------- | ---------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| url         | The URL where the document is located                                  | String       | N/A        | N/A                   | Yes      |
| filename    | The document's filename                                                | String       | N/A        | N/A                   | No       |
| provider    | Name of a provider (see [media provider](doc:whatsapp-media-provider)) | Object       | N/A        | N/A                   | No       |

- Video header

| Name        | Description                                                            | JSON Type    | Default    | Constraints           | Required |
| ----------- | ---------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| url         | The URL where the video is located                                     | String       | N/A        | N/A                   | Yes      |
| provider    | Name of a provider (see [media provider](doc:whatsapp-media-provider)) | Object       | N/A        | N/A                   | No       |

- Image header

| Name        | Description                                                            | JSON Type    | Default    | Constraints           | Required |
| ----------- | ---------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| url         | The URL where the image is located                                     | String       | N/A        | N/A                   | Yes      |
| provider    | Name of a provider (see [media provider](doc:whatsapp-media-provider)) | Object       | N/A        | N/A                   | No       |

The button body has the following field:

| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| text        | The body text                                                        | String       | N/A        | Max 1024 characters   | Yes      |

The button footer has the following field:

| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| text        | The footer text                                                      | String       | N/A        | Max 60 characters     | Yes      |

The button action has the following field:

| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| buttons     | A list of objects describing the buttons to include                  | Object array | N/A        | [1, 3] button objects | Yes      |

Each button object has the following fields:

| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| type        | Constant value `reply`                                               | Object array | N/A        | N/A                   | Yes      |
| id          | The button's id                                                      | String       | N/A        | Max 230 characters    | Yes      |
| title       | The button's title                                                   | String       | N/A        | Max 20 characters     | Yes      |


```json
{
  "to" : [
    "46732001122"
  ],
  "message" : {
    "type" : "interactive",
    "message" : {
      "type" : "button",
      "header" : {
        "type" : "video",
        "url" : "https://example.com/video.mp4"
      },
      "body" : {
        "text" : "Body text"
      },
      "footer" : {
        "text" : "Footer text"
      },
      "action" : {
        "buttons" : [ {
          "type" : "reply",
          "title" : "Title 1",
          "id" : "Id 1"
        }, {
          "type" : "reply",
          "title" : "Title 2",
          "id" : "Id 2"
        }, {
          "type" : "reply",
          "title" : "Title 3",
          "id" : "Id 3"
        } ]
      }
    }
  }
}
```

#### List message
This message type provides the recipient with a list of choices. Pressing one of the choices causes a response to be sent back in a callback. See [Interactive list reply message](doc:whatsapp-callback#interactive-list-reply-message)


| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| type        | Constant value `list`                                                | String       | N/A        | N/A                   | Yes      |
| header      | See decription below                                                 | Object       | false      | Described below       | No       |
| body        | See decription below                                                 | Object       | false      | Described below       | Yes      |
| footer      | See decription below                                                 | Object       | false      | Described below       | No       |
| action      | See decription below                                                 | Object       | false      | Described below       | Yes      |

The list header has the following field:

| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| type        | Constant value `text`                                                | String       | N/A        | N/A                   | Yes      |
| text        | The header text                                                      | String       | N/A        | Max 60 characters     | Yes      |

The list body has the following field:

| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| text        | The body text                                                        | String       | N/A        | Max 1024 characters   | Yes      |

The list footer has the following field:

| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| text        | The footer text                                                      | String       | N/A        | Max 60 characters     | Yes      |

The list action has the following fields:

| Name        | Description                                                          | JSON Type    | Default    | Constraints             | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | ----------------------- | :------: |
| button      | The list is shown after pressing a button which displays this text   | String       | N/A        | Max 20 characters       | Yes      |
| sections    | A array of sections of rows                                          | Object array | N/A        | [1, 10] section objects | Yes      |

Each list section has the following field:

| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| title       | The title of the section                                             | String       | N/A        | Max 24 characters     | Yes      |
| rows        | Each row is an option that the recipient can choose                  | Object array | N/A        | [1, 10] row objects, but the total number of rows in all sections can be at most 10 | Yes      |

> 📘 Note
>
> The total number of rows in all secions in a list message can be at most 10.
>

Each row object has the following field:

| Name        | Description                                                          | JSON Type    | Default    | Constraints           | Required |
| ----------- | -------------------------------------------------------------------- | ------------ | ---------- | --------------------- | :------: |
| id          | The ID of the row                                                    | String       | N/A        | Max 174 characters. Each ID must be unique in the message. | Yes      |
| title       | The title of the row                                                 | String       | N/A        | Max 20 characters     | Yes      |
| description | The description of the row                                           | String       | N/A        | Max 72 characters     | No       |

```json
{
  "to": [
    "46732001122"
  ],
  "message": {
    "type": "interactive",
    "message": {
      "type": "list",
      "header": {
        "type": "text",
        "text": "Header text"
      },
      "body": {
        "text": "Body text"
      },
      "footer": {
        "text": "Footer text"
      },
      "action": {
        "button": "Press me",
        "sections": [
          {
            "title": "Section 1",
            "rows": [
              {
                "title": "A row",
                "id": "ID 1",
                "description": "Description"
              }
            ]
          },
          {
            "title": "Section 2",
            "rows": [
              {
                "title": "Another row",
                "id": "ID 2",
                "description": "Description"
              }
            ]
          }
        ]
      }
    }
  }
}
```
