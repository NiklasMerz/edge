# Inconsistent `safe-area-inset-*` and `viewport-fit` Behavior on Android WebView vs. iOS

## Overview

When using `viewport-fit` on Android WebView, the handling of safe area insets (`env(safe-area-inset-*)`) differs from iOS Safari (or iOS WebView). On iOS, the safe-area insets (e.g., `safe-area-inset-top`) are set to `0` unless `viewport-fit=cover` is explicitly used. On Android WebView, these insets are consistently set to non-zero values even if `viewport-fit=contain` or `viewport-fit=auto` is used. This mismatch leads to layout issues where Android WebView does not automatically avoid notches or rounded display cutouts, contrary to expectations and the behavior on iOS WKWebView.

For context, this issue came up in the Apache Cordova community while preparing the Cordova-Android release, which is updating its target SDK to 35 (Android 15), where edge-to-edge is enforced by default. We tested the manifest setting `<item name="android:windowOptOutEdgeToEdgeEnforcement">true</item>`and this fixes it. We would like to properly support edge-to-edge in the future and therefore are looking for a solution for this.

### Steps to Reproduce

### Test page 

There is a test page on https://niklas.merz.dev that lets you toggle between `viewport-fit=contain`and `viewport-fit=auto`. It shows the value of `--safe-area-top`. Opening this page in a full screen WebView on Android or iOS shows the inconsistent behavior between both WebViews.

The code for the testing site is on [GitHub](https://github.com/niklasmerz/edge).

Videos:

[Android.webm](https://github.com/user-attachments/assets/c71c05dd-af8f-4695-b328-dad7d134ed00)

[WKWebView.mp4](https://github.com/user-attachments/assets/9dbe864c-cafd-4f03-8786-f3448eb51c69)

### Test form Scratch

1. Create a simple HTML test page that sets:
   ```css
   :root {
     --safe-area-top: env(safe-area-inset-top, 0px);
     --safe-area-right: env(safe-area-inset-right, 0px);
     --safe-area-bottom: env(safe-area-inset-bottom, 0px);
     --safe-area-left: env(safe-area-inset-left, 0px);
   }
   ```
2. Use `<meta name="viewport" content="viewport-fit=contain">` (or `auto`) in the `<head>`.
3. Display the values of these CSS variables in the DOM (e.g., via `getComputedStyle(document.documentElement).getPropertyValue("--safe-area-top")`).
4. Compare results on:
   - **iOS** Open test page in a full screen WKWebView
   - **Android** Open test page in full screen WebView

## Expected Behavior

- According to iOS behavior (and implied by [W3C Round Display](https://www.w3.org/TR/css-round-display-1/#viewport-fit-descriptor)), when `viewport-fit=contain` or `viewport-fit=auto`, the browser should automatically avoid notches and effectively set safe-area insets to 0 (so developers don’t need to manually adjust layouts).
- Only when `viewport-fit=cover` should the safe-area insets be non-zero, allowing developers to opt in to drawing content under device cutouts.

## Actual Behavior 

- On Android WebView, safe-area insets report non-zero values even with `viewport-fit=contain` or `viewport-fit=auto`. Layouts that expect iOS-like handling (i.e., no manual inset calculations for contain/auto) end up incorrectly positioned under the notch/cutout.

## Why This Is a Problem

- It breaks parity with iOS behavior and with developer expectations that `contain` or `auto` should automatically avoid notches.
    - For WebView bases apps like Cordova this brings complications for cross-platform developers.
- Developers have to implement extra logic or conditional code for Android WebView to ensure content is properly padded or notched areas are avoided.

## References

We found some issues that might be related:

- [Google Issue 40699457](https://issuetracker.google.com/issues/40699457) (for CSS env variables, though not precisely the same issue)  
- [Chromium Issue 394337118](https://issues.chromium.org/issues/394337118) (touches on `viewport-fit` but may not be specific to WebView)  
- [W3C: CSS Round Display Level 1](https://www.w3.org/TR/css-round-display-1/#viewport-fit-descriptor) (specification that describes `viewport-fit`)
- [W3C env() CSS Draft Spec](https://drafts.csswg.org/css-env-1/) (work in progress)

## Suggested Fix or Next Steps

- Align Android WebView behavior with iOS so that `viewport-fit=contain` and `viewport-fit=auto` ignore cutouts (report safe-area insets as 0) unless developers explicitly opt in to using `viewport-fit=cover`.
- If changing the default behavior is not possible, provide clearer documentation or an explicit mechanism (similar to iOS) that distinguishes between ignoring vs. including cutouts.
