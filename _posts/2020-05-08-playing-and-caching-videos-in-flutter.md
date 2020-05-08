---
title: "Playing (and caching) Online Videos In Flutter"
date: 2020-05-08
tags: [Flutter, Mobile]
---

For the past few weekends I have been working on a new feature for my apps that will allow users to watch video tutorials inside the apps. The videos will be stored securely in the cloud which means they will have to be streamed in the apps. This presented a design challenge I had to overcome. About 99% of my users are in South Africa where data costs are relatively high (very high) so I don't want my app to be responsible for their huge data bills. This means I need to download the video just once (or twice as you shall see) from the server and cache it on the device. I also need to save bandwidth costs from my cloud provider. In this post I am going to talk about the design decisions I made and the compromises I had to make in order to achieve my goal.

## Playing Videos

Playing videos in Flutter was not the biggest challenge I needed to overcome. I used one of the most popular libraries, [chewie](https://pub.dev/packages/chewie), which is a video player plugin that uses the [video_player](https://pub.dev/packages/video_player) package under the hood and wraps it in a Material or Cupertino UI. You can follow the link if you want to learn more about it. You will see how I used it later.

## Caching The Videos

This is where I spent most of my time trying to figure out what the best way forward was. I did not want my users to download a video everytime they view it because that was going to be very expensive for the both of us. After a quick google search I came across [flutter_cache_manager](https://pub.dev/packages/flutter_cache_manager) -- a cache manager that downloads and cache files in the the cache directory of the app. All I needed to do was to put everything together in a nice and maintenable way which is what I'm going to talk about in the example below.

## Example

This example is going to be a simple Flutter app that plays and caches a video stored on a server somewhere. I'm going to make it as basic as possible to show how I managed to put everything together but leaving out some of the stuff I think are out of scope for this post. I will also not show all the code in this post. If you want to see the full example you may check it out on [GitHub](https://github.com/vince-nyanga/flutter_video_player). Let's get started.

Create a new Flutter application and add the following packages to your `pubspec.yaml` file:

```yaml
# pubspec.yaml

# ...

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^0.1.2
  bloc: ^4.0.0
  flutter_bloc: ^4.0.0
  equatable: ^1.1.1
  video_player: ^0.10.8+1
  chewie: ^0.9.10
  flutter_cache_manager: ^1.2.2
  pedantic: ^1.8.0+1
# ...
```

First, let's create a custom video player widget that will play our video.

### Video Player Widget

Create `lib/widgets/video_player_widget.dart` and add the following code:

```csharp
import 'package:chewie/chewie.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:video_player/video_player.dart';

const ASPECT_RATIO = 3 / 2;

class VideoPlayerWidget extends StatefulWidget {
  final VideoPlayerController controller;
  final String videoTitle;

  const VideoPlayerWidget({
    Key key,
    @required this.controller,
    @required this.videoTitle,
  })  : assert(controller != null),
        assert(videoTitle != null),
        super(key: key);

  @override
  _VideoPlayerWidgetState createState() => _VideoPlayerWidgetState();
}

class _VideoPlayerWidgetState extends State<VideoPlayerWidget> {
  ChewieController _chewieController;

  @override
  void initState() {
    _chewieController = ChewieController(
      videoPlayerController: widget.controller,
      aspectRatio: ASPECT_RATIO,
      autoInitialize: true,
      autoPlay: true,
      deviceOrientationsAfterFullScreen: [DeviceOrientation.portraitUp],
      materialProgressColors: ChewieProgressColors(
        playedColor: Colors.purple,
        handleColor: Colors.purple,
        backgroundColor: Colors.grey,
        bufferedColor: Colors.purple[100],
      ),
      placeholder: Container(
        color: Colors.grey,
      ),
    );
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.stretch,
      children: <Widget>[
        Chewie(
          controller: _chewieController,
        ),
        Padding(
          padding: const EdgeInsets.all(16),
          child: Text(
            widget.videoTitle,
            style: Theme.of(context).textTheme.title
                .copyWith(color: Color.fromRGBO(14, 26, 92, 1)),
          ),
        ),
      ],
    );
  }

  @override
  void dispose() {
    widget.controller.dispose();
    _chewieController.dispose();
    super.dispose();
  }
}
```

This widget's sole purpose is to play a video using `chewie`, nothing more, nothing less. When I was searching for the best way to cache videos in Flutter I came across an example where the caching was done inside the widget that played the video. Even though it works I personally feel like that's giving the widget too much responsibility and violates the Single Responsibility Principle.

So how did I deal with the caching of the videos, you may ask. I created an abstruct `VideoControllerService` that returns a `VideoPlayerController` given a `Video` model. It is in the implementation of this abstract class that I then make use of the `flutter_cache_manager` library to check whether or not the video has already been cached and return an appropriate `VideoPlayerController`. Let's do that now.

### Video Controller Service

Create `lib/services/video_controller_service.dart` and add the following code:

```csharp

import 'package:flutter_cache_manager/flutter_cache_manager.dart';
import 'package:pedantic/pedantic.dart';
import 'package:video_player/video_player.dart';
import '../models/models.dart';

abstract class VideoControllerService {
  Future<VideoPlayerController> getControllerForVideo(Video video);
}

class CachedVideoControllerService extends VideoControllerService {
  final BaseCacheManager _cacheManager;

  CachedVideoControllerService(this._cacheManager) : assert(_cacheManager != null);

  @override
  Future<VideoPlayerController> getControllerForVideo(Video video) async {
    final fileInfo = await _cacheManager.getFileFromCache(video.url);

    if (fileInfo == null || fileInfo.file == null) {
      print('[VideoControllerService]: No video in cache');

      print('[VideoControllerService]: Saving video to cache');
      unawaited(_cacheManager.downloadFile(video.url));

      return VideoPlayerController.network(video.url);
    } else {
      print('[VideoControllerService]: Loading video from cache');
      return VideoPlayerController.file(fileInfo.file);
    }
  }
}

```

The `CachedVideoControllerService` implements our abstract `VideoControllerService`. Inside the `getControllerForVideo` method I first try to get the video from the cache. If the video is not in the cache I save it to the cache and stream it simultaneously. As you may have noticed, I am downloading the video twice here -- downloading it to save to the cache as well as streaming it using the `VideoPlayerController.network(url)`. This is a compromise I was willing to make because the alternative would lead to a bad user experience.

The alternative was to use the cache manager's `getSingleFile(url)` method which would try to get the file from the cache or downloads it if it's not in the cache. This means if the video is not in the cache the user would have to wait for it to be downloaded and cached first which wouldn't be a good experience in my opinion. In addition, if the cached file is too old it will have to be downloaded again. This may be the best way to do it but in this project I don't really need to refresh the cache since my objective is to save users data costs.

Even though the video is downloaded twice it's a better option that not caching at all. I'm still looking for a better solution but I need to ship to production and I don't have time at the moment. I follow the _Make it Work - Make it Right - Make it Fast_ approach. When I find a better solution I will come back here and share it.

## BLoC

I already use the `BLoC` pattern inside the app so everything goes through the `BLoC`. I am not going to talk about it or add the code here for brevity. I encourage you to check out the complete example on [GitHub](https://github.com/vince-nyanga/flutter_video_player).

## The UI

Create `lib/pages/video_page.dart` and add the following code:

```csharp
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:flutter_video_player/services/services.dart';

import '../blocs/blocs.dart';
import '../models/models.dart';
import '../widgets/widgets.dart';

class VideoPage extends StatelessWidget {
  final Video video;

  const VideoPage({Key key, @required this.video}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: _buildVideoPlayer(),
      ),
    );
  }

  Widget _buildVideoPlayer() {
    return BlocProvider<VideoPlayerBloc>(
      create: (context) =>
          VideoPlayerBloc(RepositoryProvider.of<VideoControllerService>(context))..add(VideoSelectedEvent(video)),
      child: BlocBuilder<VideoPlayerBloc, VideoPlayerState>(
        builder: (context, state){
          return Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: <Widget>[
              _getPlayer(context, state)
            ],
          );
        },
      ),
    );
  }

  Widget _getPlayer(BuildContext context, VideoPlayerState state) {
    if (state is VideoPlayerStateLoaded) {
      return VideoPlayerWidget(
        key: Key(state.video.url),
        videoTitle: state.video.title,
        controller: state.controller,
      );
    }

    final screenWidth = MediaQuery.of(context).size.width;
    final containerHeight = screenWidth / ASPECT_RATIO;

    if (state is VideoPlayerStateError){
      return Container(
        height: containerHeight,
        color: Colors.grey,
        child: Center(
          child: Text(state.message),
        ),
      );
    }

    return Container(
      height: containerHeight,
      color: Colors.grey,
      child: Center(
        child: Text('Initialising video...'),
      ),
    );
  }
}
```

Let's go to `main.dart` and hook everything up:

```csharp
// main.dart

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:flutter_cache_manager/flutter_cache_manager.dart';

import 'models/models.dart';
import 'pages/pages.dart';
import 'services/services.dart';

void main() {
  runApp(RepositoryProvider<VideoControllerService>(
    create: (context) => CachedVideoControllerService(DefaultCacheManager()),
    child: MyApp(),
  ));
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Video Player',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        primarySwatch: Colors.purple,
      ),
      home: VideoPage(
        video: Video(
          title: 'Fluttering Butterfly',
          url: 'https://flutter.github.io/assets-for-api-docs/assets/videos/butterfly.mp4',
        ),
      ),
    );
  }
}
```

We are done. Checkout the complete project from [GitHub](https://github.com/vince-nyanga/flutter_video_player) and run it.

## Conclusion

In this post I spoke about how I managed to play and cache online videos in my apps. I showed a scaled down example of how I did it and hopefully you can take it and build on it in your own app. There are things I'm not quite happy with but for now I think the solution works well even though there is plenty room for improvement. When I improve the solution I will come back and update this post. Thanks so much and stay safe if you're reading this during the 2020 **Covid-19** pandemic.
