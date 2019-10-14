## Blank Sheet

Import an application :

define a reactModule :

```javascript
export const reactModule = {
  id: "react-app",
  url: "http://localhost:3000",
  scripts: [
    "/static/js/runtime-main.js",
    "/static/js/main.chunk.js",
    "/static/js/2.chunk.js"
  ],
  styles: ["/static/css/main.chunk.css"]
};
```

import it in page.js :

```javascript
import { reactModule } from "../config";
```

define a page component that will be the anchor for the coming app :
create the root anchor :

```javascript
// connectedCallback()
const frameworkRoot = this.createElementFromID(reactModule)
this.appendChild(frameworkRoot)

// method:
createElementFromID(appModule) {
  const root = document.createElement('div')
  root.id = appModule.id
  return root
}
```

DEMO: the div with ID

create the promiseSerial helper :

```javascript
promiseSerial(funcs, init) {
  return funcs.reduce(
    (promise, func) => promise.then(func),
    Promise.resolve(init)
  )
}
```

create the loadScript method :

```javascript
loadScripts(scripts) {
  return this.promiseSerial(
    scripts.map(scriptPath => () =>
      new Promise((resolve, reject) => {
        const previousScript = document.getElementById(scriptPath)
        if (previousScript) {
          previousScript.remove()
        }
        const script = document.createElement('script')
        script.src = scriptPath
        script.id = scriptPath
        script.defer = true
        script.type = 'text/javascript'
        script.onload = resolve
        script.onerror = reject
        document.body.appendChild(script)
      })
    )
  )
}
```

add it into the connectedCallback() :

```javascript
this.loadScripts(
  reactModule.scripts.map(scriptPath => reactModule.url + scriptPath)
);
```

DEMO: the cat is here ! but not the app style !!

add loadStyles method :

```javascript
loadStyles(styles) {
  return this.promiseSerial(
    styles.map(stylePath => () =>
      new Promise((resolve, reject) => {
        const previousStyle = document.getElementById(stylePath)
        if (previousStyle) {
          previousStyle.remove()
        }
        const style = document.createElement('link')
        style.href = stylePath
        style.id = stylePath
        style.rel = 'stylesheet'
        style.onload = resolve
        style.onerror = reject
        document.head.appendChild(style)
      })
    )
  )
}
```

call it from connectedCallback :

```javascript
this.loadStyles(
  reactModule.styles.map(stylePath => reactModule.url + stylePath)
);
```

DEMO: The cat is now stylish :D

## Save : react-imported

## Master

### fixes main navigation reloading app :

Add event listener on click to catch main navigation event

```javascript
// router.js
// connectedCallback()
this.addEventListener("click", event => {
  const href = this.getLinkHref(event.target);
  if (href) {
    const { pathname } = new URL(href, location.origin);
    this.resolveRoute(pathname, event);
  }
});
```

add getLinkHref method to find 'A' link in the DOM

```javascript
getLinkHref(element) {
    if (element.tagName === 'A') {
      return element.href || ''
    } else if (element.tagName !== 'BODY') {
      return this.getLinkHref(element.parentElement)
    } else {
      return undefined
    }
  }
```

compare full current full path to new path

```javascript
// router.js
renderRoute(targetRoute, event, path) {
  // ...
  this.currentPath = path
    } else if (path === this.currentPath) {
      event.preventDefault()
  // ...
}
```

fixe React behaviour:

```javascript
// router.js
// renderRoute
} else if (targetRoute.path.startsWith(this.currentRoute.path)) {
      // react-router check if event was preventDefault and don't do anything if it is
```

## Step 1

a) Create navbar to navigate easily from our config:

```javascript
import { routes } from "../config";

export default class ContainerNavbar extends HTMLElement {
  constructor() {
    super();
  }

  static get tagName() {
    return "container-navbar";
  }

  render() {
    this.innerHTML = `
    <nav role="navigation">
      <ul>
      ${routes
        .filter(route => route.title)
        .map(
          route => `
            <li>
              <container-link
                path="${route.path}"
                title="${route.title}"
              ></container-link>
            </li>
          `
        )
        .join("")}
      </ul>
    </nav>
  `;
  }

  connectedCallback() {
    this.render();
  }
}
```

b) and the sub component link :

```javascript
export default class ContainerLink extends HTMLElement {
  constructor() {
    super();
  }

  static get tagName() {
    return "container-link";
  }

  connectedCallback() {
    this.path = this.getAttribute("path");
    this.title = this.getAttribute("title");

    this.render();
  }

  render() {
    this.innerHTML = `
      <a href="${this.path}">
        ${this.title}
      </a>
    `;
  }
}
```

c) and import them in component/index.js

```javascript
export { default as ContainerLink } from "./link";
export { default as ContainerNavbar } from "./navbar";
```

d) place them in the app component :

```javascript
  <container-navbar></container-navbar>
  <div id="app-container">

  </div>
```

## Step 2
