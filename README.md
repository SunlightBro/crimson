<h1 align="center">
  <a href="https://github.com/simc/crimson">
    <img src="https://raw.githubusercontent.com/simc/crimson/main/.github/crimson.svg" width="350">
  </a>
</h1>

<p align="center">
  <a href="https://pub.dev/packages/crimson">
    <img src="https://img.shields.io/pub/v/crimson?label=pub.dev&labelColor=333940&logo=dart">
  </a>
  <a href="https://github.com/simc/crimson/actions/workflows/test.yaml">
    <img src="https://img.shields.io/github/actions/workflow/status/simc/crimson/test.yaml?branch=main&label=tests&labelColor=333940&logo=github">
  </a>
  <a href="https://app.codecov.io/gh/simc/crimson">
    <img src="https://img.shields.io/codecov/c/github/simc/crimson?logo=codecov&logoColor=fff&labelColor=333940">
  </a>
  <a href="https://twitter.com/simonleier">
    <img src="https://img.shields.io/twitter/follow/simonleier?style=flat&label=Follow&color=1DA1F2&labelColor=333940&logo=twitter&logoColor=fff">
  </a>
</p>

<p align="center">Fast, efficient and easy-to-use JSON parser and serializer for Dart.<br>Crimson does not verify your JSON!</p>

## Features

- 🏎️ **Fast**: Like really fast. Crimson parses JSON in a single pass.
- 🌻 **Easy to use**: Crimson is designed to be easy to learn and use.
- 💃 **Flexible**: Crimson can partially parse and serialize JSON.
- 🥶 **Freezed support**: Crimson supports [freezed](https://pub.dev/packages/freezed) classes.
- 🪶 **Lightweight**: Crimson is lightweight and has no third-party dependencies.

## Usage

After adding Crimson to your `pubspec.yaml`, you can start annotating your classes with `@json` and optionally `@JsonField()`:

```dart
import 'package:crimson/crimson.dart';

part 'tweet.g.dart';

@json
class Tweet {
  DateTime? createdAt;

  String? tweet;

  int? favoriteCount;
}
```

Now you just need to run `pub run build_runner build` to generate the necessary code.

```dart
import 'package:crimson/crimson.dart';

void main() {
  final jsonBytes = downloadTweets();

  final crimson = Crimson(jsonBytes);
  final tweets = crimson.readTweetList();

  final writer = CrimsonWriter();
  writer.writeTweetList(tweets);
  final bytes = writer.toBytes();
}
```

That's it! You can now parse and serialize JSON with ease.

## Ignoring JSON fields

Annotate properties with `@jsonIgnore` to ignore them:

```dart
@json
class Tweet {
  DateTime? created_at;

  String? tweet;

  @jsonIgnore
  int? favoriteCount;
}
```

## Renaming JSON fields

Use the `@JsonName()` annotation to rename individual fields:

```dart
@json
class Tweet {
  @JsonName('created_at')
  DateTime? createdAt;

  @JsonName('text', aliases: {'alias1', 'alias2'})
  String? tweet;
}
```

The same works for enum values:

```dart
@json
enum TweetType {
  tweet,

  @JsonName('re-tweet')
  retweet,
}
```

If you want to rename all fields or enum values, you can use `@jsonKebabCase` and `@jsonSnakeCase`:

```dart
@jsonKebabCase
class Tweet {
  DateTime? createdAt; // created-at

  String? tweet; // tweet

  int? favoriteCount; // favorite-count
}

@jsonSnakeCase
enum PlaceType {
  country, // country
  largeCity, // large_city
  smallCity, // small_city
}
```

## fromJson and toJson methods

Crimson supports generating a `fromJson()` constructor and a `toJson()` extension method for your classes:

Just add the following line to your class:

```dart
@json
class Tweet {
  // ...

  factory Tweet.fromJson(Uint8List json) => _$TweetFromJson(json);
}
```

Alternatively you can use `fromBytes()` and `toBytes()`:

```dart
@json
class Tweet {
  // ...

  factory Tweet.fromBytes(Uint8List bytes) => _$TweetFromBytes(bytes);
}
```

## JSON Pointers

Crimson supports [RFC 6901 JSON pointers](https://www.rfc-editor.org/rfc/rfc6901) to access nested JSON fields:

```json
{
  "user": {
    "name": "Simon Choi"
  },
  "tweets": [
    {
      "text": "Hello World!"
    },
    {
      "text": "Hello Crimson!"
    }
  ]
}
```

We can access the `name` field of the `user` object and the `text` field of the second tweet by providing a JSON pointer:

```dart
@json
class Tweet {
  @JsonName('/user/name')
  String? userName;

  @JsonName('/tweets/1/text')
  String? secondTweet;
}
```

JSON pointers are evaluated at compile time and optimized code is generated so there is no runtime overhead.

Fields using JSON pointers are ignored for serialization.

**Important**: You can only use a pointer prefix once in a class. For example, you can't use `/user` and `/user/name` in the same class.

## Custom JSON converters

You can use custom JSON converters to convert between JSON and Dart types using the `@JsonConvert()` annotation:

```dart
@json
class Tweet {
  String? tweet;

  @JsonConvert(fromJson: jsonIntToBool, toJson: boolToJsonInt)
  bool? isFavorite;
}

bool jsonIntToBool(int json) => json >= 1;

int boolToJsonInt(bool value) => value ? 1 : 0;
```

## Freezed Support

Crimson supports classes annotated with `@freezed` from the [freezed](https://pub.dev/packages/freezed) package.

```dart
import 'package:crimson/crimson.dart';

part 'tweet.g.dart';
part 'tweet.freezed.dart';

@freezed
class Tweet with _$Tweet {
  @json
  const factory Tweet({
    DateTime? created_at,
    @JsonField(name: 'text') String? tweet,
    int? reply_count,
    int? retweet_count,
    int? favorite_count,
  }) = _Tweet;
}
```

## JSON Benchmarks

The following benchmarks give you an idea about the JSON parsing performance of Crimson. The benchmarks were run on a MacBook with M1 Pro and 32 GB of RAM using Twitter API data.

<table>
  <tr>
    <td align="center">
      <picture>
        <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/simc/crimson/main/.github/benchmarks/big_json_dark.png">
        <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/simc/crimson/main/.github/benchmarks/big_json_light.png">
        <img src="https://raw.githubusercontent.com/simc/crimson/main/.github/benchmarks/big_json_light.png">
      </picture>
      <b>Big JSON without whitespace</b>
    </td>
    <td align="center">
      <picture>
        <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/simc/crimson/main/.github/benchmarks/big_json_whitespace_dark.png">
        <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/simc/crimson/main/.github/benchmarks/big_json_whitespace_light.png">
        <img src="https://raw.githubusercontent.com/simc/crimson/main/.github/benchmarks/big_json_whitespace_light.png">
      </picture>
      <b>Big JSON with whitespace</b>
    </td>
  </tr>
  <tr>
    <td align="center">
      <picture>
        <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/simc/crimson/main/.github/benchmarks/medium_json_dark.png">
        <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/simc/crimson/main/.github/benchmarks/medium_json_light.png">
        <img src="https://raw.githubusercontent.com/simc/crimson/main/.github/benchmarks/medium_json_light.png">
      </picture>
      <b>Medium JSON without whitespace</b>
    </td>
    <td align="center">
      <picture>
        <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/simc/crimson/main/.github/benchmarks/small_json_dark.png">
        <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/simc/crimson/main/.github/benchmarks/small_json_light.png">
        <img src="https://raw.githubusercontent.com/simc/crimson/main/.github/benchmarks/small_json_light.png">
      </picture>
      <b>Small JSON without whitespace</b>
    </td>
  </tr>
</table>

### License

```
Copyright 2023 Simon Choi

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
