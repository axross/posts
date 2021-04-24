---
title: Applying Dark Mode on the Web
description: Dark mode is still not common so far, on the other hand, it's getting more common since macOS supports it in 2018. Nowadays, most major OSs has this feature. I describe how to let your websites support dark mode and what you need to do for it.
cover_image_url: https://user-images.githubusercontent.com/4289883/115945142-1d9bb180-a46f-11eb-9a22-8c5bb0d0351e.png
tags:
  - web frontend
first_published_at: '2019-08-24T00:00Z'
last_published_at: '2019-08-24T00:00Z'
---
:::callout{variant="warning"}
Since this website has updated a few times since I wrote this post, the screenshots below don't match with the current appearance. However, technical instruction on this post is still up-to-date.
:::

I just implemented Dark Mode on this website.

## What's Dark Mode?

**Dark Mode** is a new OS setting that changes UI to the black-color background. This page is supposed to show in the dark-color background for the users who turned on the dark mode at OS settings.

![Dark Mode](https://user-images.githubusercontent.com/4289883/115945189-5b98d580-a46f-11eb-940e-43999f2ea52d.png)

### Dark Mode is Accessibility

The Dark Mode is not only "color setting". The Dark Mode is an accessibility feature that shows in the color what the user requested.

Even if the user OS shows UI in dark color and only your website shows UI in white color, it's not user-friendly. Besides, it could make the user's eyes under somewhat stress.

![Dark mode setting in macOS Catalina](https://user-images.githubusercontent.com/4289883/115945235-969b0900-a46f-11eb-925d-ac467ae848c4.png)

Latest iOS and macOS has automatic dark mode setting. It enables dark mode during the night. Same as Night Light or Night Shift, which is the feature to change display color temperature from sunset to sunrise, it's going to adjust to our life.

### Dark Mode helps the battery lasts longer

A lot of new mobile devices use [OLED displays](https://en.wikipedia.org/wiki/OLED). OLED displays are consist of self-light cells instead of using the backlight. So as color is darker, the battery usage is less. According to Google's research, dark mode reduces battery usages by more than 50%.

::embed{href="https://www.gsmarena.com/google_finds_night_mode_really_helps_battery_endurance-news-34134.php"}

## Implementing Dark Mode

You can detect if the Dark Mode is enabled (requested) by a Media Query [`prefers-color-scheme`](https://developer.mozilla.org/ja/docs/Web/CSS/@media/prefers-color-scheme). With the following code, the background color will be `#000` when the Dark Mode is on, otherwise `#fff`.

```css
html {
  background-color: #fff;
}

@media (prefers-color-scheme: dark) {
  html {
    background-color: #000;
  }
}
```

I recommend you to define the color scheme first of all. In the real world, you will need to prepare colors for each text, icons, background colors, border colors, and drop shadows for buttons and text inputs and something like that.

## Detect in JavaScript

You can use [`window.matchMedia()`](https://developer.mozilla.org/ja/docs/Web/API/Window/matchMedia) to detect if the Dark Mode is on.

```js
// true when dark mode is on, false otherwise
window.matchMedia("(prefers-color-scheme: dark)").matches
```

To observe switching Dark Mode, use `.addEventListener()` method of [`MediaQueryList`](https://developer.mozilla.org/ja/docs/Web/API/MediaQueryList), which is return value of `window.matchMedia()`. The following example shows how to observe and give it to the descendant components in React.

```jsx
const DarkModeContext = React.useContext();

function DarkModeProvider({ children }) {
  const [isDarkMode, setIsDarkMode] = React.useState(false);

  React.useEffect(() => {
    // get MediaQueryList for "prefers-color-scheme: dark"
    const mediaQueryList = window.matchMedia("(prefers-color-scheme: dark)");

    // event handler that is used below
    // e.matches is true when the dark mode is on, false otherwise
    const listener = e => setDarkMode(e.matches);

    // it triggers the change event whenever dark mode switches
    mediaQueryList.addEventListener("change", listener);

    // unobserve from change event when this component gets unmounted
    () => mediaQueryList.removeEventListener("change", listener);
  });

  // pass if the dark mode is on to the descendants
  return (
    <DarkModeContext.Provider value={isDarkMode}>
      {children}
    </DarkModeContext.Provider>
  );
}

function SomeComponent() {
  // get boolean value via the context
  const isDarkMode = React.useContext(DarkModeContext);

  return isDarkMode ? <AnotherDarkComponent /> : <AnotherLightComponent />;
}
```

:::callout{variant="warning"}
If your website supports server-side rendering, I highly recommend to apply dark theme by pure CSS media query (`@media`).

Because dark mode is on end-user's device settings, which means there's no way to detect which mode the users preferred on the server.
:::

## Compatibility

All modern browsers except for Edge supports Dark Mode. Of course Internet Explorer doesn't support it.

![prefers-color-scheme Compatibility](https://user-images.githubusercontent.com/4289883/115945328-12955100-a470-11eb-8231-757522d533c1.png)

## Related Links

::embed{href="https://web.dev/prefers-color-scheme"}
