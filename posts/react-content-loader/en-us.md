---
title: Implementing Loading Placeholder with React Content Loader
description: I spent a year-end vacation to implement loading placeholders on this website. I’ll introduce it.
cover_image_url: https://user-images.githubusercontent.com/4289883/115946023-62c1e280-a473-11eb-825c-e3a3624f9ad0.png
tags:
  - react
first_published_at: '2020-01-05T15:00Z'
last_published_at: '2020-01-05T15:00Z'
---
Happy New Year! I spent a year-end vacation to implement loading placeholders on this website. I’ll introduce it.

**Loading placeholder** is an animated UI pattern that looks like the actual contents and is supposed to show during loading.

Probably you know **Spinner**, which is traditional circle-shape loading animation. Since the loading placeholder is much similar to the actual contents, it can lead the users to look at the position where the contents show. Besides, it seamlessly animates from loading animation to the actual contents because its shape is the same as the actual ones.

## React Content Loader
I’ll introduce 📦 `react-content-loader`, which is a JavaScript library that lets you implement loading placeholders of SVG by clipping path and animated gradient.

[](https://github.com/danilowoz/react-content-loader)

:::callout{variant="info"}
Also there's **[egoist/vue-content-loader](https://github.com/egoist/vue-content-loader)** for Vue and **[ngneat/content-loader](https://github.com/ngneat/content-loader)** for Angular. Usage is the same.
:::

There’s `<ContentLoader>` is exported. You will put `<rect>`s and `<circle>`s inside `<ContentLoader>`. You need to calculate the sizes and positions from the actual contents and apply them to `<rect>`. For example, this website's blogging post page during loading is like the following. You can make your device offline and change the language to try it.

![Loading Placeholder Example](https://user-images.githubusercontent.com/4289883/115946079-b2081300-a473-11eb-94ec-54c1212b98cd.png)

Generally, texts have a letter size (`font-size`) and line gap (`line-height`).    It's an excellent way to make being conscious of them.

Having said that, `<rect>` itself doesn't have `line-height`. Let's calculate `x` and `y` in considering of `font-size` and `line-height`. One way to think is like the following image:

![Layout Calculation Guide](https://user-images.githubusercontent.com/4289883/115946091-c4824c80-a473-11eb-83a6-3bddf8129bdd.png)

Expression in JSX is the following:

```css
/* blog post body css (just for an example, a little bit different from the actual) */
p {
  font-size: 16px;
  line-height: 1.75;
}
```

```jsx
// 1.75 line-height is 28px as converted
// there's (28 - 16) / 2 = 6px margin at the top and bottom
<ContentLoader>
  {/* y = 6px because there's margin at the top of text */}
  <rect x="0" y="6px" width="100%" height="16px" />

  {/* y is previous line's y + 16px + 12px for line 2 and below */}
  <rect x="0" y="34px" width="100%" height="16px" />
  <rect x="0" y="62px" width="40%" height="16px" />
</ContentLoader>
```

## Implemeting Dark Mode

It will decently look if you apply opacity to the fill colors. But when the background color is not pure black, you might feel something wrong. In fact, this website has a non-pure black background (`#11181f`), which is slightly blue. In this kind of case, it's a great idea applying different colors on `<stop>` in `<linearGradient>` element, which is SVG's fill-color definition.

```css
/* prefer to use CSS class instead of element-type selector in production */
svg > defs > linearGradient > stop:nth-of-type(2n) {
  stop-color: #e0e4e9;
}

svg > defs > linearGradient > stop:nth-of-type(2n + 1) {
  stop-color: #eff2f4;
}

@media (prefers-color-scheme: dark) {
  svg > defs > linearGradient > stop:nth-of-type(2n) {
    stop-color: #1e2730;
  }

  svg > defs > linearGradient > stop:nth-of-type(2n + 1) {
    stop-color: #2d3641;
  }
}
```

This website uses Styled Components, so each reference to the elements is on `&` symbol. [Refer here.](https://github.com/axross/kohei.dev/blob/4b3e3308a451f6445b88571895037b5624ce220b/common/components/ContentLoader.tsx#L34-L54)

## Making Responsive

SVG is something like a vector image that can be embedded in HTML. Retaining aspect ratio changes unnaturally heights. Be careful things the following:

- Set `width` and `height` of the elements that define fill colors such as `<rect>` or `<circle>` in percentage
- Keep `viewBox` and  `preserveAspectRatio` in `<svg>` default
- Calculate `height` and make it absolute value ([Sample code](https://github.com/axross/kohei.dev/blob/4b3e3308a451f6445b88571895037b5624ce220b/common/pages/BlogPostPage/ArticleLoader.tsx#L40-L47)) 。

## Known Issue with `<base>`

React Content Loader uses `url()` to refer the definition of SVG clipping path and gradient. Therefore, it doesn't work well and shows black-filled boxes if you use `<base>` element because SVG cannot reach to those definitions by URL.

This behavior only happens in Safari. So you might feel this is Safari's bug. However, this is expected behavior in [SVG Working Group's opinion.](https://www.w3.org/2015/08/25-svg-minutes.html#item08) You can avoid this using `<ContentLoader>` component's `baseUrl` props or not using a `<base>` element.

:::callout{variant="info"}
When you are using Webpack, you can set `output.publicPath` and no longer need the `<base>` element in most cases.
:::

## Conclusion

- Show loading placeholder to lead users' gaze during loading
- Make loading placeholder imitate the actual contents to lead correctly
