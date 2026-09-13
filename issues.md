# Issues with `vaft` / `video-swap-new`

## Neither script works

If you're using the uBlock Origin version of the script you need to make sure that it's set up correctly based on the instructions in the README. Check the script is active by opening your browsers developer console, refreshing a stream, and searching for `hookWorkerFetch (vaft)` / `hookWorkerFetch (video-swap-new)`. If you don't see this, then the script isn't being injected and you need to find the reason why.

If you're using Chrome with a Manifest V3 based userscript manager (e.g. Tampermonkey) you need to manually enable user script permissions in the extension settings:

- Go to `chrome://extensions`
- Click the `Details` button on the extension
- Where it says `Allow user scripts` make sure it's enabled


## Streams sometimes appear offline when ads occur

This needs to be fixed but currently the exact cause is unknown. https://github.com/pixeltris/TwitchAdSolutions/issues/477

## The player shows a loading wheel for a long time

If it says `Blocking ads (stripping)` in the top left of the stream then it's actively removing the ad segments but doesn't have a backup stream to show you. If it doesn't say `Blocking ads (stripping)` provide additional information in https://github.com/pixeltris/TwitchAdSolutions/issues/474

This is most common on midrolls. A midroll break follows your signed in session, so asking for a new access token as `embed` / `popout` / `autoplay` tends to return the same ads on every one of them, leaving nothing clean to swap to. `vaft` handles this in two ways:

- The `embed-ALT` entry in `BackupPlayerTypes` asks for the access token as a signed out viewer on a separate device id, which usually sits outside the break. It's tried last, so it only costs a request when the normal player types have all come back with ads. Remove it from `BackupPlayerTypes` if you'd rather it never requested a token without your account.
- `MaxAdStrippingTime` (30000, in milliseconds) is how long the script will sit on stripped segments before reloading the player to try for a clean stream again. Lower it to retry sooner, raise it if the reloads are more disruptive than the wait, or set it to `0` to disable the retry and wait the break out.

## The script don't work on mobile (m.twitch.tv)

There are no plans of implementing the scripts on m.twitch.tv but there are other solutions which blocking ads on Twitch for mobile. See https://github.com/JohnFawkes/TwitchAdSolutions/blob/master/full-list.md

## Long black screen during ads

The scripts can reload the player when entering/leaving ads. On some systems this may cause a long period of time where the player is black as the player is loading back in. There currently isn't any fix for this. Try a different script / solution.

## `vaft`

### Freezing / buffering / repeating segments / audio desyncs

`vaft` has a long standing problem of playback problems (freezing / buffering / repeating segments / audio desyncs).

This happens because the script forces the player to consume multiple different m3u8 files during ads and it screws with the player. Simply pressing pause/play often fixes this.

The script is configured to do this automatically for you:

https://github.com/JohnFawkes/TwitchAdSolutions/blob/89093822979f851531e5dda4e29e2c05f7a7be63/vaft/vaft.user.js#L48-L53

- If it triggers but it still freezes try setting `PlayerBufferingDoPlayerReload` to `true`. Player reloads generally have less problems.
- If you're having issues with it triggering when the player is genuinely buffering then adjust `PlayerBufferingDelay` / `PlayerBufferingSameStateCount`.
- If it triggers too frequently then increase the value of `PlayerBufferingMinRepeatDelay`. Decrease this if it triggers, freezes, then has to wait a long time to re-trigger.
- If you don't want to use this and would like to fix the buffering manually yourself you can set `PlayerBufferingFix` to `false`.
- Setting `AlwaysReloadPlayerOnAd` to `true` may reduce freezing issues when entering into ads.

### Ads aren't being blocked / the player behaves oddly during ads

Two options control how ads are detected and what's handed to the player. Both default to `true`:

- `DetectAdsBySegmentTitle` - also treats a playlist as having ads when it contains segments which aren't titled `live`, rather than relying only on the ad daterange tag. Set this to `false` if a stream is wrongly detected as being in an ad break.
- `RemoveAdTags` - removes the ad daterange tag from the playlist handed to the player, so the player doesn't run its own ad UI / ad timers over segments the script has already stripped. Set this to `false` if the player misbehaves while `Blocking ads (stripping)` is showing.

## `video-swap-new`

### Freezing / buffering during ads

There currently isn't a fix for freezing / buffering issues for `video-swap-new`. Try a different script / solution.
