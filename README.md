# cordova-plugin-playlist
A Cordova plugin for Android and iOS with native support for audio playlists, background support, and lock screen controls

## 0. Index

1. [Background](#1-background)
2. [Notes](#2-notes)
2. [Installation](#2-installation)
3. [Usage](#3-usage)
4. [Todo](#4-todo)
5. [Credits](#5-credits)
6. [License](#6-license)

## 1. Background

I have worked with [cordova-plugin-media](https://github.com/apache/cordova-plugin-media) for quite a long time and have pushed it pretty far. In the end it's not designed for certain use cases and I ultimately decided to apply what I had learned, and issues I have seen others have, to create a plugin that better addresses those use cases.

Both Android and iOS have special support for playlist-based playback, and the native implementation provides a superior user experience over attempting to implement a playlist out of an interface designed for single-item playback. Also, it is not possible to implement continuous playback using `cordova-plugin-media` on iOS, since the space between songs in the background will stop playback. This plugin addresses that, in addition to including support for command center and lock screen controls.

* Playlist support is implemented in native code
* Native playlists mean better support for continual playback in the background
* Background playback is optional - opt-in via runtime config and config.xml flags
* Includes support for lock-screen controls
* Works just as well for a single item as it does for a playlist
* Fully supports streaming URLs, with control of seek-pause behavior (does play resume from where you paused, or does it resume from current live position?)
* Compatible with [Cordova Plugman](https://github.com/apache/cordova-plugman).
* For Android and iOS

## 2. Notes

### On *Android*, utilizes a wrapper over ExoPlayer called [ExoMedia](https://github.com/brianwernick/ExoMedia). ExoPlayer is a powerful, high-quality player for Android provided by Google
### On iOS, utilizes a customized AVQueuePlayer in order to provide feedback about track changes, buffering, etc.; given that AVQueuePlayer can keep the audio session running between songs.

* This plugin intentionally does not display track cover art on the lock screen controls on iOS. Usage of the media image object on iOS is known to cause memory leaks. See the [Todo](#4-todo) section. The Swift version of that object does not (seem to) contain this memory leak, and rewriting this plugin to use Swift 4 is on the [Todo](#4-todo) list. This is fully supported on Android, however.

* This plugin is not designed to play mixable, rapid-fire, low-latency audio, as you would use in a game. A more appropriate cordova plugin for that use case is [cordova-plugin-nativeaudio](https://github.com/floatinghotpot/cordova-plugin-nativeaudio)

* Cannot mix audio; again the NativeAudio plugin is probably more appropriate. This is due to supporting the lock screen and command center controls: only an app in command of audio can do this, otherwise the controls have no meaning. I would like to add an option to do this, it should be fairly straightforward; at the cost of not supporting the OS-level controls for that invokation.

* If you are running this on iOS 9.3, this plugin requires a promise polyfill for the JS layer.

## 2. Installation

As with most cordova plugins...

```
cordova plugin add https://github.com/442623641/cordova-plugin-playlist.git
```

Android normally will give you ~2-3 minutes of background playback before killing your audio. Adding the WAKE_LOCK permission allows the plugin to utilize additional permissions to continue playing.

iOS will immediately stop playback when the app goes into the background if you do not include the `audio` `UIBackgroundMode`. iOS has an additional requirement that audio playback must never stop; when it does, the audio session will be terminated and playback cannot continue without user interaction.

## 3. Usage

Be sure to check out the examples folder, where you can find an Angular7/Ionic4 implementation of the Cordova plugin.
Just drop into your project and go.
```
  this.cdvAudioPlayer.setOptions({ verbose: true, resetStreamOnPause: true })
    .then(() => {
      this.cdvAudioPlayer.setPlaylistItems([
        { trackId: '12345', assetUrl: testUrls[0], albumArt: testImgs[0], artist: 'Awesome', album: 'Test Files', title: 'Test 1' },
        { trackId: '678900', assetUrl: testUrls[1], albumArt: testImgs[1], artist: 'Awesome', album: 'Test Files', title: 'Test 2' },
        { trackId: 'a1b2c3d4', assetUrl: testUrls[2], albumArt: testImgs[2], artist: 'Awesome', album: 'Test Files', title: 'Test 3' },
        { trackId: 'a1bSTREAM', assetUrl: testUrls[3], albumArt: testImgs[3], artist: 'Awesome', album: 'Streams', title: 'The Stream', isStream: true },
      ])
      .then(() => {
        this.cdvAudioPlayer.play();
      }).catch((err) => console.log('YourService, cdvAudioPlayer setPlaylistItems error: ', err));
    }).catch((err) => console.log('YourService, cdvAudioPlayer init error: ', err));

  this.cdvAudioPlayer.setOptions({ verbose: true, resetStreamOnPause: true });
  this.cdvAudioPlayer.setVolume(0.5);

  this.cdvAudioPlayer.onStatus.subscribe((status) => {
    console.log('YourService: Got RmxAudioPlayer onStatus: ', status);
  });

  /**
  *addAllItems
  *replacing item when the trackId exist,the positions do not change
  *
  **/
  this.cdvAudioPlayer.addAllItems([
        { trackId: '12345', assetUrl: testUrls[0], albumArt: testImgs[0], artist: 'Awesome', album: 'Test Files', title: 'Test 1' },
        { trackId: '678900', assetUrl: testUrls[1], albumArt: testImgs[1], artist: 'Awesome', album: 'Test Files', title: 'Test 2' },
        { trackId: 'a1b2c3d4', assetUrl: testUrls[2], albumArt: testImgs[2], artist: 'Awesome', album: 'Test Files', title: 'Test 3' },
        { trackId: 'a1bSTREAM', assetUrl: testUrls[3], albumArt: testImgs[3], artist: 'Awesome', album: 'Streams', title: 'The Stream', isStream: true },
      ])
```
## 4. Credits

There are several plugins that are similar to this one, but all are focused on aspects of the media management experience. This plugin takes inspiration from:
* [cordova-plugin-media](https://github.com/apache/cordova-plugin-media)
* [ExoMedia](https://github.com/brianwernick/ExoMedia)
* [PlaylistCore](https://github.com/brianwernick/PlaylistCore) (provides player controls on top of ExoMedia)
* [Bi-Directional AVQueuePlayer proof of concept](https://github.com/jrtaal/AVBidirectionalQueuePlayer)
* [cordova-music-controls-plugin](https://github.com/homerours/cordova-music-controls-plugin)