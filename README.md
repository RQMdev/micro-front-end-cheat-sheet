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
