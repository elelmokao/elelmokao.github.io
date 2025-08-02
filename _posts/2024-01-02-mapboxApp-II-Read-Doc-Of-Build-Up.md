---
title: Mapbox App II [Note] - Read Doc Of Build Up
date: 2024-01-02 00:00:00 +0800
categories: [Mapbox, MapboxDoc]
tags: [mapbox, react, doc]
# TAG names should always be lowercase
---
Update Date: 2024-01-03
## 1. Introduction
This blog is about the reading and we will go through the official documentation. 
The content might be very similar to the document for sure and add some comments I need to learn.
Now, let's dive into [Use Mapbox GL JS in a React app](https://docs.mapbox.com/help/tutorials/use-mapbox-gl-js-with-react/) [^fn1]

## 2. Set up the React app structure
The structure of file structure should be as below
```markdown
use-mapbox-gl-js-with-react/
    package.json       # Specify all of the Node packages that your app requires
    public/            # 
        index.html     # Display the rendered Mapbox map
    src/
        App.js         # Set up the React app
        index.css      # Contain styles to format the map and sidebar
        index.js       # Render the Mapbox map to the browser
```
Then navigate to `use-mapbox-gl-js-with-react/`, type `npm run` to install all Node packages and then `package-lock.json` would be generated.

### HTML page
Write down following code into `public/index.html`:
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta
      name="description"
      content="Create a React web app that uses Mapbox GL JS to render a map"
    />
    <title>Use Mapbox GL JS in a React app</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>
```
`<div>` part would randor the React app on the page.

## 3. Create the React App
Open `src/index.js`, the following code can improt 2 stylesheets and Mapbox GL JS map that you build:
```js
import React from 'react';
import ReactDOM from 'react-dom';
import 'mapbox-gl/dist/mapbox-gl.css';
import './index.css';
import App from './App';

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);
```
This stylesheet contains the Mapbox GL JS styles to display the map. And 2nd stylesheet `index.css` can add app-specific CSS.

`<App />` is the content we render and `document.getElementById('root')`是React所綁定的原始div元素。

**ReactDOM**: 

> * DOM (Document Object Model, 文件物件模型) is the program interface of HTML, XML, SVG files[^fn2]. 
* `document.getElementById`: 用id在向DOM取得元素。
* `document.getElementById().scrollTop=....`: 修改元素在DOM的scrollTop。

**React.StrictMode** [^fn3]: 

A devloping tool that can help the developer check and get warnings like: 
* 有副作用的元件
* 已棄用或不安全的生命週期方法
* 某些內置函數的不安全用法
* 列表中的重複鍵

Note that it can be removed to speed up building.

## 4. Add Mapbox GL JS
Let's improt statement to the top of the `src/App.js`:
```js
import React, { useRef, useEffect, useState } from 'react';
import mapboxgl from '!mapbox-gl'; // eslint-disable-line import/no-webpack-loader-syntax

mapboxgl.accessToken = 'YOUR_MAPBOX_ACCESS_TOKEN';
```
If you have logged in, the token above would be replace to your default one.<br>
Now, it's done for setup. We can add the app we desired below `src/App.js`

## 5. Set the app's default state
Below `src/App.js`, we can create the map by `App()` function which will return the map.
```js
export default function App() {

  // Code below gives the initial long, lat, and zoom scale.
  // They can change as user interacts with map.
  const mapContainer = useRef(null);
  const map = useRef(null);
  const [lng, setLng] = useState(-70.9);
  const [lat, setLat] = useState(42.35);
  const [zoom, setZoom] = useState(9);
  
  // Below code initialize the map.
  // Below will be invoked right after the app is inserted into the DOM tree of your HTML page. 
  // (see src/index.js: 11 and public/index.html: 14)
  useEffect(() => {
    if (map.current) return; // initialize map only once
    map.current = new mapboxgl.Map({
      container: mapContainer.current, // tells Mapbox GL JS to render the map inside a specific DOM element
      style: 'mapbox://styles/mapbox/streets-v12', // defines the map style
      center: [lng, lat],
      zoom: zoom
    });
  });
  return (
    <div>
      <div ref={mapContainer} className="map-container" />
      // If you are using hooks, you also created a map useRef to store the initialize the map. 
      // The ref will prevent the map from reloading when the user interacts with the map.
      // The function App() then return the map you initialized.
    </div>
  );
}
```

## 6. Render the map
First, the `App()` function ([see here](#5-set-the-apps-default-state)) will render the map to the `index.html`.

Second, a few styling is added to render correctly at `index.css`:
```css
.map-container {
  height: 400px;
}
.sidebar {
background-color: rgb(35 55 75 / 90%);
color: #fff;
padding: 6px 12px;
font-family: monospace;
z-index: 1;
position: absolute;
top: 0;
left: 0;
margin: 12px;
border-radius: 4px;
}
```

Save all files and run `npm start` which can give you the map you render.<br>
Note that it might warn you `no-unused-vars`, it's totally fine. It will be used afterwards.

## Reference
[^fn1]: https://docs.mapbox.com/help/tutorials/use-mapbox-gl-js-with-react/
[^fn2]: https://ithelp.ithome.com.tw/articles/10215265
[^fn3]: https://codelove.tw/@tony/post/9xLP4x